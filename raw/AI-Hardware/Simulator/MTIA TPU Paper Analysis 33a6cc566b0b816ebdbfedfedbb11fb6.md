# MTIA TPU 논문 분석

ChatTime: 2026년 1월 13일 21:22 (GMT+9)
Created time: 2026년 4월 6일 오후 11:08
ProjectName: NPU Simulator
URL: https://chatgpt.com/c/6965df64-a9dc-8331-b8b0-298dce56955c

- **You: MTIA, TPU 논문 분석 정리**
    
    MTIA, TPU 논문 분석 정리
    
- **ChatGPT: 아래는 **MTIA와 TPU를 “논문... (Part 1/2)**
    
    아래는 **MTIA와 TPU를 “논문·공개자료 기반 아키텍처 관점”에서 비교·분석한 정리**입니다.단순 소개가 아니라, **왜 이런 구조가 나왔는지**, **어떤 워크로드를 겨냥했는지**, 그리고 **NPU/컴파일러·시스템 모델링 관점에서 무엇을 배울 수 있는지**에 초점을 맞춥니다.
    
    ## 1.  entity ["company","Meta","facebook parent company"]  **MTIA (Meta Training & Inference Accelerator)**
    
    image_group {"query":["Meta MTIA accelerator die photo","Meta MTIA architecture block diagram","Meta AI accelerator MTIA board","Meta data center AI accelerator MTIA"]}
    
    ### 1.1 배경과 설계 철학
    
    • 목표
    
    • GPU 의존 탈피 → 
    
    • 핵심 문제의식:
    
    • GPU는 
    
    • Meta 워크로드는 
    
    • → 
    
    ### 1.2 아키텍처 핵심 특징
    
    ### (1) **Near-Memory Compute**
    
    • DRAM 접근 패턴 최적화
    
    • 대규모 Embedding table lookup을 위한 
    
    • GPU 대비:
    
    • Compute peak ↓
    
    • BW/Compute 비율 ↑
    
    ### (2) **INT8 중심 연산**
    
    • Training도 일부 지원하지만 실질적으로 
    
    • RecSys에서 
    
    • → Quantization-aware HW 설계
    
    ### (3) **단순한 실행 모델**
    
    • GPU처럼 범용 ISA + massive threading ❌
    
    • Static scheduling + graph-level orchestration
    
    • SW 스택:
    
    • PyTorch → 내부 compiler → MTIA IR
    
    • 런타임 오버헤드 최소화
    
    ### 1.3 MTIA의 본질 요약
    
    > 
    
    **“연산기가 아니라 메모리 시스템을 중심으로 설계된 AI 가속기”**
    
    • LLM prefill/decode 관점:
    
    • Prefill: 성능 평범
    
    • Decode: KV-cache, embedding lookup에 유리
    
    • 당신의 NPU 시뮬레이터 관점에서 보면:
    
    • DRAM–SRAM traffic 모델링이 핵심
    
    • FLOPs 모델보다 
    
    ## 2.  entity ["company","Google","search and cloud company"]  **TPU (Tensor Processing Unit)**
    
    image_group {"query":["Google TPU v1 board","Google TPU systolic array diagram","TPU v4 architecture diagram","Google TPU data center pod"]}
    
    ### 2.1 TPU의 근본적 아이디어
    
    • 논문: 
    
    • 핵심 질문:
    
    > 
    
    “**행렬곱만 빠르면 딥러닝은 끝나는가?**”
    
    • 답:
    
    • Yes (적어도 2015~2020까지는)
    
    ### 2.2 TPU 아키텍처 핵심
    
    ### (1) **Systolic Array (MXU)**
    
    • 대규모 
    
    • 데이터 재사용 극대화
    
    • 특징:
    
    • Control 거의 없음
    
    • Pipeline = Clockwork
    
    ### (2) **On-chip SRAM 극대화**
    
    • Weight/Activation을 DRAM에서 멀리 떼어냄
    
    • TPU v1:
    
    • ~28MB Unified Buffer
    
    • TPU v4:
    
    • 수백 MB급 HBM + 대형 SRAM hierarchy
    
    ### (3) **Static Compilation (XLA)**
    
    • 런타임 스케줄링 ❌
    
    • 모든 loop, tile, schedule을 
    
    • 장점:
    
    • 예측 가능한 latency
    
    • 에너지 효율 극대화
    
    ### 2.3 TPU의 본질 요약
    
    > 
    
    **“컴파일러가 마이크로아키텍처를 직접 조종하는 기계”**
    
    • TPU는 HW라기보다:
    
    • XLA + Systolic Machine
    
    • NPU 설계 교훈:
    
    • ISA보다 
    
    • Tile 크기 = 성능 결정 변수
    
    ## 3. MTIA vs TPU : 구조적 비교
    
    | 구분 | MTIA | TPU |
    | --- | --- | --- |
    | 주 타겟 | RecSys, LLM Inference | Training + Inference |
    | 중심 자원 | DRAM BW | On-chip SRAM |
    | 연산 구조 | Scalar/Vector 중심 | Dense Systolic Array |
    | 정밀도 | INT8 | bfloat16 / INT |
    | 스케줄링 | 비교적 단순 | 완전 Static (XLA) |
    | 성능 모델 | Memory-bound | Compute-bound |
    
    ## 4. Prefill / Decode 관점 재해석
    
    ### Prefill (MxM 중심)
    
    • TPU: ✔️ 매우 강함
    
    • MTIA: ❌ 비효율적
    
    ### Decode (MxV + KV-cache)
    
    • TPU: SRAM 부족 시 비효율
    
    • MTIA: ✔️ Memory-centric 구조에 유리
    
    > 
    
    **결론**MTIA는 “Decode 최적화 ASIC”TPU는 “Prefill + Training 머신”
    
    ## 5. 당신의 NPU Simulator에 주는 설계 인사이트
    
    ### 5.1 아키텍처 레벨
    
    • MTIA 스타일:
    
    • DRAM latency + BW contention 모델 필수
    
    • Embedding/Lookup operator 1급 시민화
    
    • TPU 스타일:
    
    • Tile SRAM capacity sweep
    
    • Static schedule cost model
    
    ### 5.2 컴파일러/IR 관점
    
    • TPU → 
    
    • MTIA → 
    
    • 추천:
    
    • Compute IR
    
    • Bytes moved / cycle 을 1급 metric으로
    
    ## 6. 핵심 Takeaway (한 문장씩)
    
    • MTIA
    
    • TPU
    
- **ChatGPT: 아래는 **MTIA와 TPU를 “논문... (Part 2/2)**
    
    다음 단계로 의미 있는 확장은 아래 중 하나입니다:
    
    1. MTIA vs Groq LPU vs TPU
    
    1. Prefill/Decode 분리형 
    
    1. 당신의 Py-V/NPU simulator에 맞춘 
    
    원하면 바로 **시뮬레이터 설계 스펙 문서(PRD/SDD)** 형태로 정리해줄 수 있습니다.
    
- **You: 아마존의 npu도 분석**
    
    아마존의 npu도 분석
    
- **ChatGPT: 아래는 **아마존(AWS)의 NPU ... (Part 1/2)**
    
    아래는 **아마존(AWS)의 NPU 계열(Trainium / Inferentia)을 논문·공개자료·아키텍처 관점에서 체계적으로 분석한 정리**입니다.앞서 정리한 **TPU·MTIA와의 “설계 철학 차이”**가 자연스럽게 드러나도록 구성했습니다.
    
    # entity ["company","Amazon Web Services","cloud computing company"]  NPU 분석
    
    ## (Trainium / Inferentia 아키텍처 중심)
    
    image_group {"query":["AWS Trainium chip architecture diagram","AWS Inferentia accelerator card","AWS Neuron SDK architecture","AWS Trainium server rack"]}
    
    ## 1. AWS NPU가 등장한 이유 (TPU·MTIA와 다른 출발점)
    
    AWS의 질문은 다음이었습니다:
    
    > 
    
    **“우리는 모델을 만드는 회사가 아니라,모델을 *대규모로 안정적으로* 돌려야 하는 회사다.”**
    
    따라서 AWS NPU의 목표는:
    
    • 최고 FLOPs ❌
    
    • 최저 latency ❌
    
    • Cloud TCO (Total Cost of Ownership) 최소화 ✔️
    
    • SW 스택 안정성 + 범용성 ✔️
    
    이 관점에서 AWS는 **두 개의 NPU를 분리**합니다.
    
    | 칩 | 목적 |
    | --- | --- |
    | **Inferentia** | Inference 전용 |
    | **Trainium** | Training 전용 |
    
    ## 2.  entity ["company","Amazon","e-commerce and cloud company"]  **Inferentia** (Inference NPU)
    
    ### 2.1 설계 목표
    
    • EC2 인스턴스에서 GPU 대체
    
    • LLM, CV, RecSys 
    
    • 고객 모델 다양성 대응이 핵심
    
    ### 2.2 아키텍처 특징
    
    ### (1) **NeuronCore**
    
    • Inferentia v1 기준:
    
    • 칩당 다수의 NeuronCore
    
    • 각 Core = Vector + Matrix + Control
    
    • 개념적으로:
    
    • TPU의 MXU보다 
    
    • GPU SM보다 
    
    ### (2) **INT8 / FP16 혼합**
    
    • 고객 모델 정확도 요구 다양
    
    • 완전 INT8-only ❌
    
    • Mixed-precision을 HW에서 자연스럽게 지원
    
    ### (3) **온칩 SRAM + DRAM 스트리밍**
    
    • SRAM은 “캐시”가 아니라 
    
    • 컴파일러가:
    
    • 어떤 weight를 SRAM에 둘지
    
    • 언제 DRAM에서 가져올지 결정
    
    > 
    
    TPU와 달리 **SRAM 크기를 공격적으로 키우지 않음**
    
    ### 2.3 실행 모델
    
    • Neuron SDK
    
    • PyTorch / TensorFlow frontend
    
    • Graph capture → Neuron IR
    
    • 특징:
    
    • Fully static ❌
    
    • Semi-static scheduling
    
    • 이유:
    
    • Cloud workload의 
    
    ### 2.4 Inferentia 요약
    
    > 
    
    **“GPU보다 싸고, TPU보다 유연한 Inference 엔진”**
    
    • Decode 중심 LLM:
    
    • MTIA보단 compute 강함
    
    • TPU보단 memory 효율적
    
    • 완전한 특정 모델 최적화 ❌
    
    • Cloud generality 최적화 ✔️
    
    ## 3. **Trainium** (Training NPU)
    
    image_group {"query":["AWS Trainium architecture block diagram","AWS Trainium NeuronCore v2","AWS Trainium HBM memory architecture"]}
    
    ### 3.1 Trainium의 본질적 포지션
    
    • NVIDIA A100/H100의 
    
    • TPU v4와 직접 경쟁
    
    • 그러나 목표는:
    
    • Peak 성능 ❌
    
    • /step 최소화
    
    ### 3.2 아키텍처 핵심
    
    ### (1) **NeuronCore v2**
    
    • Matrix Engine (Dense GEMM)
    
    • Vector Engine (Elementwise, Reduction)
    
    • Scalar Control
    
    → **당신이 설계 중인 TE/VE 분리 구조와 매우 유사**
    
    ### (2) **HBM 중심 구조**
    
    • 칩당 대용량 HBM
    
    • Activation/Gradient residency ↑
    
    • Off-chip traffic 최소화
    
    ### (3) **고정밀 학습 지원**
    
    • FP32 ❌
    
    • BF16 / FP16 중심
    
    • Gradient accumulation을 HW 친화적으로 설계
    
    ### 3.3 Interconnect & Scale-out
    
    • Trainium UltraCluster
    
    • 수천 개 칩을 하나의 training job으로 묶음
    
    • 설계 철학:
    
    • Chip 자체보다는 
    
    ### 3.4 Trainium 요약
    
    > 
    
    **“TPU보다 유연하고, GPU보다 예측 가능한 Training 시스템”**
    
    ## 4. AWS NPU vs TPU vs MTIA (핵심 비교)
    
    | 항목 | AWS Inferentia | AWS Trainium | TPU | MTIA |
    | --- | --- | --- | --- | --- |
    | 주 타겟 | Inference | Training | Training+Inference | Inference |
    | 철학 | 범용 Cloud | $/step | Peak 효율 | Memory BW |
    | 실행 모델 | Semi-static | Static-ish | Fully static | Static |
    | SRAM | 중간 | 중간 | 큼 | 작음 |
    | DRAM | 중요 | HBM 중심 | 덜 중요 | 최중요 |
    | 고객 | 외부 고객 | 외부 고객 | 내부+외부 | 내부(Meta) |
    
    ## 5. LLM Prefill / Decode 관점
    
    ### Prefill (Dense GEMM)
    
    • TPU ≈ Trainium >> Inferentia > MTIA
    
    ### Decode (Memory-bound)
    
    • MTIA ≈ Inferentia > Trainium > TPU
    
    AWS의 전략:
    
    • Inferentia로 Decode 비용 절감
    
    • Trainium으로 Training & Prefill 흡수
    
    → **역할 분리형 NPU 전략**
    
    ## 6. 당신의 NPU Simulator 관점에서의 교훈
    
    ### 6.1 아키텍처
    
    • AWS는 
    
- **ChatGPT: 아래는 **아마존(AWS)의 NPU ... (Part 2/2)**
    
    • TE/VE/Scalar의 균형
    
    • SRAM은 충분하지만 과하지 않음
    
    → **Design Space Exploration(DSE) 친화적 구조**
    
    ### 6.2 컴파일러 / IR
    
    • TPU: IR이 HW를 지배
    
    • AWS: HW와 IR의 타협
    
    • 시뮬레이터에 추천:
    
    ```vbnet
    IR = {
      compute_graph,
      memory_schedule,
      precision_policy
    }
    ```
    
    • Static / Semi-static 모드 스위치 가능하게
    
    ## 7. 한 문장 결론
    
    • Inferentia
    
    • Trainium
    
    • AWS 전략 전체
    
    ### 다음으로 이어가기 좋은 주제
    
    1. Trainium vs TPU v4 vs NVIDIA H100 – $/token 모델
    
    1. AWS Neuron ↔ XLA ↔ MLIR IR 구조 비교
    
    1. Prefill/Decode 분리형 Dual-NPU SoC 제안
    
    1. 당신의 NPU simulator에 맞춘 
    
    원하면 바로 **SDD/PRD 문서**나 **아키텍처 ASCII/메모리 타이밍 모델**로 구체화해줄 수 있습니다.
    
- **You: 3개 회사의 아키텍처 비교 분석 리포...**
    
    3개 회사의 아키텍처 비교 분석 리포트
    
- **ChatGPT: 아래는 **Meta · Google ...**
    
    아래는 **Meta · Google · Amazon(AWS)** 3개 회사의 **NPU 아키텍처를 ‘설계 철학–마이크로아키텍처–컴파일러–LLM 워크로드(Prefill/Decode)’ 관점에서 비교 분석한 종합 리포트**입니다.단순 나열이 아니라, **왜 서로 다른 구조를 택했는지**와 **시스템/NPU 시뮬레이터 설계에 어떤 교훈을 주는지**까지 포함합니다.
    
    # 3대 빅테크 NPU 아키텍처 비교 분석 리포트
    
    ### Meta MTIA vs Google TPU vs Amazon Trainium/Inferentia
    
    image_group {"query":["Meta MTIA architecture diagram","Google TPU v4 architecture diagram","AWS Trainium architecture block diagram"],"num_per_query":1}
    
    ## 1. Executive Summary (요약)
    
    | 회사 | NPU 이름 | 한 문장 요약 |
    | --- | --- | --- |
    | ** entity ["company","Meta","facebook parent company"] ** | MTIA | *메모리 병목을 직접 겨냥한 내부 워크로드 전용 추론 ASIC* |
    | ** entity ["company","Google","search and cloud company"] ** | TPU | *컴파일러가 지배하는 대규모 행렬 흐름 머신* |
    | ** entity ["company","Amazon Web Services","cloud computing company"] ** | Trainium / Inferentia | *클라우드 TCO 최적화를 위한 가장 균형적인 범용 NPU* |
    
    ## 2. 출발점의 차이: “누구의 문제를 푸는가”
    
    ### Meta (MTIA)
    
    • 문제 정의: 
    
    • 워크로드: RecSys, Embedding, LLM Decode
    
    • 특징: 
    
    ### Google (TPU)
    
    • 문제 정의: 
    
    • 워크로드: Training + Inference (Dense GEMM)
    
    • 특징: 
    
    ### Amazon (AWS NPU)
    
    • 문제 정의: 
    
    • 워크로드: 범용 (CV, LLM, RecSys 혼합)
    
    • 특징: 
    
    👉 **아키텍처 차이는 기술 문제가 아니라 “비즈니스 문제”의 반영**
    
    ## 3. 마이크로아키텍처 비교
    
    ### 3.1 Compute 구조
    
    | 항목 | MTIA | TPU | AWS Trainium |
    | --- | --- | --- | --- |
    | 핵심 연산 | Vector / Simple Matrix | 대형 Systolic Array | Matrix + Vector 분리 |
    | 연산 밀도 | 낮음 | 매우 높음 | 중간 |
    | 제어 로직 | 단순 | 거의 없음 | 비교적 풍부 |
    
    • TPU
    
    • MTIA
    
    • AWS
    
    ### 3.2 메모리 계층
    
    | 항목 | MTIA | TPU | AWS |
    | --- | --- | --- | --- |
    | 온칩 SRAM | 작음 | 매우 큼 | 중간 |
    | DRAM 중요도 | ★★★★★ | ★★ | ★★★★ |
    | 메모리 관리 | 정적 + 스트리밍 | 완전 정적 | SW-managed |
    
    • MTIA = 
    
    • TPU = 
    
    • AWS = 
    
    ## 4. 실행 모델 & 컴파일러 철학
    
    ### 4.1 컴파일 방식
    
    | 항목 | MTIA | TPU | AWS |
    | --- | --- | --- | --- |
    | Scheduling | Static | Fully Static | Semi-static |
    | IR의 역할 | 단순 lowering | HW를 지배 | HW와 타협 |
    | 런타임 | 매우 얇음 | 거의 없음 | 존재 |
    
    • TPU는 
    
    • AWS는 
    
    • MTIA는 
    
    ### 4.2 IR 관점 요약
    
    • TPU:
    
    ```
    IR = Schedule + Tile + Placement
    ```
    
    • MTIA:
    
    ```
    IR = Graph + Memory Flow
    ```
    
    • AWS:
    
    ```
    IR = Graph + Partial Schedule + Policy
    ```
    
    👉 **당신의 NPU Simulator에 매우 중요한 시사점**
    
    ## 5. LLM Prefill / Decode 관점 비교
    
    ### 5.1 Prefill (MxM, Dense)
    
    | 순위 | 가속기 |
    | --- | --- |
    | 1 | TPU ≈ Trainium |
    | 2 | Inferentia |
    | 3 | MTIA |
    
    ### 5.2 Decode (MxV, KV-cache, Memory-heavy)
    
    | 순위 | 가속기 |
    | --- | --- |
    | 1 | MTIA |
    | 2 | Inferentia |
    | 3 | Trainium |
    | 4 | TPU |
    
    > 
    
    **LLM은 이미 “두 개의 다른 컴퓨터 문제”**
    
    ## 6. 시스템 스케일링 전략
    
    ### Meta
    
    • 단일 노드 효율 ↑
    
    • 내부 트래픽 최적화
    
    ### Google
    
    • TPU Pod
    
    • 대규모 Systolic mesh
    
    ### AWS
    
    • UltraCluster
    
    • Fabric + SW orchestration 중심
    
    👉 **Chip보다 Interconnect의 비중이 증가**
    
    ## 7. NPU 시뮬레이터 / 시스템 아키텍트 관점의 핵심 교훈
    
    ### 7.1 성능 모델링
    
    • TPU 스타일 → 
    
    • MTIA 스타일 → 
    
    • AWS 스타일 → 
    
    ### 7.2 반드시 분리해야 할 축
    
    ```vbnet
    1) Compute topology
    2) Memory residency
    3) Scheduling policy
    4) Precision policy
    ```
    
    이 4가지를 독립 파라미터로 두면→ **TPU-like / MTIA-like / AWS-like 아키텍처를 모두 포괄 가능**
    
    ## 8. 최종 한 문장 비교
    
    • Meta MTIA
    
    • Google TPU
    
    • Amazon AWS NPU
    
    ## 9. 다음 단계 제안 (연구·설계용)
    
    1. 3-way DSE 시뮬레이션
    
    • SRAM size × DRAM BW × Tile size
    
    1. Prefill/Decode 분리형 SoC 제안
    
    1. MTIA-like Decode NPU + TPU-like Prefill NPU 이종 결합
    
    1. 이 리포트를 기반으로:
    
    • 📄 
    
    • 📊 
    
    • 🧪 
    
    원하시면 이 내용을 **PDF 리포트 / PRD·SDD 문서 / ASCII·Mermaid 아키텍처 다이어그램** 형태로 바로 고정 문서화해드릴 수 있습니다.
    
- **ChatGPT: 아래는 앞서 제시한 **“Archit... (Part 1/2)**
    
    아래는 앞서 제시한 **“Architecture Whitepaper”**를 실제로 **연구·내부 제안·특허·로드맵 문서로 바로 사용할 수 있도록 확장·구조화한 설명**입니다.즉, *“무엇을 쓰는 문서인가”*가 아니라 **“어떻게 써야 경쟁력이 생기는가”**에 초점을 둡니다.
    
    # Architecture Whitepaper란 무엇인가
    
    ### (MTIA · TPU · AWS NPU 비교 맥락에서)
    
    ## 1. Whitepaper의 정체성
    
    **Architecture Whitepaper**는 다음 중 어느 것도 아닙니다:
    
    • ❌ 마케팅 자료
    
    • ❌ 단순 기술 소개
    
    • ❌ 구현 상세(SDD)
    
    대신 다음을 목표로 합니다:
    
    > 
    
    **“왜 이 아키텍처가 이 문제에 최적인가를경쟁 아키텍처와의 비교를 통해 증명하는 문서”**
    
    즉,
    
    • Decision document
    
    • Design philosophy document
    
    • Research → Product 전환용 문서
    
    입니다.
    
    ## 2. 이 Whitepaper의 핵심 질문
    
    이번 리포트(Meta / Google / AWS 비교)를 기준으로 하면,Architecture Whitepaper는 반드시 아래 질문에 답해야 합니다.
    
    ### Q1. 왜 하나의 NPU로는 LLM을 최적화할 수 없는가?
    
    • Prefill ≠ Decode
    
    • Compute-bound ≠ Memory-bound
    
    • Static ≠ Dynamic
    
    → **문제의 분해 자체가 핵심 기여**
    
    ### Q2. 기존 3대 빅테크는 어떤 선택을 했는가?
    
    • ** entity ["company","Meta","facebook parent company"] **
    
    • 내부 RecSys / Decode 비용 절감
    
    • → Memory-centric, MTIA
    
    • ** entity ["company","Google","search and cloud company"] **
    
    • Dense training 효율 극대화
    
    • → Systolic + Static, TPU
    
    • ** entity ["company","Amazon Web Services","cloud computing company"] **
    
    • Cloud TCO, 범용성
    
    • → Balanced, Trainium / Inferentia
    
    → **모두 옳지만, 모두 다른 문제를 풀고 있음**
    
    ### Q3. 그렇다면 새로운 아키텍처의 설계 기준은 무엇인가?
    
    여기서 Whitepaper의 *Original Contribution*이 등장합니다.
    
    ## 3. Architecture Whitepaper의 권장 목차 (실전용)
    
    아래 구조는 **학술 논문 + 산업계 화이트페이퍼 중간 지점**입니다.
    
    ### 1. Introduction
    
    • LLM inference/training의 구조적 분리:
    
    • Prefill vs Decode
    
    • 기존 가속기의 한계 요약
    
    > 
    
    핵심 문장 예:
    *“Modern LLM workloads expose a fundamental mismatch between compute-centric accelerators and memory-dominated decoding phases.”*
    
    ### 2. Workload Decomposition
    
    • LLM 연산을 다음 축으로 분해:
    
    ```
    (A) GEMM-dominant
    (B) KV-cache-dominant
    (C) Embedding / Lookup
    (D) Elementwise / Control-heavy
    ```
    
    👉 **이 분해 자체가 논문/특허 포인트**
    
    ### 3. Comparative Architecture Analysis
    
    (= 지금까지 대화한 내용의 정제 버전)
    
    • MTIA / TPU / AWS NPU를
    
    • Compute topology
    
    • Memory hierarchy
    
    • Scheduling model
    
    • Compiler coupling
    
    기준으로 비교
    
    중요:
    
    • 수치 나열 ❌
    
    • Trade-off 구조를 그림과 표로 설명 ✔️
    
    ### 4. Key Observation (핵심 관찰)
    
    예시:
    
    1. Static schedule은 Prefill에는 최적이나 Decode에는 비효율
    
    1. SRAM 확장은 Decode latency를 선형으로 줄이지 않음
    
    1. DRAM BW 최적화는 Decode에서 FLOPs 증가보다 효과적
    
    → **“왜”에 대한 설명이 핵심**
    
    ### 5. Proposed Architecture (가장 중요한 장)
    
    여기서 선택지는 여러 개가 가능합니다.
    
    ### Option A. Dual-NPU Architecture
    
    • Prefill NPU (TPU-like)
    
    • Decode NPU (MTIA-like)
    
    ### Option B. Mode-switchable NPU
    
    • Static Mode / Streaming Mode
    
    • Compiler policy로 전환
    
    ### Option C. Tile-centric Unified NPU
    
    • Tile SRAM residency를 1급 개념으로 승격
    
    ⚠️ Whitepaper에서는 **구현 상세를 쓰지 않습니다**
    
    • “어떻게” ❌
    
    • “왜 이 구조가 합리적인가” ✔️
    
    ### 6. Compiler & System Implication
    
    • IR 분리 제안:
    
    ```
    Compute IR
    Memory IR
    Schedule Policy IR
    ```
    
    • Static / Semi-static / Dynamic의 공존
    
    👉 **HW 단독이 아니라 시스템 제안임을 강조**
    
    ### 7. Evaluation Methodology (수치 없이도 가능)
    
    • 실제 칩 성능 ❌
    
    • 대신:
    
    • Bytes/op
    
    • SRAM residency ratio
    
    • DRAM transaction count
    
    • Prefill/Decode latency model
    
    → **시뮬레이터 기반 평가 정당화**
    
    ### 8. Conclusion
    
    • 단일 NPU 시대의 종료
    
    • Workload-aware architecture의 필요성
    
    ## 4. 이 Whitepaper가 강력해지는 이유
    
    ### (1) “우리가 최고다”를 말하지 않는다
    
    → 대신
    
    • “문제가 이렇게 다층적이다”
    
    • “기존 해법은 이 지점에서 막힌다”
    
    ### (2) 특정 회사 비판 ❌
    
- **ChatGPT: 아래는 앞서 제시한 **“Archit... (Part 2/2)**
    
    → 구조적 한계만 제시
    
    ### (3) 구현이 아니라 **사고 프레임**을 제안
    
    → 장기적으로 매우 강함 (특허·연구·조직 설득)
    
    ## 5. 당신의 NPU Simulator와의 직접적 연결
    
    이 Whitepaper는 단독 문서가 아니라:
    
    • 📄 Whitepaper
    
    • 📐 Architecture Spec
    
    • 🧪 Simulator (DSE)
    
    의 **상위 개념 문서**입니다.
    
    즉:
    
    > 
    
    *“왜 이런 시뮬레이터가 필요한가”*를 설명하는 문서
    
    ## 6. 다음에 바로 할 수 있는 작업
    
    원하면 다음 중 하나로 바로 진행할 수 있습니다:
    
    1. 실제 Whitepaper 초안 작성
    
    • 학술 톤 / 산업 톤 중 선택
    
    1. Whitepaper → 
    
    1. Whitepaper → 
    
    1. Prefill/Decode 분리 아키텍처를 
    
    어느 방향으로 확장할지 말씀 주시면,**“바로 제출 가능한 문서” 수준으로 만들어드리겠습니다.