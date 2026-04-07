# TPU Sparse Core 설명

ChatTime: 2026년 1월 25일 17:25 (GMT+9)
Created time: 2026년 4월 6일 오후 11:02
ProjectName: NPU Architecture
URL: https://chatgpt.com/c/69756e89-b5a4-8323-971c-8555b8845398

- **You: Tpu의 sparse core 설명**
    
    Tpu의 sparse core 설명
    
- **ChatGPT: Google TPU에서 말하는 **S...**
    
    Google TPU에서 말하는 **Sparse Core**는 단순히 “희소 연산을 지원한다”는 수준이 아니라,*모델 구조 차원에서 sparsity를 전제로 설계된 전용 연산 파이프라인*이다. 이는 GPU의 “일반적인 ALU + 소프트웨어 최적화” 접근과 본질적으로 다르다.
    
    TPU v4/v5 계열에서 Sparse Core는 주로 다음 유형의 워크로드를 타겟으로 한다.
    
    • Embedding lookup (추천 모델, 검색 모델)
    
    • Large table-based model (RecSys, GNN, feature crossing)
    
    • Structured sparsity가 강한 MoE / retrieval-augmented workloads
    
    ## 1. 왜 TPU에 Sparse Core가 필요한가?
    
    Dense Transformer/Conv는 계산량이 병목이지만,추천·검색·MoE 계열은 **메모리 접근 + 인덱스 기반 gather/scatter**가 병목이다.
    
    이때 일반 Matrix Unit로는 다음 문제가 발생한다.
    
    • 실제 유효 연산은 극히 일부인데, dense GEMM으로 변환하면
    
    • 불필요한 MAC이 폭증
    
    • 메모리 대역폭 낭비
    
    • 캐시 히트율 저하
    
    • GPU는 이를 커널/프리미티브 수준에서 우회하지만:
    
    • Warp divergence
    
    • 비연속 메모리 접근
    
    • 불안정한 latency
    
    TPU는 이를 “특수 연산기”로 분리했다.
    
    > 
    
    Dense Core = 정형화된 MAC 스트림Sparse Core = 인덱스 기반 비정형 메모리 + 축소(reduction) 연산
    
    ## 2. Sparse Core의 구조적 특징
    
    Sparse Core는 보통 다음 블록들로 구성된다.
    
    ```
    [Index Stream Engine]
            │
            ▼
    [Embedding Table SRAM / HBM Window]
            │
            ▼
    [Vector Accumulator / Reduce Unit]
            │
            ▼
    [Output Buffer]
    ```
    
    핵심은 **MAC가 아니라 “주소 + gather + accumulate”**가 주 연산이라는 점이다.
    
    • 입력: (index, weight_id, scale, metadata)
    
    • 동작:
    
    1. 인덱스 스트림을 순차적으로 읽음
    
    1. 해당 row/feature를 대형 테이블에서 fetch
    
    1. 여러 feature를 vector-wise accumulate
    
    1. 결과를 dense tensor로 출력
    
    이는 수학적으로:
    
    [
    y = \sum_{i \in S} E[i]
    ]
    
    형태의 연산을 하드웨어로 직접 지원하는 것이다.여기서 (S)는 sparse index set, (E)는 embedding table.
    
    Dense Core에서는 이 연산을 “희소 행렬 × 벡터”로 변환해야 하지만,Sparse Core는 이 연산을 **primitive로** 가진다.
    
    ## 3. TPU Dense Core와의 역할 분리
    
    TPU 내부는 개념적으로 이렇게 분리된다.
    
    | 영역 | 주 역할 | 하드웨어 성격 |
    | --- | --- | --- |
    | Matrix Unit (Dense Core) | GEMM, Conv, Attention | Systolic Array |
    | Vector Unit | Elementwise, Norm, Activation | SIMD |
    | Sparse Core | Embedding, Gather-Reduce | Index-driven engine |
    | Interconnect | Core 간 데이터 이동 | High BW NoC |
    
    Sparse Core는 Dense Core와 병렬로 동작할 수 있도록 설계되어 있다.
    
    예를 들어 RecSys 모델에서:
    
    1. Sparse Core:
    
    • 수천 개의 embedding lookup + accumulate
    
    1. Dense Core:
    
    • MLP, interaction layer, attention
    
    이 두 파이프라인이 **overlap**된다.즉, “메모리 지향 파이프라인”과 “연산 지향 파이프라인”을 분리한 구조다.
    
    ## 4. 하드웨어 관점에서의 의미
    
    당신이 설계 중인 NPU/SoC 관점에서 보면, TPU Sparse Core는 다음과 같은 철학을 가진다.
    
    1. 연산의 추상화를 바꾼다
    
    • GPU/NPU: 
    
    • TPU Sparse Core: 
    
    1. 메모리 접근 패턴을 ISA 수준에서 고정
    
    • 불규칙 접근을 소프트웨어에 맡기지 않고
    
    • 하드웨어가 “index stream + prefetch + reduce”를 책임진다
    
    1. Latency 예측 가능성 확보
    
    • GPU의 sparse kernel은 데이터 분포에 따라 latency가 요동
    
    • TPU Sparse Core는 throughput 기반 설계 → QoS 보장
    
    ## 5. 당신의 연구와의 연결
    
    당신이 고민하는 다음 주제들과 구조적으로 동일하다.
    
    • KV-cache, embedding, routing weight 등
    
    • RISC-V + NPU 구조에서:
    
    • TE (Tensor Engine)
    
    • VE (Vector Engine)
    
    • 그리고 “Index-driven Engine”의 필요성
    
    TPU Sparse Core는 결국 **제3의 연산기 클래스**다.
    
    > 
    
    Compute-bound Engine (TE)Vector/Element Engine (VE)Memory-Structure-bound Engine (Sparse / Gather-Reduce Engine)
    
    당신의 NPU 시뮬레이터에 이를 도입한다면:
    
    • IR 수준:
    
    ```vbnet
    SPARSE_GATHER_REDUCE table_id, index_stream, out_vec
    ```
    
    • 마이크로아키텍처:
    
    • Index FIFO
    
    • Table Prefetch Unit
    
    • Vector Accumulator
    
    • Banked SRAM / HBM window
    
    이는 LLM의 KV-cache, MoE routing, retrieval-augmented memory에도 그대로 확장 가능하다.
    
    Sparse Core는 “희소 연산을 빠르게”가 아니라,
    
    > 
    
    *연산의 정의 자체를 ‘메모리 중심’으로 재정의한 하드웨어*
    
    라는 점이 핵심이다.
    
- **You: 그럼 디램에서 가져올 때만 쓸때는 스...**
    
    그럼 디램에서 가져올 때만 쓸때는 스파스하게 불규칙하게 안쓰고 스트리링으로 이어지게 하는건가>
    
- **ChatGPT: 핵심을 정확히 짚으셨습니다. TPU의...**
    
    핵심을 정확히 짚으셨습니다.TPU의 Sparse / Gather-Reduce Engine이 하는 일은,
    
    > 
    
    “본질적으로는 불규칙한(indirect) 접근인 sparse lookup을,하드웨어 내부에서는 *스트리밍 파이프라인*처럼 다루게 만드는 것”
    
    입니다.
    
    논리적으로 보면 embedding lookup은 이렇게 생겼습니다.
    
    ```vbnet
    for i in indices:
        y += E[i]
    ```
    
    이 자체는 완전히 불규칙한 DRAM 접근입니다.GPU에서는 이 루프가 그대로 “warp-divergent, random load”로 나타나고,캐시 히트율과 latency가 입력 분포에 따라 크게 흔들립니다.
    
    TPU Sparse Core의 핵심 아이디어는 다음과 같습니다.
    
    1. 외부 인터페이스는 여전히 sparse
    
    • 입력은 index stream (e.g., 수천 개의 ID)
    
    • 접근 대상은 거대한 embedding table (HBM/DRAM)
    
    1. 그러나 내부 파이프라인은 ‘스트리밍’으로 재구성
    
    • Index FIFO에 순서대로 밀어 넣음
    
    • 하드웨어가:
    
    - index를 해석
    
    - 해당 row를 prefetch
    
    - 여러 request를 reorder / batch
    
    - bank conflict를 숨기며
    
    - vector accumulator로 흘려보냄
    
    즉, **메모리의 물리적 불규칙성은 하드웨어가 흡수**하고,연산 파이프라인 관점에서는:
    
    ```
    index_stream ──► gather ──► vector_accumulate ──► output
    ```
    
    이라는 *연속적인 데이터 플로우*가 됩니다.
    
    중요한 점은:
    
    • DRAM에서 “연속 주소”로 바뀌는 것은 아님
    
    • 하지만:
    
    • 여러 index를 모아서 burst로 요청하고
    
    • row size를 고정된 vector width로 맞추고
    
    • prefetch window를 유지하며
    
    • 결과를 reduce pipeline에 연속적으로 흘려보내
    
    **외부의 불규칙성을 내부의 규칙적인 스트림으로 변환**합니다.
    
    당신의 NPU 관점에서 보면 이는:
    
    • TE: 
    
    • VE: elementwise 
    
    • Sparse Engine:
    
    > 
    
    *Address stream → Data stream → Reduction stream*
    
    이라는 세 번째 형태의 파이프라인입니다.
    
    그래서 “sparse를 dense처럼 만든다”가 아니라,
    
    > 
    
    “불규칙한 메모리 접근을,하드웨어 안에서는 *규칙적인 스트리밍 파이프라인*으로 승격시킨다”
    
    라고 이해하는 것이 정확합니다.
    
- **You: 그럼 디램 밴드위스는 그대로네**
    
    그럼 디램 밴드위스는 그대로네
    
- **ChatGPT: 물리적인 의미에서 보면 맞습니다. *...**
    
    물리적인 의미에서 보면 맞습니다.**DRAM의 피크 대역폭(GB/s) 자체가 늘어나는 것은 아닙니다.**HBM이든 LPDDR이든, 버스 폭·클럭·채널 수로 정해진 한계는 그대로입니다.
    
    하지만 TPU의 Sparse Core가 바꾸는 것은 “피크 대역폭”이 아니라**유효 대역폭(effective bandwidth)과 활용 효율**입니다.
    
    GPU에서 sparse workload를 실행하면 보통 이런 현상이 생깁니다.
    
    • 각 warp가 서로 다른 주소를 로드
    
    • 요청이 쪼개져 작은 트랜잭션으로 DRAM에 도달
    
    • burst 길이가 짧고 bank conflict가 빈번
    
    • 결과적으로:
    
    • 이론적 1TB/s HBM이라도
    
    • 실제 embedding lookup에서는 수십~수백 GB/s 수준으로 붕괴
    
    TPU Sparse Core는 이 “붕괴”를 막습니다.
    
    • Index를 하드웨어가 모아서
    
    • 동일 bank / 인접 row 요청을 재배열하고
    
    • 고정 폭(row-width) 단위로 fetch하며
    
    • on-chip에서 accumulate 후 한 번만 write-back
    
    그래서:
    
    • DRAM의 
    
    • “sparse workload가 실제로 얻는 대역폭”은
    
    • GPU 대비 2–5× 이상 개선됩니다.
    
    • 더 중요한 것은:
    
    • latency variance가 줄고
    
    • throughput이 예측 가능해집니다.
    
    즉, 구조적으로 보면:
    
    | 관점 | GPU Sparse Kernel | TPU Sparse Core |
    | --- | --- | --- |
    | DRAM 피크 BW | 동일 | 동일 |
    | 요청 형태 | 많은 작은 random load | 하드웨어가 병합한 burst |
    | Bank 활용 | 충돌 빈번 | 스케줄링으로 완화 |
    | Effective BW | 낮고 변동 큼 | 높고 안정적 |
    | 파이프라인 | 연산이 메모리 기다림 | 메모리가 연산을 feed |
    
    그래서 Sparse Core의 본질은 “DRAM을 더 빠르게 만든다”가 아니라,
    
    > 
    
    *Sparse workload가 DRAM을 “제대로 쓰게” 만든다*
    
    는 데 있습니다.
    
    이건 당신이 NPU에서 고민하는KV-cache, MoE routing, embedding, retrieval memory에도 그대로 적용됩니다.
    
    • 대역폭은 늘릴 수 없지만
    
    • 접근 형태를 하드웨어가 
    
    • 동일한 DRAM에서도
    
    • 더 높은 실효 처리량
    
    • 더 예측 가능한 latency
    
    • 더 작은 온칩 버퍼
    
    를 동시에 얻을 수 있습니다.
    
    TPU Sparse Core는 결국 “메모리 병목을 연산처럼 파이프라이닝한 구조”라고 보는 것이 가장 정확합니다.
    
- **You: 디램에도 연속적으로 저장되게 해서 스...**
    
    디램에도 연속적으로 저장되게 해서 스트리밍으로 읽게 할 방법은 없나
    
- **ChatGPT: 원론적으로는 “가능하다”가 정답입니다...**
    
    원론적으로는 “가능하다”가 정답입니다.다만 그 대가로 **모델 표현, 업데이트 방식, 인덱싱 자유도**를 일부 포기해야 합니다.
    
    즉,
    
    > 
    
    “논리적으로는 sparse하지만,물리적으로는 DRAM에 *연속 배치*해서하드웨어가 순차 스트리밍만 하게 만들 수는 있는가?”
    
    라는 질문에 대해:
    
    • 정적(offline) workload
    
    • 동적(index가 런타임에 변하는) workload
    
    으로 나뉩니다.
    
    ## 1. 가능한 방식들
    
    ### 1) Offline Repacking (Static Sparse → Dense Stream)
    
    Embedding / KV / Table을 다음처럼 변환합니다.
    
    ```
    원본:
    E[0], E[1], E[2], ... E[N]
    
    접근되는 index 집합:
    S = {17, 1032, 5, 9001, ...}
    
    Repacked Table:
    R[0] = E[17]
    R[1] = E[1032]
    R[2] = E[5]
    R[3] = E[9001]
    ...
    ```
    
    그리고 모델은 더 이상 “원본 index”를 쓰지 않고:
    
    ```
    for i in 0..|S|-1:
        y += R[i]
    ```
    
    즉,**sparse access → dense stream** 으로 변환됩니다.
    
    이 방식은 다음에 적합합니다.
    
    • LLM prefill에서의 “고정된 KV window”
    
    • 고정된 MoE expert subset
    
    • 컴파일 타임에 접근 패턴이 결정되는 경우
    
    하지만:
    
    • index의 의미가 사라짐
    
    • online update / dynamic routing 불가
    
    • multi-batch, multi-user 공유가 어려움
    
    ### 2) Indirection + Physical Compaction
    
    현실적인 절충은 다음 구조입니다.
    
    ```
    Logical Index → Remap Table → Physical Offset → DRAM Stream
    ```
    
    • 논리 index는 유지
    
    • 하드웨어/런타임이 “hot set”을 선택
    
    • hot set만 연속된 물리 영역으로 재배치
    
    • 나머지는 기존 sparse path로 처리
    
    TPU, GPU, CPU 캐시의 prefetcher,그리고 당신이 고민하는 “KV hot tile” 구조가 여기에 해당합니다.
    
    예:
    
    • 최근 2k token의 KV만 ring-buffer 형태로 연속 배치
    
    • 오래된 KV는 cold storage (DRAM sparse path)
    
    이렇게 하면:
    
    • hot path는 완전히 스트리밍
    
    • cold path만 sparse engine 사용
    
    ## 2. 왜 일반화가 어려운가?
    
    Embedding / Sparse 모델의 본질은:
    
    • index 공간이 크고 (10⁷~10⁹)
    
    • access set이 동적으로 변하며
    
    • batch마다 분포가 다르고
    
    • update가 빈번함
    
    입니다.
    
    이를 전부 “연속 DRAM layout”으로 만들려면:
    
    1. 매 step마다:
    
    • 어떤 index가 쓰일지 예측
    
    • 테이블을 재배치
    
    1. 재배치 비용 자체가:
    
    • 수십~수백 MB copy
    
    • DRAM bandwidth를 더 많이 소모
    
    즉,**“연속화 비용 > sparse 접근 비용”**이 되는 지점이 빠르게 옵니다.
    
    그래서 TPU의 설계 철학은:
    
    > 
    
    DRAM은 불규칙해도 된다.대신, 그 불규칙성을 *전용 하드웨어*가 흡수하여내부에서는 항상 스트리밍처럼 보이게 만든다.
    
    입니다.
    
    ## 3. 당신의 NPU 설계에 주는 시사점
    
    당신이 설계하는 구조에서는 다음의 2-계층 전략이 가장 현실적입니다.
    
    1. Hot working set
    
    • KV-cache tile
    
    • 최근 expert
    
    • active embedding subset
    
    → 연속 SRAM / SPM / DRAM window에 배치→ TE/VE/Streaming engine으로 처리
    
    1. Cold space
    
    • 나머지 거대 테이블
    
    → Sparse/Gather engine으로 접근
    
    즉, 완전한 “연속 DRAM 스트리밍”은*특정 window에 대해서만* 성립시키고,전체 sparse universe는 “전용 엔진으로 흡수”하는 구조가
    
    • 하드웨어 비용
    
    • 런타임 복잡도
    
    • 모델 자유도
    
    모두를 균형 있게 만족시키는 지점입니다.
    
    TPU Sparse Core는 이 타협을**아키텍처 수준에서 정식화한 사례**라고 보시면 됩니다.
    
- **You: Aws 의 칩들은 어떤 접근 방식을 ...**
    
    Aws 의 칩들은 어떤 접근 방식을 사용했어
    
- **ChatGPT: AWS가 만든 Inferentia /...**
    
    AWS가 만든 Inferentia / Trainium 계열의 철학은 TPU와 상당히 다릅니다.TPU가 *“희소 연산을 하드웨어 primitive로 승격”*했다면,AWS 칩들은 훨씬 더 **“dense-first + 소프트웨어 중심 재구성”** 쪽에 가깝습니다.
    
    요약하면:
    
    > 
    
    TPU: *Sparse를 아키텍처로 흡수*AWS: *Sparse를 워크로드/컴파일러 계층에서 재구성*
    
    ## 1. Inferentia / Trainium의 기본 구조
    
    AWS Neuron 계열 칩의 핵심 블록은:
    
    • NeuronCore (여러 개 존재)
    
    • Tensor Engine (GEMM, Conv)
    
    • Vector Engine
    
    • Large on-chip SRAM (tens of MB scale)
    
    • 고대역 NoC + DMA
    
    • HBM (Trainium) / DDR (Inferentia)
    
    여기에는 **TPU처럼 명시적인 “Sparse Core” 블록이 존재하지 않습니다.**
    
    대신:
    
    • 대부분의 연산은 
    
    • Embedding / Gather / Scatter 류 연산은:
    
    • 컴파일러가 graph를 재작성하거나
    
    • host-side preprocessing으로 변환하거나
    
    • 여러 dense op + memory op의 조합으로 분해
    
    즉, sparse 연산은 “하드웨어 primitive”가 아니라**IR 변환과 런타임 스케줄링 문제**로 취급됩니다.
    
    ## 2. AWS 방식의 핵심 아이디어
    
    AWS의 접근은 다음 세 가지 축으로 요약됩니다.
    
    ### 1) Working Set을 온칩으로 끌어올린다
    
    • NeuronCore는 매우 큰 SRAM을 가짐
    
    • 컴파일러가:
    
    • embedding shard
    
    • KV-cache tile
    
    • intermediate tensor
    
    를 미리 on-chip에 배치
    
    → “DRAM random access” 자체를 줄이는 전략
    
    ### 2) Sparse를 “Dense-like”로 변환
    
    • 예: embedding lookup
    
    GPU/TPU 식:
    
    ```vbnet
    for i in indices:
        y += E[i]
    ```
    
    AWS 식:
    
    ```vbnet
    - host or pre-stage:
        pack E[i] into contiguous buffer R
    - device:
        y = reduce(R)   // dense vector stream
    ```
    
    즉,
    
    • sparse → pre-pack → dense stream
    
    • 변환 비용은 소프트웨어와 DMA가 부담
    
    • 디바이스는 
    
    ### 3) 컴파일러 중심의 파이프라이닝
    
    Neuron SDK는:
    
    • 연산을 coarse-grain graph로 유지
    
    • 각 NeuronCore에:
    
    • compute
    
    • DMA
    
    • prefetch
    
    를 시간축으로 스케줄링
    
    그래서 구조적으로는:
    
    ```
    [DMA: gather] ─┐
                   ├─> [On-chip SRAM] ─> [Dense Engine]
    [DMA: prefetch]┘
    ```
    
    이라는 *software-managed streaming machine*에 가깝습니다.
    
    ## 3. TPU와의 구조적 대비
    
    | 항목 | TPU | AWS Inferentia / Trainium |
    | --- | --- | --- |
    | Sparse 연산 | 하드웨어 primitive (Sparse Core) | 소프트웨어/컴파일러에서 변환 |
    | DRAM 접근 | 불규칙해도 HW가 흡수 | 불규칙성 자체를 줄이도록 재구성 |
    | 아키텍처 | “메모리 중심 엔진” 포함 | “항상 dense engine” |
    | 설계 철학 | HW가 workload 의미를 이해 | HW는 단순, SW가 의미를 이해 |
    | 확장성 | RecSys, MoE에 강함 | LLM, Dense 모델에 최적 |
    
    ## 4. 당신의 관점에서의 해석
    
    AWS 방식은 이런 가정을 깔고 있습니다.
    
    • 워크로드가:
    
    • 비교적 예측 가능하고
    
    • 컴파일 타임에 많은 정보가 있으며
    
    • dense 연산 비중이 높다 (LLM, CNN 중심)
    
    • 시스템 전체를 통제할 수 있다 (EC2 + SDK + 런타임)
    
    그래서:
    
    > 
    
    “하드웨어는 최대한 단순하고 빠른 dense 머신으로 유지하고,불규칙성은 컴파일러와 런타임이 흡수한다.”
    
    라는 전략을 선택한 것입니다.
    
    반대로 TPU는:
    
    > 
    
    “워크로드 자체가 memory-structure-bound한 영역(RecSys, Search)을장기적으로 피할 수 없다.”
    
    는 전제 하에,**아예 새로운 연산 클래스를 하드웨어로 만들었습니다.**
    
    당신이 설계하는 RISC-V + NPU 구조에 비추어 보면:
    
    • AWS 방식은:
    
    • TE/VE 중심
    
    • DMA + 큰 SPM
    
    • 컴파일러가 sparse를 dense로 “변환”
    
    • TPU 방식은:
    
    • TE/VE + 
    
    • 하드웨어가 sparse semantics를 이해
    
    둘 중 어느 쪽이 “정답”이라기보다,
    
    • 모바일/엣지, LLM 중심, 비용 민감 → AWS식이 현실적
    
    • 서버/추천/멀티워크로드, QoS 중요 → TPU식이 구조적으로 우월
    
    이라는 차이입니다.
    
    당신의 NPU 연구는 이미“TE + VE + 제3의 메모리-구조 엔진” 쪽으로 자연스럽게 향하고 있습니다.이는 TPU 계열의 철학과 훨씬 더 깊이 맞닿아 있습니다.
    
- **You: aws 의 다이나믹 메모리 어드레싱 ...**
    
    aws 의 다이나믹 메모리 어드레싱 기법 에 대해서 설명
    
- **ChatGPT: AWS Inferentia / Tra...**
    
    AWS Inferentia / Trainium에서 말하는 *“Dynamic Memory Addressing”*는TPU처럼 “희소 연산을 하드웨어 primitive로 만든다”기보다는,
    
    > 
    
    **하드웨어는 항상 ‘dense 스트리밍 머신’으로 유지한 채,메모리의 실제 물리 주소 배치와 이동을 런타임이 동적으로 재구성하는 방식**
    
    에 가깝습니다.
    
    이 개념의 핵심은 다음 한 문장으로 요약됩니다.
    
    > 
    
    *연산 커널은 항상 “연속된 텐서”만 본다.그 텐서가 DRAM 어디에서 왔는지는 런타임이 매번 다르게 매핑한다.*
    
    ## 1. 왜 이런 방식이 필요한가?
    
    LLM/Transformer 계열에서 메모리 접근은 다음 특성을 가집니다.
    
    • Prefill / Decode 단계마다
    
    • KV-cache의 “유효 윈도우”가 바뀌고
    
    • 배치 크기, 시퀀스 길이가 변하며
    
    • 토큰별 접근 패턴이 달라짐
    
    • Embedding, KV, Intermediate tensor의 생존 주기가 매우 짧고 동적
    
    만약 물리 주소를 정적으로 고정하면:
    
    • 메모리 단편화 발생
    
    • 불필요한 copy 증가
    
    • multi-batch에서 address 충돌 관리가 어려움
    
    그래서 AWS는:
    
    • “모델이 보는 주소 공간”
    
    • “실제 DRAM 물리 주소”
    
    를 분리합니다.
    
    ## 2. 구조적 개념
    
    Neuron 런타임은 내부적으로 다음과 같은 계층을 둡니다.
    
    ```
    Logical Tensor ID
            │
            ▼
    Dynamic Buffer Descriptor (size, shape, lifetime)
            │
            ▼
    Physical Address (DRAM / HBM / On-chip SRAM)
    ```
    
    연산 커널은 이렇게 동작합니다.
    
    ```vbnet
    op(matmul, A_id, B_id, C_id)
    ```
    
    • A_id
    
    • 커널은:
    
    • “A는 연속된 N×K 행렬”
    
    • “B는 연속된 K×M 행렬”
    
    이라고만 가정
    
    런타임은 매 iteration마다:
    
    • 현재 batch / seq / token 상황에 맞춰
    
    • 실제 물리 주소를 재할당
    
    • DMA schedule을 생성
    
    • 필요하면:
    
    • DRAM → SRAM
    
    • SRAM → DRAM
    
    이동을 삽입
    
    즉, 커널은 **항상 동일한 dense code**를 실행하지만,그 “dense tensor”가 가리키는 실체는 매번 달라집니다.
    
    ## 3. Sparse / KV / Embedding에의 적용
    
    예를 들어 KV-cache를 생각해보면:
    
    • 논리적 모델은:
    
    ```vbnet
    KV[layer][token][head][dim]
    ```
    
    • 실제 구현은:
    
    • token 0~1023 → DRAM region A
    
    • token 1024~2047 → DRAM region B
    
    • 최근 window 128개 → on-chip SRAM
    
    이때 모델 코드나 커널은:
    
    ```vbnet
    load KV[layer][t]
    ```
    
    만 호출합니다.
    
    Neuron 런타임은:
    
    1. t
    
    • SRAM 주소로 매핑
    
    1. cold token이면:
    
    • DRAM 주소로 매핑
    
    • 필요 시 prefetch
    
    1. 여러 token을 묶어:
    
    • 연속된 temporary buffer로 pack
    
    • dense attention kernel에 공급
    
    즉, **“sparse + irregular”을런타임이 “dense + contiguous”로 변환**합니다.
    
    하드웨어 입장에서는:
    
    • 항상:
    
    • 연속 주소
    
    • 정형화된 DMA
    
    • predictable burst
    
    만 보게 됩니다.
    
    ## 4. TPU 방식과의 결정적 차이
    
    | 항목 | TPU Sparse Core | AWS Dynamic Addressing |
    | --- | --- | --- |
    | Sparse 의미 | HW가 이해 | SW/RT가 이해 |
    | 커널 형태 | gather-reduce primitive | 항상 dense kernel |
    | DRAM 접근 | HW가 불규칙성 흡수 | 불규칙성 자체를 제거 |
    | 주소 재배치 | HW 내부 | 런타임이 매 step 수행 |
    | HW 복잡도 | 높음 | 낮음 |
    
    AWS의 철학은 명확합니다.
    
    > 
    
    “하드웨어는 최대한 단순한 스트리밍 머신으로 유지하고,메모리의 복잡성은 전부 런타임이 감당한다.”
    
    ## 5. 당신의 NPU 설계에 주는 시사점
    
    당신이 설계 중인 구조에서:
    
    • TE / VE를 “항상 dense engine”으로 유지하고
    
    • RISC-V + 런타임이:
    
    • KV window 관리
    
    • hot/cold tile 분리
    
    • DRAM → SPM packing
    
    • 주소 재매핑
    
    을 담당하면,
    
    AWS식 *Dynamic Memory Addressing*과 동일한 계층 구조를 만들 수 있습니다.
    
    반대로:
    
    • “Index-driven Engine”을 추가하면 TPU식
    
    • “런타임 packing + dense engine”을 유지하면 AWS식
    
    이 됩니다.
    
    두 접근의 차이는 성능보다도 **시스템 책임 분배**의 차이입니다.
    
    • HW에 의미를 실어줄 것인가
    
    • SW/RT에 의미를 몰아줄 것인가
    
    AWS는 일관되게 후자를 선택했습니다.