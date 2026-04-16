---
title: NPU Simulator & Compiler
type: topic
status: canonical
last_compiled: 2026-04-12
---

# NPU Simulator & Compiler

*last_compiled: 2026-04-12 | sources: 8*

---

## Summary [coverage: high -- 8 sources]

NPU 시뮬레이터와 컴파일러는 AI 하드웨어 가속기 개발의 두 핵심 축이다. **컴파일러**는 DNN 프레임워크(PyTorch, JAX, TensorFlow)에서 출발하여 표준 IR(ONNX, StableHLO, MLIR)을 거쳐 타일링·스케줄링·코드젠을 수행하고 최종적으로 NPU가 실행할 바이너리 또는 command packet을 생성한다. **시뮬레이터**는 실리콘 이전 단계에서 컴파일된 워크로드를 사이클 단위로 실행해 성능과 에너지를 예측한다.

이 토픽에서 다루는 주요 플랫폼·접근:

| 플랫폼 | 타깃 | 핵심 철학 |
|---|---|---|
| **Google Coral NPU** | Edge AI (mW급) | RISC-V + MLIR 풀스택 오픈소스 |
| **Google TPU** | Training + Inference | XLA 정적 컴파일 + Systolic Array |
| **Meta MTIA** | RecSys / LLM Inference | 메모리 중심(DRAM BW 우선) |
| **AWS Inferentia / Trainium** | Cloud Inference / Training | Neuron SDK, Semi-static scheduling |
| **HyperAccel LPU** | LLM Inference (Decode) | Dataflow + 메모리 스트리밍 |
| **Groq LPU** | Deterministic Inference | 컴파일러가 모든 cycle을 고정 |
| **슈퍼노드(Supernode)** | 개념/컴파일러 추상화 | HW-aware 연산 묶음 단위 |
| **Transformer C 구현** | 교육적 Low-level 참조 | 프레임워크 없이 Transformer 구현 |

---

## Core Concepts [coverage: high -- 8 sources]

### 슈퍼노드(Supernode) — "그래프와 하드웨어 사이의 실행 계약"

슈퍼노드는 NPU 컴파일러에서 **그래프 레벨 최적화와 하드웨어 친화적 실행 단위를 연결하는 핵심 추상화**다. 단순한 연산 fusion이 아니라, 메모리·스케줄·ISA·마이크로아키텍처 제약을 동시에 만족하도록 설계된 실행 블록이다.

```
Supernode
 ├─ Input Tile Load (DMA)
 ├─ Tensor Engine (MatMul)
 ├─ Vector Engine (Add / Bias / Norm)
 ├─ Activation Unit (GELU / ReLU)
 ├─ Optional Reduce / Scale
 └─ Output Tile Store (DMA)
```

중간 텐서는 외부 메모리(DRAM)로 나오지 않고 NPU 내부 로컬 SPM(Scratchpad Memory)에만 존재한다. **슈퍼노드 경계 = 메모리 트래픽 경계**다.

**슈퍼노드 생성 과정(컴파일러)**:
1. 그래프 분석 (topological order, shape/stride/broadcast)
2. Fusion 후보 탐색 (producer-consumer chain, elementwise+reduction 패턴)
3. HW 제약 필터링 — SPM 용량, tile 크기, TE↔VE 파이프라인 연결 가능성, precision 호환성 확인

**개념 비교**:

| 개념 | 의미 | 한계 |
|---|---|---|
| Operator | 단일 연산 | 오버헤드 큼 |
| Fusion | 단순 연산 결합 | HW 제약 고려 부족 |
| **Supernode** | HW-aware 실행 블록 | 메모리·파이프라인·ISA 반영 |

### Prefill / Decode 슈퍼노드 분리

LLM 추론은 본질적으로 두 개의 전혀 다른 워크로드다. 동일한 슈퍼노드 구조를 쓰면 둘 중 하나는 반드시 손해를 본다.

| 구분 | Prefill Supernode | Decode Supernode |
|---|---|---|
| 크기 | 큼 (coarse-grained) | 작음 (fine-grained) |
| 주 연산 | GEMM | GEMV |
| 병목 | Compute | Memory (KV-cache) |
| SPM 사용 | Aggressive | Minimal |
| 스케줄러 | Static | Dynamic / token-driven |
| 목표 | Throughput 최적화 | Token latency 최소화 |

IR 레벨에서는 슈퍼노드 타입을 명시적으로 분리한다:
```
npu.supernode @attn_prefill  { mode = "prefill",  tile = (Tq, Tk) }
npu.supernode @attn_decode   { mode = "decode",   streaming = true }
```

### 타일 기반 NPU와 슈퍼노드의 관계

타일 기반 NPU에서 슈퍼노드 = "타일 라이프사이클을 공유하는 연산 묶음"이다. 슈퍼노드 내부에서만 double buffering, partial sum accumulation, data reuse가 일어난다. 이를 슈퍼노드 단위 성능 모델링의 기본 단위로 삼으면 compute-bound / memory-bound 판별과 energy per supernode 추정이 가능하다.

### 컴파일러 IR 계층 구조

커스텀 NPU를 위한 SW 스택의 표준적 계층:

```
Framework (TF / PyTorch / JAX)
        ↓
  StableHLO / ONNX (외부 표준)
        ↓
   MLIR Middle-end (타일링·스케줄링·버퍼라이제이션)
        ↓
   Target-specific Lowering (NPU MLIR Dialect)
        ↓
  Binary / Command Packet
```

각 계층은 독립적으로 교체 가능하다. NPU-specific pass와 back-end를 명확히 분리함으로써 시뮬레이터→컴파일러→런타임이 단일 통일된 파이프라인을 구성한다.

### 정적(Static) 컴파일 vs 동적(Dynamic) 스케줄링

| 방식 | 대표 사례 | 특징 |
|---|---|---|
| 완전 정적 (XLA) | TPU, Groq | 모든 loop/tile/schedule을 컴파일 시 결정. 예측 가능한 latency |
| Semi-static | AWS Inferentia | 정적 컴파일 기반이지만 cloud 워크로드 다양성 고려 |
| 메모리 중심 정적 | HyperAccel LPU | dataflow planner 역할, memory access를 사전 계산 |
| 유연 (TVM) | 연구/프로토타이핑 | 자동 튜닝(autotune) + 커널 자동 생성 |

TPU의 본질은 "컴파일러가 마이크로아키텍처를 직접 조종하는 기계"이며, Groq LPU는 이를 극단까지 밀어붙여 worst-case latency = best-case latency를 실현한다.

### Transformer C 구현의 의의

C 언어로 Transformer를 직접 구현(llm.c 방식)하는 것은 어텐션, 선형 변환, 소프트맥스 등의 핵심 메커니즘을 프레임워크 없이 low-level로 이해하는 교육적 출발점이다. NPU 시뮬레이터 설계에서는 이 수준의 연산 흐름 이해가 컴파일러 백엔드 설계와 ISA 정의의 기반이 된다.

---

## Architecture [coverage: high -- 8 sources]

### 메모리 계층 설계

NPU/LPU 설계에서 메모리 계층 선택은 성능의 본질이다.

| 계층 | 역할 | 설계 선택 |
|---|---|---|
| Local SPM (Tile SRAM) | 슈퍼노드 내 중간 텐서, activation buffer | 용량 vs reuse 트레이드오프 |
| On-chip Unified Buffer | Weight/Activation 캐싱 | TPU v1: 28MB, v4: 수백 MB급 |
| DRAM (LPDDR5X / HBM) | Weight + KV cache 상주 | HyperAccel: LPDDR5X ($/token 효율), GPU/TPU: HBM (BW 우선) |

**LPU 카드 구성 (HyperAccel)**:
```
┌───────────────────────────────┐
│           LPU Card             │
│  ┌───────────────┐            │
│  │   LPU ASIC    │            │
│  │ (Compute Core)│            │
│  └───────┬───────┘            │
│  ┌───────▼───────┐            │
│  │   Local DRAM  │ ← LPDDR5X  │
│  │ (Weight + KV) │            │
│  └───────────────┘            │
│  PCIe / ESL Interface         │
└───────────────────────────────┘
```

각 카드(= pipeline stage)는 DRAM을 독립적으로 소유한다. 카드 간에는 activation만 이동한다.

**DRAM 용량 산정 공식 (LPU 카드)**:
- Weight(INT8, d=4096): ~201 MB/layer → 8 layers/stage = ~1.6 GB
- KV cache(S=4096 tokens): 8 KB/token/layer × 8 layers × 4096 ≈ 0.25 GB
- 합계: ~2.0 GB/stage → 실제 제품은 8~16 GB DRAM이 자연스러운 선택

### 데이터플로우 패턴

**Systolic Array (TPU 방식)**:
- 대규모 행렬곱 가속. 데이터 재사용 극대화. 제어 로직 거의 없음. 완전 정적 파이프라인.

**Dataflow & Memory-centric (HyperAccel LPU 방식)**:
- 순차적 weight streaming, KV cache access pattern 고정, 정확한 prefetch
- `DRAM → Stream Buffer → Tile SRAM → Compute` 파이프라인
- DRAM BW utilization ~90% 달성 가능

**Near-Memory Compute (MTIA 방식)**:
- Embedding table lookup 최적화. BW/Compute 비율을 GPU 대비 높임. FLOPs 모델보다 bytes/cycle 모델이 핵심.

### Coral NPU 아키텍처 (RISC-V 기반 풀스택)

Coral NPU의 하드웨어 구성:
- **Scalar Core** (경량 RISC-V): C 코드로 제어/관리 루프 실행. run-to-completion 실행 모델.
- **Vector Execution Unit** (RISC-V Vector): SIMD 기반 벡터 연산 가속.
- **Matrix Execution Unit** (MAC Engine): 양자화 중심의 행렬 연산 가속.

이 Scalar + Vector + Matrix 조합은 "프로그래머블 NPU"의 원형(proto-NPU)이다. Edge AI 타깃이므로 단일 스트림, 작은 모델, SRAM 중심, DRAM 접근 최소화를 전제로 설계됐다.

### LPU Core Tile 구조 (HyperAccel)

```
┌─────────────────────────┐
│ LPU Core Tile           │
│  ┌──────────────┐       │
│  │ MatMul Engine│  ← GEMM / MxV 최적화
│  └──────────────┘       │
│  ┌──────────────┐       │
│  │ Vector Engine│  ← RMSNorm, RoPE, Softmax
│  └──────────────┘       │
│  ┌──────────────┐       │
│  │ Local SRAM   │  ← KV / activation buffer
│  └──────────────┘       │
└─────────────────────────┘
```

설계 철학: 범용성 감소, 제어 복잡도 감소, latency predictability 증가.

### DMA와 스크래치패드 사용 패턴

슈퍼노드 구조에서 DMA는 tile load/store를 담당하며, 슈퍼노드 내부 연산 중에 다음 타일을 prefetch하는 **double buffering** 패턴이 기본이다. 클러스터 확장 시에는 SRAM banking + AXI outstanding 튜닝이 성능 보장의 핵심이 된다.

---

## Key Findings [coverage: high -- 8 sources]

### HyperAccel LPU — Decode-first 철학

- LLM 추론(decode)은 본질적으로 **memory-bound streaming workload**다. 연산보다 KV-cache fetch + weight streaming이 지배적이다.
- LPDDR5X를 의도적으로 선택한 이유: HBM 대비 pJ/bit 효율 우위, $/token 최적화. Decode 단계는 HBM급 BW가 반드시 필요하지 않다.
- **ESL (Expandable Synchronization Link)**: NVLink/PCIe가 아닌 token-level pipeline link. 레이어 기준 pipeline parallel로 모델을 분할하며, 카드 간에는 activation만 전달.
- Prefill은 GPU 또는 다수 LPU 병렬에 맡기고, Decode 전용 가속에 집중하는 것이 현실적 선택이다.

### Groq LPU — 컴파일러가 시간을 지배

- 완전 정적(static) 스케줄링: 컴파일 시 모든 연산의 시작 cycle, 메모리 이동 타이밍, 리소스 사용을 완전히 고정.
- Runtime scheduler 거의 없음 → latency variance ≈ 0, worst-case latency = best-case latency.
- 대형 온칩 SRAM으로 DRAM 접근 최소화. Prefill(GEMM) 성능이 매우 강함.
- 한계: 모델/컨텍스트 크기 제약, 확장성 제한.

**Groq vs HyperAccel 핵심 차이**:
> Groq는 "시간을 고정해 연산기를 100% 채우는" 단일칩 가속기이고, HyperAccel은 "메모리와 파이프라인으로 비용을 최소화하는" 시스템형 가속기다.

### Google Coral NPU — 풀스택 오픈소스의 교훈

- RISC-V 기반 NPU IP + MLIR/StableHLO 통합 컴파일러 + 개발자 경험 통합이 오픈소스로 제공된다.
- "커스텀 NPU + 오픈소스 SW 스택" 설계의 가이드레일 역할: 각 계층이 독립적이며 시뮬레이터→컴파일러→런타임이 단일 파이프라인.
- Coral 구조를 고성능 모바일 AP에 그대로 적용할 수 없는 이유: 단일 실행 모델, SRAM 전제, run-to-completion은 멀티 워크로드·DRAM/SLC 공유·LLM prefill/decode 요구사항과 불일치.
- 단, Coral의 핵심 철학(MLIR 중심 IR 설계, Scalar-controlled NPU)은 모바일 AP·서버 NPU 설계에서도 유효하다.
- 멀티 MatMul 클러스터로 Coral을 확장하려면 메모리 계층/대역폭, 클러스터 인터커넥트(NoC), 컴파일러 tile+cluster mapping, 런타임 큐/이벤트를 함께 설계해야 한다.

### Meta MTIA — 메모리 시스템 중심 가속기

- "연산기가 아니라 메모리 시스템을 중심으로 설계된 AI 가속기"
- RecSys Embedding table lookup에 특화. GPU 대비 BW/Compute 비율 높음, Compute peak 낮음.
- LLM 관점: Prefill 성능은 평범하고, Decode(KV-cache, embedding lookup)에서 유리.
- NPU 시뮬레이터 설계 시사점: FLOPs 모델보다 DRAM-SRAM traffic 모델링이 핵심. bytes/cycle을 1급 metric으로.

### Google TPU — 컴파일러-HW 공진화의 교과서

- 핵심 질문: "행렬곱만 빠르면 딥러닝은 끝나는가?" → 2015~2020 기간 동안 Yes였음.
- Systolic Array(MXU): 데이터 재사용 극대화, 제어 로직 최소화, 파이프라인이 clockwork처럼 동작.
- 완전 정적 컴파일(XLA): 런타임 스케줄링 없음. ISA보다 컴파일러 IR 설계가 우선.
- Tile 크기 = 성능 결정 변수. Prefill(GEMM) 매우 강함. Decode(MxV + KV-cache)에서 SRAM 부족 시 비효율.

### Transformer C 구현 — low-level 기반 이해

- C 언어로 직접 구현(llm.c)함으로써 어텐션, 소프트맥스, 행렬 변환의 계산 흐름을 프레임워크 없이 파악할 수 있다.
- 하드웨어 친화적 사고(hardware-friendly thinking)와 컴파일러 백엔드 영향 분석의 기반.

---

## Connections [coverage: medium -- 4 sources]

- [[../../raw/AI-Hardware/Architecture]] — NPU SoC 아키텍처 (버스/인터페이스, 메모리 계층) 토픽과 직접 연결됨. 슈퍼노드 경계 = 메모리 트래픽 경계는 SoC 메모리 설계까지 연결된다.
- [[../../wiki/concepts]] — 양자화(PTQ/QAT), pruning 등 GenAI/Compression 기법은 컴파일러의 precision 호환성 필터링 및 슈퍼노드 precision 정책과 연동된다.
- LLM Prefill/Decode 워크로드 특성 — `wiki/topics/` 내 LLM 추론 관련 토픽 (KV-cache layout, GQA/MQA, PagedAttention)과 연결.
- HW-Friendly 모델 설계 — GenAI/HW-Friendly 카테고리의 FlashAttention, RMSNorm, RoPE 등은 슈퍼노드 내 Vector Engine 설계와 직접 연관.
- RISC-V + VCIX 확장 ISA — Coral NPU의 RISC-V Vector ISA는 커스텀 opcode(VCIX) 기반 슈퍼노드 lowering과 연결.

---

## Open Questions [coverage: medium -- 3 sources]

1. **슈퍼노드 자동 분할 전략**: SPM pressure 기반 adaptive split, prefill vs decode용 다른 분할 기준을 어떻게 컴파일러에서 자동화할 것인가?

2. **슈퍼노드 단위 성능·에너지 모델**: compute-bound / memory-bound 판별, energy per supernode 추정 방법론. ROB retire 단위로 슈퍼노드를 사용하는 OOO dispatch 모델은 실현 가능한가?

3. **Groq식 cycle-level 결정성을 커스텀 NPU에서 얼마나 달성할 수 있는가**: 완전 정적 스케줄링의 이점을 가져오면서도 Groq의 모델 크기 제약을 극복하는 아키텍처 설계 방향은?

4. **Coral 기반 멀티 MatMul 클러스터의 성능 보장 조건**: Roofline 모델 → cycle/이벤트 시뮬 → end-to-end 컴파일+실행의 3단계 검증에서 각 단계의 허용 오차 기준은?

5. **LPU의 핵심 아이디어를 GPU에서 부분 구현할 때의 현실적 한계**: pipeline parallel과 memory streaming 최적화는 GPU에서도 가능하지만, cycle-level 결정성과 GEMV 특화는 구조적으로 불가능하다고 분석됐다. 중간 지점(예: CUDA Graph + static kernel fusion)의 실효성은?

6. **Prefill/Decode 분리형 NPU 스케줄러 설계**: KV-cache layout(GQA/MQA, interleaved heads)과 dual-path 스케줄러를 SoC 수준에서 어떻게 구현할 것인가?

7. **Edge → Mobile AP → Server NPU로의 확장 로드맵**: Coral(Stage 0)에서 LLM-aware 모바일 AP NPU(Stage 3), 나아가 서버 NPU(분산·가상화·서빙·신뢰성)로 진화하는 각 단계의 설계 결정 포인트는?

---

## Sources [coverage: high -- 8 sources]

1. [[../../raw/AI-Hardware/Simulator/MTIA TPU Paper Analysis 33a6cc566b0b816ebdbfedfedbb11fb6]]
2. [[../../raw/AI-Hardware/Simulator/NPU Supernode Concept 33a6cc566b0b81aca099f1e6a9a8061e]]
3. [[../../raw/AI-Hardware/Simulator/Transformer C Implementation Analysis 33a6cc566b0b818585ddf76217de5112]]
4. [[../../raw/AI-Hardware/Simulator/coral npu/coral npu (Part 1 2) 33a6cc566b0b813d98cee31bdda3eac4]]
5. [[../../raw/AI-Hardware/Simulator/coral npu/coral npu (Part 2 2) 33a6cc566b0b81d8a159d87e7e1b114c]]
6. [[../../raw/AI-Hardware/Simulator/HyperAccel LPU Explanation/HyperAccel LPU Explanation (Part 1 of 3) 33a6cc566b0b81349b96fd84f1659f77]]
7. [[../../raw/AI-Hardware/Simulator/HyperAccel LPU Explanation/HyperAccel LPU Explanation (Part 2 of 3) 33a6cc566b0b8184b3e7c80f64b99235]]
8. [[../../raw/AI-Hardware/Simulator/HyperAccel LPU Explanation/HyperAccel LPU Explanation (Part 3 of 3) 33a6cc566b0b81dbb100c4737c051345]]
