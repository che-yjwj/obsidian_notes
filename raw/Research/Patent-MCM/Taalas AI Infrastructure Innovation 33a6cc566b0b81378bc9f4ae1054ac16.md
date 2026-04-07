# Taalas AI 인프라 혁신

ChatTime: 2026년 2월 23일 12:57 (GMT+9)
Created time: 2026년 4월 6일 오후 11:00
ProjectName: [Patent-Paper] MCM
URL: https://chatgpt.com/c/699a594d-473c-83ab-bd3b-7a039ce4438e

- **You: https://www.bricksum...**
    
    [https://www.bricksum.com/taalas-the-path-to-ubiquitous-ai/](https://www.bricksum.com/taalas-the-path-to-ubiquitous-ai/)
    
- **ChatGPT: 요약 및 기술적 해석: **Taala...**
    
    요약 및 기술적 해석: **Taalas: AI 모델을 실리콘에 “하드와이어(Hard-wire)”하는 전략과 AI 인프라 패러다임 전환**
    
    아래는 이번에 공개된 *“The Path to Ubiquitous AI”* 의 주요 내용과 기술적 의의를 논리·체계적으로 정리한 것입니다. cite turn0view0 turn0search0
    
    ## 1. 문제 정의: 현재 AI 인프라의 구조적 한계
    
    현대 AI 추론(특히 LLM)의 확산은 **범용 GPU 기반 데이터센터 인프라**를 전제로 진행돼 왔습니다.그러나 이 구조는 본질적으로 다음과 같은 한계가 있습니다:
    
    • 메모리 병목
    
    • 비용·전력 효율
    
    • 지연(latency)
    
    이러한 문제는 “AI 모델 규모가 커질수록 인프라가 비례해서 확장돼야 한다”는 구조적 불균형을 야기합니다.
    
    ## 2. Taalas의 핵심 주장: **모델이 곧 컴퓨터**
    
    Taalas는 기존의 *모델을 소프트웨어로 실행하는 방식*에서 벗어나, **AI 모델 자체를 실리콘으로 구현한다는 개념적 전환**을 내세웁니다.  0 turn0search3
    
    ### 주요 철학
    
    1. Complete Specialization (완전 특수화)
    
    • 개별 AI 모델을 대상으로 최적화된 ASIC을 설계하여,
    
    • 범용 하드웨어 대비 
    
    1. Merge Compute and Storage (연산·저장 통합)
    
    • 전통적 시스템에서 분리된 계산 유닛과 메모리 계층을 
    
    • DRAM 수준의 밀도를 가진 저장소와 연산을 실리콘 내부에서 공존시키는 구조.  0
    
    1. Radical Simplification (구조 단순화)
    
    • HBM, 고속 I/O, 복잡한 패키지 기술, 냉각 시스템과 같은 
    
    • 설계/제조 파이프라인을 단순화하여 전체 TCO 절감.  0
    
    ## 3. 기술 구조와 구현 전략
    
    ### ■ “Hardcore Models”
    
    Taalas는 특정 AI 모델(예: Llama 3.1 8B) 전체를 **ASIC에 하드와이어**한 프로세서를 설계·제조했습니다.  0
    
    • 이 칩은 
    
    • 소프트웨어가 아닌 “실리콘 위에서 직접 실행”되는 방식으로 설계됨.  0
    
    ### ■ DRAM 레벨 메모리·연산 융합
    
    Taalas는 외부 DRAM이나 HBM 대신:
    
    • 칩 내부에 저장과 연산이 공존하는 구조
    
    • 이로 인해 메모리 이동 비용과 지연이 획기적으로 줄어듦.  6
    
    즉, 메모리-연산 간 **통합 메모리 아키텍처**가 시스템 병목을 줄여 inference 효율을 극대화합니다.
    
    ## 4. 성능·효율 지표
    
    Taalas가 공개한 벤치마크에 따르면:
    
    • 약 17,000 tokens/sec 처리 성능
    
    • 기존 GPU 대비 ≈10× 빠른 inference
    
    • 전력/제조 비용 ≈10×/20× 절감 규모
    
    이는 **HC1** 보드 기준이며, PCIe 카드 형태로도 제공됩니다.
    
    ## 5. 제조 및 생태계 전략
    
    ### ■ “Foundry-Optimized Flow”
    
    Taalas는 TSMC 파운드리와 협력하여:
    
    • 모델 → 실리콘 변환  → 제조
    
    이는 일반 custom ASIC 설계 주기(수개월–수년)를 훨씬 뛰어넘는 전략입니다.
    
    ## 6. 기술적 Trade-offs
    
    ### ✔ 장점
    
    • 극단적 전력/성능 효율
    
    • 메모리 병목 최소화
    
    • 데이터센터 규모 확대 없이 로컬/엣지 AI 가능성
    
    ### ⚠ 단점
    
    • 유연성 제한
    
    • 스케일업 한계
    
    ## 7. AI 인프라 패러다임 전환의 의미
    
    **핵심 질문**:
    
    > 
    
    “AI 인프라가 범용 하드웨어 위주에서 벗어나, 특정 모델용 특수화 ASIC 기반으로 전환될 수 있는가?”
    
    Taalas는 이를 위해:
    
    • Inference workload를 컴퓨팅 아키텍처의 중심
    
    • 연산 효율·지연·TCO를 혁신적 수준으로 개선
    
    • Model-centric hardware design 철학을 제안. cite turn0view0
    
    이 접근은 *모델이 하드웨어를 주도하는 AI 시스템*이라는 개념적 전환을 의미합니다.
    
    ## 8. 연구/산업적 함의 (창훈 님 관점에서)
    
    ### ■ 시스템 아키텍처 관점
    
    • 모델-실리콘 매핑 파이프라인(ONNX/IR → HDL/RTL 변환)의 자동화는 필수적.
    
    • Memory-Compute unified architecture 설계가 핵심 병목 제거 전략임.
    
    ### ■ NPU/컴파일러 설계
    
    • Taalas 수준의 특수화는 
    
    • 모델 가중치/연산을 실리콘 레벨에서 최적화하는 HW/SW Co-design 과제.
    
    ### ■ 연구 주제 제안
    
    • 자동화된 
    
    • Reconfigurable hardwired architectures (부분 가중치 업데이트 지원)
    
    • 통합 메모리/로직 최적화 및 
    
    ## 결론
    
    Taalas의 비전은 “AI가 범용 컴퓨팅에서 벗어나 모델 중심의 특수화 컴퓨팅으로 진화한다”는 **구조적 전환**을 제시합니다. 모델을 silicon에 embedding 함으로써 inference의 효율성과 비용 구조를 급진적으로 개선할 수 있다는 주장은, AI 시스템 아키텍처의 다음 세대를 예고합니다.  0
    
    원문을 참고하는 것이 기술적 설계나 사업적 기회 파악에 도움이 되면, 추가 분석/도식화, 수식 기반 성능 모델링, 그리고 비교 아키텍처 분석을 이어서 도와드릴 수 있습니다.
    
- **You: Mcm과 연결**
    
    Mcm과 연결
    
- **ChatGPT: 좋습니다. **Taalas의 “모델-... (Part 1/2)**
    
    좋습니다.**Taalas의 “모델-하드와이어 ASIC” 전략을 MCM(Multiplierless / Multiple Constant Multiplication 기반 설계)과 연결**해서 구조적으로 정리해보겠습니다.
    
    당신이 추구하는 방향(모델→IR→ISA→RTL 자동화, 양자화/압축, 메모리-연산 통합)과 직접적으로 연결되는 지점이 있습니다.
    
    # 1️⃣ Taalas = “Model → Silicon Direct Mapping”
    
    Taalas 접근을 한 줄로 정리하면:
    
    > 
    
    **Trained Model Graph를 거의 변형 없이 실리콘에 고정**
    
    즉,
    
    ```
    PyTorch Model
       ↓
    Weight Fixed
       ↓
    ASIC Netlist (hardwired)
    ```
    
    여기서 중요한 점:
    
    • Weight는 고정
    
    • Layer topology도 고정
    
    • Runtime programmability 최소화
    
    • Memory hierarchy 단순화
    
    이건 “범용 NPU”가 아니라 **모델 특화 데이터플로우 머신**입니다.
    
    # 2️⃣ 여기서 MCM이 왜 핵심인가?
    
    Transformer/LLM에서 대부분의 연산은:
    
    ```
    Y = W × X
    ```
    
    Weight W는 inference에서 상수입니다.
    
    즉,
    
    > 
    
    Weight는 constant→ Multiplication by constant→ MCM (Multiple Constant Multiplication) 문제
    
    # 3️⃣ MCM = Hardwired Weight Architecture의 수학적 기반
    
    MCM은 다음 문제입니다:
    
    ```
    여러 개의 상수 {c1, c2, ..., cn}에 대해
    곱셈기를 최소화하고
    Shift + Add 네트워크로 구현
    ```
    
    LLM Linear layer:
    
    ```
    Y_i = Σ (W_ij × X_j)
    ```
    
    여기서 W_ij는 constant.
    
    즉:
    
    • 곱셈기 필요 없음
    
    • Shift + Add DAG로 변환 가능
    
    • Booth multiplier 제거 가능
    
    • DSP block 제거 가능
    
    이게 바로:
    
    > 
    
    MCM = Hardwired Model Acceleration의 핵심 수학
    
    # 4️⃣ Taalas vs GPU vs MCM ASIC
    
    | 구조 | Weight 처리 | Multiply | 메모리 |
    | --- | --- | --- | --- |
    | GPU | DRAM 로드 | 범용 MAC | HBM |
    | 일반 NPU | SRAM 로드 | MAC array | SRAM |
    | Taalas | 실리콘 고정 | 거의 없음 | On-chip |
    | MCM ASIC | 실리콘 고정 | Shift/Add | On-chip |
    
    Taalas가 말하는 “model as hardware”는
    
    > 
    
    MCM을 극단적으로 확장한 개념
    
    이라고 볼 수 있습니다.
    
    # 5️⃣ MCM이 Taalas를 더 발전시킬 수 있는 이유
    
    Taalas는 현재:
    
    • 모델을 하드와이어
    
    • 메모리-연산 통합
    
    하지만 여전히 일부는 multiplier 기반일 가능성이 큽니다.
    
    MCM을 적용하면:
    
    ### ✅ 1. DSP 제거
    
    • multiplier → adder tree
    
    • 전력 감소
    
    • area 감소
    
    ### ✅ 2. Weight 공유 최적화
    
    • 동일 상수 coefficient reuse
    
    • DAG merging
    
    ### ✅ 3. Quantized weight와 결합 가능
    
    • INT4/INT2 → MCM cost 감소
    
    • Bit-sliced 구조와 잘 맞음
    
    # 6️⃣ MCM + Model Hardwiring = "Computational ROM"
    
    극단적 형태:
    
    ```
    Input → Shift/Add Graph → Output
    ```
    
    이건 거의:
    
    > 
    
    논리 합성된 계산용 ROM
    
    과 유사합니다.
    
    특히:
    
    • TinyLLaMA
    
    • Edge LLM
    
    • Vision Transformer
    
    • ISP CNN
    
    이런 구조에 매우 적합합니다.
    
    # 7️⃣ 당신의 연구와 직접 연결
    
    당신이 지금까지 진행한:
    
    • Multiplierless CNN
    
    • ChaosActivation
    
    • Quantization
    
    • xNPU ISA
    
    • ONNX → IR 자동화
    
    이걸 연결하면:
    
    ### 🔷 Phase 1
    
    ONNX → Constant Folding → MCM Graph Extraction
    
    ### 🔷 Phase 2
    
    MCM DAG Optimization
    
    • common subexpression elimination
    
    • adder depth minimization
    
    • pipeline balancing
    
    ### 🔷 Phase 3
    
    RTL Auto-generation
    
    • Verilog emitter
    
    • Floorplan-aware clustering
    
    • SRAM co-location
    
    # 8️⃣ MCM + Memory-Compute Fusion
    
    Taalas는 compute와 storage를 통합합니다.
    
    MCM을 결합하면:
    
    • Weight 저장 필요 없음
    
    • coefficient가 wiring으로 존재
    
    • 메모리 bandwidth = 0 (for weights)
    
    • input activation만 필요
    
    LLM decode 단계에서 특히 강력:
    
    > 
    
    KV cache만 외부 메모리Weight는 칩에 고정
    
- **ChatGPT: 좋습니다. **Taalas의 “모델-... (Part 2/2)**
    
    # 9️⃣ LLM Prefill vs Decode 관점
    
    ### Prefill
    
    • 대량 matmul
    
    • Hardwired MCM 매우 유리
    
    ### Decode
    
    • 작은 batch
    
    • 메모리 latency 민감
    
    • Shift-add 구조는 매우 전력 효율적
    
    Groq LPU, HyperAccel LPU와도 연결 가능.
    
    # 🔟 한 단계 더 나가면?
    
    Taalas + MCM + Partial Reconfiguration
    
    문제:
    
    • 모델 바뀌면 재제작?
    
    해결 아이디어:
    
    > 
    
    Coefficient Programmable MCM Fabric
    
    • Shift network는 고정
    
    • sign/shift mask만 변경
    
    • LUT 기반 coefficient bank
    
    이건 FPGA와 ASIC 중간 영역.
    
    # 11️⃣ 연구 논문 프레임 제안
    
    제목 예시:
    
    > 
    
    “Hardwired Transformer via MCM-Based Multiplierless Architecture”
    
    구성:
    
    1. Taalas paradigm 분석
    
    1. LLM Linear layer MCM 변환 수학
    
    1. Adder graph 최적화
    
    1. Energy/Area 모델링
    
    1. GPU/NPU 비교
    
    1. Silicon synthesis 결과
    
    # 12️⃣ 전략적 결론
    
    Taalas는:
    
    > 
    
    모델 특화 ASIC
    
    MCM은:
    
    > 
    
    그 특화를 극단적으로 밀어붙이는 기술
    
    즉,
    
    > 
    
    MCM은 Taalas를 이론적으로 완성시키는 수단
    
    원하시면 다음 단계로:
    
    • LLaMA Linear layer를 실제 MCM DAG로 변환
    
    • 토큰당 에너지 모델 수식화
    
    • NPU vs MCM ASIC 비교 시뮬레이션 모델 설계
    
    • “MCM-based Hardcore Model” 아키텍처 문서 초안 작성
    
    어느 방향으로 깊게 들어갈까요?
    
- **You: https://news.hada.io...**
    
    [https://news.hada.io/topic?id=26896](https://news.hada.io/topic?id=26896)
    
- **ChatGPT: 요청하신 ** link_title T...**
    
    요청하신 ** link_title Taalas 칩 뉴스** turn0search0  내용을 근거로, 기술적 근거와 산업적 맥락을 바탕으로 **핵심 포인트를 정리**합니다.
    
    ## 📌 Taalas 칩 핵심 내용 요약
    
    ### 1️⃣ 모델을 실리콘에 직접 “하드와이어링”
    
    • **  0 **
    
    • 실리콘 위에 모델의 32개 계층이 순차적인 데이터 패스를 
    
    • 결과적으로 
    
    ### 2️⃣ 메모리 구조의 재정의
    
    • **  0 **
    
    • 이 구조는 traditional AI 인프라에서 가장 큰 장애 요인이던 대용량 메모리 계층과 통신 비용을 근본적으로 줄입니다.
    
    ### 3️⃣ 실리콘 제작 속도와 방법
    
    • **  0 **
    
    • 모델 특화 ASIC의 상용화에 대한 
    
    ### 4️⃣ 성능·비용/전력 효율
    
    • **  2 **
    
    ### 5️⃣ 한계점
    
    • 현재 구현은 
    
    ## 📍 기술적 해석 — 핵심 혁신 요소
    
    ### ✔ 모델을 실리콘 구조로 합성
    
    기존 아키텍처:
    
    ```
    소프트웨어 실행 (GPU/NPU)
        • 가중치 로딩
        • MAC 연산
        • 외부 메모리 왕복
    ```
    
    Taalas 구조:
    
    ```
    AI 모델 그래프 —> 실리콘 하드웨어 회로
        • 가중치 = 트랜지스터 + 연결
        • 연산 = 물리적 신호 전달
        • 계층 간 인터커넥션은 배선 파이프라인
    ```
    
    **특징**
    
    • 가중치가 SRAM이나 DRAM이 아닌 
    
    • 계층 간 중간 결과가 RAM에 저장되지 않음 → 
    
    ## 📍 왜 이것이 중요한가?
    
    ### ① 메모리 병목 완전 해소
    
    AI 인퍼런스에서 가장 병목을 만드는 요소는 **대용량 행렬 가중치의 메모리 이동 비용** 입니다.
    
    • CPU/GPU/NPU 전통 구조에서는 DRAM/HBM → 연산 → 쓰기/읽기 과정이 반복됩니다.
    
    • Taalas는 가중치가 
    
    ### ② 추론 지연(latency) 극소화
    
    • Taalas 구조는 각 층(계층)의 연산이 
    
    • 이는 웹/로보틱스/엣지 응용에서 요구되는 
    
    ### ③ 메모리-연산 통합
    
    • 기존 NPU/TPU 등은 메모리 대역폭과 연산 유닛 간 최적화가 주요 난제지만,
    
    • Taalas는 가중치 저장과 연산을 
    
    ## 📍 산업적 의미
    
    ### ✔ ASIC 중심의 신 AI 컴퓨팅 카테고리 탄생
    
    범용 GPU/AI 가속기:
    
    • 하나의 HW로 여러 모델 실행
    
    • 높은 유연성, 낮은 특수화 효율
    
    Taalas식 특화 ASIC:
    
    • 특정 모델 체인을 
    
    • 극단적 성능/전력 효율 획득
    
    → 이 둘은 **완전히 다른 시스템 클래스** 입니다.
    
    ### ✔ 제품 수명주기 관점
    
    Taalas의 접근은 칩을 신속히 재구성하는 워크플로우를 가정함으로써:
    
    • AI 모델 변경 → 칩 마스크 변경 → 2개월 생산 → 재배포
    
    이 순환은 산업 표준 디자인플로우와 비교하면 **빠르지만 여전히 ASIC 계열의 제한성**이 존재합니다.
    
    ## 📍 기술적 이슈 — 부가 분석
    
    ### ❗ Quantization/Approximation 영향
    
    Taalas가 일부 **3~6비트 양자화**를 활용했다는 보도도 있습니다.
    
    • 이는 모델 정확도/성능 트레이드-오프가 존재함을 시사합니다.
    
    • 다만 
    
    ### ❗ 모델 변경 비용/속도
    
    • 완전 하드와이어 ASIC은 
    
    • 이것은 
    
    ## 📌 종합 결론
    
    Taalas 칩 뉴스는 기술·산업적으로 다음과 같은 의미를 갖습니다:
    
    1. AI 모델을 실리콘으로 물리적 합성
    
    1. 메모리 병목 완전 제거
    
    1. 특정 모델 특화 ASIC
    
    1. 제작 주기 최적화
    
    필요하다면 **Taalas 구조의 정확한 RTL/데이터플로우 분석**, **비용·전력 모델링**, **프로그래머블 NPU 대비 성능/유연성 트레이드-오프**를 더욱 정량적으로 정리해드릴 수 있습니다.