# Memory Hierarchy in AI

ChatTime: 2026년 1월 20일 23:10 (GMT+9)
Created time: 2026년 4월 6일 오후 11:02
ProjectName: NPU Architecture
URL: https://chatgpt.com/c/696f840f-336c-832f-a895-336d3af50eae

- **You: Nvidia H100 & B100/Q...**
    
    Nvidia H100 & B100/Qualcomm AHPM/AMD MI300 등에서 사용된 memory hierarchy
    
- **ChatGPT: 아래에서는 ** entity ["co...**
    
    아래에서는 ** entity ["company","NVIDIA","gpu company"]  H100 / B100**, ** entity ["company","Qualcomm","semiconductor company"]  AHPM**, ** entity ["company","AMD","semiconductor company"]  MI300** 계열을 중심으로, **AI 가속기에서 실제로 채택된 memory hierarchy**를 **연산–데이터 이동–전력/면적 트레이드오프** 관점에서 정리한다.(창훈님의 관심사인 **NPU 시스템 모델링·성능 모델·타일링·KV-cache**까지 바로 연결되도록 해석을 덧붙인다.)
    
    ## 1. NVIDIA H100 / B100 계열: “HW-managed cache + SW-managed shared memory”의 정점
    
    image_group {"query":["NVIDIA H100 memory hierarchy diagram","NVIDIA H100 HBM3 memory stack","NVIDIA GPU shared memory L2 cache diagram"]}
    
    ### (1) 계층 구조 요약
    
    ```
    Register File (per SM)
      ↓
    Shared Memory (SW-managed, scratchpad)
      ↓
    L1 Cache (부분 통합, load/store 캐시)
      ↓
    L2 Cache (대형, HW-managed)
      ↓
    HBM3 / HBM3e (off-chip)
    ```
    
    ### (2) 핵심 포인트
    
    • Register File
    
    • warp 단위 연산의 ultimate locality
    
    • Tensor Core MMA operand staging
    
    • Shared Memory (Scratchpad)
    
    • 프로그래머 제어
    
    • FlashAttention, tiled GEMM, softmax fusion의 핵심
    
    • L2 Cache
    
    • 수십 MB 규모
    
    • KV-cache reuse
    
    • HBM
    
    • 대역폭 극대화 (H100 ~3TB/s급)
    
    • 용량·전력·비용 부담 큼
    
    ### (3) 아키텍처적 철학
    
    • “프로그래머가 locality를 만들면 HW는 이를 증폭”
    
    • HW cache는 
    
    ➡️ **NPU 시사점**
    
    • TE/VE에 
    
    • L2급 공유 SRAM은 
    
    ## 2. Qualcomm AHPM: 모바일·엣지형 “on-chip SRAM 중심”
    
    image_group {"query":["Qualcomm AI accelerator memory hierarchy","Qualcomm NPU SRAM architecture","Qualcomm mobile AI memory subsystem"]}
    
    ### (1) 계층 구조 요약
    
    ```
    Register / Vector RF
      ↓
    Local SRAM (Tensor/Vector Engine 전용)
      ↓
    Shared On-chip SRAM (NPU cluster)
      ↓
    System Cache (SoC 공용, 선택적)
      ↓
    LPDDR
    ```
    
    ### (2) 핵심 포인트
    
    • HBM 없음
    
    • SRAM 용량/배치가 성능의 1차 결정 요인
    
    • DMA 기반 명시적 이동 (compiler/runtime 주도)
    
    ### (3) 아키텍처적 철학
    
    • “DRAM을 안 쓰는 것이 곧 성능”
    
    • 모델을 
    
    ➡️ **NPU 시사점**
    
    • SRAM partitioning + NoC
    
    • 컴파일러의 
    
    ## 3. AMD MI300 (MI300A / MI300X): “패키지 안에서 모든 걸 해결”
    
    image_group {"query":["AMD MI300 memory hierarchy diagram","AMD MI300 HBM memory architecture","AMD MI300 chiplet memory system"]}
    
    ### (1) 계층 구조 요약
    
    ```
    Register File
      ↓
    L1 / LDS (Local Data Share, scratchpad)
      ↓
    L2 Cache
      ↓
    HBM3 (on-package, massive capacity)
    ```
    
    ### (2) 핵심 포인트
    
    • HBM을 사실상 Last-Level Cache처럼 사용
    
    • MI300X: 대용량 HBM (192GB급)
    
    • CPU+GPU 통합(MI300A) → 
    
    ### (3) 아키텍처적 철학
    
    • “메모리를 숨기지 말고, 충분히 크게”
    
    • programming model 단순화 (HIP, unified address space)
    
    ➡️ **NPU 시사점**
    
    • 서버용 NPU라면 
    
    • 단, 
    
    ## 4. 세 아키텍처의 Memory Hierarchy 비교
    
    | 항목 | NVIDIA H100/B100 | Qualcomm AHPM | AMD MI300 |
    | --- | --- | --- | --- |
    | 최상위 DRAM | HBM3/3e | LPDDR | HBM3 |
    | 온칩 SRAM | Shared Mem + L2 | 대형 SRAM 중심 | LDS + L2 |
    | Cache 정책 | HW + SW 혼합 | 거의 없음 | HW cache |
    | SW 제어 | 매우 강함 (CUDA) | 매우 강함 (DMA) | 상대적으로 약함 |
    | KV-cache 전략 | L2 + HBM | SRAM fit | HBM 상주 |
    
    ## 5. 창훈님 연구(NPU 시스템 모델링)로의 직접 연결
    
    ### (1) **Simulator에서 반드시 모델링해야 할 요소**
    
    • SRAM vs DRAM traffic 분리
    
    • 타일당:
    
    • load bytes
    
    • reuse factor
    
    • spill 발생 여부
    
    • KV-cache lifetime 모델
    
    ### (2) 추천 NPU memory hierarchy (중립적)
    
    ```
    PE RF
     ↓
    Local Scratchpad (TE/VE 전용)
     ↓
    Cluster Shared SRAM (SW-managed)
     ↓
    Optional L2 Cache (HW-managed)
     ↓
    DRAM / HBM
    ```
    
    ### (3) 성능 모델 핵심 식 (개념)
    
    ```
    Latency ≈ max(
      Compute_cycles,
      SRAM_bandwidth_bound,
      DRAM_transfer / overlap_efficiency
    )
    ```
    
    → **NVIDIA식**: overlap_efficiency ↑→ **Qualcomm식**: DRAM_transfer ≈ 0→ **AMD식**: DRAM_transfer ↓ (HBM)
    
    ## 6. 결론 (아키텍처 철학 요약)
    
    • H100/B100
    
    • AHPM
    
    • MI300
    
    👉 **Custom NPU 설계에서는**
    
    > 
    
    “**SRAM을 얼마나, 어떻게, 누가 관리하느냐**”가 곧 성능이다.
    
    원하시면 다음 단계로:
    
    • KV-cache 전용 SRAM vs L2 설계
    
    • Prefill/Decode memory traffic 모델
    
    • Tile scheduler + memory contention 모델
    
    을 바로 이어서 정리해드릴 수 있다.
    
- **You: 왜 위 구조는 sram만 이용해서 구...**
    
    왜 위 구조는 sram만 이용해서 구현하지 않고 복잡한 캐시를 사용한거야?
    
- **ChatGPT: 이 질문은 **“왜 현대 AI 가속기... (Part 1/2)**
    
    이 질문은 **“왜 현대 AI 가속기는 *순수 SRAM scratchpad*로 가지 않고, 굳이 복잡한 cache hierarchy를 유지하는가?”**라는, **아키텍처 철학의 핵심을 찌르는 질문**이다.결론부터 말하면 **“SRAM만으로는 *성능·범용성·생산성*을 동시에 만족할 수 없기 때문”**이다. 아래에서 **물리적 한계 → 워크로드 특성 → 시스템/조직 현실** 순서로 논리적으로 정리한다.
    
    ## 1. 물리 법칙: SRAM은 “빠르지만 작고 비싸다”
    
    image_group {"query":["SRAM vs DRAM area comparison","CPU cache hierarchy die area","HBM vs SRAM die area comparison"]}
    
    ### (1) 면적·전력 스케일링
    
    • SRAM (6T)
    
    • 면적: DRAM 대비 
    
    • leakage 큼
    
    • 대용량화 시 die size 급증
    
    • Cache(L2/L3)
    
    • SRAM이지만 
    
    • working set 전체를 저장하지 않아도 “충분히 잘 동작”
    
    👉 **H100 L2 수십 MB**는 이미 “SRAM 한계선” 근처👉 “HBM 크기의 SRAM”은 **현실적으로 불가능**
    
    ## 2. 워크로드 특성: AI는 “완벽히 예측 가능한 스트리밍”이 아니다
    
    image_group {"query":["transformer attention memory access pattern","LLM KV cache access pattern","irregular memory access AI workloads"]}
    
    ### (1) Scratchpad 가정이 깨지는 지점
    
    SRAM-only 설계는 암묵적으로 가정한다:
    
    > 
    
    “데이터 접근 패턴을 **컴파일 타임에 100% 안다**”
    
    하지만 현실의 AI 워크로드:
    
    • LLM:
    
    • variable sequence length
    
    • dynamic batch
    
    • KV-cache access가 head/step마다 다름
    
    • Vision:
    
    • early-exit, conditional compute
    
    • Multimodal:
    
    • modality-dependent branching
    
    ➡️ **정적 DMA + scratchpad만으로는 coverage 부족**
    
    ### (2) Cache는 “불확실성 흡수층”
    
    • cache = 
    
    • “이 데이터, 다시 쓸지도 몰라”를 
    
    ## 3. 생산성의 벽: SRAM-only는 “컴파일러 지옥”
    
    image_group {"query":["AI compiler memory scheduling diagram","DMA scratchpad compiler complexity","GPU shared memory programming complexity"]}
    
    ### (1) SRAM-only의 숨겨진 비용
    
    순수 scratchpad 기반 설계에서 필요한 것:
    
    • perfect tiling
    
    • perfect double buffering
    
    • perfect liveness analysis
    
    • 모든 corner case 대응
    
    이건:
    
    • 컴파일러 난이도 ↑↑
    
    • bring-up 시간 ↑
    
    • 모델 바뀔 때마다 schedule 재작성
    
    ### (2) Cache는 조직적 보험
    
    대형 기업 입장에서는:
    
    • 모든 모델을 perfect하게 튜닝하는 것보다
    
    • cache로 80% 성능을 자동 확보
    
    ➡️ NVIDIA/AMD가 cache를 버리지 않는 이유
    
    ## 4. 그래서 나온 “혼합 구조” (중요)
    
    > 
    
    **SRAM-only vs Cache-only는 false dichotomy**
    
    ### (1) 실제 채택된 구조
    
    ```
    Register / Local SRAM      ← 100% deterministic
    Shared Memory / LDS        ← SW-managed
    L2 Cache                   ← HW-managed safety net
    HBM / DRAM                 ← capacity layer
    ```
    
    • 핵심 연산 경로
    
    • 불확실성 / 모델 변화
    
    ### (2) H100의 본질
    
    • 성능 핵심: 
    
    • cache는:
    
    • KV-cache
    
    • weight reuse
    
    • multi-SM sharing
    
    👉 cache가 없으면 **프로그래밍 모델이 붕괴**
    
    ## 5. “그럼 Qualcomm은 왜 가능한가?”
    
    image_group {"query":["mobile NPU SRAM architecture","edge AI accelerator scratchpad design"]}
    
    Qualcomm AHPM이 SRAM 중심인 이유:
    
    • 워크로드 제한:
    
    • mobile vision / audio / small LLM
    
    • batch 작음
    
    • sequence 짧음
    
    • 모델 fixed
    
    ➡️ **문제 공간을 줄였기 때문에 가능**
    
    ## 6. NPU 설계자를 위한 핵심 정리 (중요)
    
    ### (1) SRAM-only가 가능한 조건
    
    • 모델 구조 고정
    
    • sequence length 고정
    
    • batch 고정
    
    • compiler/runtime 완전 통제
    
    → **Edge / Robot reflex / Control loop**
    
    ### (2) Cache가 필요한 조건
    
    • LLM
    
    • multimodal
    
    • dynamic batching
    
    • server inference
    
    ## 7. 당신의 연구에 대한 직접적인 해석
    
    창훈님이 고민 중인:
    
    • TE/VE + SRAM + NoC
    
    • explicit tile scheduler
    
    • cycle-level simulator
    
    이건 **“Qualcomm + NVIDIA의 hybrid”**에 해당한다.
    
    > 
    
    ❗ 단, **L2-like shared SRAM or cache**는 반드시 필요
    
    ### 추천 설계적 타협
    
    • “cache”라는 이름이 아니어도 됨
    
    • Victim buffer / software-visible cache
    
    • or 
    
    ## 8. 결론 (한 문장)
    
    > 
    
    **Cache는 성능을 올리기 위한 장치가 아니라,‘SRAM-only 설계가 감당하지 못하는 현실을 흡수하는 장치’다.**
    
    원하시면 다음 중 하나를 바로 이어가자:
    
    1. SRAM-only NPU가 실패하는 실제 케이스 시뮬레이션
    
- **ChatGPT: 이 질문은 **“왜 현대 AI 가속기... (Part 2/2)**
    
    1. L2 cache vs Shared SRAM 면적·전력 비교 모델
    
    1. KV-cache 전용 hierarchy 설계 (decode 최적화)
    
- **You: llm의 복잡한 데이터 접근 패턴에서...**
    
    llm의 복잡한 데이터 접근 패턴에서 하드웨어가 대신 판단 가능해? 어떤 형태의 하드웨어가 필요해?
    
- **ChatGPT: 질문의 핵심은 이것이다: > **LL... (Part 1/2)**
    
    질문의 핵심은 이것이다:
    
    > 
    
    **LLM처럼 복잡하고 동적인 데이터 접근에서“다시 쓸지 모른다”를 하드웨어가 실제로 판단할 수 있는가?가능하다면 어떤 *형태*의 하드웨어가 필요한가?**
    
    결론부터 명확히 말하면,
    
    > 
    
    **하드웨어는 ‘의미’를 이해하지 못한다.하지만 “재사용 가능성(reuse likelihood)”은 통계적으로 판단할 수 있다.**
    
    그리고 현대 GPU/NPU는 **의미 판단이 아니라,① 패턴 감지 + ② 실패 비용이 낮은 추측(speculation)**으로 이 문제를 해결한다.
    
    아래에서 **LLM 데이터 접근 → HW가 관찰하는 신호 → 필요한 하드웨어 블록** 순서로 체계적으로 설명한다.
    
    ## 1. LLM의 “복잡한 데이터 접근”을 분해하면 사실 3가지다
    
    image_group {"query":["LLM KV cache access pattern","transformer attention memory access","LLM decode phase memory access pattern"]}
    
    LLM에서 문제가 되는 접근은 막연히 랜덤이 아니다. 실제로는:
    
    ### (1) **Phase-dependent locality**
    
    • Prefill
    
    • Q,K,V를 순차적으로 streaming
    
    • tile 단위 재사용 높음
    
    • Decode
    
    • 과거 KV 전체를 반복 접근
    
    • temporal locality 매우 강함
    
    👉 HW는 “지금은 decode phase”라는 걸 **명시적으로 알 필요가 없다**👉 **접근 패턴 변화**로 감지 가능
    
    ### (2) **Stride + broadcast 패턴**
    
    • KV-cache:
    
    • 동일 K/V가
    
    • 여러 head, 여러 matmul에서 반복 사용
    
    • 주소는:
    
    • stride 고정
    
    • 반복 주기 명확
    
    👉 이는 **cache prefetcher / reuse detector가 매우 잘 잡는 패턴**
    
    ### (3) **불규칙성은 ‘경계’에서만 발생**
    
    • sequence 길이 변화
    
    • batch 합쳐짐/쪼개짐
    
    • speculative decoding rollback
    
    👉 이런 경우는 **항상 소수**👉 cache miss 비용이 전체 성능을 지배하지 않음
    
    ## 2. “HW가 대신 판단”한다는 말의 정확한 의미
    
    중요한 오해부터 정리해야 한다.
    
    ❌ HW가 하는 것:
    
    • “이건 KV-cache다”
    
    • “이건 다음 토큰에서도 쓰인다”
    
    ✅ HW가 실제로 하는 것:
    
    • “이 주소가 
    
    • “이 라인에 
    
    • “최근 N-cycle 동안 
    
    즉,
    
    > 
    
    **의미 기반 판단 ❌통계 + 관찰 기반 판단 ⭕**
    
    ## 3. 이를 가능하게 하는 하드웨어 구성요소
    
    아래는 **실제로 산업계에서 쓰이는/검증된 블록들**이다.
    
    ### 3.1 Reuse distance 감지 하드웨어 (Cache tag logic)
    
    image_group {"query":["CPU cache reuse distance concept","cache hit miss timeline diagram"]}
    
    ### 핵심 메커니즘
    
    • 각 cache line:
    
    • last access timestamp
    
    • HW는 implicit하게 계산:
    
    • “얼마 만에 다시 접근됐는가”
    
    ### 결과
    
    • 짧으면 → 
    
    • 길면 → 
    
    👉 LLM KV-cache는:
    
    • decode 단계에서 
    
    • 자동으로 L2에 남음
    
    ### 3.2 다중 consumer 감지 (MSHR / scoreboard 확장)
    
    LLM에서 중요한 특징:
    
    • 같은 데이터가 여러 연산에 fan-out
    
    하드웨어는 이를 이렇게 감지한다:
    
    • 동일 cache line에 대해
    
    • multiple outstanding requests
    
    • MSHR entry 공유
    
    👉 결과:
    
    • “이 데이터는 hot”
    
    • eviction 우선순위 ↓
    
    ### 3.3 Stream vs Reuse 분리 (Bypass logic)
    
    image_group {"query":["cache bypass logic diagram","streaming access cache bypass"]}
    
    하드웨어는 다음을 구분한다:
    
    • 한 번 읽고 끝
    
    • 짧은 주기로 반복
    
    방법:
    
    • 연속 접근 + no re-reference → bypass
    
    • re-reference 발생 → cacheable
    
    👉 이게 없으면:
    
    • weight streaming이 cache를 오염시킴
    
    ### 3.4 Sector / sub-line cache (KV-cache에 매우 중요)
    
    KV-cache 특징:
    
    • head 단위 접근
    
    • 벡터 일부만 사용
    
    하드웨어 대응:
    
    • sector cache
    
    • partial line valid bits
    
    👉 “라인 전체를 다 쓰지 않아도 보존 가치 판단 가능”
    
    ## 4. 그럼 NPU에는 어떤 형태가 필요할까?
    
    창훈님의 NPU 관점에서 **현실적인 답**을 정리하면:
    
    ### 4.1 반드시 필요한 것 (최소 세트)
    
    ```
    [Shared L2-like SRAM]
     ├─ Tag + reuse tracking
     ├─ Multi-consumer aware MSHR
     ├─ Streaming bypass
    ```
    
    이건 “CPU급 cache”가 아니라:
    
    • LLM 특화 victim-aware buffer
    
    ### 4.2 선택적으로 강력해지는 요소
    
    • Decode 단계 힌트:
    
    • compiler가 “likely reuse” bit 제공
    
    • KV region pinning:
    
    • eviction 금지 영역
    
    • Age-based eviction:
    
    • decode token 경계에서 age reset
    
- **ChatGPT: 질문의 핵심은 이것이다: > **LL... (Part 2/2)**
    
    👉 이건 **cache + scratchpad의 중간 형태**
    
    ## 5. 중요한 결론 (아주 중요)
    
    > 
    
    **LLM에서 하드웨어는 ‘판단’하지 않는다.다만 ‘틀려도 괜찮은 추측’을 매우 싸게 반복할 뿐이다.**
    
    • cache hit → 큰 이득
    
    • miss → 성능 일부 손실
    
    • correctness 영향 ❌
    
    그래서:
    
    • HW speculation이 성립
    
    • SRAM-only 설계에서는 이게 불가능
    
    ## 6. 한 문장 요약 (설계자 관점)
    
    > 
    
    **LLM의 복잡성은 ‘의미’가 아니라 ‘통계’이고,통계는 하드웨어가 가장 잘 다룬다.**
    
    ### 다음으로 이어가기 좋은 주제
    
    1. KV-cache 전용 L2 설계 (pinning / aging / sectoring)
    
    1. SRAM-only NPU에서 이 기능을 흉내 내는 방법
    
    1. Prefill ↔ Decode phase 전환을 HW가 감지하는 방법
    
    원하시면 바로 이어서 깊게 파겠습니다.