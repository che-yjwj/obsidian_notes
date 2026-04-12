# NPU Architecture

*last_compiled: 2026-04-12 | sources: 8*

---

## Summary [coverage: high -- 8 sources]

NPU(Neural Processing Unit) 아키텍처는 AI 추론 워크로드를 대상으로 범용 GPU/CPU 대비 전력·면적·지연 효율을 극대화하는 도메인 특화 가속기 설계 분야다. 이 토픽은 연산 구조(Tensor Engine / Vector Engine / Sparse Engine), 메모리 계층(SRAM scratchpad vs. cache hybrid), 인터커넥트(AMBA AXI / NoC), 컴파일러-HW 공동 설계, 그리고 현실 제품 레퍼런스(Tesla FSD 칩, AMD Versal ACAP, Google TPU, Xilinx FINN, Qualcomm AHPM, NVIDIA H100)를 아우른다.

핵심 주제가 중요한 이유: 자율주행·로봇·엣지 추론·서버 LLM 추론 등 응용 도메인이 확장되면서 단순 TOPS 경쟁을 넘어 **메모리 트래픽 통제, 정적 스케줄 기반 결정론적 실행, 혼합 정밀도 표현 설계**가 NPU 경쟁력의 실질적 축으로 부상하고 있다.

---

## Core Concepts [coverage: high -- 8 sources]

### 실행 모델: 정적 vs 동적 스케줄

NPU 설계의 가장 근본적인 선택은 실행 모델이다. 범용 GPU는 런타임 dynamic dispatch(warp scheduling)를 사용하는 반면, 도메인 특화 NPU는 **컴파일 타임 정적 스케줄(static schedule + micro-code)**로 worst-case latency를 예측 가능하게 만든다. Tesla FSD AI5 칩이 대표적 사례이며, AMD Versal AIE와 Xilinx FINN 모두 같은 철학을 공유한다. 정적 스케줄은 검증 가능한 상태공간을 축소해 tape-out 주기를 단축하는 부수 효과도 있다.

### 연산 엔진 분류: TE / VE / Sparse Engine

산업계와 연구에서 수렴하는 세 가지 연산기 클래스:

- **Tensor Engine (TE)**: Dense GEMM / Conv / Attention — systolic array 또는 대규모 MAC array
- **Vector Engine (VE)**: Activation, normalization, elementwise, BEV 후처리 — SIMD 기반
- **Sparse / Gather-Reduce Engine**: Embedding lookup, MoE routing, KV-cache 인덱스 접근 — index stream 기반

Google TPU v4/v5의 Sparse Core는 세 번째 클래스의 구현 사례다. Dense Core와 병렬 동작하여 추천 모델(RecSys)·MoE·retrieval 워크로드를 커버한다. AMD Versal AIE는 VLIW SIMD 벡터 타일 중심(VE 강함, TE 상대적으로 약함)이고, Tesla AI6는 Tensor + Vector + Scalar Control Core의 명시적 분리를 지향한다.

### 데이터플로우 vs 커맨드 기반

- **Dataflow-first**: FINN의 핵심 철학. 레이어 간 중간 결과를 DRAM에 저장하지 않고 on-chip FIFO로 스트리밍. 지연 최소(수 µs), 유연성 낮음, QNN 특화.
- **Command + SRAM/DRAM**: 범용 NPU의 전형. 유연성 높고 LLM 대응 가능.

### 혼합 정밀도 (Mixed Precision)

단순한 bit-width 혼합이 아닌 **Representation + Transport + Reconstruction의 공동 설계**. Tesla 특허(US20260017019A1)의 Mixed-Precision Bridge는 데이터를 로그(logarithm) 도메인으로 변환해 8-bit로 운반하고, 목적지에서 Horner's Method로 32-bit 정확도를 복원한다. 핵심은 "8-bit로 계산"이 아니라 "8-bit로 이동하고 필요한 지점에서만 고정밀 복원"이다.

### 메모리 계층의 핵심 선택: SRAM scratchpad vs Cache

순수 SRAM scratchpad 설계는 컴파일 타임에 데이터 접근 패턴을 100% 예측할 수 있어야 가능하다. LLM의 variable sequence length, dynamic batch, KV-cache 불규칙 접근은 이 가정을 깬다. Cache는 "성능을 올리는 장치"가 아니라 **SRAM-only 설계가 감당하지 못하는 불확실성을 흡수하는 안전망**이다. 실용적 설계는 두 방식의 혼합: 핵심 연산 경로는 SW-managed scratchpad, 나머지는 HW-managed cache.

---

## Architecture [coverage: high -- 8 sources]

### 연산 구조

**Tesla AI5 / AI6 아키텍처**

AI5는 AI4(FSD HW4)에서 확립된 구조의 확장·정교화로, 핵심 특징은:
- 대규모 MAC Array, INT8 / mixed precision 중심, vision backbone 최적화된 systolic-like dataflow
- 학습용이 아닌 추론 전용 (누산 정밀도만 국소적 확장)
- 정적 스케줄 + micro-code: GPU dynamic warp scheduling의 반대 방향. 컴파일러가 타이밍을 결정하여 worst-case latency 예측 가능
- Streaming-friendly DMA: 카메라 → ISP → NPU → Planner 파이프라인 연속 연결

AI6 예상 진화 방향:
- **Chiplet / Partitioned Architecture**: Camera-heavy SKU vs. Robot-heavy SKU를 동일 ISA, 다른 조합으로 빠른 파생 생산
- **Tensor + Vector + Scalar의 명시적 분리** — TE/VE/Control Core 3-tier 구조
- **Mixed-Precision Bridge의 구조적 내재화**: 로그 도메인 / 스케일 분리, 저정밀 datapath + 고정밀 메타데이터

9개월 세대 전환이 가능한 이유: 아키텍처가 단순할수록(고정 ISA, 정적 스케줄, SRAM scratchpad, 제한된 상태공간) 검증 범위가 좁아지고 tape-out 주기가 단축된다.

**AMD Versal ACAP (AI Engine)**

Versal은 CPU + AI Engine + FPGA Fabric + Hard NoC를 단일 실리콘에 통합한 ACAP 플랫폼이다:
- **AI Engine (AIE) 타일**: 각 타일이 VLIW SIMD 벡터 프로세서. Local SRAM(scratchpad), deterministic pipeline, 2D Mesh interconnect, 컴파일 타임 정적 라우팅
- **Programmable Logic (PL)**: Custom datatype(INT4, custom float), 특수 연산(FFT, codec), AIE와 스트리밍 연결 — NPU의 "custom execution unit 슬롯"
- **Hard NoC**: AXI 기반, 다중 QoS, deadlock-free routing, latency predictable(컴파일 시 반영), 캐시 일관성 SoC가 아닌 SPM + NoC 기반 deterministic memory system
- Versal의 Hard NoC 물리 토폴로지(라우터 위치, 링크 구조)는 변경 불가. 논리적 연결·QoS·트래픽 구조는 광범위하게 변경 가능.

Versal은 "Transformer 이전 시대의 AI"(streaming, DSP, fixed graph)에 최적이며, KV-cache random access가 필요한 decode-heavy LLM에는 구조적으로 불리하다.

**Xilinx FINN (FPGA 기반 QNN 가속기 생성기)**

컴파일 스택:
```
PyTorch/QKeras → ONNX → FINN IR (Dataflow Graph) → HLS (C++ Templates) → Vivado/Vitis → FPGA Bitstream
```
HW 특징:
- Streaming Conv/FC Engine: 입력 스트림 → MAC/비트연산 → 출력 스트림
- Activation Fusion: BN + ReLU/Sign을 하나의 파이프라인 단계로 병합
- FIFO-Centric Interconnect: NoC/캐시 없이 레이어 간 FIFO로 연결
- Clock-by-clock Determinism: 고정 지연 파이프라인, 최소 지연(수 µs)
- INT1~4 중심, LUT 기반 비트연산 최적화, DSP 사용 최소화
- Folding 파라미터: 레이어별 병렬도(PE 수) / 정밀도 / 파이프라인 깊이를 SW에서 설정

한계: Transformer/Attention 미적합(Softmax, KV Cache, 고정폭 스트리밍 한계), 동적 Shape 처리 곤란.

**Google TPU Sparse Core**

구조:
```
[Index Stream Engine]
        ↓
[Embedding Table SRAM / HBM Window]
        ↓
[Vector Accumulator / Reduce Unit]
        ↓
[Output Buffer]
```
핵심 동작: 외부적으로는 sparse index stream이지만, 내부 파이프라인은 index → gather → vector_accumulate → output의 연속 스트리밍. 하드웨어가 index를 모아 인접 row를 prefetch하고 burst로 fetch하여 bank conflict를 완화. DRAM 피크 BW는 동일하나 유효 BW가 GPU 대비 2–5× 개선되고 latency variance가 감소한다.

### 메모리 계층

**레퍼런스 아키텍처 비교**

| 항목 | NVIDIA H100/B100 | Qualcomm AHPM | AMD MI300 |
|---|---|---|---|
| 최상위 DRAM | HBM3/3e | LPDDR | HBM3 |
| 온칩 SRAM | Shared Mem + L2 | 대형 SRAM 중심 | LDS + L2 |
| Cache 정책 | HW + SW 혼합 | 거의 없음 | HW cache |
| SW 제어 | 매우 강함 (CUDA) | 매우 강함 (DMA) | 상대적으로 약함 |
| KV-cache 전략 | L2 + HBM | SRAM fit | HBM 상주 |

H100의 철학: "프로그래머가 locality를 만들면 HW는 이를 증폭." Shared Memory(scratchpad)가 FlashAttention, tiled GEMM, softmax fusion의 핵심이고, L2 cache는 KV-cache reuse와 multi-SM sharing을 담당.

Qualcomm AHPM의 철학: "DRAM을 안 쓰는 것이 곧 성능." 모바일 vision/audio/small LLM처럼 문제 공간을 줄였기 때문에 SRAM-only 접근이 가능하다.

**추천 NPU 메모리 계층 (중립적 합성)**

```
PE RF
 ↓
Local Scratchpad (TE/VE 전용)
 ↓
Cluster Shared SRAM (SW-managed)
 ↓
Optional L2 Cache (HW-managed safety net, LLM/multimodal 대응)
 ↓
DRAM / HBM
```

성능 모델 핵심 식:
```
Latency ≈ max(Compute_cycles, SRAM_bandwidth_bound, DRAM_transfer / overlap_efficiency)
```

**LLM KV-cache의 HW 판단 메커니즘**

LLM의 데이터 접근은 막연히 랜덤이 아니다:
- Prefill phase: Q, K, V 순차 streaming, tile 단위 재사용 높음
- Decode phase: 과거 KV 전체 반복 접근, temporal locality 매우 강함

HW가 하는 것은 의미 기반 판단이 아닌 통계 + 관찰 기반 판단: "이 주소가 N회 이상 참조됐는가", "이 라인에 여러 consumer가 대기 중인가". 구현 블록: Reuse distance tracking, Multi-consumer aware MSHR, Stream vs Reuse 분리(Bypass logic), Sector/sub-line cache.

### AMBA 버스 인프라 (성능 티어별)

NPU 성능별 AMBA 구성의 기본 패턴: **(1) Control-plane / Data-plane 분리, (2) NPU 근처는 로컬/클러스터 인터커넥트로 짧고 굵게, (3) DRAM/캐시 일관성은 필요한 범위만.**

| 성능 티어 | 추천 인프라 |
|---|---|
| ~10 TOPS (Edge) | AXI4 + APB control, 단일 크로스바, SMMU 선택적 |
| 10–50 TOPS (Mobile) | L0(NPU 내부 로컬 버스) + L1(AXI Switch, NPU 서브시스템) + L2(SoC NoC), QoS 2레벨 |
| 50–200 TOPS (고성능 SoC) | SoC Top은 NoC, NPU는 다중 포트 AXI, DRAM 채널 인터리빙 |
| 200 TOPS+ (서버/칩렛) | 온패키지 NoC + 다이-다이 인터커넥트 + PCIe/CXL, AMBA는 다이 내부 로컬 패브릭 |

멀티레벨 버스 권장 템플릿:
```
[CPU Cluster]──[SLC/L3]──[SoC NoC (L2)]──[NPU Subsys AXI Switch (L1)]──[NPU Cluster Bus / TE/VE/SRAM (L0)]
```

버스 성능 결정 요소: Read/Write 포트 분리, 트래픽 성격별 포트 분리(Weight / Activation-KV / Writeback), QoS를 정책으로 설계, Outstanding/ID 폭 설정, Coherency는 정확히 필요한 곳만(non-coherent DMA → ACE-Lite → CHI 순서로 점진 확장).

### Tesla Mixed-Precision Bridge 아키텍처 상세

특허(US20260017019A1) 핵심 구조:
```
Sensor / Memory
   ↓
[ Log-Encoded 8b Transport ]
   ↓
Local Accumulator / Geometry Unit
   ↓
[ 32b Reconstruction (Horner's Method) ]
```
- **Domain Transform**: FP32 linear → Log/Piecewise-log 공간. 지각·기하 정보는 절대값보다 비율·스케일 변화에 민감하므로 로그 공간에서 8-bit 표현으로도 정보 보존이 가능.
- **Transport 분리**: 8-bit는 "연산용"이 아니라 "운반용". DRAM BW, NoC, SRAM access 에너지를 동시에 절감.
- **Horner's Method**: 다항식 복원을 곱셈 최소화, 누산기 친화, 파이프라인화 가능한 형태로 구현. 32-bit 계산이 아니라 "32-bit 정확도를 갖는 결과를 생성".
- **Attention Sink**: 장시간 동작 시 로그 공간에서 곱셈이 덧셈으로 변환되어 분산 폭발이 억제되고 attention score drift가 감소. 구조적 필연.
- 전력 절감(500W → <100W for Optimus)의 대부분은 연산기가 아닌 **메모리 계층**에서 발생.

추론 가능한 미래 NPU 블록:
```
[ Log Encode Unit ]
[ Low-bit Transport Fabric ]
[ Polynomial Reconstruction Unit ]
[ Stable Attention Accumulator ]
```

### 시스템 모델링 도구: Synopsys Platform Architect

SoC/시스템 수준 아키텍처 탐색 도구. RTL 이전 단계에서 CPU·NPU·GPU·DSP·메모리·인터커넥트(AXI/NoC)를 트랜잭션/행위 수준으로 모델링하여 정량적 병목 분석을 수행한다.

NPU 설계 관련 활용 시나리오:
- TE/VE 분리 구조, Prefill vs. Decode 트래픽 모델, DMA vs. NoC 선택
- 온칩 SRAM 분할/분산 탐색, LLC 크기 영향, 압축/양자화 효과
- AXI 멀티레벨, NoC 토폴로지·VC·QoS 정책, AI burst 트래픽 모델

한계: Cycle-accurate가 아님, 모델 정확도에 의존, 학습 비용 존재.

권장 워크플로우: Platform Architect로 상위 시스템 탐색 → 후보 아키텍처를 cycle-accurate Python/RTL 시뮬레이터로 정밀 검증 → 결과를 다시 상위 모델에 피드백.

---

## Key Findings [coverage: high -- 8 sources]

1. **TOPS보다 중요한 것은 Latency Predictability와 Memory Locality**: Tesla의 교훈. 워크로드 고정 + 컴파일 타임 정적 스케줄 + SRAM scratchpad 구조는 TOPS가 낮아도 실질 성능에서 우위를 가질 수 있다.

2. **아키텍처 단순성이 세대 전환 속도를 결정**: Tesla AI5→AI6의 9개월 주기 목표는 "ISA 고정 + 정적 스케줄 + 제한된 상태공간"에서 비롯된다. GPU/TPU 대비 검증 범위가 극히 제한적이어서 가능하다.

3. **Mixed Precision의 본질은 bit-width가 아니라 좌표계 재정의**: "고정밀 지능은 고정밀 연산이 아니라, 정보가 안정적으로 머무는 좌표계에서 나온다." 로그 도메인 변환은 압축이 아닌 좌표계 변경이다.

4. **Edge AI의 본질은 연산 축소가 아니라 Memory Traffic 통제**: Tesla Mixed-Precision Bridge의 전력 절감은 연산기 개선이 아닌 메모리 계층(DRAM BW, NoC, SRAM access 에너지) 절감에서 발생한다.

5. **SRAM-only는 false dichotomy**: "SRAM-only vs. Cache-only"는 잘못된 이분법. 현실의 최적 설계는 핵심 연산 경로는 SW-managed scratchpad, 불확실성 흡수는 HW-managed cache인 혼합 구조다.

6. **TPU Sparse Core = 연산 추상화의 재정의**: "희소 연산을 빠르게"가 아닌 "불규칙한 메모리 접근을 하드웨어 내부에서 규칙적인 스트리밍 파이프라인으로 승격"한 구조. DRAM 피크 BW는 동일하나 유효 BW는 GPU 대비 2–5× 개선.

7. **Versal은 LLM 시대의 경계선**: Static, deterministic, streaming-centric 아키텍처로 "Transformer 이전 시대의 AI"에 최적. KV-cache random access와 dynamic shape에 구조적으로 불리. 그러나 tile 기반 NPU, compiler-driven architecture, SoC-level NoC 통합을 현실에서 검증한 유일한 상용 플랫폼.

8. **컴파일러가 NPU의 절반 이상**: FINN의 Folding, Versal의 Vitis, Tesla의 static schedule이 공통적으로 보여주는 교훈. "HW는 고정, 유연성은 컴파일러에서."

9. **LLM의 복잡성은 의미가 아니라 통계**: HW는 LLM 데이터 접근의 "의미"를 이해하지 못하지만, 재사용 가능성(reuse likelihood)은 통계적으로 판단 가능하다. Cache가 작동하는 이유.

---

## Connections [coverage: medium -- 4 sources]

- [[../../wiki/AI-Hardware/Architecture]] — NPU 아키텍처 카테고리 상위 노드
- **Quantization / Mixed Precision**: Tesla Mixed-Precision Bridge는 GenAI 압축 연구([[../../wiki/GenAI/Compression]])와 직접 연결. 로그 도메인 표현은 PTQ/QAT 기법과 공통 관심사.
- **KV-Cache 최적화**: Decode-phase 메모리 접근 패턴, KV-cache pinning/aging/sectoring 설계는 LLM 서빙 최적화 토픽과 겹침.
- **RISC-V NPU SoC 시뮬레이터**: Tesla의 "RISC-V + NPU + 정적 스케줄" 방향과 [[../../wiki/AI-Hardware/Simulator]] 카테고리의 연구 주제가 철학적으로 일치.
- **NoC / AMBA Bus**: AMBA Recommendation 파일에서 다룬 멀티레벨 버스 설계는 [[../../wiki/concepts]] 의 인터커넥트 토픽으로 확장 가능.
- **Chiplet / MCM**: AI6의 Chiplet 방향과 Research/Patent-MCM 카테고리의 특허 연구가 연결됨.
- **Synopsys Platform Architect**: RTL 이전 시스템 탐색 도구로서 NPU 시뮬레이터 연구와 상보적 위치.

---

## Open Questions [coverage: medium -- 4 sources]

1. **AI6 Chiplet 아키텍처의 실제 구현 방식**: Camera-heavy vs. Robot-heavy SKU의 조합 방법, 동일 ISA 유지 하에서 chiplet 간 인터페이스 설계 (die-to-die bandwidth, coherency).

2. **LLM decode phase에서의 SRAM-only NPU 한계 극복**: "L2-like shared SRAM"을 cache라고 부르지 않고도 reuse tracking / streaming bypass를 구현하는 최소 HW 세트가 무엇인가?

3. **Tesla Mixed-Precision Bridge의 학습 호환성**: log-domain transport와 Horner reconstruction이 학습(QAT) 단계에서도 gradient 안정성을 유지할 수 있는가?

4. **Versal의 LLM 확장 가능성**: KV-cache random access를 AIE tile local SRAM으로 처리하기 위한 타일 수/크기 요구사항. dynamic shape 처리를 위한 PL 활용 전략.

5. **TPU Sparse Core의 NPU 이식**: "Index-driven Engine"을 TE/VE 분리형 NPU에 추가할 때의 아키텍처 오버헤드(면적, 전력, NoC 포트 수)와 워크로드 커버리지.

6. **Horner Reconstruction과 Chaos/Polynomial 확장의 결합**: Log-domain 변환 + chaos 기반 동역학 + polynomial correction을 하나의 NPU ISA 확장으로 통합할 때의 실험적 검증 방법.

7. **AMBA QoS와 AI burst 트래픽의 실시간 공존**: Display/ISP 같은 실시간 클라이언트와 NPU의 대용량 burst 트래픽이 동일 NoC를 공유할 때 QoS 정책 최적화.

8. **Sparse Engine에서 DRAM 연속 배치(Offline Repacking)의 범용성 한계**: 정적 workload에서는 sparse→dense stream 변환이 가능하지만, 런타임에 인덱스가 변하는 동적 workload에서의 한계와 대안.

---

## Sources [coverage: high -- 8 sources]

1. [[../../raw/AI-Hardware/Architecture/Xilinx FINN Overview]] — FINN 프레임워크: dataflow-first FPGA QNN 가속기 생성, 컴파일 스택, HW 아키텍처 특징, 범용 NPU와의 비교
2. [[../../raw/AI-Hardware/Architecture/Tesla AI Chip Roadmap]] — Tesla AI5/AI6 로드맵: 아키텍처 분석, 9개월 세대 전환 이유, TE/VE/Scalar 분리 방향, 삼성 파운드리 연계
3. [[../../raw/AI-Hardware/Architecture/Tesla Edge AI Innovation]] — Tesla Mixed-Precision Bridge 특허 분석: 로그 도메인 변환, Horner 복원, Attention Sink, 메모리 트래픽 중심 전력 절감, NPU 블록 설계 함의
4. [[../../raw/AI-Hardware/Architecture/Synopsys Platform Architect Introduction]] — Synopsys Platform Architect: SoC 수준 아키텍처 탐색, NPU/NoC/메모리 모델링, Synopsys 생태계 내 위치
5. [[../../raw/AI-Hardware/Architecture/AMD Versal ACAP Overview]] — AMD Versal ACAP: AIE 타일 구조, Hard NoC, PL 슬롯, 컴파일러 중심 SoC 플랫폼, LLM 한계와 streaming AI 강점, 버스 변경 가능 범위
6. [[../../raw/AI-Hardware/Architecture/NPU Performance-based AMBA Recommendation]] — NPU 성능 티어별 AMBA 버스 인프라 추천, 멀티레벨 버스 템플릿, QoS 정책, Coherency 선택 기준
7. [[../../raw/AI-Hardware/Architecture/Memory Hierarchy in AI]] — H100/B100/Qualcomm AHPM/AMD MI300 메모리 계층 비교, SRAM vs Cache 선택 철학, LLM 접근 패턴 HW 판단 메커니즘, KV-cache 특화 HW 블록
8. [[../../raw/AI-Hardware/Architecture/TPU Sparse Core Explanation]] — Google TPU Sparse Core: 구조, Dense vs Sparse Core 역할 분리, 불규칙 접근의 내부 스트리밍 변환, 유효 대역폭 개선, NPU 이식 방향
