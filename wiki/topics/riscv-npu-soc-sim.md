# RISC-V NPU SoC Simulator

*last_compiled: 2026-04-12 | sources: 70+ (sampled 20 key docs)*

---

## Summary [coverage: high -- 8 sources]

`RISCV_NPU_SoC_SIM`은 RISC-V 기반 SoC 위에서 동작하는 NPU 아키텍처를 대상으로 하는 **정적 스케줄 기반 시뮬레이터 & 오프라인 컴파일러** 프로젝트다. 타깃 워크로드는 모바일·엣지 환경의 LLM Inference(특히 Prefill/Decode 분리와 KV cache 재사용 시나리오)이며, ONNX 모델을 입력으로 받아 내부 IR → 타일링 → 정적 스케줄링 → CMDQ 생성 → 시뮬레이션까지 전 과정을 다룬다.

프로젝트 목적:
- **분석용 시뮬레이터**: latency, bandwidth, utilization 등 NPU 자원 특성을 빠르게 예측. cycle-accurate RTL 시뮬레이터가 아닌 resource-accurate 분석 도구 (목표 latency 오차 ±10~15%)
- **오프라인 컴파일러**: ONNX → NPU IR → TileGraph → CMDQ 생성 파이프라인
- **LLM-friendly 아키텍처 실험**: Mixed-precision quantization, KV-cache, Prefill/Decode 분리 시나리오 지원
- **시각화·프로파일링**: Gantt timeline, bandwidth heatmap, engine utilization 등

**현재 상태 (Last Updated: 2025-12-02):**
| 영역 | 상태 | 비고 |
|---|---|---|
| 문서 (Spec/Design/Test) | Stable Draft | SDD 기반 문서 우선 정비 완료 |
| 오프라인 컴파일러 (`src/compiler/`) | Planned | 스켈레톤만 존재 |
| NPU 시뮬레이터 (`src/simulator/`) | Planned | Global cycle loop 구현 전 |
| 공통 유틸/테스트 (`src/common/`, `tests/`) | Planned | 문서로만 정의된 상태 |

프로젝트는 **Spec-Driven Development(SDD)** 원칙을 채택: 스펙 문서 업데이트 → 설계 문서 업데이트 → 코드 구현 → 테스트 순서를 강제한다. 70+ 문서가 코드보다 먼저 완성된 상태이며, 구현은 순차적으로 진행될 예정이다.

---

## Core Concepts [coverage: high -- 12 sources]

### NPU IR (Intermediate Representation)

NPU IR은 오프라인 컴파일러 전체의 단일 소스 오브 트루스(SSoT)이며, 3개의 주요 데이터 구조로 구성된다:

```
NPU_IR
 ├── Graph        → LayerIR 노드의 DAG
 ├── TensorTable  → TensorIR 메타데이터
 └── QConfig      → Quantization policy
```

**LayerIR**: ONNX 노드보다 더 구조적이며 tile/schedule-friendly한 연산 단위. 주요 op_type:
- `GEMM`, `MATMUL`: DRAM → SPM DMA Load → TE_GEMM_TILE → DMA Store로 매핑
- `LAYER_NORM`, `RMS_NORM`, `SOFTMAX`, `GELU`: VE tile 연산으로 매핑
- `QKV_PROJ`, `ATTN_SCORES`, `ATTN_OUTPUT`: LLM attention 블록
- `KV_UPDATE`, `KV_CACHE_RESIZE`: KV cache 증분 업데이트 (qbits_kv 별도 관리)

각 LayerIR에는 `qbits_weight`, `qbits_activation`, `qbits_kv` 세 개의 독립 bitwidth 필드가 있다. **Quantization annotation은 IR에만 삽입**되며 실제 scale/zero-point 처리는 구현 범위가 아니다.

**TensorIR**: `id`, `shape`, `dtype` (fp32/fp16/int8/int4/int2), `role` (activation/weight/kv/intermediate), `layout`, `qbits`, `producer/consumers` 정보를 담는 텐서 메타데이터.

**NPU IR Core Reference** (TOG-Compatible Design): PyTorchSim TOG(Tile Operation Graph) 개념과 호환되는 코어 IR 설계 원칙:
1. Tile is the atomic execution unit
2. Memory movement is explicit (DmaLoad/DmaStore 노드)
3. Synchronization is explicit (Tag + Wait)
4. Loops are structural IR (PARALLEL / ACCUMULATION / INNER)
5. Compute latency is deterministic
6. IR is backend-agnostic

### Tile Execution Model

타일은 스케줄링·데이터 이동·연산의 최소 단위이며 아래 라이프사이클을 거친다:

```
Allocated → Produced → Handed-off → Consumed → (Reused | Freed)
```

**메모리 계층 책임 경계:**
- **DRAM**: 입력/최종 출력/KV cache 영속 저장소. 엔진 간 중간 결과 전달 경로로 사용 금지.
- **SPM (Scratchpad Memory)**: 타일 payload의 유일한 공유 저장소. 캐시가 아니며 자동 eviction 없음. multi-bank 구조, 직접 소프트웨어 관리.
- **엔진 로컬 버퍼**: 엔진 내부 scratch. IR 관점에서 주소화되지 않으며 외부로 노출 불가.

**TE-VE 데이터플로우 (STB Semantics):**
- TE(Tensor Engine): GEMM/MAC 중심 2D 고밀도 연산. 결과를 SPM에 기록하고 타일 디스크립터를 handoff.
- VE(Vector Engine): softmax/LN/reduction/activation 등 후처리. STB 경계를 통해 디스크립터를 수신 후 SPM에서 payload 로드.
- 타일 payload 데이터는 SPM에만 존재하며, 엔진 간 전달은 타일 디스크립터로만 이루어진다.

### ISA / Command Queue (CMDQ)

CMDQ는 **NPU가 실행해야 할 모든 연산을 정적으로 서술한 명령 스트림**이다. 오프라인에서 생성되며 런타임에 NPU 내부에는 복잡한 스케줄러가 필요 없다. CMDQ는 시뮬레이터의 **유일한 입력**이다.

CMDQ 명령 카테고리:
| Class | 명령 예시 |
|---|---|
| DMA | `DMA_LOAD_TILE`, `DMA_STORE_TILE` |
| Tensor Engine (TE) | `TE_GEMM_TILE`, `TE_CONV_TILE`, `TE_QKT_TILE`, `TE_AV_TILE` |
| Vector Engine (VE) | `VE_LAYERNORM_TILE`, `VE_SOFTMAX_TILE`, `VE_GELU_TILE` |
| Control/Sync | `BARRIER`, `NOP`, `END` |
| LLM-Specific | `KV_STORE_TILE`, `KV_LOAD_TILE`, Q/K/V projection tile 등 |

각 CMDQ 엔트리는 `opcode`, `engine_type`, `engine_id` (dma_id/te_id/ve_id), `deps_before` (선행 완료 조건 ID 배열), `spm_bank/spm_offset`, `qbits_*` 필드를 포함한다. JSON 포맷으로 표현된다.

**CMDQ 실행 모델:** `ControlFSM → 각 엔진(DMA/TE/VE) → TraceEngine`
- ControlFSM은 CMDQ를 한 줄씩 fetch하고 deps_before 조건이 만족되면 해당 엔진에 issue.
- END opcode를 만나면 시뮬레이션 종료.
- 동일 CMDQ + 동일 config + 동일 초기 상태이면 항상 동일 결과 (결정론적 실행 보장).

### Quantization Model

레이어별 Weight/Activation/KV bitwidth를 독립적으로 관리하는 mixed-precision 정책을 채택한다.

```yaml
quantization:
  defaults:
    weight: 4      # INT4 weight 기본값
    activation: 8  # INT8 activation 기본값
    kv: 4          # KV cache용 INT4
  overrides:
    layer.ffn_out: { weight: 8, activation: 8 }
```

Quantization 메타데이터는 downstream 전 단계에 영향을 미친다:
- **DMA bytes**: qbits 기반 `total_bytes = num_elements × qbits / 8` 계산
- **SPM allocation**: bitwidth-aware capacity 산정
- **TE/VE latency**: weight compression 효과 반영
- **DRAM bandwidth occupancy**: bandwidth saturation 분석

KV cache는 activation/weight와 독립적인 bitwidth(일반적으로 INT4)를 가지며, append/concat 기반 seq 증가 모델을 사용한다.

### Static Scheduler

StaticScheduler는 TileGraph + SPM allocation + 엔진 구성 정보를 기반으로 **정적 tile-level 실행 순서와 deps를 생성**한다.

스케줄러 불변 규칙:
1. **의존성 보존**: TDG의 의존성을 100% 보존. predecessor가 완료되기 전에 issue 불가.
2. **메모리 규칙**: tile payload는 SPM에 존재해야 하며 DRAM을 중간 전달 경로로 사용 금지.
3. **엔진 귀속**: 각 entry는 정확히 하나의 engine_type에 귀속, ID는 유효 범위 내.
4. **결정론**: 동일 입력이면 동일 스케줄. 랜덤/RNG 기반 tie-break 금지.

엔진 배정 전략: 라운드로빈 또는 load-balance heuristic으로 TE/VE/DMA에 타일을 분배. DMA LOAD를 우선 배치하여 TE/VE idle을 최소화.

LLM-aware 스케줄링:
- **Prefill**: TE/VE 완료 직후 KV_STORE_TILE enqueue, head/t_start 순서로 KV append 보장
- **Decode**: 토큰 step마다 KV_LOAD_TILE을 TE/VE보다 먼저 배치하여 fetch latency를 숨김

### Tiling

TilingPlanner는 LayerIR을 tile 단위 연산으로 분해하여 TileGraph(tile-level DAG)를 구성한다.

| Layer 유형 | 기본 타일링 축 | 전략 |
|---|---|---|
| GEMM/FFN | M, N | SPM 용량 내에서 M_tile×N_tile을 크게, K는 가능한 한 크게 유지 |
| Conv 2D | H, W, C | SPM에 IFM/WGT/OFM이 동시에 들어가도록 H/W 우선 타일링 |
| LayerNorm/RMSNorm | H (hidden) | hidden size를 SIMD lane 수에 맞춰 타일링 |
| Attention (Q/K/V) | T (seq), H (head) | KV cache는 T 기준, 다중 head는 병렬 유지 |
| KV Cache Update | T | 토큰/토큰 블록 단위로 append/copy 비용 최소화 |

GEMM 타일 크기 결정 제약: `bytes_ifm_tile + bytes_wgt_tile + bytes_ofm_tile ≤ spm_capacity_per_engine`

---

## Architecture [coverage: high -- 9 sources]

### 시스템 구성 (4개 상위 컴포넌트)

```
[Host CPU (RISC-V)]
    - MMIO/CSR, NPU Launch (write_csr(NPU_CMDQ_BASE, addr); write_csr(NPU_START, 1))
    - DRAM Controller / SoC Interconnect (CPU/NPU 공유 DRAM)
          |
          v
[Offline Compiler]
    - ONNX Loader / IR Builder
    - Quantization Annotator (W/A/KV bit)
    - Tiling Planner / SPM Allocator
    - Static Scheduler
    - CMDQ Generator
          |
          v
[NPU Simulator Core (Cycle-Based)]
    - Control FSM + CMDQ Executor
    - DMA Engine
    - Multi TE (Tensor Engines, ≥ 2)
    - Multi VE (Vector Engines, ≥ 2)
    - SPM / DRAM / Bus / NoC Models
    - Trace Engine
          |
          v
[Visualization & Profiler]
    - Gantt Timeline / BW Heatmap / Utilization
    - Export: JSON, PNG/SVG, CSV
```

**모듈 간 의존성 규칙 (단방향):**
- Offline Compiler → NPU Simulator (CMDQ만 전달)
- NPU Simulator → Visualization (Trace만 전달)
- Host CPU → NPU Simulator (launch-only)
- Compiler는 Simulator 내부를 참조하지 않으며, Simulator는 Compiler 산출물(CMDQ)만 사용.

### 오프라인 컴파일러 파이프라인

```
ONNX Model
   ↓
IrBuilder          → NPU IR (LayerIR/TensorIR)
   ↓
QuantizationAnnotator → qbits W/A/KV 주입된 Annotated IR
   ↓
TilingPlanner      → TileGraph (tile-level DAG)
   ↓
SpmAllocator       → TileGraph + SPM bank/offset map
   ↓
StaticScheduler    → ScheduleDAG (tile-level 순서 + deps)
   ↓
CmdqGenerator      → CMDQ JSON (시뮬레이터 입력)
```

Python 인터페이스 예시:
```python
def compile(onnx_path: str, hw_config: HwConfig, qconfig: QConfig) -> CmdqArtifact:
    ir = IrBuilder.build(onnx_path)
    ir = QuantizationAnnotator.annotate(ir, qconfig)
    tile_graph = TilingPlanner.plan(ir, hw_config.spm, hw_config.engines)
    tile_graph = SpmAllocator.allocate(tile_graph, hw_config.spm)
    schedule = StaticScheduler.schedule(tile_graph, hw_config.engines)
    cmdq = CmdqGenerator.generate(schedule, tile_graph, ir, hw_config)
    return CmdqArtifact(cmdq=cmdq, ir=ir, tile_graph=tile_graph, schedule=schedule)
```

### NPU Simulator Core

CMDQ를 입력으로 받아 cycle 기반으로 DMA/TE/VE/메모리/Trace를 통합 시뮬레이션한다.

**Global Cycle Loop 구조:**
```pseudo
cycle = 0
while not control_fsm.is_finished() and cycle < max_cycles:
    for tickable in [ControlFSM, DMAEngineCluster, TECluster, VECluster, MemoryModel, TraceEngine]:
        tickable.tick(cycle)
    engine_events = collect_engine_events()
    control_fsm.consume_engine_events(engine_events)
    issue_reqs = control_fsm.step_issue(cycle)
    for req in issue_reqs:
        engines[req.engine_type][req.engine_id].enqueue(req.cmdq_entry)
    trace_engine.step(cycle)
    cycle += 1
```

**SimulatorConfig 주요 파라미터:** `n_te`, `n_ve`, `n_dma`, DRAM/NoC/SPM 파라미터, quantization/timing 모델 선택, 각 엔진의 clock period (cpu_period, dma_period, te_period, ve_period 등 multi-clock 지원).

**CMDQ Entry 상태 전이:** `NOT_ISSUED → READY → ISSUED → COMPLETED`

### DMA Engine

DRAM ↔ SPM 데이터 이동의 latency/bandwidth 모델링 담당.

Job 처리 흐름: `DmaJob QUEUED → TRANSFERRING → COMPLETED`

Latency 계산: `bytes_total = num_elements × qbits / 8` → bus width, burst size, contention 모델 적용. 복수 DMA가 동시 실행 시 `effective_bw = peak_bw / N` (shared bandwidth 모델).

KV-aware 처리: `KV_STORE_TILE`/`KV_LOAD_TILE`은 `tensor_role="kv"`, `head_id`, `kv_kind` (k/v), `kv_range` (t_start/t_len/d_start/d_len) 필드를 가진다. KV DMA 채널에 우선순위 가중치(+δ)를 부여하여 TE idle 방지.

### Memory & NoC Model

**메모리 계층:**
```
[DRAM]  →  [Bus/NoC]  →  [SPM (multi-bank)]  →  [TE/VE Engines]
```

**Bus/NoC Timing 공식:**
```
dram_cycles = bytes / effective_bw + t_setup
noc_cycles  = hops × hop_latency_cycles + serialization_cost
bus_latency = max(dram_cycles, noc_cycles)
```

**Arbitration Policy:** `weighted_rr_v1` (기본값). 토큰 기반 weighted round-robin. 결정론을 위해 동률 시 `master_id` 오름차순 tie-break 고정. Master 타입 우선순위: `DMA=0, TE=1, VE=2, TRACE=3`.

**Queue-based Stall 모델:** 각 master는 `queue_depth_per_master` 만큼 outstanding transaction 허용. 초과 시 stall 상태로 STALL_EVENT를 Trace에 기록. RESUME_EVENT는 grant 재개 시 기록.

### Trace & Visualization

TraceEngine이 수집하는 정보:
- `ENGINE_EVENT`: 각 엔진의 start_cycle/end_cycle, opcode, bytes
- `MEM_ACCESS_EVENT`: DRAM/SPM 주소, 크기, 방향, source_engine
- `bandwidth_samples`: window 단위 read/write bytes 합산
- `STALL_EVENT`/`RESUME_EVENT`: stall 사유(queue_full/backpressure 등) 및 지속 시간
- layer별 latency breakdown, per-window DRAM bandwidth usage, SPM bank conflict count

Visualization 출력: Gantt chart (TE/VE/DMA timeline), DRAM bandwidth heatmap, SPM bank conflict map, TE/VE utilization bars, KV cache growth graph (LLM-only), Quantization impact graph.

---

## Key Findings [coverage: high -- 11 sources]

### 핵심 설계 결정

**정적 스케줄 모델 선택 이유:** Mobile/Edge NPU는 전력·면적·제어부 단순화를 위해 런타임 스케줄러를 사용하지 않는다. 모든 스케줄은 offline에서 완성되며 CMDQ로 serialize된다. 이를 통해 Control FSM이 단순해지고 성능 분석 및 병목 파악이 용이해진다.

**Resource-accurate (Not Cycle-accurate):** 정확한 cycle-level RTL 시뮬레이터가 아닌 NPU 구조의 병목/자원 사용률/메모리 traffic을 정확히 반영하는 분석용 시뮬레이터를 목표로 한다. 목표: latency 예측 오차 ±10~15%, bandwidth/traffic 패턴 분석 정확도 높음.

**결정론적 시뮬레이션:** 동일 CMDQ + 동일 config + 동일 초기 상태이면 항상 동일 결과를 보장한다. Bus/NoC arbitration에 랜덤/RNG 사용 금지. tie-break 규칙을 문서로 고정(master_id 오름차순).

**Bitwidth-driven Memory Model:** FP32 → INT8 → INT4 → INT2로 줄어들수록 메모리 transaction이 줄어드는 구조를 정밀하게 반영. Weight/Activation/KV bitwidth를 각각 독립적으로 설정 가능.

**CMDQ = 오프라인 컴파일러와 시뮬레이터 사이의 ISA-level 계약:** CMDQ는 IR/Tiling/Scheduling의 모든 결정 사항을 포함한 "작은 프로그램"이며, 시뮬레이터는 CMDQ를 실행하는 executor에 불과하다.

### 주요 수치 및 스펙

| 항목 | 값 |
|---|---|
| 목표 latency 예측 오차 | ±10~15% |
| TE 최소 구성 | ≥ 2개 |
| VE 최소 구성 | ≥ 2개 |
| 기본 Weight bitwidth | INT4 |
| 기본 Activation bitwidth | INT8 |
| 기본 KV cache bitwidth | INT4 |
| Bus arbitration 기본 정책 | `weighted_rr_v1` |
| Tie-break 규칙 | `by_master_id` (오름차순) |

### 컴파일러 Lowering 규칙 (요약)

| IR TileOp | 필수 CMDQ 시퀀스 |
|---|---|
| MatMul / Conv | `DMA_LOAD_TILE (IFM/WGT)` → `SYNC_WAIT` → `TE_MATMUL_TILE` → `SYNC_WAIT` → `DMA_STORE_TILE` |
| FFN 1st MatMul + Bias + GELU | `LOAD X/W/B` → `TE_MATMUL_TILE` → `VE_ADD_TILE` → `VE_GELU_TILE` |
| LayerNorm/RMSNorm | `LOAD X` → `VE_RMSNORM_TILE` → `STORE Y` |
| Softmax | `LOAD Scores` → `VE_SOFTMAX_TILE` → `STORE` |
| KV-Attention (Decode) | Q/K/V Projection → KV_STORE → KV_LOAD → `TE_QKT_TILE` → `VE_SOFTMAX_TILE` → `TE_AV_TILE` → Output Projection → `DMA_STORE` |

### LLM 워크로드 특성

- **Prefill**: throughput 중심 (대규모 S×S attention). TE 타일들이 병렬로 최대한 활용됨.
- **Decode**: latency 중심 (1×T attention + KV load 지배). KV_LOAD_TILE이 임계 경로를 지배.
- **KV cache 트래픽**: seq 증가에 따라 DRAM traffic이 선형으로 증가. INT4 KV quantization으로 완화 가능.
- **Prefill/Decode 분리 CMDQ**: Manifest 기반으로 각 phase를 독립 CMDQ 세그먼트로 생성. `MARKER_EVENT("PREFILL_DONE")` / `MARKER_EVENT("DECODE_DONE")`으로 phase 경계 표시.

---

## Connections [coverage: medium -- 3 sources]

이 vault 내의 관련 토픽:

- **양자화 (GenAI/Compression)**: 이 시뮬레이터의 quantization model(INT4/INT8/KV-specific bitwidth, mixed precision)은 PTQ/QAT 연구 결과를 NPU 타이밍 분석에 직접 연결한다. QConfig per-layer override 메커니즘은 양자화 실험 결과를 하드웨어 성능 분석으로 브리지.
- **NPU/가속기 아키텍처 (AI-Hardware/Architecture)**: TE(Systolic Array 계열 GEMM 가속기) + VE(SIMD 벡터 유닛) + DMA 구조는 전형적인 Mobile NPU 아키텍처 패턴. SPM(Scratchpad) 기반 메모리 모델, AXI Bus/NoC 구조도 AI 가속기 아키텍처 공통 설계.
- **KV-Cache 및 LLM HW-Friendly 설계 (GenAI/HW-Friendly)**: KV cache tile semantics, Prefill/Decode 분리 실행 모델, INT4 KV quantization은 LLM 추론을 하드웨어 효율적으로 만드는 핵심 기법들.

---

## Open Questions [coverage: medium -- 3 sources]

**구현 진행 관련:**
- Stage A (Foundation): 코드 스켈레톤 구축 + MatMul-only E2E 파이프라인 완성이 첫 번째 마일스톤. 현재 이 단계도 미완성(Planned).
- Offline Compiler의 `src/compiler/` 구현 시작 시점과 우선순위 미결정.
- NPU Simulator의 Global Cycle Loop 및 엔진 모델 구현 시작 시점 미결정.

**설계 오픈 이슈:**
- MLIR backend 연동 가능성 언급되었으나 구체적 설계 없음 (Stage E 이후).
- Auto-tuning 기반 tile optimizer: 현재는 heuristic round-robin 기반이며, critical path 기반 우선순위 스케줄링은 향후 확장 항목.
- 2D/ND DMA stride 지원: 현재 1D 기반이며 2D stride DMA는 향후 확장.
- Multi-channel DRAM, channel interleaving 모델: 현재는 단일 채널 모델.
- Sparsity engine 추가: 언급만 있으며 구체적 스펙 없음.
- Compression engine (compressed tensor 전송) DMA 지원: 미구현.

**LLM 특화 오픈 이슈:**
- Sliding-window attention 타일링 전략의 구체적 스펙 미정.
- Rotary Positional Embedding (RoPE) 처리용 opcode의 구체적 설계 미완.
- Mobile LPDDR vs Server HBM 등 프로파일별 메모리/NoC 구성 비교 섹션 추가 예정.

**로드맵 단계별 미결정:**
- Stage A-B 우선 수행 후 Stage D(LLM) vs Stage C(Conv/Attention) 우선순위가 명시되지 않음.
- DSE(Design Space Exploration) 플랫폼(Stage E): Config Sweep/Scenario YAML/CLI 설계 미시작.

---

## Sources [coverage: high -- 20 sources]

아래 파일들을 직접 읽어 이 문서를 작성하였다.

**Overview:**
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/README]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/README_SPEC]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/overview/system_architecture]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/overview/system_architecture_overview]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/overview/dataflow_overview]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/overview/memory_noc_overview]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/overview/module_responsibilities]]

**Design:**
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/design/npu_simulator_core_design]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/design/npu_ir_core_reference]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/design/static_scheduler_design]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/design/dma_engine_design]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/design/ir_builder_design]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/design/tiling_planner_design]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/design/offline_compiler_design]]

**Spec:**
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/ir/npu_ir_spec]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/isa/cmdq_overview]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/scheduling/static_scheduler_semantics_spec]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/quantization/quantization_model_overview]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/timing/bus_and_noc_model]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/architecture/tile_semantics_spec]]

**Process:**
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/project_roadmap]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/spec_driven_development_workflow]]

---

Full project documentation: 70+ files in `raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/`
