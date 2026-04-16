---
title: Project Helios Edge / Physical AI Custom SoC Platform Strategy
type: deep-dive
tags: [project-helios, edge-ai, physical-ai, custom-soc, NPU, ISP, modem, NoC, platform-strategy]
updated: 2026-04-15
---

# Project Helios Edge / Physical AI Custom SoC Platform Strategy Report

Canonical topics: [[topics/npu-architecture]], [[topics/npu-simulator-compiler]]
Related concepts: [[concepts/memory-bandwidth-bottleneck]], [[concepts/static-scheduling-determinism]]

## Version 1.0 (Final)

# 1. Executive Summary

Custom SoC 시장의 경쟁 축은 더 이상 단일 IP의 피크 성능 경쟁에 머물지 않는다.
고객은 "더 높은 TOPS" 자체보다, **빠른 제품화**, **낮은 설계 리스크**, **워크로드 특화 데이터플로우**, **SW까지 포함된 제품화 가능성**을 더 중시한다. 특히 Edge / Physical AI 영역에서는 sensor-to-AI-to-control path 전체가 제품 경쟁력의 핵심이 되며, NPU 하나의 성능만으로는 차별화가 어렵다.

Broadcom과 Marvell은 공통 플랫폼 자산과 Anchor 고객 중심 반복 수주 구조를 통해 빠르게 커스텀 제품을 확장하고 있다. Arm은 CPU IP 공급자를 넘어 CSS(Compute Subsystem) 기반의 platform provider로 진화하며, 고객의 SoC 개발 리스크와 bring-up 시간을 줄이고 있다. AMD/Xilinx Versal ACAP은 adaptive dataflow 구조를 통해 sensor -> preprocess -> AI -> control의 시스템 경로를 칩 내부에서 최적화하는 것이 얼마나 중요한지 보여준다.

이러한 환경에서 Project Helios의 전략은 단순히 경쟁사의 장점을 모방하는 것이 아니라, 고유 강점을 결합해 다음과 같이 정의되어야 한다.

> **Arm CSS 기반의 검증된 공통 플랫폼 위에, Helios NPU / ISP / Modem / Mobile-grade low-power integration을 계층적으로 결합하고, 고객 세그먼트별 Fabric Preset을 통해 Vision / Compute / Connected Edge를 빠르게 파생시키는 Edge / Physical AI Custom SoC 플랫폼을 구축한다.**

이 전략의 본질은 다음 세 가지다.

1. **Platformization** — 고객마다 완전히 새로운 칩을 설계하지 않는다. 공통 플랫폼과 공통 설계 자산 위에서 파생형 SKU를 생성한다.
2. **Differentiation** — Arm 공통 플랫폼 위에 Helios 고유 IP와 시스템 통합 역량을 결합해 차별화한다.
3. **Fabric Preset-based Customization** — 단순한 IP ON/OFF가 아니라, NoC / Bus topology 자체를 세그먼트별로 재사용 가능한 preset으로 제공한다.

## Strategic Thesis

> **Arm은 검증된 compute backbone과 software ecosystem을 제공하고, Helios는 vision, connectivity, mobile power, high-efficiency AI acceleration을 통해 고객이 실제 제품을 만들 수 있는 differentiation을 제공한다.**

이 문서의 핵심 결론은 다음과 같다.

- Project Helios는 단일 NPU vendor가 아니라 **Edge / Physical AI Platform Provider**가 되어야 한다.
- Arm CSS는 공통 backbone으로 활용하되, Helios의 경쟁력은 **NPU / ISP / Modem / Runtime / Fabric Preset**에서 확보해야 한다.
- Vision, Robotics, Connected Edge, Compute Gateway를 하나의 monolithic bus 구조로 커버하려 하기보다, **재사용 가능한 Fabric Preset 집합**으로 대응해야 한다.
- 플랫폼 성공의 핵심은 HW block의 개수보다 **runtime-first execution**, **NoC/dataflow 최적화**, **Anchor 고객 반복 수주 모델**에 있다.

# 2. Strategy Context and Competitive Benchmark

## 2.1 Why this strategy is needed

Custom SoC 시장에서 단기간 경쟁 우위를 확보하려면, 단순히 "좋은 NPU"를 만드는 것으로는 부족하다. 실제 고객이 원하는 것은 다음과 같다.

- 개발 리스크가 낮은 검증된 베이스
- 빠른 PoC와 빠른 Tape-out 가능성
- 특정 워크로드에 최적화된 차별화 IP
- 고객별 요구를 수용할 수 있는 플랫폼형 구조
- software stack과 runtime까지 포함된 제품화 가능성

즉, 승부 포인트는 단일 IP 성능이 아니라 다음의 결합이다: 공통 플랫폼, 차별화 블록, 빠른 파생 구조, 통합된 SW/HW 실행 환경.

## 2.2 Broadcom benchmark

### 핵심 특징

- Anchor 고객 중심의 반복 수주 구조
- RF / Connectivity / Custom silicon 중심
- 고객 맞춤형 SKU 파생 능력
- 공통 설계 자산의 재사용

### 시사점

Broadcom의 강점은 "고객마다 완전히 새로운 구조를 매번 만드는 것"이 아니라, **공통 플랫폼 자산을 고객 요구에 맞게 빠르게 변형하는 운영 모델**에 있다.

### 약점 / Helios의 기회

Broadcom은 무선 연결성과 커스텀 실리콘 측면에서는 강하지만, **edge-specific vision pipeline**, **ISP와 NPU의 tight integration**, **sensor-to-AI streaming 구조** 측면에서는 Helios 대비 구조적 우위를 갖기 어렵다.

### Helios에 주는 인사이트

- Anchor 고객 1곳을 잡으면, 그 안에서 세대별 / 용도별 / 전력별 SKU를 확장할 수 있어야 한다.
- SoC 전략은 "칩 하나"가 아니라 "제품군 생성 엔진"이어야 한다.
- Broadcom이 강한 SKU 파생 운영 모델은 벤치마크하되, Helios는 **ISP + modem + low-power edge integration** 쪽에서 차별화해야 한다.

## 2.3 Marvell benchmark

### 핵심 특징

- Data center / infra 중심 custom ASIC
- IP catalog 기반 설계
- SerDes, memory, networking, packaging 자산의 조합
- Time-to-market을 위한 공통 자산 재사용
- OCTEON / OCTEON Fusion 기반 carrier / edge infra 확장
- 5G O-RAN / Baseband 시장에서의 존재감

### 시사점

Marvell은 커스텀 반도체를 "새로운 설계 프로젝트"라기보다 **플랫폼 기반의 조합 문제**로 다룬다. 특히 네트워킹, baseband, carrier infrastructure에서 축적된 자산을 IP catalog 형태로 조합해 빠르게 고객 맞춤형 솔루션을 제공한다.

### 경쟁 구도 분석

| 영역 | Marvell | Helios |
|---|---|---|
| O-RAN / Baseband Infra | 강함 | 제한적 |
| Device-side Modem | 약함 | 강함 |
| Mobile / Edge Integration | 상대적으로 약함 | 강함 |
| Vision-centric Edge Platform | 제한적 | 강점 가능 |
| Carrier / Networking Fabric Heritage | 강함 | 상대적으로 약함 |

### Helios에 주는 인사이트

- NPU, ISP, CPU, memory subsystem, NoC, I/O, package를 **catalog화**해야 한다.
- 설계 산출물뿐 아니라 verification asset, performance model, compiler/runtime interface도 재사용 자산으로 축적해야 한다.
- Marvell이 infra-side에서 강하다면, Helios는 **device-side modem + mobile-grade integration + vision edge**를 중심으로 차별화할 수 있다.

## 2.4 Arm benchmark

### 핵심 특징

- CPU IP 공급자에서 platform provider로 진화
- CSS(Compute Subsystem) 기반 사전 검증된 구조 제공
- software ecosystem와 함께 제공
- 고객의 설계 리스크와 bring-up 시간을 줄이는 모델

### 시사점

Arm의 핵심 메시지는 다음과 같다.

> "고객이 SoC를 더 빨리, 더 적은 리스크로 만들 수 있게 한다."

### Helios에 주는 인사이트

- Arm을 단순 CPU vendor로 보면 안 된다.
- Arm CSS는 Helios custom SoC의 **공통 backbone**으로 활용해야 한다.
- Helios의 가치는 Arm과 경쟁하는 것이 아니라, Arm 기반 플랫폼 위에 **Helios differentiation layer**를 얹는 데 있다.
- Project Helios는 Arm SME2 ecosystem의 adopter로 이미 참여하고 있으며, 이는 CPU matrix path와 Helios NPU의 계층형 전략 실현 가능성을 높인다.

## 2.5 AMD/Xilinx Versal ACAP benchmark

### 핵심 특징

- CPU + programmable logic + AI engine + NoC
- sensor -> preprocess -> AI -> control의 adaptive dataflow
- latency, bandwidth, streaming 효율에 강점
- 워크로드별 데이터플로우 최적화

### 시사점

ACAP의 핵심 가치는 "FPGA가 있다"가 아니다. 진짜 가치는 다음이다.

- sensor-to-AI-to-control path를 칩 내부에서 최적화
- DRAM round-trip 최소화
- streaming 중심 시스템 구조
- workload-adaptive architecture

### Helios에 주는 인사이트

Helios가 full FPGA/ACAP를 만들 필요는 없다. 하지만 다음 철학은 반드시 흡수해야 한다.

- streaming path 분리
- shared SRAM 중심의 on-chip reuse
- vision/AI/control domain 분리
- direct sensor-to-NPU path
- 데이터 이동 구조를 아키텍처 차별화 자산으로 다루는 접근

## 2.6 Benchmark summary

| Company | Primary Strength | Business Model | Helios takeaway |
|---|---|---|---|
| Broadcom | Anchor account + fast SKU derivation | Customer-specific repeated variants | 고객 1곳당 다수 SKU 파생 구조 필요 |
| Marvell | IP / platform reuse + infra lineage | Catalog-based custom ASIC | IP + interconnect + SW stack까지 catalog화 필요 |
| Arm | Validated subsystem + SW ecosystem | Platform provider | CSS를 공통 backbone으로 활용 |
| AMD/Xilinx | Adaptive dataflow | Heterogeneous streaming architecture | sensor/AI/control path와 streaming fabric 구조 흡수 |

# 3. Strategic Positioning for Project Helios

## 3.1 Core positioning

Project Helios가 취해야 할 포지션은 다음과 같다.

> **Edge AI accelerator vendor**가 아니라 **Edge / Physical AI Platform Provider**

즉, 고객에게 제공해야 하는 것은 단일 NPU가 아니라 다음의 조합이다: Arm 기반 공통 compute platform, Helios 차별화 IP, Fabric preset 기반 SKU 파생 구조, runtime / compiler / profiling을 포함한 software-ready platform.

## 3.2 Helios-specific strengths

Helios가 Broadcom, Marvell, Arm, AMD/Xilinx와 비교해 차별화할 수 있는 포인트는 다음과 같다.

### 현재 확보 역량

**1. Mobile-grade integration competence** — 전력 최적화, 모바일/엣지 환경에서의 integration 경험, 실제 제품화 가능한 thermal / power budget 대응.

**2. ISP / multimedia strength** — camera ingest, denoise / HDR / resize / warp, vision front-end와 AI 연계 가능성.

**3. Modem / connectivity option** — always-connected edge device, XR / wearable / smart camera / mobile edge 대응, 고객에 따라 optional integration 가능.

**4. Helios NPU differentiation** — tile-based SRAM-centric architecture, ISP direct stream path와의 결합 가능성, deterministic latency 중심 최적화.

### 잠재적 확장 역량

**5. Potential package / memory / foundry alignment** — memory / package-aware architecture planning, 공정 / 패키징 / 대역폭 전략과의 상호 최적화 가능성. 이 항목은 현재 확보된 역량이 아닌 향후 실현 가능한 잠재적 우위로 분류한다.

## 3.3 Strategic thesis

Helios의 차별화 전략은 다음 문장으로 정리할 수 있다.

> **Arm은 공통 플랫폼의 속도와 안정성을 제공하고, Helios는 vision, connectivity, mobile power, and high-efficiency NPU를 통해 고객이 실제 제품을 만들 때 필요한 differentiation을 제공한다.**

# 4. Target Market and Customer Segmentation

## 4.1 Why not compete head-on in every market

Broadcom/Marvell과 동일하게 모든 고객, 모든 시장을 한 번에 겨냥하는 것은 비효율적이다. 초기 전략은 **Edge / Physical AI에 특화된 세그먼트**를 중심으로 설정해야 한다.

## 4.2 Recommended target segments

### Segment A: Vision Edge

예: smart camera, industrial vision, retail analytics camera, AI inspection device

특징: multi-camera ingest, ISP 중요, low-latency sensor-to-AI path 중요, NPU와 ISP의 직접 연결 가치 큼.

### Segment B: Physical AI / Robotics

예: AMR, drone, service robot, industrial robot controller adjunct

특징: real-time control 중요, vision optional, RT CPU + NPU + sensor fusion 중요, deterministic latency 중요.

### Segment C: Connected Mobile Edge

예: XR device, wearable AI, connected smart device, field AI terminal

특징: modem / connectivity 중요, low power 중요, camera / AI / network가 함께 존재, mobile-grade integration이 경쟁력.

### Segment D: Compute / Gateway Edge

예: AI gateway, local inference box, sensor hub gateway, industrial controller with AI

특징: camera 미탑재 가능, connectivity / PCIe / Ethernet 비중 큼, NPU↔DDR↔I/O path 최적화 중요.

## 4.3 Customer strategy

### 핵심 원칙

- 다수 고객을 얕게 상대하지 않는다.
- Anchor 고객 1~2개를 깊게 가져가고, 그 안에서 SKU를 확장한다.

### 목표 운영 모델

- Customer A -> Vision variants
- Customer B -> Robotics variants
- Customer C -> Connected edge variants

> **"고객 수 확대"보다 "고객별 SKU 확장"이 우선이다.**

# 5. Platform Architecture Overview

## 5.1 Layered architecture model

전체 플랫폼은 다음 3개 계층으로 정의한다.

**Layer 1. Arm Base Platform Layer** — 공통 backbone. 변경을 최소화하고 검증 자산을 재사용하는 층.

**Layer 2. Helios Differentiation Layer** — Helios 고유 IP와 최적화 자산이 위치하는 층. 고객 차별화와 성능/전력 우위를 만드는 핵심 층.

**Layer 3. Customization / Fabric Preset Layer** — 고객별 / 세그먼트별로 변형되는 층. 단순 IP ON/OFF뿐 아니라 fabric topology, QoS, bandwidth shape까지 포함.

## 5.2 Platform block view

```text
+-------------------------------------------------------------------+
| Project Helios Edge / Physical AI Custom SoC Platform             |
+-------------------------------------------------------------------+
| Layer 1: Arm Base Platform Layer                                  |
|  - Application CPU Cluster                                        |
|  - Real-Time / Control CPU Cluster                                |
|  - Coherent System Fabric                                         |
|  - System Cache / Memory Controller / Security / Debug            |
+-------------------------------------------------------------------+
| Layer 2: Helios Differentiation Layer                             |
|  - Helios NPU Subsystem                                           |
|  - Helios ISP / Vision Front-End                                  |
|  - Helios Modem / Connectivity Option                             |
|  - Helios DMA / Compression / Low-Power Optimization              |
+-------------------------------------------------------------------+
| Layer 3: Customization / Fabric Preset Layer                      |
|  - Vision-centric topology                                        |
|  - Compute-centric topology                                       |
|  - Connected-edge topology                                        |
|  - QoS / SRAM bank / bridge / DMA sequencer configuration         |
+-------------------------------------------------------------------+
```

# 6. Arm Base Platform Layer

## 6.1 Role of Arm CSS

Arm CSS는 이 플랫폼의 공통 기반이다. 이 층은 "차별화"보다 "검증"과 "재사용"이 중요하다.

포함 요소: application CPU cluster, control / RT CPU cluster, coherent interconnect, cache hierarchy, SMMU / security root, boot / firmware / standard driver baseline, Linux / Android / RTOS bring-up base.

목적: 개발 속도 향상, integration risk 감소, ecosystem 활용, software-first 접근의 기반 마련.

## 6.2 What should remain stable in the Arm layer

Arm Base Platform Layer는 SKU가 달라져도 최대한 유지되어야 한다.

유지 대상: CPU subsystem, coherent backbone, boot / security flow, base interrupt / debug infrastructure, basic memory controller structure, software stack entry point.

즉, 이 층은 platform reuse의 핵심이다.

## 6.3 What should not be overloaded into the Arm layer

다음은 Arm layer에 과도하게 넣지 않는 것이 좋다: vision-specific direct stream path, Helios NPU local dataflow policy, modem-specific power / peripheral domain detail, segment-specific QoS tuning.

이들은 Helios Differentiation 또는 Fabric Preset layer에서 처리하는 것이 적절하다.

# 7. Helios Differentiation Layer

## 7.1 Why this layer matters

Arm CSS만으로는 Helios가 고객에게 선택받을 이유가 부족하다. Helios의 differentiation layer는 다음 질문에 답해야 한다.

- 왜 Arm reference만 쓰지 않고 Helios와 해야 하는가?
- 왜 Broadcom / Marvell이 아니라 Helios여야 하는가?
- 왜 Helios 플랫폼이 edge / mobile / vision에서 더 유리한가?

## 7.2 Helios NPU subsystem

Helios NPU는 platform의 핵심 차별화 요소다. 단순 Ethos 대체품이 아니라, Ethos로 커버하기 어려운 high-efficiency / high-bandwidth / streaming AI path를 담당해야 한다.

### 아키텍처 구성

**Scalar Engine (SE)** — command decode, micro-sequence control, loop control, address generation, synchronization / barrier, VE / TE orchestration, DMA trigger / completion handling, irregular op control. SE는 NPU 내부의 작은 제어 프로세서다. Transformer, multimodal, reduction-heavy workload에서 필수적이다.

**Vector Engine (VE)** — activation, layernorm / rmsnorm, softmax, elementwise op, reduction, dequant / requant, data rearrangement. VE는 AI용 DSP / SIMD 벡터 엔진 역할을 담당한다.

**Tensor Engine (TE)** — GEMM, Conv, attention core matmul, dense MAC compute, tile-based matrix operation. TE는 peak compute를 담당하는 핵심 연산 블록이다.

**Memory / Dispatcher subsystem** — local SRAM / scratchpad, tile buffer, DMA / prefetch, dependency tracking, format conversion, compression / decompression, command queue / scoreboard. 실제 NPU 성능은 compute array보다 이 subsystem에서 크게 좌우된다.

### SRAM target and differentiation

- target SRAM 범위: 1~4MB (Phase 1에서 확정)
- 이는 Arm Ethos-U 계열(29~267KB 내장 RAM) 대비 현저히 큰 on-chip working set과 SRAM-centric dataflow를 지향하는 차별화 포인트로 활용 가능하다.

### Deterministic latency의 의미

Helios NPU는 단순히 높은 피크 처리량보다, 타일 기반 로컬 SRAM 재사용, 정해진 데이터 이동 경로, 예측 가능한 DMA/prefetch 동작을 통해 latency variability를 줄이는 방향으로 설계되어야 한다.

## 7.3 Helios ISP / Vision front-end

이 블록은 Vision Edge 고객에서 매우 중요하다.

역할: camera ingest, HDR, denoise, temporal filtering, resize / crop / CSC, optical flow / stereo / front-end CV, stream-to-SRAM ingress.

전략적 가치: sensor-to-AI latency 감소, DRAM round-trip 최소화, NPU 앞단 pre-processing offload, camera-centric customer 대응력 강화.

## 7.4 Helios Modem / Connectivity option

모든 SKU에 modem을 넣는 것은 비효율적일 수 있다. 하지만 Helios는 modem / connectivity를 optional differentiation asset으로 활용할 수 있다.

적용 가능 고객: XR, smart camera with backhaul, connected AI terminal, wearable / field robotics.

전략: modem always-on 필수 아님, platform option으로 제공, connected edge preset에서 강하게 활용.

## 7.5 Helios-specific optimization assets

다음 요소도 differentiation layer에 포함되어야 한다: streaming DMA sequencer, memory bandwidth reduction logic, compression support, power island optimization, mobile-grade DVFS hooks, platform-aware runtime tuning.

# 8. Compute Hierarchy Strategy

## 8.1 Recommended compute hierarchy

단일 compute path로 통일하면 고객 스펙트럼을 커버하기 어렵다. 따라서 다음 4-tier compute hierarchy가 적절하다.

1. Arm CPU General Path
2. Arm Matrix-Accelerated CPU Path
3. Ethos Baseline NPU Path
4. Helios High-Performance NPU Path

## 8.2 CME / SME2 조건 명시

Arm CPU Matrix Path는 모든 Arm CSS에서 자동으로 성립하는 것이 아니다. 이 경로는 선택한 CSS / core 옵션에 따라 가능 여부가 달라진다.

문서의 기본 가정은 다음과 같다.

> CME는 Lumex CSS의 C1-Pro Core 계열 선택 시 활성 가능한 matrix acceleration path로 간주한다.

Project Helios는 Arm SME2 ecosystem의 adopter로 이미 참여하고 있으며, 이는 이 전략의 실현 가능성을 높이는 근거로 활용할 수 있다.

참고: C1-Pro(mid-tier)와 C1-Ultra(flagship)는 SME2 기반 대형 모델 추론이 가능하나, C1-Nano(효율 코어)는 1~3B 파라미터 이하의 소형 모델이 현실적 상한이다. 따라서 CPU Matrix Path의 실제 처리 범위는 채택하는 코어 구성에 따라 달라진다.

## 8.3 Role definition

| Compute path | Role | Strength | Limitation |
|---|---|---|---|
| Arm CPU General Path | control, fallback, software portability | 개발 유연성, 일반성 | efficiency 낮음 |
| Arm CPU Matrix Path | small-mid matrix-heavy task, on-CPU SLM inference | dedicated matrix acceleration, SW 친화성 | core/CSS option 의존 |
| Ethos Baseline NPU Path | fast bring-up, baseline AI SKU | 빠른 TTM, Arm SW 연계 | customization 한계 |
| Helios High-Performance NPU Path | high-efficiency AI / streaming path | 성능/전력/데이터플로우 최적화 | HW/SW co-design 필요 |

## 8.4 Strategic use of each path

**Arm CPU General Path** — control-heavy task, fallback execution, light AI, software-defined pipeline, irregular operation.

**Arm CPU Matrix Path** — small-mid GEMM, matrix-heavy control-adjacent task, 코어 구성에 따라 약 1B~7B급 SLM 범위를 담당(C1-Nano: ~3B, C1-Pro/Ultra: ~7B), low-latency on-CPU inference path. 정확한 처리 범위는 선택한 core / CSS 조합과 memory budget에 따라 달라지며, 최종 범위는 후속 cycle spec 및 simulator에서 확정해야 한다.

**Ethos Baseline NPU Path** — 빠른 제품화가 필요한 entry/mid SKU, low-risk 고객 대응, Arm 기반 compiler/runtime와의 빠른 연계.

**Helios High-Performance NPU Path** — vision-heavy AI, edge multimodal, robotics / low-latency control-adjacent AI, SRAM-centric tile execution, direct stream path 활용.

## 8.5 Dispatch threshold concept

Unified runtime 설계의 출발점으로, 다음 개념적 threshold를 둔다.

> Threshold = single tile 내에서 유리하게 처리 가능한 GEMM 크기 상한

초기 정책 예시는 다음과 같다: small GEMM / tile-contained GEMM -> CPU Matrix Path, baseline CNN / transformer inference -> Ethos Path, large streaming / high-throughput / vision-heavy path -> Helios NPU Path.

정확한 수치는 simulator와 cycle model 기반으로 Phase 2에서 확정한다.

## 8.6 Strategic principle

Helios NPU는 Ethos를 단순히 대체하는 것이 아니라, Ethos로는 어려운 high-performance, high-efficiency, streaming-centric use case를 담당해야 한다.

# 9. Memory Hierarchy and Dataflow

## 9.1 Recommended memory hierarchy

```text
[NPU Local SRAM / Scratchpad]
        ↓
[Shared SRAM / Inter-engine Buffer]
        ↓
[External DRAM: LPDDR5X baseline]
```

향후 확장 버전에서는 memory speed / width / package option에 따라 scalable memory option을 검토할 수 있으나, 첫 플랫폼 세대의 baseline은 LPDDR5X로 두는 것이 현실적이다.

## 9.2 Design philosophy

핵심 철학: compute보다 data movement가 더 중요하다. DRAM round-trip을 줄여야 한다. local reuse와 streaming path를 적극 활용해야 한다.

## 9.3 Canonical dataflow

**Vision-centric canonical flow**

```text
Sensor -> ISP / Vision Front-End -> Shared SRAM / Stream Buffer -> Helios NPU -> Shared SRAM -> Arm CPU / RT CPU / Output
```

**Compute-centric canonical flow**

```text
External IO / Storage / Host -> DMA -> Shared SRAM / DRAM -> Helios NPU or Ethos -> CPU / Peripheral Path
```

## 9.4 CME impact on memory / fabric planning

CME / CPU Matrix Path는 Fabric A(Coherent Fabric)에서 동작하므로, 대량 matrix 연산 시 다음과 같은 영향을 준다: Fabric A bandwidth 점유, CPU cache traffic과의 contention, system cache / DRAM path 간섭 가능성.

따라서 Fabric A bandwidth 산정 시, worst-case matrix burst traffic을 반드시 포함해야 한다.

## 9.5 Suggested design metrics

후속 상세 설계에서는 다음 정량 지표를 반드시 정의해야 한다: GB/s per TOPS, SRAM reuse ratio, DRAM spill ratio, average DMA chain utilization, NPU idle percentage due to memory stall, sensor-to-NPU ingress latency, shared SRAM bank conflict rate.

이 지표들은 이후 performance simulator와 NoC 모델의 baseline이 된다.

# 10. Hierarchical Multi-NoC Architecture

## 10.1 Core conclusion

Arm CSS와 Helios IP는 Hierarchical Multi-NoC 구조로 분리 가능하며, 분리하는 것이 플랫폼화에 유리하다.

다만 "서로 완전히 독립된 NoC 여러 개"보다는, **Arm CSS backbone + Helios domain-specific sub-fabrics + gateways / bridges** 구조가 현실적이다.

## 10.2 Recommended fabric decomposition

**Fabric A: Coherent System Fabric** — Arm CPU cluster, system cache, coherent memory access, OS-managed shared memory. 특징: coherence 중심, control plane / software plane의 기반, 상대적으로 일반적이고 재사용 가능.

**Fabric B: High-Bandwidth AI Fabric** — Helios NPU, shared SRAM, tensor DMA, command DMA, high-throughput memory path. 특징: non-coherent 또는 restricted-coherent / IO-coherent 후보, deterministic throughput 지향, large burst / tile transfer에 최적화.

**Fabric C: Vision Streaming Fabric** — camera ingress, ISP, line buffer, stream preprocess, direct stream to SRAM / NPU bridge. 특징: 지속적 stream latency가 중요, burst bandwidth보다 pipeline continuity 중요, Vision 고객에서 차별화의 핵심.

**Fabric D: Peripheral / Connectivity Fabric** — modem, PCIe, USB, Ethernet, UFS, low-speed peripheral. 특징: isolation, QoS, power-domain granularity 중요.

## 10.3 Lumex CSS adoption implication

Lumex CSS 채택 시 Arm 측 interconnect 자산(SI L1 interconnect, NoC S3 등)을 기본 backbone으로 활용할 수 있으므로, 설계 범위는 더 구체화된다.

- Fabric A는 Arm CSS 기본 interconnect 자산 중심으로 구성 가능
- Helios의 실질적 차별화 포인트는 Fabric B/C/D 및 gateway/bridge 설계에 집중될 수 있음
- 이는 일정 리스크와 설계 범위를 줄이는 근거가 된다

즉, Helios는 Arm backbone을 최대한 활용하고, Helios domain-specific fabric에서 차별화하는 구조가 가장 현실적이다.

## 10.4 Fabric interaction model

```text
[Arm Coherent Fabric]
        |
   [System Bridge / Gateway]
        |
+------------------------------+
| Helios Domain-specific       |
| - AI Fabric                  |
| - Vision Streaming Fabric    |
| - Peripheral Fabric          |
+------------------------------+
```

## 10.5 Key design principle

> **Control Path ≠ Data Path ≠ Streaming Path**

이 원칙이 중요한 이유: CPU 중심 제어 트래픽과 NPU 대역폭 트래픽은 성격이 다르다. Vision stream은 또 다른 latency 요구를 가진다. Peripheral / Modem path는 isolation과 QoS 요구가 다르다.

하나의 monolithic bus로 모든 것을 처리하려 하면: arbitration 복잡도 증가, worst-case latency 증가, customer-specific optimization 어려움, SKU 파생이 어려워진다.

## 10.6 Bridge latency and NoC IP selection

**Bridge latency assumption** — bridge latency baseline: 5~15 cycles (single-hop 기준). 이 수치는 후속 NoC latency model의 초기 가정으로 사용한다.

**NoC IP selection** — Phase 0 또는 Phase 1에서 다음 중 선택 범위를 확정해야 한다: Arm backbone interconnect(SI L1 등) 활용, Helios domain fabric용 별도 NoC IP(Arm NI / Arteris FlexNoC 등) 활용, custom gateway / bridge 설계. 이 결정은 일정, verification scope, risk profile에 직접 영향을 준다.

## 10.7 Phase 1 mandatory decisions

Phase 1에서 반드시 확정해야 할 구조적 결정은 다음과 같다: Fabric B coherency mode(non-coherent vs restricted-coherent / IO-coherent), bridge latency assumption, Fabric bandwidth budget, CME burst traffic의 Fabric A 영향 분석, NoC IP selection 범위.

# 11. Fabric Preset Strategy for Vision / Non-Vision / Connected Edge

## 11.1 Why IP ON/OFF is not enough

Vision 고객과 Non-Vision 고객의 차이는 단순히 ISP의 유무가 아니다. 핵심 차이는 트래픽 성격과 메모리 경로 요구사항이다.

따라서 플랫폼화는 다음 수준까지 가야 한다: IP enable / disable, DMA structure, SRAM bank shape, NoC topology, arbitration policy, QoS priority, bridge enable / disable.

> **버스 자체가 플랫폼 파생의 핵심 자산이 되어야 한다.**

## 11.2 Fabric Preset A: Vision-centric topology

**목표 고객** — smart camera, industrial vision, multi-camera edge device, XR vision-heavy device.

**특징** — ISP 필수, sensor ingress 대역 중요, line-buffer / stream buffer 중요, ISP→Shared SRAM→NPU path 최적화 중요.

**권장 fabric 설정** — Vision Streaming Fabric 활성화, ISP↔NPU direct streaming bridge 활성화, shared SRAM bank 수 증가, sensor ingress QoS 최우선, DDR path는 spill / fallback 용도 강화, NPU prefetch와 stream ingestion 동시 지원.

## 11.3 Fabric Preset B: Compute-centric topology

**목표 고객** — AI gateway, compute edge box, robotics without camera-heavy path, industrial inference controller.

**특징** — camera path 약하거나 없음, DDR / NPU / shared SRAM path 중요, PCIe / GbE / UFS 같은 ingress 비중 큼, external model / data streaming 중심.

**권장 fabric 설정** — Vision fabric 제거 또는 최소 stub, AI Fabric 중심 구조, peripheral bandwidth 강화, DRAM arbitration을 NPU / IO 중심으로 최적화, line-buffer 성격의 SRAM 영역 축소, DMA path를 compute ingest 중심으로 구성.

## 11.4 Fabric Preset C: Connected Edge topology

**목표 고객** — XR, wearable AI, connected smart camera, mobile edge terminal.

**특징** — modem / connectivity 중요, power sensitivity 높음, camera + AI + network의 동시성 존재 가능.

**권장 fabric 설정** — Vision fabric 일부 유지, peripheral / modem QoS 강화, power island 세분화, Arm CSS와 Helios domain 간 power gating granularity 강화, low-power always-on path와 burst AI path를 분리.

## 11.5 RTL-level vs Config-level preset separation

**RTL-level preset parameters (mask 영향 가능)** — link width, port count, SRAM bank count, bridge 존재 여부.

**Config-level preset parameters (동일 mask 내 변형 가능)** — QoS policy, arbitration priority, DMA sequencer preset, power gating enable / disable.

이 구분은 "몇 개의 mask로 몇 개의 SKU를 커버할 수 있는가"라는 비즈니스 질문에 직접 연결된다. 즉, 플랫폼 유연성을 단순 설계 자유도가 아니라 확장 가능한 SKU business model로 번역하는 핵심 구조다.

## 11.6 Strategic statement

> The platform is not defined only by a reusable set of IP blocks, but by a reusable set of fabric presets: Vision-centric, Compute-centric, and Connected Edge-centric topologies.

본 플랫폼의 핵심은 IP 블록 재사용 자체만이 아니라, 고객 세그먼트별로 재사용 가능한 Fabric Preset(Vision-centric / Compute-centric / Connected Edge-centric Topology)에 있다.

# 12. SKU Strategy

## 12.1 Platform SKU principle

SKU는 단순히 TOPS나 power bin만으로 나누지 않는다. 다음 세 축을 동시에 고려해야 한다.

1. Compute capability
2. Fabric preset
3. Domain option (ISP / modem / RT 강화 여부)

## 12.2 Example SKU matrix

| SKU Type | CPU | Matrix Path | Ethos | Helios NPU | ISP | Modem | Fabric Preset |
|---|---|---|---|---|---|---|---|
| Ultra-Lite | O | O* | X | X | X | Optional | Compute-centric |
| Lite Vision | O | O | O | X | O | Optional | Vision-centric |
| Mid Vision Pro | O | O | O | O | O | Optional | Vision-centric |
| Robotics Mid | O | O | X | O | Optional | X | Compute-centric |
| Connected Edge Pro | O | O | O | O | Optional | O | Connected Edge |
| Gateway Compute | O | O | O or X | O | X | Optional | Compute-centric |

*Ultra-Lite의 Matrix Path 활성화는 CME-capable 코어(C1-Pro 이상) 채택을 전제로 한다. IoT/sensor hub 등 극도로 cost-sensitive한 세그먼트에서는 C1-Nano 기반으로 Matrix Path가 제한될 수 있으며, 이 경우 Phase 1에서 CME-only SKU의 TOPS/W 타겟 및 실현 가능성을 검증해야 한다.

## 12.3 Parameterization dimensions

**Compute parameters** — CPU core count, Matrix path enable / disable, Ethos enable / disable, Helios NPU size, RT CPU level.

**Memory parameters** — external memory width, shared SRAM size, SRAM bank count, local scratchpad size.

**Fabric parameters** — link width, port count, bridge enable / disable, QoS policy, stream path enable, arbitration preset.

**Domain parameters** — ISP level, modem option, peripheral mix, security / safety enhancement.

# 13. Software Platform Strategy

## 13.1 Why SW-first matters

플랫폼이 성공하려면 HW block보다 runtime이 먼저 정의되어야 한다. 고객은 결국 다음을 본다: 프레임워크 입력이 되는가, compiler가 있는가, profiling이 되는가, multi-engine dispatch가 되는가, SW bring-up이 쉬운가.

## 13.2 Recommended software stack view

```text
Framework / Model Import
    ↓
Graph Lowering / Partitioning
    ↓
Unified Compiler / Runtime
    ↓
Dispatch to:
  - Arm CPU General Path
  - Arm CPU Matrix Path
  - Ethos Baseline NPU Path
  - Helios High-Performance NPU Path
    ↓
Profiler / Trace / Performance Model
```

Supported framework direction: ONNX, TFLite / LiteRT, PyTorch / ExecuTorch, Android NNAPI (Connected Mobile Edge 대응 시).

## 13.3 Runtime responsibilities

graph partitioning, memory planning, DMA scheduling, stream path orchestration, CPU / CPU Matrix / Ethos / Helios NPU dispatch, fallback handling, profiling / trace generation.

## 13.4 Initial dispatch heuristic

초기 runtime 정책의 출발점은 다음과 같다.

- control-heavy / irregular / fallback -> Arm CPU General Path
- single-tile 또는 small GEMM 중심 -> Arm CPU Matrix Path
- baseline CNN / transformer inference -> Ethos Path
- streaming-heavy / vision-heavy / high-throughput path -> Helios NPU Path

정확한 분할 기준은 후속 simulator / cycle model로 보정한다.

## 13.5 Strategic principle

> **Arm은 software baseline과 ecosystem을 제공하고, Helios는 platform-aware runtime과 performance-oriented dispatch로 differentiation을 만든다.**

# 14. Execution Plan

## 14.1 Recommended timeline

### Phase 0 (0~3 months): Strategy lock

목적: 플랫폼의 범위와 기본 가정 확정.

주요 항목: target segment 선정, platform scope 정의, Arm 협력 범위 설정, Fabric preset 개념 확정, key customer hypothesis 수립, process node baseline 설정 (4nm baseline, 3nm은 차세대 파생형에서 검토), CSS / core 후보군 정의, NoC IP selection 후보 정의.

### Phase 1 (3~9 months): Architecture lock

목적: 기술 전제조건과 시스템 구조 확정.

주요 항목: Arm base layer 정의, Helios differentiation layer spec 정의, NPU / ISP / Fabric architecture 문서화, runtime requirements 정의, SKU matrix 초안 정의, coherency boundary 결정, SRAM 범위 확정, matrix path(CME/SME2) 사용 여부 확정, NoC/gateway 구조 결정.

**Gate criteria** — Phase 1 종료 시 다음이 완료되어야 한다: 선택한 CSS / core 구성이 명확해질 것, CME / Matrix path의 유효성 가정이 확정될 것, Fabric B coherency mode가 확정될 것, shared SRAM / local SRAM 범위가 확정될 것, Fabric topology와 preset 구성이 lock될 것, SKU matrix 초안과 dispatch 초안이 도출될 것.

### Phase 2 (9~15 months): Modeling and integration

목적: 설계 가정을 검증 가능한 모델로 변환.

주요 항목: performance model, cycle-based simulator, NoC latency / bandwidth model, dispatch threshold 보정, RTL subsystem integration 준비, FPGA or emulation-based PoC.

### Phase 3 (15~21 months): Verification and pre-silicon platform

목적: pre-silicon 수준에서 실행 가능성 확보.

주요 항목: verification ramp, software bring-up, SDK pre-release, customer PoC enablement, profiling / trace 연계.

### Phase 4 (21~24 months): Tape-out and productization

목적: 제품화 및 고객 enablement 마무리.

주요 항목: final closure, tape-out, customer enablement package, derivative SKU planning, next-gen platform feedback 수집.

## 14.2 3-month PoC concept

고객 대응 속도를 높이려면 다음 구조가 필요하다.

- Day 0: customer requirement intake
- Day 15: preset + SKU mapping
- Day 30: performance estimate
- Day 60: workload mapping + runtime feasibility
- Day 90: emulation / prototype evidence

즉, 3개월 PoC는 실칩 개발 완료가 아니라, **플랫폼 기반으로 고객에게 신뢰할 수 있는 성능 / 구성 / 개발 가능성 증거를 제시하는 단계**로 정의해야 한다.

# 15. Key Success Factors

**15.1 Platformization over one-off design** — 고객마다 새 SoC를 설계하지 않는다. 공통 platform + fabric preset + configurable domains로 대응한다.

**15.2 Arm partnership with clear boundary** — Arm을 적극 활용하되 종속되지는 않는다. Arm은 backbone, Helios는 differentiation 역할을 맡는다.

**15.3 Helios-specific domain excellence** — ISP, modem / connectivity option, mobile-grade power integration, Helios NPU + runtime.

**15.4 Fabric as a product asset** — NoC / bus topology는 구현 상세가 아니라 product differentiation asset이다. topology preset이 SKU 전략의 핵심이 된다.

**15.5 Runtime-first execution** — compiler / runtime / profiler를 HW와 동등한 핵심 자산으로 다뤄야 한다.

**15.6 Anchor customer model** — 고객 수 확대보다 anchor customer 반복 수주 모델이 중요하다.

# 16. Risks and Mitigations

## 16.1 Risk: Arm dependency becomes too strong

**위험**: Arm backbone 의존도가 지나치게 높아질 경우, Helios platform의 독자성이 약화될 수 있다. 고객이 Helios differentiation보다 Arm baseline을 더 크게 인식할 수 있다.

**대응**: Helios differentiation layer를 명확히 분리한다. Helios NPU / ISP / runtime / fabric preset에서 독자성을 확보한다. Arm은 backbone, Helios는 differentiated execution domain이라는 역할 구분을 문서와 조직 모두에서 유지한다.

## 16.2 Risk: Ethos와 Helios NPU의 내부 역할 충돌

**위험**: Ethos와 Helios NPU가 동일한 workload를 겨냥하면 조직과 제품 전략이 충돌할 수 있다. compiler / runtime / marketing 메시지가 혼선에 빠질 수 있다.

**대응**: Ethos는 baseline SKU / low-risk customer path로 위치시킨다. Helios NPU는 high-performance / high-efficiency / streaming path를 담당한다. runtime dispatch 정책 문서에서 역할 경계를 명확히 규정한다.

## 16.3 Risk: Fabric complexity explodes

**위험**: customer-specific customization을 과도하게 허용하면 interconnect 구조가 지나치게 복잡해질 수 있다. verification cost와 schedule risk가 급증할 수 있다.

**대응**: 자유도 무한정 허용 금지. preset 기반 topology만 제공. Vision / Compute / Connected Edge 3개 중심으로 제한. RTL-level과 Config-level preset을 분리해 mask 증가를 억제한다.

## 16.4 Risk: Software stack readiness 부족

**위험**: HW가 준비되어도 compiler / runtime / profiler가 준비되지 않으면 고객 대응이 지연된다. multi-engine dispatch가 성숙하지 않으면 platform value가 약해진다.

**대응**: runtime / trace / profiler를 초기 phase부터 포함한다. compiler integration plan을 HW spec과 병행한다. 90일 PoC 단계에서 SW feasibility를 반드시 증명 대상으로 포함한다.

## 16.5 Risk: 일정 과도 낙관

**위험**: 12~18개월 수준의 과도한 낙관 일정은 내부 자원 계획과 고객 기대치를 동시에 왜곡할 수 있다. PoC와 productization의 개념이 혼동될 수 있다.

**대응**: 24개월 기준 계획 수립. PoC와 full productization을 구분. Phase gate와 deliverable을 명확히 정의해 일정 리스크를 조기 가시화한다.

## 16.6 Risk: Process node uncertainty

**위험**: process node가 확정되지 않으면 area / power / SRAM density / bandwidth 가정이 모두 불안정해진다. NPU 크기, SRAM 용량, fabric preset feasibility가 연쇄적으로 흔들린다.

**대응**: Phase 0에서 baseline node를 확정한다. 첫 세대는 4nm baseline, 이후 파생형에서 finer node를 검토하는 방식으로 리스크를 분산한다. performance model과 area budget을 process node 가정과 함께 관리한다.

## 16.7 Risk: Security / Safety architecture 부족

**위험**: Connected Edge와 Robotics 영역에서는 secure boot, trusted runtime, safety island 요구가 발생한다. 이를 늦게 반영하면 fabric / power / memory architecture 재작업 가능성이 커진다.

**대응**: secure / non-secure transaction boundary를 조기에 정의한다. RT / safety island 가능성을 architecture phase에서 반영한다. Coherency & Power Domain Spec에서 security / safety domain을 별도 명시한다.

# 17. Recommended Next Technical Deliverables

다음 단계에서 반드시 만들어야 하는 문서는 아래와 같다.

**1. NPU Cycle-Level Spec** — SE / VE / TE overlap model, SRAM access model, command timing spec, dependency / scoreboard behavior, tile size와 dispatch threshold의 관계.

**2. Multi-Fabric NoC Spec** — fabric role definition, bridge / gateway behavior, bandwidth and latency assumptions, contention model, CME burst traffic의 Fabric A 영향.

**3. Fabric Preset Spec** — Vision-centric topology, Compute-centric topology, Connected Edge-centric topology, parameter table and enable conditions, RTL-level vs Config-level preset mapping.

**4. Unified Runtime Architecture Spec** — graph partitioning, dispatch policy, memory planner, DMA scheduler, CPU / CPU Matrix / Ethos / Helios NPU orchestration.

**5. Performance Simulator Spec** — Python 중심 cycle-based simulator, graph input schema, command stream model, latency / throughput / bandwidth report format.

**6. Coherency & Power Domain Spec** — Fabric A~D 간 coherency boundary, flush / invalidate / snoop 정책, power island 정의, DVFS domain 분리, security / safety domain 반영.

# 18. Final Conclusion

Project Helios의 Edge / Physical AI용 Custom SoC 전략은 단순한 NPU 경쟁이 아니라, **Arm 기반 공통 플랫폼 + Helios 차별화 IP + Fabric Preset 기반 제품군 생성 구조**로 정의되어야 한다.

핵심은 다음과 같다.

1. Arm CSS를 공통 backbone으로 활용한다.
2. Helios NPU / ISP / Modem / mobile power integration을 differentiation layer로 둔다.
3. Vision / Non-Vision / Connected Edge 고객 대응을 위해 Fabric Preset을 플랫폼 자산으로 만든다.
4. SKU는 compute block의 조합뿐 아니라 interconnect topology의 조합으로 정의한다.
5. runtime / compiler / profiling까지 포함한 platform business로 접근한다.

최종적으로 이 전략은 다음 문장으로 요약된다.

> **Helios의 Edge Custom SoC 플랫폼은 재사용 가능한 IP 집합이 아니라, 재사용 가능한 Arm backbone과 Helios differentiated domains, 그리고 고객 세그먼트별 재사용 가능한 Fabric Preset의 조합으로 정의된다.**

*Document Version: 1.0 Final*  
*Review History: v0.1 -> v0.7 (7 iterations, cross-reviewed)*
