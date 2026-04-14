---
title: Flash-Resident LLM and HBF for Edge Inference
type: topic
sources:
  - raw/AI-Hardware/Architecture/Memory Hierarchy in AI 33a6cc566b0b81f4b111e6d0e5d21553.md
  - raw/AI-Hardware/Simulator/HyperAccel LPU Explanation/HyperAccel LPU Explanation (Part 1 of 3) 33a6cc566b0b81349b96fd84f1659f77.md
  - raw/GenAI/HW-Friendly/KV-cache Optimization Explanation 33a6cc566b0b811890a0df4bc615b114.md
  - raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/architecture/kv_cache_semantics_spec.md
tags: [LLM, edge-inference, memory-hierarchy, HBF, flash, KV-cache, NPU]
updated: 2026-04-14
---

# Flash-Resident LLM and HBF for Edge Inference

엣지 AI 추론 시스템의 핵심은 더 이상 "최대 TOPS"가 아니다. 실제 병목은 모델 가중치, KV cache, 멀티모달 자산, 개인화 adapter를 어떤 메모리 계층에 배치하고 언제 이동시키는가에 있다. 이 문서는 Apple의 `LLM in a Flash`가 보여준 flash-resident weight streaming 철학과, 최근 업계가 제시하는 `HBF(High Bandwidth Flash)` 계층을 연결해, 엣지 추론용 전체 시스템 아키텍처를 하나의 관점으로 정리한다.

핵심 결론은 단순하다. 엣지 LLM/VLM 시스템은 compute-centric NPU가 아니라 memory-centric inference system으로 설계되어야 한다. SRAM, LPDDR 또는 HBM-lite, HBF, SSD/UFS로 이어지는 계층형 메모리 위에서 weight, KV, adapter, expert를 분리 배치하고, prefill/decode/multimodal burst를 구분하는 runtime이 이들을 지속적으로 승격(promote)하거나 축출(evict)해야 한다.

## 1. 용어와 범위

먼저 용어를 정확히 맞출 필요가 있다.

- `LLM in a Flash`는 Apple이 2024년 8월 공개한 limited-memory inference 접근이다. 핵심은 DRAM에 모두 올릴 수 없는 모델의 가중치를 flash에 두고, 실행 시 필요한 부분만 DRAM으로 가져오되, 전송량을 줄이고 큰 연속 청크를 읽도록 runtime과 데이터 배치를 최적화하는 데 있다.
- `HBF`는 현재 공개 자료 기준 `High Bandwidth Flash`로 해석하는 것이 가장 자연스럽다. Sandisk와 SK hynix는 이를 AI 추론 시대를 위한 새로운 메모리 계층으로 제시하고 있으며, 2025년 8월 MOU, 2025년 10월 `AIN B` 공개, 2026년 2월 OCP 기반 표준화 협력까지 이어졌다.
- 다만 HBF는 아직 범용 상용 메모리로 굳어진 상태가 아니다. 따라서 이 문서에서는 HBF를 "현재 즉시 모든 모바일 SoC에 들어가는 부품"이 아니라, 향후 엣지 서버와 프리미엄 엣지 장치에 도입될 가능성이 높은 `read-mostly warm memory tier`로 다룬다.

이 정의는 중요하다. HBF를 DRAM의 단순 대체재로 보면 설계가 잘못된다. HBF의 가치는 "working memory 전체를 맡는 것"이 아니라, LPDDR/HBM과 SSD/UFS 사이에서 모델 저장과 대역폭 사이의 간극을 메우는 데 있다.

## 2. 왜 기존 엣지 AI 구조로는 부족한가

기존 모바일 NPU 또는 엣지 가속기 구조는 대체로 다음과 같은 형태였다.

```text
CPU + GPU + NPU
        |
      LPDDR
```

이 구조는 CNN, detector, small encoder, 고정형 ASR에는 잘 맞는다. 하지만 LLM과 VLM으로 넘어오면 병목의 성격이 달라진다.

기존 병목은 주로 compute throughput이었다. 큰 GEMM를 얼마나 빨리 처리하느냐가 중심이었다. 반면 현재의 병목은 다음과 같이 바뀐다.

- weight residency
- KV cache growth
- DRAM bandwidth
- host-device copy
- model switching
- expert routing

특히 decode 단계는 FLOPs보다 weight fetch와 KV access가 더 지배적이기 쉽다. 이 때문에 엣지 추론 시스템에서 중요한 것은 단순히 NPU MAC 배열을 키우는 일이 아니라, 가중치와 KV가 메모리 계층 사이를 어떻게 이동하는지 제어하는 것이다. `LLM in a Flash`는 바로 이 점을 정면으로 다루며, 최근 HBF 논의도 본질적으로 같은 문제를 다른 계층에서 확장한다.

## 3. 아키텍처의 기본 원칙

이 문서가 제안하는 원칙은 다음 한 문장으로 정리할 수 있다.

> 모델은 "메모리에 상주시킨다"가 아니라 "메모리 계층에 배치한다".

즉, 엣지 AI 시스템은 단일 DRAM 상주 모델을 전제로 해서는 안 된다. 대신 모델과 상태를 다음처럼 나눠서 배치해야 한다.

- 현재 연산에 필요한 activation과 hot tile은 on-chip SRAM/SPM에 둔다.
- 현재 세션에서 반복 접근하는 live KV와 active layer weight는 LPDDR/HBM-lite에 둔다.
- read-mostly 성격의 대형 weight, shared precomputed KV, cold expert, frozen adapter bank는 HBF에 둔다.
- 장기 저장과 업데이트, 설치 이미지는 UFS/NVMe/SSD에 둔다.

이 관점에서는 연산기보다 `Memory Residency Manager`가 더 중요해진다. 어떤 데이터를 어느 tier에 두고, 어느 시점에 승격할지, 어떤 데이터는 압축한 채 보관할지, 어떤 상태는 요약 후 버릴지를 결정하는 runtime이 시스템 경쟁력을 좌우한다.

## 4. 제안하는 전체 시스템 블록도

권장 아키텍처는 다음과 같은 구조를 갖는다.

```text
Application Layer
  Assistant / VLM / ASR / OCR / Camera / Local Agent
          |
          v
AI Runtime & Orchestration Layer
  - Model planner
  - Memory residency manager
  - Token-phase scheduler
  - KV-cache manager
  - Multimodal graph executor
  - Thermal / power / QoS governor
          |
          v
Compute Complex
  - CPU control cluster
  - Vector Engine
  - Tensor Engine
  - Sparse / routing micro-engine
  - Streaming DMA / dequant / decompress unit
          |
          v
On-chip Memory Fabric
  - Register files
  - Local SRAM
  - Shared SRAM / scratchpad
  - KV SRAM slices
          |
          v
Tiered Memory Subsystem
  T0: Register / local SRAM
  T1: Shared SRAM / SPM
  T2: LPDDR or HBM-lite
  T3: HBF
  T4: UFS / NVMe / SSD
```

여기서 중요한 점은 HBF path를 일반 CPU coherent fabric과 같은 경로로 취급하지 않는 것이다. HBF는 캐시 일관성 중심 경로가 아니라, 큰 연속 청크를 빠르게 읽어오는 streaming path로 설계되어야 한다. HBF 트래픽이 일반 CPU 메모리 트래픽과 뒤섞이면 이 계층의 장점을 살리기 어렵다.

## 5. 메모리 계층과 데이터 배치

### T0/T1: Register, Local SRAM, Shared SRAM

이 계층은 여전히 가장 중요하다. flash-resident 설계라고 해서 on-chip memory의 중요성이 줄어드는 것은 아니다. 오히려 flash에서 가져온 데이터를 실제 성능으로 바꾸려면 on-chip reuse가 더 중요해진다.

이 계층에는 다음이 올라간다.

- active tile
- hot activation
- layernorm, RoPE, small constants
- 현재 decode에 필요한 small vector state
- 극히 최근에 접근한 hot KV fragment

### T2: LPDDR 또는 HBM-lite

이 계층은 시스템의 실질적인 working memory다. 엣지 장치에서는 보통 LPDDR이 현실적이고, 더 큰 장치나 엣지 서버에서는 HBM-lite 또는 근접 DRAM이 가능하다.

여기에는 다음이 올라간다.

- 현재 active layer weights
- live KV cache
- 현재 세션에서 반복 사용하는 adapter
- multimodal intermediate
- draft model 또는 speculative decoding state

중요한 점은 `live KV cache`를 기본적으로 이 계층에 둔다는 것이다. KV는 decode 동안 매 토큰 접근되고 계속 늘어나는 상태이므로, 일반적인 인터랙티브 추론에서는 HBF보다 LPDDR/HBM-lite가 더 자연스러운 주 저장소다.

### T3: HBF

HBF는 이 아키텍처에서 가장 오해받기 쉬운 계층이다. HBF의 역할은 "느린 DRAM"이 아니라 `read-mostly warm reservoir`다.

여기에 적합한 데이터는 다음과 같다.

- read-mostly model weights
- 자주 쓰지는 않지만 용량이 큰 vision tower weights
- cold MoE experts
- frozen adapter bank
- shared precomputed KV cache
- 세션 간 공유 가능한 prefix asset

여기서 `shared precomputed KV cache`라는 표현은 중요하다. 2026년 공개된 H3 방향도 `gigantic read-only data`와 `shared precomputed key-value cache`를 주된 대상으로 삼고 있다. 반대로 사용자 세션마다 실시간으로 append되는 live KV를 기본적으로 HBF에 두는 것은 지연시간과 쓰기 특성 측면에서 좋은 기본값이 아니다.

따라서 HBF에 대한 실무적 원칙은 다음과 같다.

- live, write-heavy state는 LPDDR/HBM-lite에 둔다.
- large, read-mostly state는 HBF에 둔다.
- SSD/UFS보다는 훨씬 빠르지만, DRAM처럼 취급하지는 않는다.

### T4: UFS, NVMe, SSD

이 계층은 cold archive다.

- 모델 패키지 저장
- 체크포인트
- 업데이트 이미지
- 낮은 우선순위 멀티모달 asset
- 장기 로그와 오프라인 캐시

실행 중에는 가능하면 직접 접근하지 않고, HBF 또는 LPDDR로 먼저 승격한 뒤 사용해야 한다.

## 6. 컴퓨트 복합체는 최소 네 가지로 나뉘어야 한다

엣지 LLM 시스템에서는 NPU를 하나의 dense GEMM 엔진으로 보는 관점이 부족하다. 권장되는 구성은 최소 다음 네 가지다.

### CPU control cluster

역할은 단순 host가 아니라 control plane이다.

- graph dispatch
- DMA orchestration
- token loop control
- QoS / thermal / battery policy
- interrupt handling

### Vector Engine

decode 단계에서 자주 등장하는 비대형 벡터 연산을 담당한다.

- layernorm
- softmax
- RoPE
- sampling
- dequant
- packing / unpacking
- 작은 GEMV와 elementwise fusion

### Tensor Engine

prefill과 encoder-heavy path의 주력 엔진이다.

- dense GEMM
- projection
- FFN matmul
- vision/audio encoder

### Sparse / routing micro-engine

앞으로의 엣지 모델은 dense 하나로 고정되기보다, small dense core 위에 sparse router, MoE-lite, adapter bank를 얹는 방향으로 갈 가능성이 크다. 따라서 sparse routing, expert prefetch hint 생성, block-sparse steering을 처리하는 작은 전용 엔진이 유효하다.

이 네 엔진 사이를 연결하는 핵심은 compute scheduler가 아니라 `phase-aware memory scheduler`다.

## 7. Prefill, Decode, Multimodal Burst를 분리해서 다뤄야 한다

엣지 추론 런타임은 하나의 "inference mode"만 가정하면 안 된다. 최소 세 가지 실행 모드를 나눠야 한다.

### Prefill

특징은 compute-heavy, 병렬도 높음, KV 생성이다. 이 단계는 Tensor Engine 중심이고, 큰 burst read와 activation reuse가 중요하다.

### Decode

특징은 memory-bound, 작은 batch, token latency 민감성이다. 이 단계에서는 hot layer pinning, next-layer prefetch, KV page cache, dequant + GEMV fusion이 핵심이다. 실효 성능은 MAC 수보다 weight/KV fetch 경로가 좌우한다.

### Multimodal Burst

카메라 프레임, 음성 입력, 텍스트 응답이 동시에 걸리는 순간이다. vision/audio encoder는 frame-based이고 language head는 token-based이므로, shared DRAM contention과 DMA 우선순위를 runtime이 직접 관리해야 한다.

이 구분이 필요한 이유는 간단하다. prefill 최적화와 decode 최적화는 거의 다른 시스템 문제이기 때문이다.

## 8. Weight, KV, Adapter의 상주 정책

### Weight Placement Policy

weight는 hot, warm, cold로 나눠야 한다.

- hot weights: T1/T2
- warm weights: T3
- cold weights: T4

hotness 판단 기준은 최근 토큰에서의 참조 빈도, layer criticality, 사용자 workload profile, decode path locality다.

예를 들어 항상 켜져 있는 assistant 기기라면 다음은 hot 쪽으로 갈 가능성이 높다.

- base LLM core
- small ASR decoder
- tokenizer / embedding
- 가벼운 projector

반대로 드물게 쓰는 대형 vision backbone, domain-specific adapter, cold expert는 HBF에 더 적합하다.

### KV-cache Placement Policy

KV는 가중치보다 더 민감하다. 기본 원칙은 다음과 같다.

- recent KV: T1/T2
- active session KV: T2
- compressed snapshot or read-mostly shared KV: T3
- archival or summarized context: T4

즉, KV 전체를 무한정 LPDDR에 두는 것도 비효율적이고, 반대로 live KV를 HBF로 곧바로 밀어넣는 것도 좋지 않다. 세션 문맥이 길어질수록 runtime은 압축, 요약, 스냅샷, eviction을 조합해야 한다.

이를 위해 필요한 하드웨어/런타임 기능은 다음과 같다.

- KV page table
- page-level quantization
- append-friendly layout
- gather-friendly read path
- age-based eviction policy

### Adapter / Personalization Bank

온디바이스 AI가 개인화되면 base model 하나보다 adapter bank가 더 중요해진다.

- base model: T2/T3
- frozen adapter bank: mostly T3
- session-hot adapter: T2

이 방식은 "거대한 단일 모델"보다 "적당한 base model + 많은 adapter"가 엣지에서 더 현실적이라는 가정을 반영한다.

## 9. LLM in a Flash를 하드웨어 관점으로 확장하기

Apple의 포인트는 단순히 "플래시에 둔다"가 아니다. 성능을 만든 핵심은 `transfer volume reduction`과 `large contiguous read`다. 따라서 엣지용 HBF 아키텍처도 같은 철학을 가져야 한다.

필요한 하드웨어 블록은 다음과 같다.

### Row-group streaming unit

- weight를 row 또는 block bundle 단위로 읽음
- 작은 random read를 큰 burst로 재구성
- alignment-aware DMA를 발행

### On-the-fly dequant / decompress engine

- HBF에는 압축 또는 양자화된 형식으로 저장
- LPDDR 또는 SRAM으로 올리면서 바로 해제
- 별도 복사 패스를 줄임

### Prefetch predictor

- 다음 토큰에서 필요할 확률이 높은 layer block, expert, adapter를 예측
- decode path에 맞춘 prefetch hint 생성

핵심은 DMA가 단순 복사기가 아니라 `semantic DMA`가 되어야 한다는 점이다. 무엇을 언제 읽는지가 성능을 결정한다.

## 10. HBF를 쓸 때 반드시 지켜야 할 설계 원칙

HBF는 매력적이지만, 잘못 쓰면 시스템을 느리게 만든다. 따라서 다음 원칙이 필요하다.

첫째, HBF를 `working memory`로 보지 않는다. working set은 LPDDR/HBM-lite에 두고, HBF는 그 바깥의 warm tier로 본다.

둘째, HBF에는 기본적으로 `read-mostly` 데이터를 둔다. 특히 H3에서 강조되는 시나리오도 gigantic read-only data와 shared precomputed KV에 가깝다. live decode path 전체를 HBF 위에 얹는 것이 아니다.

셋째, HBF의 장점은 `large sequential movement`에서 나온다. random access가 많아질수록 장점이 줄어든다.

넷째, 제품군별 현실성을 구분한다. 2026년 현재 스마트폰 클래스에서 HBF를 기본 전제로 두기보다는, 엣지 서버와 상위 엣지 장치에서 먼저 받아들이고 모바일은 이를 수용 가능한 확장 아키텍처로 준비하는 편이 현실적이다.

## 11. 소프트웨어 스택은 메모리 계층을 직접 다뤄야 한다

하드웨어만으로는 부족하다. 실전 경쟁력은 runtime과 compiler가 메모리 계층을 얼마나 명시적으로 다루느냐에 달려 있다.

권장되는 IR primitive는 다음과 같다.

- `WEIGHT_PREFETCH(block_id, tier_hint)`
- `WEIGHT_PROMOTE(block_id, from_tier, to_tier)`
- `KV_PAGE_PROMOTE(page_id)`
- `KV_PAGE_EVICT(page_id, compress_mode)`
- `EXPERT_ROUTE(token_group, expert_set)`
- `ADAPTER_PROMOTE(adapter_id)`
- `STREAM_DEQUANT(stream_id, format)`

runtime planner는 최소 다음 상태를 알아야 한다.

- current phase: prefill / decode / multimodal
- thermal headroom
- battery state
- bandwidth pressure
- model residency map
- KV pressure
- deadline or latency target

이 정보가 있어야 per-token 또는 per-frame 수준에서 "지금 latency mode로 갈지", "vision encoder를 지금 돌릴지", "cold expert를 HBF에서 올릴지", "dense fallback으로 갈지" 같은 결정을 내릴 수 있다.

## 12. 제품군별 권장 변형

### Smartphone class

현실적인 현재형은 다음과 같다.

- CPU + GPU + NPU
- SRAM + LPDDR + UFS
- 선택적으로 package-near flash tier를 고려할 수 있음

이 구성은 3B~8B급 compact LLM, 경량 VLM, always-on ASR, 개인 비서에 적합하다. 여기서 중요한 것은 HBF 자체보다도, 나중에 HBF-like tier가 들어와도 수용 가능한 runtime 구조를 미리 설계해 두는 것이다.

### XR / robotics edge

이 계층은 더 공격적인 메모리 구성이 가능하다.

- CPU + NPU + ISP + DSP
- larger SRAM
- LPDDR + optional HBF
- SSD

센서 융합, 장시간 추론, 로컬 autonomy가 중요하므로 memory residency 관리의 효과가 크게 나타난다.

### Edge server / personal workstation

가장 먼저 HBF 효과가 드러날 제품군이다.

- CPU + larger TE/VE complex
- HBM-lite 또는 wide LPDDR
- HBF
- NVMe SSD

20B 이상 local assistant, multi-user on-prem agent, long-context inference에 적합하다.

## 13. 시뮬레이터 관점에서 무엇을 모델링해야 하는가

이 아키텍처는 전통적인 "matmul latency 중심 NPU simulator"보다 `memory residency simulator + token phase simulator`에 더 가깝다.

필요한 컴포넌트 모델은 다음과 같다.

- Tensor Engine latency model
- Vector Engine latency model
- LPDDR controller model
- HBF controller model
- tier-aware DMA model
- KV pager model
- prefetch predictor model

워크로드 모델은 최소 다음을 포함해야 한다.

- prefill / decode 분리
- sequence length growth
- multimodal overlap
- adapter switching
- expert imbalance
- shared prefix KV reuse

핵심 메트릭은 다음과 같다.

- token latency p50 / p99
- joule per token
- bytes per token by tier
- HBF hit ratio
- DRAM hot-set residency
- KV eviction rate
- prefetch usefulness
- stall reason breakdown
- HBF sequentiality ratio

이 메트릭들이 있어야 HBF가 실제로 대역폭을 유효하게 쓰는지, 단순히 느린 외부 메모리 하나가 더 추가된 것인지 구분할 수 있다.

## 14. 결론

엣지 AI 추론의 방향은 명확하다. LPDDR 또는 HBM-lite를 working memory로, HBF를 read-mostly warm reservoir로, SSD/UFS를 cold archive로 두고, phase-aware runtime이 weight, KV, adapter, expert를 계층 간에 재배치하는 구조가 가장 설득력 있다.

이 구조는 다음과 같은 전환을 의미한다.

- compute-centric NPU -> memory-centric inference system
- single-tier DRAM -> tiered memory hierarchy
- static model load -> dynamic residency management
- dense-only engine -> dense + vector + sparse + streaming engine

즉, 미래 엣지 LLM 시스템의 성패는 "연산기가 얼마나 큰가"보다 "모델과 상태를 얼마나 잘 배치하고 움직이는가"에 달려 있다. `LLM in a Flash`는 이 전환의 소프트웨어적 출발점이고, HBF는 이를 시스템 메모리 계층 차원으로 확장할 수 있는 다음 후보 계층이다.

## External References Checked on 2026-04-14

- Apple, `Running LLMs on-device with limited memory`, 2024-08: https://machinelearning.apple.com/research/efficient-large-language
- Sandisk and SK hynix HBF MOU, 2025-08-06: https://investor.sandisk.com/news-releases/news-release-details/sandisk-collaborate-sk-hynix-drive-standardization-high
- SK hynix AIN B / HBF announcement, 2025-10-27: https://news.skhynix.com/sk-hynix-presents-next-generation-nand-storage-product-strategy-at-ocp-2025/
- SK hynix and Sandisk HBF standardization kickoff, 2026-02-26: https://news.skhynix.com/sk-hynix-and-sandisk-begin-global-standardization-ofnext-generation-memory-hbf/
- `H3: Hybrid Architecture Using High Bandwidth Memory and High Bandwidth Flash for Cost-Efficient LLM Inference`, IEEE Computer Architecture Letters early access abstract index: https://eurekamag.com/research/103/475/103475220.php
