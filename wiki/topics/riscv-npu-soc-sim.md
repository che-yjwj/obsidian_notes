---
title: RISC-V NPU SoC Simulator
type: topic
status: canonical
last_compiled: 2026-04-16
---

# RISC-V NPU SoC Simulator

*last_compiled: 2026-04-16 | sources: 70+ (re-scoped umbrella topic)*

---

## Summary [coverage: curated umbrella]

`RISCV_NPU_SoC_SIM`은 RISC-V 기반 SoC 위에서 동작하는 NPU 아키텍처를 대상으로 하는 **정적 스케줄 기반 시뮬레이터 & 오프라인 컴파일러** 프로젝트다. 타깃 워크로드는 모바일·엣지 환경의 LLM Inference(특히 Prefill/Decode 분리와 KV cache 재사용 시나리오)이며, ONNX 모델을 입력으로 받아 내부 IR → 타일링 → 정적 스케줄링 → CMDQ 생성 → 시뮬레이션까지 전 과정을 다룬다.

이 페이지는 이제 프로젝트 전체를 한 장에서 잡아주는 **umbrella topic**이다. 세부 내용은 아래의 하위 canonical topic으로 분리했다.

| Topic | 역할 |
|---|---|---|
| [[topics/npu-architecture-spec]] | execution-spec 계층 전체를 라우팅하는 umbrella topic |
| [[topics/ir-cmdq-contract]] | IR schema, tensor metadata, CMDQ format/opcodes, deterministic lowering contract |
| [[topics/tile-semantics-quantization]] | tile lifecycle, mixed precision, KV semantics, prefill/decode policy contract |
| [[topics/npu-timing-memory-model]] | DMA/TE/VE/SPM/Bus/NoC와 cycle timing, bandwidth, contention 모델 |
| [[topics/simulation-validation]] | golden trace, test plan, integration/unit/perf validation protocol |
| [[topics/trace-visualization]] | trace schema, timeline, heatmap, utilization visualization |
| [[topics/npu-doc-process]] | SDD workflow, milestone, roadmap, naming, review process |

프로젝트 목적은 여전히 같다.

- **분석용 시뮬레이터**: latency, bandwidth, utilization 등 NPU 자원 특성을 빠르게 예측
- **오프라인 컴파일러**: ONNX → NPU IR → TileGraph → CMDQ 생성 파이프라인
- **LLM-friendly 아키텍처 실험**: Mixed-precision, KV-cache, Prefill/Decode 분리 시나리오 지원
- **시각화·프로파일링**: Gantt timeline, bandwidth heatmap, engine utilization 등

---

## Project Status

| 영역 | 상태 | 비고 |
|---|---|---|
| 문서 (Spec/Design/Test) | Stable Draft | SDD 기반 문서 우선 정비 완료 |
| 오프라인 컴파일러 (`src/compiler/`) | Planned | 스켈레톤만 존재 |
| NPU 시뮬레이터 (`src/simulator/`) | Planned | Global cycle loop 구현 전 |
| 공통 유틸/테스트 (`src/common/`, `tests/`) | Planned | 문서로만 정의된 상태 |

프로젝트는 **Spec-Driven Development(SDD)** 원칙을 채택한다. 즉, 스펙 문서 업데이트 → 설계 문서 업데이트 → 코드 구현 → 테스트 순서를 강제한다. 이 때문에 `riscv-npu-soc-sim`은 단순 코드 저장소가 아니라 문서 중심 프로젝트 운영 모델까지 함께 포함하는 연구/개발 허브다.

---

## What Stays in the Umbrella Topic

이 umbrella page에는 아래만 남긴다.

- 프로젝트 전체 목적과 범위
- 컴파일러 + 시뮬레이터 + 시각화 + 문서 프로세스의 큰 그림
- 하위 canonical topic으로의 라우팅
- cross-cutting concept와 현재 개발 단계 요약

다음은 하위 topic으로 넘긴다.

- 규범적 execution semantics와 IR/ISA/quantization rule
- timing, SPM, DRAM, Bus/NoC contention, DMA/engine latency
- trace schema와 visualization 상세
- integration/unit/performance validation protocol
- roadmap, milestone, naming, contribution workflow

---

## System Picture

```text
[Host CPU (RISC-V)]
    - MMIO/CSR, NPU Launch
    - DRAM Controller / SoC Interconnect
          |
          v
[Offline Compiler]
    - ONNX Loader / IR Builder
    - Quantization Annotator
    - Tiling Planner / SPM Allocator
    - Static Scheduler
    - CMDQ Generator
          |
          v
[NPU Simulator Core]
    - Control FSM (CMDQ Executor)
    - DMA / TE / VE Engines
    - SPM / DRAM / NoC Models
    - Trace Engine
          |
          v
[Visualization & Profiler]
```

핵심 아키텍처 아이디어는 세 가지다.

- **정적 스케줄 + 결정론적 실행**
- **SRAM/SPM 중심 데이터 재사용**
- **LLM-friendly execution path**: mixed precision, KV cache, prefill/decode 분리

---

## Cross-Cutting Concepts

- [[concepts/static-scheduling-determinism]]
- [[concepts/prefill-decode-duality]]
- [[concepts/tile-semantics-contract]]
- [[concepts/mixed-precision-policy]]
- [[concepts/kv-cache-dram-residency]]
- [[concepts/trace-first-design]]

---

## Source Families

- `docs/overview/system_architecture.md`, `docs/overview/dataflow_overview.md`, `docs/overview/module_responsibilities.md`
  -> [[topics/npu-architecture-spec]]
- `docs/spec/architecture`, `docs/spec/ir`, `docs/spec/isa`, `docs/spec/quantization`, `docs/spec/scheduling`
  -> [[topics/npu-architecture-spec]]
- `docs/spec/timing`, `docs/overview/memory_noc_overview.md`, `docs/design/dma_engine_design.md`, `docs/design/tile_rt_analysis.md`
  -> [[topics/npu-timing-memory-model]]
- `docs/test/*`, golden traces, validation protocol
  -> [[topics/simulation-validation]]
- `docs/spec/trace/*`, `docs/design/visualizer_design.md`
  -> [[topics/trace-visualization]]
- `docs/process/*`, roadmap, milestone, contribution guide
  -> [[topics/npu-doc-process]]

---

## Next Split Focus

- `docs/process/archive/*`를 process mainline과 분리해 archive 계층으로 내려야 한다.
- validation 쪽이 계속 커지면 `simulation-validation` 안에서 `golden-trace`와 `performance-validation`을 다시 나눌 수 있다.
- architecture spec topic이 계속 커지면 `IR/CMDQ`와 `tile semantics/quantization`을 추가 분리 후보로 본다.

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
