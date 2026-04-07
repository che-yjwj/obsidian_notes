# coral npu (Part 2/2)

- **You: 지금까지 대화내용을 정리해줘.**
    
    지금까지 대화내용을 정리해줘.
    
- **ChatGPT: 아래는 **지금까지의 전체 대화 내용... (Part 1/2)**
    
    아래는 **지금까지의 전체 대화 내용을 “의사결정에 바로 쓰는 구조적 요약”**으로 정리한 것입니다.(엣지 → 모바일 → 클라우드 NPU로 확장되는 **설계 철학·SW 스택·아키텍처 진화**를 한 번에 볼 수 있게 구성)
    
    # 1. 출발점: Coral NPU의 정체성
    
    ### Coral NPU란?
    
    • 초저전력 Edge AI
    
    • RISC-V 기반 하드웨어 + MLIR/StableHLO 중심 컴파일러 + 매우 단순한 런타임
    
    • 타깃 워크로드:
    
    • CV, Audio, Context-aware AI
    
    • 작은 모델, 정적 그래프, single-stream
    
    ### Coral의 핵심 가치
    
    • IR 중심 설계
    
    • 프로그래머블 NPU
    
    • AOT 컴파일 + deterministic 실행
    
    ➡️ Coral은 **“Proto-NPU (원형)”**에 해당
    
    # 2. 핵심 질문 ①
    
    ## “Coral 구조가 고성능 모바일 AP에도 맞는가?”
    
    ### 결론
    
    • ❌ 
    
    • ✅ 
    
    ### 이유
    
    • 모바일 AP는:
    
    • 멀티 워크로드 (Camera + LLM + Vision)
    
    • DRAM/SLC 공유
    
    • LLM Prefill / Decode
    
    • QoS / preemption / 지속 성능
    
    ➡️ Coral의
    
    • 단일 실행 모델
    
    • SRAM 전제
    
    • run-to-completion
    
    은 **모바일 요구사항과 불일치**
    
    # 3. Coral → Mobile AP NPU 진화 트리
    
    ### Stage 정리
    
    | Stage | 의미 |
    | --- | --- |
    | Stage 0 | Coral NPU (엣지, proto) |
    | Stage 1 | 멀티 유닛 확장 |
    | Stage 2 | 타일/클러스터 기반 모바일 NPU |
    | Stage 3 | **LLM-aware 모바일 AP NPU** |
    | Stage 4 | **Compiler-defined NPU (미래)** |
    
    ### 핵심 변화
    
    • 단일 NPU → 
    
    • SRAM-only → 
    
    • 정적 실행 → 
    
    • 연산 중심 → 
    
    # 4. 핵심 질문 ②
    
    ## “Coral 소프트웨어 스택을 모바일 AI에서 그대로 쓸 수 있는가?”
    
    ### 결론
    
    • ❌ 
    
    • ✅ 
    
    ### 재사용 가능
    
    • StableHLO / MLIR 기반 
    
    • IR 중심 풀스택 철학
    
    • AOT 컴파일 모델
    
    ### 반드시 재설계 필요
    
    • Runtime (multi-queue, event/fence, QoS)
    
    • 메모리 모델 (KV cache, spill, lifetime)
    
    • LLM Decode 대응
    
    ➡️ **“Coral 컴파일러 + 모바일 전용 런타임”**이 정답
    
    # 5. 모바일 AI 확장을 위한 구체적 방법론 (요약)
    
    ### (1) IR 확장
    
    • 연산 IR → 
    
    • Prefill / Decode / Always-on 
    
    ### (2) Runtime 교체
    
    • run-to-completion ❌
    
    • Multi-queue + priority + preemption
    
    ### (3) Prefill / Decode 분리
    
    • Prefill: throughput, 큰 GEMM
    
    • Decode: latency, KV cache, 작은 GEMM
    
    ### (4) 메모리는 “크기”가 아니라 “정책”
    
    • SRAM banking
    
    • lifetime 기반 allocation
    
    • KV locality hint
    
    ### (5) Heterogeneous 실행 경로
    
    • Large GEMM path
    
    • Small GEMM / KV path
    
    • Vector / elementwise path
    
    # 6. Vendor 매핑 (모바일 AP 관점)
    
    | 회사 | 위치 | 특징 |
    | --- | --- | --- |
    | Apple | Stage 3 → 4 | SoC 통합, UMA, 시스템 레벨 최적화 |
    | Qualcomm | Stage 3 | Micro-tile, concurrency, fused AI engine |
    | Samsung | Stage 3 | Heterogeneous NPU core 강조 |
    
    ➡️ 3사 모두 **Coral 이후 세계(Stage 3)**에 있음
    
    # 7. 핵심 질문 ③
    
    ## “클라우드 서버형 NPU로 확장하려면?”
    
    ### 패러다임 전환
    
    • 병목이 
    
    ### 필수 고려사항
    
    1. HBM / 근접 메모리
    
    1. Chip-to-chip / Node-to-node 인터커넥트
    
    1. Collective 통신 (AllReduce 등)
    
    1. 멀티테넌시 / 가상화 / QoS
    
    1. RAS (ECC, 오류 격리, 복구)
    
    1. 서빙 스택 (KV paging, batching, admission control)
    
    ➡️ 서버 NPU는 **“가속기”가 아니라 “분산 시스템의 노드”**
    
    # 8. 전체 흐름을 한 문장으로
    
    > 
    
    **Coral은 엣지 AI를 위한 “NPU의 원형”이고,모바일 AP는 이를 타일·클러스터·메모리 중심으로 재구성한 Stage-3 구조이며,클라우드 서버형 NPU는 여기에 분산·가상화·서빙·신뢰성을 얹은 시스템이다.**
    
    # 9. 창훈님 프로젝트 관점에서의 위치
    
    이미 하고 계신 것:
    
    • IR 다단계 분리
    
    • Prefill / Decode 관심
    
    • Dependency / ROB / Queue 모델
    
    • NPU Simulator + 성능 모델링
    
    👉 **이미 Coral 이후(Stage-3~서버 초입) 세계에 있음**👉 Coral은 **참고용 출발점**, 현재 방향은 **정확**
    
    # 10. 다음 단계로 가장 자연스러운 선택
    
    이 요약을 기반으로 바로 이어가기 좋은 작업:
    
    1. 모바일/서버 공용 NPU IR 설계 문서
    
    1. Prefill/Decode 성능 모델 수식화
    
    1. 서버형 NPU Runtime & Serving 아키텍처
    
    1. Edge–Mobile–Server 통합 NPU 로드맵
    
- **ChatGPT: 아래는 **지금까지의 전체 대화 내용... (Part 2/2)**
    
    원하시면 이 중 하나를 **바로 문서/스펙/다이어그램 수준**으로 만들어드릴게요.
    
- **You: 코랄 npu 프로젝트를 활용해서 멀티...**
    
    코랄 npu 프로젝트를 활용해서 멀티 matmul연산기로 구성된
    클러스터도 구현 가능? 성능을 보장하면서
    
- **ChatGPT: 가능합니다. **다만 “Coral N...**
    
    가능합니다. **다만 “Coral NPU RTL을 그대로 복제해서 matmul 엔진을 여러 개 달면 자동으로 성능이 보장된다”는 형태는 어렵고**, 성능을 “보장”하려면 **(1) 메모리/인터커넥트, (2) 컴파일러 스케줄링, (3) 런타임 큐/QoS**까지 **클러스터 전용으로 재설계**가 필요합니다.
    
    아래는 **Coral NPU 프로젝트를 기반으로 멀티 MatMul(=Matrix) 연산기 클러스터를 구현**하는 것이 기술적으로 어떤 의미인지, 그리고 **성능 보장을 위해 반드시 해야 하는 것**을 “실행 가능한 방법론”으로 정리한 것입니다.
    
    ## 1) “가능”의 범위: Coral이 제공하는 출발점
    
    Coral NPU는 오픈소스 IP로, 기본적으로 **Scalar core + RISC-V Vector + Matrix(quantized) engine**으로 구성된 ML 추론 가속기입니다.   1 turn0search3 turn0search6 turn0search15또한 Google의 가이드/튜토리얼은 Coral NPU를 대상으로 **커널 작성(예: 벡터 add), 빌드, RISC-V 어셈블리 확인** 같은 “프로그래머블” 개발 플로우를 제공합니다.   4
    
    즉, **단일 코어/단일 매트릭스 엔진을 전제로 한 풀스택**은 이미 갖춰져 있고, 여기서 **스케일아웃(클러스터화)**를 추가하는 구조로 접근하면 됩니다.
    
    ## 2) 멀티 MatMul 클러스터가 “성능 보장”을 깨뜨리는 3가지 병목
    
    멀티 matmul을 붙였는데 성능이 안 나오는 이유는 거의 아래 3개로 귀결됩니다.
    
    ### (A) 메모리 대역폭/재사용이 matmul 수를 못 따라감 (Roofline 병목)
    
    • matmul 엔진을 N배로 늘리면 
    
    • 하지만 SRAM 뱅크/포트/NoC/DMA가 그대로면 
    
    ✅ 성능 보장 조건(필수):
    
    • 타일 SRAM banking + 포트 설계 + DMA 다중 outstanding
    
    • cluster당 
    
    ### (B) 단일 Scalar/단일 명령 스트림이 클러스터 프론트엔드 병목이 됨
    
    Coral은 “Scalar가 데이터플로우/제어를 관리”하는 철학이 강합니다.   15 turn0search1클러스터로 확장할 때 Scalar가 단일이면:
    
    • issue/dispatch가 bottleneck
    
    • 동기화/이벤트 처리로 scalar가 바빠짐
    
    ✅ 성능 보장 조건:
    
    • (권장) 
    
    • 최소한 
    
    ### (C) 컴파일러가 “멀티 엔진 매핑”을 못 하면 실성능이 안 나옴
    
    멀티 matmul은 결국
    
    • 분할(tile/partition)
    
    • 배치(placement)
    
    • 스케줄(time)
    
    이 3가지를 컴파일러가 “정적으로” 잘해줘야 성능이 나옵니다.
    
    ✅ 성능 보장 조건:
    
    • MLIR 단계에서 
    
    • matmul을 
    
    ## 3) Coral 기반 “멀티 MatMul 클러스터” 구현 로드맵 (간략/실행형)
    
    ### Step 1) 목표 워크로드를 먼저 고정 (성능 보장의 기준점)
    
    • LLM prefill용 GEMM? (대형 K, 높은 reuse)
    
    • decode용 작은 GEMM? (메모리 지배)
    
    • CV용 conv→gemm?
    
    여기서부터 **tile shape**가 결정되고, SRAM/NoC 요구가 결정됩니다.
    
    ### Step 2) RTL: Matrix engine 복제보다 “클러스터 패브릭”부터 설계
    
    • [각 matmul 엔진] + [로컬 SRAM slice]를 **타일(tile)**로 정의
    
    • 타일들을 연결하는 
    
    • DMA 엔진을 
    
    ### Step 3) 메모리 정책: SRAM을 “크기”가 아니라 “동시성”으로 설계
    
    • banking, interleave, multi-ported(또는 복제) 전략
    
    • double buffering 기본
    
    • read/write outstanding 파라미터를 클러스터 기준으로 튜닝
    
    (이 부분은 Coral 이슈에서도 AXI outstanding 튜닝 문제가 언급됩니다.   19 )
    
    ### Step 4) 컴파일러: MLIR에 Cluster mapping 계층을 추가
    
    • StableHLO/ONNX → (기존) Tensor-level 최적화
    
    • Tile IR
    
    • Cluster IR
    
    • Codegen: command packet 생성
    
    ### Step 5) 런타임: 단일 실행 → multi-queue + 이벤트 모델
    
    • Prefill queue (throughput)
    
    • Decode queue (latency priority)
    
    • 모델/테넌트 분리(최소한 context 분리)
    
    ## 4) “성능 보장”을 현실적으로 정의하는 방법
    
    하드웨어에서 “보장”은 보통 이렇게 정의합니다.
    
    1. 피크 성능 보장
    
    1. 대표 워크로드에서
    
    1. **상한 대비 X% 이상(예: 70~85%)**을 달성
    
    이를 위해 권장하는 3단계 검증:
    
    • (1) Roofline 모델
    
    • (2) Cycle/이벤트 시뮬
    
    • (3) End-to-end compile+run
    
    ## 5) 결론
    
    • Coral NPU 프로젝트를 활용해 멀티 matmul 연산기 클러스터 구현은 가능
    
    • 하지만 성능을 “보장”하려면 
    
    • 메모리 계층/대역폭(은행/포트/DMA)
    
    • 클러스터 인터커넥트(NoC)
    
    • 컴파일러의 tile+cluster mapping
    
    • 런타임 큐/이벤트
    
    를 같이 올려야 합니다.
    
    원하시면, 창훈님이 상정하는 클러스터(예: **TE 4개/8개**, SRAM 크기, 목표 클럭, 타겟 모델이 LLM prefill 중심인지)를 제가 “가정값”으로 잡고, **(1) 요구 BW/ SRAM banking 수, (2) 추천 타일 크기, (3) MLIR에 넣을 Cluster IR 필드, (4) simulator 파라미터 세트**까지 한 장짜리 스펙으로 바로 내려드릴게요.