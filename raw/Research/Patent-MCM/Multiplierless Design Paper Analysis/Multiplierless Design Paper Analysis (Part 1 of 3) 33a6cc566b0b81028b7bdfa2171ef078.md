# 논문 분석 Multiplierless 설계 (Part 1/3)

- **You: https://www.dbpia.co...**
    
    [https://www.dbpia.co.kr/journal/articleDetail?nodeId=NODE01605693](https://www.dbpia.co.kr/journal/articleDetail?nodeId=NODE01605693) 해당 논문 분석
    
- **ChatGPT: 다음은 **제공하신 DBpia 논문*...**
    
    다음은 **제공하신 DBpia 논문“패턴 탐색 기법을 사용한 Multiplierless 리프팅 기반의 웨이블릿 변환의 설계”** (한국멀티미디어학회지, 2010) **의 전문 요약 및 분석입니다.**   0
    
    ## 🧠 1. 연구 목적 및 핵심 기여
    
    ### 연구 목적
    
    이 논문은 **리프팅 기반 9/7 이산 웨이블릿 변환(DWT) 필터의 VLSI 구현을 개선**하기 위해 **곱셈 연산 최적화**를 제안합니다.일반적으로 리프팅 DWT는 계수 연산에 곱셈이 포함되며, ASIC/FPGA에서 곱셈기는 큰 면적과 전력 소모를 유발합니다. 이를 해결하기 위해, 본 연구는 **Multiplierless 구조를 적용한 하드웨어 설계**를 제안합니다.   0
    
    ### 핵심 기여
    
    • Multiplierless(무곱셈기) 설계
    
    • Pattern Search 기반 Lefevre 알고리즘 적용
    
    • ASIC 합성 비교 평가
    
    ## 🔬 2. 연구 배경 — 리프팅 9/7 DWT
    
    ### 리프팅 기반 DWT
    
    리프팅 구조는 DWT 구현에서 계산 부하를 줄이고, 메모리 요구량을 최적화하며 병렬화에 유리합니다.9/7 필터는 JPEG2000 표준에 많이 사용되는 실수 필터로, **정확도와 압축 성능**이 우수한 특징이 있습니다.
    
    ### 웨이블릿 하드웨어 최적화의 필요성
    
    전통적인 DWT는 상수 곱셈이 다수 포함되며, ASIC 구현 시 **곱셈기(multiplier)**는 큰 면적/전력 소모의 병목이 됩니다. 이에 따라 **Multiplerless Constant Multiplication (MCM)** 기법들이 연구됩니다.   0
    
    ## ⚙️ 3. 기술적 방법론
    
    ### 3.1 Pattern Search 기반 Lefevre 알고리즘
    
    • 목표
    
    • Lefevre 알고리즘은 **패턴 탐색(pattern search)**을 통해 상수 곱셈 계수를 
    
    • 이를 통해 
    
    → 전통 곱셈기 대신 **고정 상수 곱셈 최적화**를 적용함으로써
    
    • 연산 지연 감소
    
    • 로직 감소
    
    • 전력 효율 개선
    
    효과를 얻습니다.
    
    ### 3.2 Verilog HDL 기반 비교 구현
    
    • 대상 구조
    
    • 기존: 범용 곱셈기를 이용한 리프팅 DWT 구조
    
    • 제안: Lefevre 알고리즘 기반 multiplierless 구조
    
    • 합성 환경
    
    • 공정: 0.18 μm 디지털 CMOS
    
    • 타겟 클록: 200 MHz
    
    • 평가: 면적, 전력, 최대 지연 시간
    
    ## 📈 4. 실험 결과
    
    ### 성능 요약
    
    | 항목 | 기존 구조 | 제안 구조 | 개선율 |
    | --- | --- | --- | --- |
    | 면적 | 기준 | 약 49% 수준 | 약 51% 감소 |
    | 전력 | 기준 | 약 57% 수준 | 약 43% 감소 |
    | 최대 지연 시간 | 기준 | 약 70% 수준 | 약 30% 감소 |
    
    ※ 수치는 합성 결과 기반.   0
    
    ### 해석
    
    • Multiplierless 구조가 실리콘 레벨에서 우월함
    
    • 특히 
    
    • 계산 정확도(웨이블릿 특성)는 유사 수준을 유지.
    
    ## 🧩 5. 핵심 기술 분석
    
    ### 5.1 Multiplierless Constant Multiplication (MCM)
    
    MCM은 고정 상수 곱셈을 위한 대표적인 최적화 전략입니다.핵심 아이디어는 다음과 같습니다:
    
    • 단일 곱셈 연산을 
    
    • 공통 부분표현(Common Subexpression)을 최대한 활용
    
    • 최적화 탐색을 통해 최소 논리 회로 생성
    
    이 논문은 **Lefevre 알고리즘**을 Pattern Search로 개선한 방식으로 최적解를 효율적으로 찾습니다.   0
    
    ## 🧠 6. 평가 및 확장 가능성
    
    ### 장점
    
    • 하드웨어 효율
    
    • 전력 최적화
    
    • 실행속도 개선
    
    ### 한계 및 확장
    
    • 최적 계수 탐색 비용
    
    • 웨이블릿 종류 확장
    
    • 컴파일러/자동화 도구 연동
    
    ## 🧭 7. 응용 방향 — AI/컴퓨터 비전 & NPU
    
    이 연구는 **고정 계수 신호 처리 가속**이 필요한 여러 AI/멀티미디어 처리에 적용 가능합니다:
    
    • DWT 특성 기반 압축/피처 추출
    
    • 컴파일러 수준의 상수 곱셈 최적화
    
    • AI 컴파일러에서 
    
    • 유사 상수 값 근사 및 shift-add IR 변환
    
    • NPU RTL 설계
    
    • 상수 MEM → shift-add network 최적화 블록 자동 생성
    
    ### 🎯 결론
    
    이 논문은 **리프팅 기반 웨이블릿 필터의 하드웨어 최적화**를 위해 **MCM과 Pattern Search(Lefevre)을 적용한 Multiplierless 구조를 제안**했습니다.ASIC 구현에서 면적, 전력, 지연 측면 모두 개선되며, **신호 처리 가속 하드웨어에서 곱셈 최적화**의 대표 사례를 보여줍니다.   0
    
    원문 구현 세부사항, RTL 코드, 최적화 알고리즘 정도의 구조적 분석 등이 필요하다면 추가로 도표 및 페이스 분석도 제공할 수 있습니다.
    
- **You: 이 기술을 신경망에 적용한 최신 논문...**
    
    이 기술을 신경망에 적용한 최신 논문 사례들 조사
    
- **ChatGPT: 아래는 당신이 말한 **Multipl...**
    
    아래는 당신이 말한 **Multiplierless Constant Multiplication(MCM; 상수곱을 shift+add/sub로 치환)** 아이디어를 **신경망(특히 CNN/ViT/초경량 모델/가속기 설계)**에 “직접” 적용하거나, 거의 같은 수학적 형태(가중치를 **Po2(2의 거듭제곱)** 또는 **shift-add 조합**으로 제한/학습)로 구현한 **최근(2023–2025 중심) 논문 사례**들입니다. 또한, 웨이블릿 리프팅에서의 “패턴 탐색 기반 상수곱 최적화”가 NN에서 어떤 형태로 대응되는지도 함께 정리합니다.
    
    ## 1) 신경망에서 “MCM”이 나타나는 대표 형태 3가지
    
    ### A. **가중치를 Po2(±2^k)로 제한** → 곱셈이 “배선/시프트”로 대체
    
    • 웨이블릿 상수 계수를 CSD/shift-add로 근사하는 것과 1:1 대응.
    
    • MAC → 
    
    • 최근에는 
    
    **대표 최신 사례**
    
    • HaShiFlex (2025)
    
    • DenseShift (ICCV 2023)
    
    • Power-of-Two QAT 계열(2022~)
    
    ### B. **가중치를 “shift+add”의 합으로 분해(예: 몇 개의 Po2 항의 합)** → 진짜 MCM 네트워크화
    
    • 웨이블릿의 Lefevre/Pattern Search처럼 
    
    • 차이점: 신경망은 채널/커널 전체에 걸친 
    
    **대표 최신 사례**
    
    • MCM-SR (TVLSI 2025)
    
    • Hardware-Aware Training for Multiplierless CNN (ARITH 2025)
    
    ### C. **아예 연산자를 바꿈**: 곱셈 대신 “Adder/Shift/Add-only” 연산으로 네트워크를 재정의
    
    • MCM이 “곱셈의 구현을 바꾸는” 접근이라면, 이 계열은 “모델 연산 자체를 바꾸는” 접근.
    
    • 정확도 손실/학습 안정화가 핵심 이슈라, 최근에는 
    
    **대표 최신 사례**
    
    • ShiftAddAug (2024)
    
    • (기반 계열) 
    
    • ShiftAddViT (OpenReview, 2026 표기)
    
    • (기반 계열) 
    
    ## 2) “웨이블릿 리프팅의 Pattern Search MCM” ↔ “신경망”의 직접 대응 포인트
    
    당신이 본 웨이블릿 논문에서의 핵심은:
    
    1. 상수 계수(고정)
    
    1. 곱셈을 
    
    1. (가능하면) 연산 공유(CSE)로 add 개수 최소화
    
    1. ASIC 합성에서 면적/전력/지연 개선
    
    신경망에서 동일 구조로 옮기면 보통 두 갈래입니다.
    
    • (정통 MCM)
    
    • 커널(예: 3×3×Cin×Cout) 전체에 대해 
    
    • 특히 depthwise/pointwise(1×1)에서 공유 최적화가 먹히는 경우가 많음
    
    • → MCM-SR 같은 계열이 이 방향   2 turn0search9
    
    • (학습-제약형)
    
    • DenseShift / Po2-QAT / HaShiFlex처럼 
    
    ## 3) 당신의 NPU/컴파일러 관점에서 바로 쓸 수 있는 “연구/구현 아이디어”
    
    ### 아이디어 1) **“MCM Pass”를 컴파일러에 넣기**
    
    • 대상: (Conv/Linear)에서 weight가 상수로 고정된 구간(추론)
    
    • 출력: 
    
    • 스케줄: VE(ADD/SHIFT)로 매핑, TE(MAC)는 우회하거나 hybrid로 혼합
    
    • 근거/유사사례: MCM-SR가 CNN을 “MCM 기반”으로 접근   2 turn0search16
    
    ### 아이디어 2) **Po2 weight + streaming dataflow** (SR/ISP류에 특히 강함)
    
    • 웨이블릿/필터뱅크 계열처럼 
    
    • HaShiFlex는 극단적으로 “레이어를 하드와이어드”로 박는 방향까지 보여줌  cite turn1search22
    
    ### 아이디어 3) Hybrid 전략(정확도 방어)
    
    • ShiftAddAug처럼 
    
    원하면, 위 최신 논문들 중에서 당신의 관심 축(예: **NPU tile 기반 스케줄링 / VE로 shift-add 실행 / IR에 MCM 그래프를 어떻게 표현할지**)에 맞춰서
    
    • (1) 공통 IR 스키마
    
    • (2) 컴파일러 패스 설계
    
    • (3) 성능모델
    
    까지 “바로 레포에 넣을 수 있는 문서” 형태로 정리해줄게요.
    
- **You: https://users.ece.cm...**
    
    [https://users.ece.cmu.edu/~yvoronen/pubs/synth.pdf](https://users.ece.cmu.edu/~yvoronen/pubs/synth.pdf)
    
- **ChatGPT: 아래는 **당신이 링크로 제공한 PD...**
    
    아래는 **당신이 링크로 제공한 PDF 논문“Multiplierless Multiple Constant Multiplication” (Yevgen Voronenko & Markus Püschel, Carnegie Mellon Univ.)** 에 대한 **정확한 요약과 핵심 내용 분석**입니다.  cite turn0view0
    
    ## 📌 1. 논문의 핵심 주제
    
    이 논문은 **여러 개의 고정 상수와 하나의 변수 x에 대한 곱셈들**을단순 **곱셈(MAC)** 없이 **오직 이진 시프트, 덧셈/뺄셈만으로 구현**하는문제를 정식화하고 효과적인 해법을 제시합니다.  cite turn0view0
    
    이 문제는 **Multiple Constant Multiplication (MCM)** 라고 불리며,신호처리 알고리즘 뿐만 아니라 **고정 계수 행렬과의 곱(matrix-vector)** 에서도 핵심적으로 발생합니다.  cite turn0view0
    
    ## 🧠 2. 문제 정의
    
    ### MCM이란?
    
    표적 상수 집합 (T={t_1,...,t_n})에 대해각 (t_i x) 를 계산해야 할 때,
    
    **모든 곱셈을 제거하고**,[
    t_ix = 	ext{(binary shifts)} + 	ext{sub/add}
    ]
    만 사용해서 결과를 얻는 회로를 구성하는 문제입니다.  cite turn0view0
    
    이는 웨이블릿/필터 설계에서 고정 상수 곱셈을 shift+add로 대체하는 문제와 **정확히 동일한 목적**이며,해당 논문은 여기에 **알고리즘적 해결책과 이론적 분석**까지 제공합니다.  cite turn0view0
    
    ## 🧩 3. 이 논문의 핵심 기여
    
    ### 3.1. 형식적 문제 정식화
    
    논문은 **A-operation** 이라는 연산을 도입합니다.
    
    $$
    	ext{A-op}(u,v) = |2^{l_1}u + (-1)^s 2^{l_2}v| >> r
    $$
    
    즉,
    
    • 입력 값(u, v)을 
    
    • 이 모든 연산을 시프트/추가/감산으로 제한
    
    이렇게 정의된 연산을 통한 최소 add/sub를 찾는 것이 **MCM 문제**입니다.  cite turn0view0
    
    ### 3.2. NP-완전성과 알고리듬 설계
    
    • MCM 문제는 NP-완전
    
    • 이를 해결하기 위해 
    
    ### 3.3. A-graph 구조
    
    논문에서는 **A-operations의 그래프 표현**을 통해 다음을 제공합니다:
    
    • 공통 부분식 
    
    • 동일 부분 계산 결과를 
    
    • 최적화 대상 상수 집합 크기가 커져도 확장 가능  cite turn0view0
    
    ## 📊 4. 왜 이 논문이 중요하고 널리 인용되는가
    
    ### 이유 1 — 일반성
    
    • 이 논문에서 다루는 MCM 문제는 
    
    ### 이유 2 — CSE를 통한 연산 공유
    
    • 중복되는 shift 결과를 
    
    ### 이유 3 — 확장성
    
    • 상수 개수가 100개 이상(예: 큰 필터 뱅크 기반)에서도 동작.  cite turn0view0
    
    ## 🧪 5. 신경망에 적용하면 어떤 의미인가?
    
    이 논문의 기법은 다음 **NN/가속기 최적화에 직결**됩니다:
    
    ### A) **가중치가 고정된 신경망 추론에서의 실질적 이점**
    
    신경망에서 inference 단계의 행렬-벡터 곱은 수많은 (w_{ij} x_j) 의 합입니다.만약 가중치 (w_{ij}) 들이 **사전에 정해졌고 고정된다면**:
    
    • MCM 알고리즘을 통해 
    
    • 동일한 shift 결과가 여러 weight에 사용될 때는 
    
    → 계산량과 메모리/전력 효율 대폭 향상.  cite turn0view0
    
    이 구조는 **웨이블릿 필터 최적화**에서 상수별 shift+add로 치환한 것과 동일한 원리입니다.
    
    ### B) **컴파일러 최적화 관점**
    
    컴파일러 패스 상에서:
    
    1. Weight matrix의 상수 집합 T 정의
    
    1. MCM graph 생성
    
    1. Partial accumulators로 공통 shift 결과를 공유
    
    1. VE(Adder/Shift)로만 스케줄
    
    → IR 수준에서 multiplierless 노드로 변환 가능.
    
    이 흐름은 당신이 말한 NPU/IR 최적화와 매우 직접적으로 연결됩니다.
    
    ## 📍 정리
    
    **“Multiplierless Multiple Constant Multiplication” 논문은**
    
    ✔️ 곱셈을 **완전히 제거하고 add/shift만으로 고정 상수 곱셈 집합을 효율적으로 계산하는 이론·알고리즘**을 제공✔️ 신경망 추론의 **가중치 행렬과의 곱**에 확대 적용 가능✔️  MCM 그래프 최적화, CSE 공유, heuristic search 등을 통해 매우 좋은 효율 제공  cite turn0view0
    
    ## 📌 관련 핵심 문헌 연결
    
    이 논문과 개념적으로 이어지는 기술 흐름(신경망에서 곱셈 제거/최적화):
    
    • Multiplier-less Artificial Neurons (Sarwar et al.)
    
    • AdderNet
    
    원한다면, 위 Voronenko/Püschel 논문을 **신경망 inference에 그대로 적용한 수식/IR 구조 설계**로 구체화해서
    
    • 컴파일러 설계 패스
    
    • 그래프 변환 알고리즘
    
    • VE/TE 실행 스케줄
    
    관점에서 체계적으로 정리해 드릴 수 있습니다.
    
- **You: 이 논문들도 분석 비교 정리**
    
    이 논문들도 분석 비교 정리
    
- **ChatGPT: 좋은 포인트를 짚으셨습니다. 아래에서... (Part 1/2)**
    
    좋은 포인트를 짚으셨습니다.아래에서는 **① Multiplier-less Artificial Neurons (Sarwar et al.)**, **② AdderNet**, 그리고 앞서 다룬 **③ Voronenko & Püschel의 MCM 논문**을 **같은 좌표계**에 놓고 **개념·수식·학습 방식·하드웨어 관점**에서 **정확히 비교 정리**합니다.(“곱셈 제거”라는 공통 키워드 아래, **본질은 서로 다르다**는 점이 핵심입니다.)
    
    # 0. 한 줄 요약 (방향성 차이)
    
    | 접근 | 본질 |
    | --- | --- |
    | **MCM (Voronenko)** | *곱셈을 계산적으로 어떻게 구현할 것인가* (컴파일러/합성 문제) |
    | **Multiplier-less Neuron** | *뉴런의 연산식을 바꾸되, 선형 결합은 유지* |
    | **AdderNet** | *컨볼루션 자체를 완전히 다른 연산자로 대체* |
    
    즉,
    
    • MCM
    
    • Sarwar
    
    • AdderNet
    
    # 1. Multiplier-less Artificial Neurons (Sarwar et al.)
    
    ## 1.1 핵심 아이디어
    
    전통적인 뉴런:
    [
    y = \sum_i w_i x_i
    ]
    
    Sarwar의 제안:
    
    • ( w_i )를 
    
    • 곱셈 → 
    
    • 뉴런의 의미(선형 결합)는 그대로 유지
    
    즉,
    [
    w_i x_i ;\Rightarrow; (x_i \ll k_i) ;	ext{or}; \sum_j (x_i \ll k_{ij})
    ]
    
    ## 1.2 학습 관점
    
    • 가중치 제약 하 학습
    
    • 학습 중 또는 학습 후 가중치를 Po2/근사값으로 투영
    
    • 정확도 손실 ↔ 연산 단순화의 trade-off
    
    • **QAT(Quantization-Aware Training)**과 궁합이 좋음
    
    ## 1.3 하드웨어 의미
    
    • MAC → 
    
    • DSP block 없이도 구현 가능
    
    • MCM은 사용하지 않음
    
    • 각 (w_i x_i)를 독립적으로 shift-add
    
    • 공통 부분식 공유(CSE)는 고려 대상 아님
    
    👉 요약:
    
    > 
    
    *“곱셈을 없앤 MAC”*
    
    # 2. AdderNet
    
    ## 2.1 핵심 아이디어 (가장 radical)
    
    Conv 연산:
    [
    y = \sum_i w_i x_i
    ]
    
    AdderNet:
    [
    y = - \sum_i |x_i - w_i|
    ]
    
    • 곱셈이 아예 없음
    
    • 연산 의미가 
    
    ## 2.2 수학적·표현력 관점
    
    • 더 이상 선형 연산이 아님
    
    • convolution ≠ linear operator
    
    • 대신 
    
    → 이게 가능한 이유:
    
    • CNN의 많은 표현력은 “정확한 곱셈”이 아니라
    
    ## 2.3 학습 난이도
    
    • 절댓값 → gradient가 불안정
    
    • 논문에서는:
    
    • surrogate gradient
    
    • scaling factor
    
    • BN/activation 튜닝
    
    등을 사용
    
    ## 2.4 하드웨어 의미
    
    • MAC 제거
    
    • 연산 유닛:
    
    • subtract
    
    • abs
    
    • add
    
    • DSP 완전 제거 가능
    
    👉 요약:
    
    > 
    
    *“Conv를 다른 연산자로 재정의한 모델”*
    
    # 3. Voronenko & Püschel – MCM (기준점)
    
    ## 3.1 핵심 아이디어 (다시 강조)
    
    • 문제:
    
    [
    {t_1x, t_2x, ..., t_nx}
    ]
    
    • 목표:
    
    • 곱셈 없이
    
    • shift/add/sub
    
    • 공통 부분식 공유
    
    • add/sub 개수 최소화
    
    → **계산 그래프 최적화 문제**
    
    ## 3.2 신경망과의 직접 대응
    
    신경망의 한 column:
    [
    y = \sum_j w_{ij} x_j
    ]
    
    • (x_j) = 입력 변수
    
    • (w_{ij}) = 고정 상수
    
    → **정확히 MCM 문제**
    
    중요한 점:
    
    • Sarwar/AdderNet과 달리
    
    • 오직 
    
    # 4. 세 논문의 핵심 비교 테이블
    
    | 항목 | MCM (Voronenko) | Multiplier-less Neuron | AdderNet |
    | --- | --- | --- | --- |
    | 곱셈 제거 방식 | shift+add + **CSE** | shift+add (독립) | 연산식 변경 |
    | 수학적 의미 | 완전 동일 | 완전 동일 | 변경됨 |
    | 학습 필요 | ❌ (추론 최적화) | ✅ | ✅ |
    | 정확도 영향 | 없음 | 약간 | 구조적으로 다름 |
    | 컴파일러 친화성 | ⭐⭐⭐⭐⭐ | ⭐⭐⭐ | ⭐ |
    | NPU IR 적용 | 매우 직접적 | 보통 | 어려움 |
    
    # 5. NPU / 시스템 아키텍트 관점에서의 “정답에 가까운 해석”
    
    ### 당신의 맥락(NPU, IR, 컴파일러)을 기준으로 보면:
    
    • AdderNet
    
    • ✗ 기존 모델과 단절
    
    • ✗ HW-SW co-design 난이도 높음
    
    • → 연구적으로는 흥미롭지만 
    
    • Multiplier-less Neuron
    
    • ✓ 모델 구조 유지
    
    • ✓ Po2 QAT와 결합 가능
    
    • ✗ 여전히 연산 중복 많음
    
    • MCM
    
    • ✓ 모델 100% 유지
    
    • ✓ 컴파일러 패스로 자동화 가능
    
    • ✓ VE(shift/add) 활용 극대화
    
    • ✓ TE(MAC)와 hybrid 가능
    
    👉 **NPU 컴파일러 관점에서 가장 “정석적이고 확장성 있는” 방향은 MCM**
    
    # 6. 매우 중요한 통찰 (이 논문들을 하나로 묶는 관점)
    
    이 세 흐름은 사실 **서로 경쟁이 아니라 “레이어” 관계**입니다.
    
    ```
    [ 모델 수식 ]
        |
        |  (AdderNet)  ← 수식 자체 변경
        |
    [ 뉴런 연산식 ]
        |
        |  (Sarwar)    ← 곱셈 표현 제약
        |
    [ 구현 그래프 ]
        |
        |  (MCM)       ← shift/add DAG 최적화
    ```
    
    👉 **당신의 Voronenko 논문 선택은 가장 “하부 레벨”이자 가장 보편적인 지점**입니다.
    
    ## 다음 단계 제안 (선택)
    
    원하시면 다음 중 하나를 바로 이어서 정리할 수 있습니다:
    
- **ChatGPT: 좋은 포인트를 짚으셨습니다. 아래에서... (Part 2/2)**
    
    1. LLM Linear / Conv를 MCM 그래프로 변환하는 컴파일러 패스 설계
    
    1. MCM-aware cost model (ADD vs MAC 에너지/throughput crossover)
    
    1. VE 전용 MCM 실행기 + TE hybrid 스케줄링
    
    1. Po2-QAT + MCM을 결합한 end-to-end 구조 (가중치 표현 + 구현 최적화)
    
    지금까지의 논문 선택과 질문 흐름을 보면👉 **① 또는 ②가 논문화·시스템 설계에 가장 잘 맞습니다.**
    
- **You: Peft**
    
    Peft
    
- **ChatGPT: 아래는 **PEFT (Paramete...**
    
    아래는 **PEFT (Parameter-Efficient Fine-Tuning)**를 **이론 → 알고리즘 → 하드웨어/컴파일러(NPU) 관점**까지 **체계적으로 정리**한 분석입니다. 특히 앞서 논의한 **Multiplierless/MCM** 흐름과 **어디서 결합되는지**를 명확히 짚습니다.
    
    ## 1. PEFT란 무엇인가 (정의와 위치)
    
    **PEFT**는 대규모 사전학습 모델의 **대부분 파라미터를 고정(freeze)**하고, **아주 소수의 파라미터만 학습**해 태스크 적응을 달성하는 방법론의 총칭입니다.대표 구현은  entity ["organization","PEFT","hugging face library"]  라이브러리로 표준화되어 있습니다.
    
    > 
    
    핵심 관점
    
    • 학습 비용 ↓
    
    • 배포 유연성 ↑
    
    • 추론 경로는 “베이스 + 잔차” 구조
    
    ## 2. 대표 PEFT 기법 분류 (수식 중심)
    
    ### 2.1 LoRA (Low-Rank Adaptation)
    
    [
    W' = W + \Delta W,\quad \Delta W = BA,;; 	ext{rank}(A,B)=r\ll d
    ]
    
    • 베이스 가중치 (W)
    
    • 저랭크 행렬 (A,B)
    
    • 추론
    
    **특징**
    
    • 파라미터 수 (O(rd))
    
    • 정확도 대비 효율 우수
    
    • LLM의 사실상 표준
    
    ### 2.2 Adapters
    
    [
    h' = h + f_{	ext{adapter}}(h)
    ]
    
    • 레이어 사이에 
    
    • LoRA보다 파라미터는 많지만 구조적 분리 명확
    
    ### 2.3 Prefix / Prompt Tuning
    
    [
    ext{Attention}(Q, [P;K], [P;V])
    ]
    
    • 가중치 미변경
    
    • 입력 시퀀스(또는 KV)에 학습된 프리픽스 추가
    
    • 하드웨어 변경 거의 없음
    
    ## 3. PEFT의 본질적 구조 해석 (중요)
    
    PEFT의 공통 구조는 다음으로 요약됩니다:
    
    [
    \boxed{
    y = Wx ;+; \underbrace{g_	heta(x)}_{	ext{작은 보정항}}
    }
    ]
    
    • (Wx)
    
    • (g_	heta(x))
    
    👉 이 구조는 **하드웨어 최적화 관점에서 매우 “의도적으로 분리”된 형태**입니다.
    
    ## 4. PEFT × Multiplierless/MCM 결합 포인트
    
    ### 4.1 베이스 경로에 MCM 적용 (가장 강력)
    
    • (W)는 
    
    • → 
    
    • 곱셈 제거
    
    • shift/add DAG
    
    • 공통 부분식(CSE) 공유
    
    [
    Wx ;\Rightarrow; 	ext{Shift/Add DAG}(x)
    ]
    
    **의미**
    
    • 정확도 
    
    • 베이스 연산의 
    
    • 특히 
    
    ### 4.2 LoRA 경로는 “정밀·소형”으로 유지
    
    [
    \Delta W x = B(Ax)
    ]
    
    • 연산량 작음 (rank (r))
    
    • MAC 유지
    
    • 빈번한 모델 교체/업데이트 대응 용이
    
    👉 **Hybrid 실행**이 최적:
    
    • TE(MAC)
    
    • VE(Shift/Add)
    
    ### 4.3 시스템 분해 관점 (NPU 아키텍처)
    
    ```
               ┌──────────────┐
    x ───────► │ Base Path     │ ──┐
               │ (MCM DAG)     │   │
               │ Shift/Add VE  │   │
               └──────────────┘   │
                                   ├─ add ─► y
               ┌──────────────┐   │
    x ───────► │ PEFT Path     │ ──┘
               │ (LoRA / Adpt) │
               │ Small MAC TE  │
               └──────────────┘
    ```
    
    ## 5. PEFT가 “Multiplierless 전략과 잘 맞는 이유”
    
    | 항목 | 이유 |
    | --- | --- |
    | 베이스 고정 | MCM 전제 조건 충족 |
    | 잔차 구조 | 하드웨어 분리 최적 |
    | 반복 사용 | CSE/공유 이득 극대화 |
    | 추론 중심 | 정확도 손실 허용 불가 → MCM 최적 |
    
    > 
    
    **AdderNet / Multiplier-less Neuron과 달리**→ *모델 수식은 그대로*, *구현만 바꿈*
    
    ## 6. LLM 추론 워크로드 기준의 실질적 이득
    
    ### Decode 단계 (token-by-token)
    
    • (Wx) 반복 호출
    
    • PEFT 보정은 상대적으로 미미
    
    • → 
    
    ### KV-cache와의 궁합
    
    • KV는 고정/양자화 경향
    
    • 베이스 projection에 MCM 적용 가능
    
    • PEFT는 attention bias 수준으로 유지
    
    ## 7. 연구/제품화 관점에서의 포지셔닝
    
    ### “PEFT + MCM”은 다음을 가능하게 함
    
    • 모델 호환성 100% 유지
    
    • 학습 파이프라인 변경 없음
    
    • 컴파일러/백엔드 최적화로 차별화
    
    • 서버/엣지 모두 적용 가능
    
    이는 단순한 양자화(INT8)보다:
    
    • 더 
    
    • 더 
    
    • 더 
    
    ## 8. 다음 단계로 바로 확장 가능한 주제
    
    원하시면 다음 중 하나를 바로 이어서 **설계 문서 수준**으로 내려갈 수 있습니다:
    
    1. PEFT-aware MCM 컴파일러 패스
    
    1. LoRA rank vs MCM depth의 에너지 trade-off 분석
    
    1. Prefill/Decode 분리 스케줄링 (MCM은 decode 집중)
    
    1. ONNX/MLIR에서 PEFT + MCM 표현 스키마
    
    지금까지의 흐름을 보면,👉 **PEFT는 “Multiplierless 추론을 실전에서 쓰게 해주는 열쇠”**에 가깝습니다.
    
- **You: [https://research.go...**
    
    [https://research.google/blog/coral-npu-a-full-stack-platform-for-edge-ai/](https://research.google/blog/coral-npu-a-full-stack-platform-for-edge-ai/) 이런 종류의 작은 칩의 응용으로 무곱셈기 기반으로 어때?
    
- **ChatGPT: 결론부터 말하면, **Coral NP...**
    
    결론부터 말하면, **Coral NPU 같은 “작은 엣지 칩”에 무곱셈기(shift/add-only) 기반을 ‘주력 경로’로 두는 건 조건부로만 유리**합니다. 다만 **특정 클래스의 모델/레이어를 대상으로 “보조 엔진(또는 컴파일러 패스)”로 넣으면 꽤 매력적인 설계 포인트**가 됩니다.
    
    아래는 Coral NPU(구글 리서치)와 Edge TPU 스펙을 기준으로, **무곱셈기 기반을 어디에/어떻게 쓰면 좋은지**를 하드웨어·컴파일러 관점에서 정리한 내용입니다.
    
    ## 1) Coral NPU/Edge TPU가 전통적인 MAC(곱셈) 엔진을 쓰는 이유
    
    구글 리서치 블로그의 Coral NPU는 **RISC-V 스칼라 코어 + RVV 벡터 유닛 + 양자화 MAC 기반 매트릭스 유닛**을 중심으로 잡고, IREE/MLIR(StableHLO) 기반 컴파일 흐름을 제시합니다.  cite turn1view0기존 Coral Edge TPU 쪽도 “INT8에서 4 TOPS, 2 TOPS/W” 같은 포지셔닝이 널리 알려져 있고, 모듈 스펙으로도 확인됩니다.   6 turn0search2
    
    핵심은 이거예요:
    
    • INT8 dot-product
    
    • LLM/CNN에서 
    
    • shift/add-only는 “곱셈기 제거” 대신 **덧셈기·배선·시프트·(중간값 저장/재사용)**이 늘어나며, 결국 
    
    즉 “곱셈을 없애면 무조건 전력↓”이 아니라, **무곱셈기에서는 ‘연산수’보다 ‘데이터 이동과 누산 구조’가 더 비싸지는 경우**가 많습니다.
    
    ## 2) 그럼에도 무곱셈기가 “엣지 초저전력”에서 먹히는 경우
    
    무곱셈기가 유리해지는 조건은 대체로 다음 중 하나를 만족할 때입니다.
    
    ### A. 가중치가 Po2/Shift-Add 형태로 강하게 제한되는 모델
    
    • 예: Shift-only/Shift-add 네트워크, hardened shifter 계열
    
    • 최근에는 “레이어를 하드와이어드(shifter)로 박아 넣고, 일부만 유연성 유지” 같은 극단적 설계도 나옵니다(예: HaShiFlex 2025).   3
    
    **이 경우 장점**
    
    • MAC 배열을 크게 만들 필요가 없고,
    
    • 아주 작은 전력/면적으로 
    
    **단점**
    
    • 모델 자유도/정확도/업데이트성(배포 유연성)이 급격히 떨어짐.
    
    ### B. “상수 곱셈 집합(MCM)”처럼 공통 부분식 공유(CSE)로 큰 이득이 나는 경우
    
    Voronenko & Püschel류 MCM의 강점은 **CSE로 shift/add를 공유해서 ‘덧셈 수’를 줄이는 것**인데, 이게 잘 먹히는 레이어는 생각보다 제한적입니다.
    
    • 동일 입력 (x)에 대해 여러 상수 (t_ix)를 만드는 구조(필터뱅크/변환/고정 프리프로세싱)에서 특히 좋습니다.
    
    • CNN/LLM의 큰 GEMM은 각 weight가 제각각이라 
    
    ### C. “항상 켜져 있어야(always-on) 하는 센싱”에서 처리량보다 mW 이하가 최우선일 때
    
    Coral NPU 블로그도 ‘웨어러블/always-on/ambient sensing’ 같은 목표를 강조합니다.  cite turn1view0이 영역에서는 TOPS보다 **절대전력(mW)**이 KPI가 되는 경우가 많아서,
    
    • 소형 shifter + 누산 기반의 특화 경로가 의미가 생깁니다.
    
    ## 3) Coral 같은 “풀스택 NPU”에 적용한다면 가장 현실적인 아키텍처
    
    Coral NPU가 이미 **Scalar + RVV + Matrix(MAC)**의 3단 구성을 제시한 점이 중요합니다.  cite turn1view0여기에 무곱셈기를 “대체재”로 넣기보다는, **4번째 옵션(특화 경로)**으로 넣는 게 맞습니다.
    
    ### 추천 구조: “MAC 주력 + Multiplierless 보조(옵션)”
    
    • Matrix(MAC) 유닛
    
    • Vector(RVV) 유닛
    
    • (추가) Shift/Add 전용 uEngine
    
    • Po2/shift-only 레이어
    
    • MCM으로 압축된 고정 상수 연산(프리/포스트 프로세싱, 특정 1×1/Depthwise 변형 등)
    
    • tiny model의 일부 레이어를 “하드와이어드”로 태워서 항상 켜짐 경로 최적화
    
    **컴파일러(MLIR/IREE) 관점**
    
    • 패스 1: “이 레이어가 multiplierless로 이득인가?”를 cost-model로 판정
    
    • 패스 2: 가능하면 weight를 
    
    • 패스 3: MCM 그래프 생성 + scheduling(레지스터/스칼라 scratchpad/VE 활용)
    
    ## 4) “무곱셈기 기반이 Edge TPU(4 TOPS/2W) 같은 제품 스펙을 이길 수 있나?”
    
    대부분의 범용 DNN(특히 CNN/Transformer)에서는 **그냥은 어렵습니다.**Edge TPU류는 이미 INT8 MAC을 극도로 효율화해 “2 TOPS/W” 레벨을 노립니다.   6 turn0search2
    
    무곱셈기로 역전하려면 보통:
    
    • 모델이 shift-only/저랭크/코드북
    
    • on-chip에서 
    
    • 레이어를 하드와이어드로 박아서 
    
    가능성이 생깁니다. (HaShiFlex류가 그 방향을 보여줌)   3
    
    ## 5) 당신의 연구(컴파일러+NPU 모델링) 관점에서 “좋은 연구 주제”로 만드는 법
    
    **가장 설득력 있는 연구 프레이밍**은 이겁니다:
    
    1. Coral 스타일의 풀스택(MLIR/IREE) NPU에서  cite turn1view0
    
    1. 특정 클래스(ambient sensing / tiny transformer / keyword spotting 등)에 대해
    
    1. “MAC 엔진 + Multiplierless 엔진”의 하이브리드 오프로딩
    
    1. cost-model로 자동 선택하게 하며
    
    1. 전력/면적/지연을 system-level로 비교
    
    이렇게 가면,
    
    • “완전 multiplierless”의 비현실성을 피하면서
    
    • “특정 워크로드에서의 확실한 이득”을 만들 수 있습니다.
    
    원하면, 다음 중 하나로 더 깊게 들어가서 **당신 NPU 시뮬레이터/IR에 바로 붙일 수 있는 수준**으로 내려가 정리할게요.
    
    1. Multiplierless 오프로딩 판단용 cost model
    
    1. MLIR 패스 설계 스케치
    
    1. Coral NPU의 “scalar+RVV+matrix” 구조에 “shift-engine”을 추가했을 때의 
    
- **You: [https://research.go...**
    
    [https://research.google/blog/coral-npu-a-full-stack-platform-for-edge-ai/](https://research.google/blog/coral-npu-a-full-stack-platform-for-edge-ai/) 이런 종류의 작은 칩의 응용으로 무곱셈기 기반으로 어때?
    
- **ChatGPT: “Coral NPU 같은 **초소형 ...**
    
    “Coral NPU 같은 **초소형 엣지 칩(수 mW~수십 mW, 512 GOPS급 목표)**”에 **무곱셈기(shift/add-only, 또는 MCM 기반)**를 쓰는 건 **‘대체 아키텍처’로는 대부분 불리**하고, 대신 **특정 연산군을 겨냥한 ‘보조 경로/옵션’**으로 넣으면 꽤 설득력 있습니다.
    
    Coral NPU 자체가 이미 “작은 칩에서의 현실적인 정답”에 가까운 구성을 취하고 있어요:**Scalar(RISC-V) + RVV Vector + Quantized MAC 기반 Matrix 엔진**의 3단 융합 구조입니다.  cite turn1view0 turn1view2
    
    ## 1) 왜 Coral류는 “무곱셈기”가 아니라 “Quantized MAC(곱셈)”을 코어로 두나
    
    Coral NPU는 “ML matrix engine을 우선”으로 설계하면서, **Matrix execution unit을 ‘quantized outer-product MAC 엔진’**이라고 명시합니다.  cite turn1view0이 선택은 단순합니다:
    
    • 엣지에서 가장 지배적인 커널(Conv/Linear/Attention)은 
    
    • INT8/저정밀 MAC은 면적/전력 효율이 매우 좋아서, 
    
    • 반면 shift/add-only는 곱셈기를 없애는 대신
    
    즉, “곱셈 제거 = 무조건 저전력”이 아니라 **엣지에서는 오히려 메모리·배선 비용이 더 크게 튀는** 경우가 많습니다.
    
    ## 2) 그럼에도 무곱셈기가 ‘이기는’ 영역이 있다 (조건부)
    
    ### (A) Always-on / Ambient sensing에서 “절대전력(mW)”이 KPI일 때
    
    Coral NPU가 목표로 하는 것도 **ultra-low-power, always-on, ambient sensing**입니다.  cite turn1view0이 경우 “최대 처리량”보다 “항상 켜져 있어도 배터리를 안 먹는” 쪽이 중요해져서,
    
    • 소형·특화된 shift/add 경로
    
    • 특정 프리프로세싱/특정 레이어를 거의 고정 동작으로 처리
    
    같은 전략이 의미가 생깁니다.
    
    ### (B) 가중치를 Po2/Shift-only로 강하게 제한한 모델(또는 코드북/클러스터링)
    
    • 모델이 애초에 
    
    • 작은 칩에서 “곱셈기 제거 + 단순 데이터패스”가 진짜 이득이 됩니다.
    
    • 다만 이건 
    
    ### (C) MCM(공통 부분식 공유)이 강하게 먹히는 “상수곱 집합” 형태
    
    Coral도 벡터/스칼라를 함께 두는 이유가 “fused execution(후처리, elementwise, 제어)” 때문인데,  cite turn1view1 turn1view2여기에 **MCM 그래프(CSE 공유)**가 잘 맞는 연산군(필터뱅크/변환/고정 전처리)이 있으면 “무곱셈기”가 꽤 경쟁력 있습니다.
    
    ## 3) 현실적인 결론: “MAC 주력 + Multiplierless 보조 엔진/패스”가 정답
    
    Coral NPU의 기본 철학이 이미 “행렬(MAC) 중심 + 벡터/스칼라 융합”입니다.  cite turn1view0 turn1view2여기에 무곱셈기를 **대체재**로 넣기보다는, 아래처럼 **옵션으로 붙이는 게 가장 설득력** 있어요.
    
    ### 추천 아키텍처
    
    • Matrix(MAC) 엔진
    
    • Vector(RVV) 엔진
    
    • (옵션) Shift/Add 전용 uEngine
    
    • Po2/shift-only 레이어
    
    • MCM으로 최적화된 고정 상수 연산(전처리/후처리/일부 depthwise/1×1 변형)
    
    • always-on 경로에서의 초저전력 고정 파이프라인
    
    ### 컴파일러(StableHLO→IREE/MLIR) 관점
    
    Coral는 StableHLO→IREE→하드웨어 플러그인 기반 progressive lowering 흐름을 제시합니다.  cite turn1view0여기에 넣을 “무곱셈기” 전략은:
    
    1. Candidate 탐지
    
    1. cost-model 판정
    
    1. MCM 그래프 생성 + 스케줄링
    
    ## 4) 당신의 NPU 연구 관점에서 “좋은 연구/제품화 포인트”
    
    당신이 이미 **IR lowering + 타일 기반 스케줄링 + VE/TE 분리**를 하고 있으니, 여기서 가장 깔끔한 연구 주제는 이겁니다:
    
    • Coral류 “작은 칩”에서
    
    이렇게 하면 “무곱셈기 올인”의 리스크(정확도/범용성)를 피하면서도, **항상-온 엣지**에서 확실한 효율 이득을 만들 수 있습니다. (Coral의 목표 앱과도 정합)  cite turn1view0
    
    원하면 다음 중 하나를 바로 만들어드릴게요(문서/스펙 형태로):
    
    1. Multiplierless 오프로딩 cost model
    
    1. MLIR 패스 설계
    
    1. 타일 스케줄링
    
- **You: 위 아이디어를 기반으로 vit나 ge...**
    
    위 아이디어를
    기반으로 vit나 gemma3n에 학습부터 추론까지 가능한 새로운 신경망을 만들어볼까?
    
- **ChatGPT: 가능합니다. 다만 “ViT / Gem... (Part 1/2)**
    
    가능합니다. 다만 “ViT / Gemma3n 급”에 **순수 무곱셈기(shift/add-only)로 학습·추론 전체를 커버**하려면 성공 확률이 낮고, 대신 **(1) MAC 코어는 유지**하면서 **(2) 무곱셈기 경로를 ‘주력 연산의 일부’로 끌어올리는 하이브리드 신경망**이 현실적이면서도 새롭습니다.
    
    아래는 당신이 앞에서 쌓아온 아이디어(Edge급 small chip, Coral류 풀스택, MCM, PEFT)를 그대로 엮어 **“학습부터 추론까지 end-to-end로 가능한 새 모델/시스템”** 설계안입니다.(목표: **ViT급 비전 + Gemma3n급 LLM(또는 SLM) 라인에 적용 가능한 공통 원리**)
    
    # 1) 제안 모델 개념: HyMME (Hybrid MCM + MAC Engine)
    
    핵심은 한 레이어를 이렇게 분해하는 겁니다.
    
    [
    y = \underbrace{W_{	ext{base}} x}*{	ext{멀티플라이어리스 후보}} ;+; \underbrace{\Delta W x}*{	ext{작은 보정(PEFT/LoRA)}}
    ]
    
    • (W_{	ext{base}})
    
    • (\Delta W)
    
    • 추론 시:
    
    • Base 경로는 
    
    • 보정 경로는 
    
    이렇게 하면:
    
    • 모델의 표현력 손실을 
    
    • 하드웨어에서는 “큰 GEMM의 일부”를 
    
    • Coral 같은 “scalar+vector+matrix” 구조에 
    
    # 2) “무곱셈기화”를 학습 가능하게 만드는 핵심: Base weight를 구조적으로 제한
    
    ## 2.1 제한 방식 A: Po2(±2^k) + 작은 코드북 혼합
    
    베이스 가중치를 다음 형태로 제한합니다.
    
    • Po2-only
    
    • 또는 
    
    학습은 QAT처럼 “실수 shadow weight”를 두고, forward에서는 projection:
    
    [
    w_{	ext{base}} = \Pi_{	ext{Po2/Codebook}}(	ilde{w})
    ]
    
    > 
    
    이 방식의 장점: 모델 학습은 정상적으로 되면서, 추론 때는 무곱셈기 경로가 생김.
    
    ## 2.2 제한 방식 B: “MCM-friendly”를 직접 loss로 넣기
    
    컴파일러/하드웨어 비용을 프록시로 만들어 loss에 넣습니다.
    
    • 예: 한 레이어에서 “고유 상수 개수”, “Po2 비율”, “근사 shift-add 깊이” 등을 penalty로
    
    • 목표: 
    
    # 3) ViT에 적용하는 구체 설계안
    
    ViT에서 가장 비싼 건 대략:
    
    • QKV projection (Linear)
    
    • Attention의 matmul (QKᵀ, AV)
    
    • MLP(FFN)의 두 Linear
    
    여기서 “곱셈 없는 쪽으로 유리”한 건 **Linear(가중치 상수)** 입니다.Attention matmul은 입력×입력이라 MCM이 직접 안 먹힙니다.
    
    ## 3.1 ViT-HyMME 블록 (추천)
    
    • QKV/FFN의 
    
    • Attention matmul은 기존 MAC/Matrix 엔진 사용
    
    • 각 Linear마다 
    
    구조:
    
    • (Q = (W^Q_{	ext{base}}x) + 	ext{LoRA}_Q(x))
    
    • (K, V, FFN)도 동일
    
    이렇게 하면 ViT에서 “큰 비중을 차지하는 Linear들”에 대해:
    
    • Base 경로를 multiplierless화
    
    • LoRA는 작은 MAC
    
    # 4) Gemma3n/SLM(LLM) 쪽에 적용하는 구체 설계안
    
    LLM에서 지배 항은 대부분 **Linear(GEMM)** 입니다.
    
    • Attention의 QKV, O projection
    
    • FFN up/gate/down projection
    
    여기서도 마찬가지로:
    
    • 입력×입력인 attention score matmul은 그대로 MAC
    
    • Linear는 무곱셈기 후보
    
    ## 4.1 LLM-HyMME 블록 (추천)
    
    • 모든 Linear에 대해:
    
    • (W_{	ext{base}}): Po2/코드북 제한 (추론 시 shift-add/MCM)
    
    • (\Delta W): LoRA/Adapter (추론 시 MAC)
    
    • 특히 
    
    # 5) “학습부터 추론까지” 전체 파이프라인 제안
    
    ## 단계 0: 목표/지표 정의
    
    • 정확도: ViT는 top-1 / mAP, LLM은 ppl/MT-bench류(가능 범위 내)
    
    • 시스템: latency/token, energy proxy, SRAM traffic proxy
    
    • 하드웨어 비용 프록시:
    
    • MAC count vs ADD/SHIFT count
    
    • unique constants / codebook size
    
    • MCM DAG depth(근사)
    
    ## 단계 1: 베이스 모델 준비
    
    • ViT: DeiT-tiny/small부터 시작 추천
    
    • LLM: Gemma3n은 꽤 빡세므로, “TinyLlama급”에서 먼저 검증 후 scaling
    
    ## 단계 2: Base 경로 제약 학습(QAT 스타일)
    
    • shadow weight 유지
    
    • forward에서 Po2/코드북 투영
    
    • backward는 STE(추정 기울기) 또는 smooth projection
    
    ## 단계 3: PEFT(LoRA) 결합
    
    • 초기에는 LoRA rank를 조금 크게(r=16~32) 두고 수렴 확인
    
    • 이후 rank를 줄이면서 base 제약을 강화
    
    ## 단계 4: 컴파일러 백엔드(당신 강점)
    
    • 그래프에서 Linear를 분해:
    
    • base: MCM graph 생성(또는 Po2 shifter network 생성)
    
    • lora: 작은 GEMM kernel
    
    • 스케줄:
    
    • base는 VE/uEngine
    
    • lora는 TE
    
    ## 단계 5: 추론 최적화
    
    • decode 전용 최적화(LLM)
    
    • weight prepacking
    
    • base MCM graph를 “레이어별 고정 커널”로 캐시
    
    # 6) 이게 “새로운 신경망”이 되는 이유 (논문화 포인트)
    
    단순히 “Po2 네트워크”나 “LoRA”가 아니라,
    
    • (A) 베이스는 multiplierless로 
    
    • (B) PEFT를 ‘정확도 보정 장치’로 구조화
    
    • (C) 컴파일러가 MCM 그래프를 생성하여 실행 엔진을 분기
    
    • (D) 작은 칩(코랄류)에서 scalar/vector/matrix의 하이브리드 실행으로 시스템 이득을 보임
    
    즉, 모델-학습-컴파일러-하드웨어가 하나의 story로 이어집니다.
    
    # 7) 실패 조건(미리 박아두면 설계가 단단해짐)
    
    • 베이스 weight 제한이 너무 강하면 LoRA가 흡수 못함 → 정확도 붕괴
    
    • MCM 공유가 생각보다 안 생김(상수가 너무 다양) → shift/add만 늘고 이득 없음
    
    • 메모리/누산 트래픽이 증가 → 에너지 이득 상쇄
    
    • 결국 “MAC가 더 싸다”가 되는 crossover가 존재
    
    → 그래서 “완전 무곱셈기”가 아니라 **하이브리드**가 안전합니다.
    
    # 8) 당신에게 추천하는 “가장 현실적인 1차 실험 세트”
    
    1. ViT-DeiT-tiny
    
    • QKV/FFN Linear만 HyMME 적용
    
- **ChatGPT: 가능합니다. 다만 “ViT / Gem... (Part 2/2)**
    
    • Attention matmul은 그대로
    
    • base는 Po2 + codebook(예: M=16)
    
    • LoRA rank 8~16
    
    1. TinyLlama급 SLM
    
    • 모든 Linear에 동일 적용
    
    • decode 성능(토큰당 latency)과 SRAM 트래픽 프록시 측정
    
    1. 결과로 “crossover chart” 제시
    
    • (정확도 vs MAC 감소율 vs ADD/SHIFT 증가율 vs SRAM traffic)
    
    원하면, 다음을 바로 만들어 드릴게요(당신 레포/시뮬레이터에 넣기 좋게):
    
    • HyMME 레이어 스펙 문서
    
    • 컴파일러 패스 설계
    
    • cost model 초안
    
    • 1차 실험 프로토콜
    
    어느 쪽부터 갈까요?
    
    • ViT 쪽(비전)부터 빠르게 검증
    
    • LLM(Tiny→Gemma3n)로 바로 가기
    
    둘 중 하나로 바로 “실행 가능한 설계”로 내려가겠습니다.
    
- **You: Vit먼저. 카메라 isp 대신 mc...**
    
    Vit먼저. 카메라 isp 대신 mcm기반 end to end 신경망 하드웨어를 칩에 구현하는것도 고려
    
- **ChatGPT: 좋습니다. **ViT부터 시작**하되... (Part 1/2)**
    
    좋습니다. **ViT부터 시작**하되, 방향을 “그냥 ViT를 MCM으로 최적화”가 아니라 **카메라 ISP를 일부 대체/흡수하는 ‘MCM-친화 End-to-End Vision HW 파이프라인’**으로 잡으면 칩 레벨에서 스토리가 훨씬 강해집니다. 핵심은:
    
    • (A) 입력단에서 ISP가 하던 ‘고정 필터/변환/정규화/색공간’은 원래 MCM이 가장 잘 먹히는 영역
    
    • (B) ViT의 본체는 MAC이 지배적이므로, ‘Base(고정·제약) + LoRA(보정)’ 하이브리드로 MCM을 끌어올리고
    
    • (C) 작은 칩(Coral류)의 Scalar+Vector+Matrix 구조에 “MCM uEngine”을 1-class citizen으로 넣어 end-to-end로 묶는 것
    
    아래는 **바로 실험/스펙으로 내려갈 수 있는 설계안**입니다.
    
    # 1) 전체 시스템 컨셉: ISP→ViT를 하나의 “MCM-First E2E Vision”으로
    
    기존:
    **Sensor → ISP(고정) → RGB/YCbCr → CNN/ViT**
    
    제안:
    **Sensor → MCM-ISP(고정 변환) → Tokenizer(부분 MCM) → ViT-HyMME(부분 MCM) → Head**
    
    여기서 “MCM-ISP”는 ISP를 전부 대체하자는 게 아니라,
    
    • 학습 가능한 end-to-end pipeline을 위해 최소한의 고정 변환을 유지하면서도
    
    • 하드웨어에서는 
    
    • ViT가 안정적으로 학습되도록 “신호 품질”을 챙기는 계층입니다.
    
    # 2) ViT 쪽: HyMME 블록을 최소 변경으로 적용
    
    ViT에서 MCM이 직접 잘 먹는 건 **Linear(상수 weight)** 입니다.
    Attention의 QKᵀ는 입력×입력이라 MCM이 안 맞습니다.
    
    ## 2.1 ViT-HyMME 레이어 정의 (추천)
    
    각 Linear를 아래처럼 분해합니다.
    
    [
    y = (W_{	ext{base}}x) + \underbrace{B(Ax)}_{	ext{LoRA}}
    ]
    
    • (W_{	ext{base}}): 
    
    • LoRA(저랭크): 정확도 보정, TE(MAC)로 처리
    
    ### Base 제약의 1차 실험용 선택지
    
    • Codebook(예: 16 또는 32개 상수)
    
    • 장점: Po2보다 정확도 유지가 쉬움
    
    • MCM 공유(CSE) 유도 가능(상수 종류가 제한되니까)
    
    • Po2-only는 2차에서(난이도↑)
    
    # 3) “카메라 ISP 대신”을 노릴 때, MCM이 가장 강한 지점
    
    ISP에서 흔히 등장하는 연산은 사실상 “상수 필터 + piecewise 비선형”입니다.
    
    ## 3.1 MCM-ISP 후보 블록 (곱셈 없이 구현 가능)
    
    1. Color space / 3×3 CCM
    
    # [
    \begin{bmatrix}R'\G'\B'\end{bmatrix}
    
    C
    \begin{bmatrix}R\G\B\end{bmatrix}
    ,\quad C \in \mathbb{R}^{3	imes 3}
    ]
    
    • 상수 행렬 → MCM 최적화에 매우 적합
    
    • 칩에서 “3채널 선형 변환”은 자주 등장
    
    1. 간단한 Denoise / Sharpen / Edge (3×3, 5×5)
    
    • depthwise conv 형태로 제한하면 하드웨어가 매우 단순해짐
    
    • 고정 커널(또는 소수 코드북 커널)로 구성 가능
    
    1. Local normalization / gain / white balance
    
    • gain이 상수 또는 저비트면 shift/add로 근사 가능
    
    • nonlinearity는 LUT(벡터 엔진)로 처리
    
    즉, **전통 ISP에서 ‘곱셈’이 많은 부분은 대부분 상수곱이라 MCM으로 잘 눌립니다.**
    그리고 이 블록들을 “학습 가능한 front-end”로 설계하면,
    
    • ISP가 하던 역할을 신경망이 흡수하고
    
    • 대신 하드웨어는 여전히 싸게 유지 가능합니다.
    
    # 4) 하드웨어 아키텍처 제안: Coral류 + “MCM uEngine” 추가
    
    Coral NPU가 지향하는 구조(Scalar + RVV + Matrix)를 기준으로,  cite turn1view0 turn1view2여기에 MCM용 uEngine을 붙이면 그림이 깔끔해집니다.
    
    ## 4.1 실행 유닛 분담
    
    • Matrix(MAC) 엔진
    
    • Vector(RVV) 엔진
    
    • MCM uEngine
    
    • MCM-ISP(3×3 CCM, 고정 필터뱅크)
    
    • Tokenizer의 일부(예: patch embedding의 코드북/shift-add 버전)
    
    • ViT Linear의 base 경로(코드북 제한된 상수곱 → CSE 공유)
    
    핵심은 “MCM uEngine이 항상 동작할 연산이 충분히 많아야” 칩이 설득력 있습니다.그래서 **프론트(ISP대체) + ViT base**를 동시에 밀어야 합니다.
    
    # 5) 학습부터 추론까지 “실행 가능한” 1차 실험 계획 (ViT 우선)
    
    ## 5.1 데이터/모델
    
    • DeiT-tiny
    
    • 데이터: CIFAR-100 → ImageNet-1k 순으로 스케일업(빠른 검증)
    
    ## 5.2 모델 구성 (1차 MVP)
    
    1. MCM-ISP front-end (학습 가능 옵션 포함)
    
    • 기본은 고정(상수)로 시작하되, 선택적으로 “코드북 커널”을 학습 가능하게
    
    • 출력은 ViT 입력으로 바로 연결
    
    1. Patch embedding
    
    • 먼저는 일반 Linear/Conv로 두고,
    
    • 2차에서 코드북 제약 + MCM화
    
    1. ViT Linear들(QKV/FFN)
    
    • (W_{	ext{base}}): codebook 제한(예: 16 values) + QAT
    
    • LoRA rank: 8~16
    
    ## 5.3 학습 레시피(간단)
    
    • stage1: 베이스 ViT + front-end를 float로 학습/수렴 확인
    
    • stage2: base weight를 codebook으로 투영(QAT/STE)
    
    • stage3: LoRA를 붙여 정확도 회복
    
    • stage4: MCM 비용 프록시를 loss에 약하게 추가(선택)
    
    ## 5.4 평가 지표(하드웨어 친화)
    
    • 정확도(top-1)
    
    • MAC 감소율
    
    • 유니크 상수 개수 / 코드북 재사용률
    
    • MCM DAG cost proxy
    
    • SRAM traffic proxy(activation/weight read/write)
    
    이걸로 “왜 칩에서 의미 있는지”가 숫자로 나옵니다.
    
    # 6) 칩 구현을 염두에 둔 설계 포인트(당장 결정해야 할 것들)
    
    ## 6.1 MCM uEngine의 ISA/마이크로옵
    
    • 입력 스트림 x에 대해
    
    • 특정 상수 집합에 대한 (t_ix)들을
    
    • shift/add 그래프로 생성 → 누산
    
    필수 기능:
    
    • shift-left/right, add/sub
    
    • abs/clip(optional)
    
    • partial sum 재사용(레지스터 파일 또는 small SPM)
    
    • 출력 스트라이드/벡터화(채널 병렬)
    
    ## 6.2 메모리 계층
    
    MCM은 연산 자체보다 “중간값 저장/재사용”이 성능을 좌우합니다.
    
    • 레지스터 파일(작은 PRF) + scratchpad(수 KB~수십 KB)
    
    • CSE 공유가 많을수록 PRF 재사용률이 올라가고 energy가 떨어짐
    
    ## 6.3 컴파일러 패스(당신 강점)
    
    • 그래프에서 “MCM 후보” 추출
    
    • 코드북/Po2 weight를 상수 집합으로 모으고
    
    • Voronenko-style MCM 그래프 생성 + scheduling
    
    • VE/uEngine으로 lowering
    
    # 7) 권장 “논문/프로젝트” 타이틀 스토리라인
    
    > 
    
    **“MCM-First End-to-End Vision: Replacing ISP and Accelerating ViT with Multiplierless Constant-Multiplication Graphs and PEFT Residuals on Edge NPUs”**
    
    이 스토리의 장점:
    
- **ChatGPT: 좋습니다. **ViT부터 시작**하되... (Part 2/2)**
    
    • ISP 대체(또는 축소)라는 “시장/제품” 관점의 임팩트
    
    • MCM이라는 “회로/컴파일러” 관점의 기여
    
    • ViT + LoRA라는 “모델/학습” 관점의 실증
    
    • 작은 칩(Coral류)에서의 “풀스택” 설계 정합
    
    ## 다음 액션 (제가 바로 만들어줄 수 있는 산출물)
    
    1. ViT-HyMME + MCM-ISP 모델 스펙
    
    1. 컴파일러 패스 설계 문서
    
    1. MCM uEngine 마이크로아키텍처/ISA 초안
    
    1. 1차 실험 프로토콜
    
    원하는 순서는 보통 **(1) 모델 스펙 → (4) 실험 프로토콜 → (2) 컴파일러 → (3) 하드웨어**가 가장 빠릅니다.
    
- **You: Mcm-isp를 vit가 아닌 cnn...**
    
    Mcm-isp를 vit가 아닌 cnn으로 한다면? Sesr이나 quicksenet을 활용
    
- **ChatGPT: 가능하고, **ViT보다 CNN이 “... (Part 1/2)**
    
    가능하고, **ViT보다 CNN이 “MCM-ISP와 훨씬 더 자연스럽게 결합**됩니다. 특히 **SESR / QuickSeNet 같은 초경량 네트워크**는 (1) 고정/반고정 필터(상수곱)와 (2) depthwise/pointwise 구조가 많아서 **MCM(CSE 공유) + shift/add-only 경로**를 설계하기 좋은 편입니다.
    
    아래는 **MCM-ISP + CNN(end-to-end)** 관점에서 “왜 CNN이 더 맞는지”, “SESR/QuickSeNet을 어떻게 쓰는지”, “칩에 어떻게 얹는지”를 정리한 설계안입니다.
    
    # 1) 왜 CNN이 MCM-ISP에 더 잘 맞나
    
    ## 1.1 “ISP 성격” 자체가 CNN에 가깝다
    
    ISP에서 중요한 블록(denoise, sharpen, demosaic 주변, edge enhancement 등)은 본질적으로
    
    • local neighborhood(3×3/5×5) 처리
    
    • 공간적으로 반복되는 필터
    
    • 스트리밍 데이터플로우
    
    라서 CNN, 특히 **depthwise conv**와 거의 동형입니다.
    
    → ViT는 patch/token 단위로 넘어가면서 “global mixing”이 중심이라, ISP 대체/흡수라는 스토리에서 CNN이 더 직접적입니다.
    
    ## 1.2 CNN에서는 “상수곱 공유(CSE)”가 더 쉽게 생김
    
    • 작은 커널(3×3)과 반복되는 channel 패턴
    
    • 코드북(kernel clustering)을 걸면 
    
    • 그러면 MCM 그래프를 “커널 단위”로 공유하기 쉬움
    
    # 2) MCM-ISP + CNN 아키텍처: 두 가지 노선
    
    ## 노선 A) “ISP 블록을 CNN 앞단으로 넣고 end-to-end 학습”
    
    **Sensor/RGB → MCM-ISP(고정 또는 코드북 학습) → Tiny CNN(분류/검출/SR)**
    
    • MCM-ISP는 “고정 상수 변환 + 약간의 비선형(LUT)” 정도로 시작
    
    • CNN이 나머지 보정/해석을 담당
    
    • 칩에서는 MCM-ISP는 거의 “항상 켜짐(always-on)” 파이프라인으로 최적화 가능
    
    ## 노선 B) “CNN 자체를 ISP+비전까지 흡수하는 구조로 재설계”
    
    **(Front depthwise stack = ISP 역할) + (Back stack = task 역할)**
    
    • 앞 몇 레이어를 “ISP 성격의 필터뱅크”로 규정하고
    
    • 그 레이어에 강한 MCM 제약(코드북/Po2)을 적용
    
    • 뒷단은 일반 INT8 MAC 또는 약한 제약 + SE 블록 등
    
    # 3) SESR / QuickSeNet을 어떻게 활용할까
    
    여기서 중요한 건 “이 두 모델을 그대로 가져다 쓰는 것”보다,
    **어떤 구조가 MCM-ISP와 잘 붙는지**입니다.
    
    ## 3.1 SESR (초경량 SR 계열)로 가는 경우
    
    SR은 ISP의 연장선(특히 sharpen/denoise/edge)이라 스토리가 강해집니다.
    
    **추천 구조**
    
    • MCM-ISP: (CCM + 간단 denoise + sharpening)
    
    • SESR: 초경량 업스케일/디테일 복원
    
    **왜 MCM에 유리?**
    
    • SR 모델은 종종
    
    • 3×3 conv 반복
    
    • 잔차 구조
    
    • 채널 수가 작음
    
    이라서 kernel codebook을 걸기 쉽고, MCM 최적화가 잘 먹습니다.
    
    **칩 관점**
    
    • MCM-ISP는 스트리밍 처리(라인버퍼)로 완전 고정
    
    • SESR는 MAC 엔진 중심이되, 앞단/일부 레이어는 MCM 엔진으로 오프로딩
    
    ## 3.2 QuickSeNet (경량 분류/인식)로 가는 경우
    
    QuickSeNet은 모바일 경량 CNN 계열(Depthwise + SE 류) 패턴을 기대할 수 있어,
    **“MCM-ISP + always-on 인식”** 흐름에 잘 맞습니다.
    
    **추천 구조**
    
    • MCM-ISP: 밝기/색/노이즈/edge를 “최소 비용으로 안정화”
    
    • QuickSeNet: 후단에서 분류/검출의 feature extraction
    
    **왜 MCM에 유리?**
    
    • Depthwise는 커널이 작고, 채널별 독립이라 
    
    • Pointwise(1×1)는 MAC이 지배적이므로, 여기는 그냥 INT8 GEMM으로 두는 게 보통 유리합니다.
    
    • 단, 1×1에서도 codebook/low-rank를 걸어 base 일부를 MCM화하는 하이브리드도 가능
    
    # 4) “MCM-ISP”를 CNN으로 할 때의 핵심 설계: Kernel Codebook + MCM Graph
    
    CNN에서 MCM을 진짜로 이득 보려면 “무작정 shift/add”가 아니라
    
    1. 커널/가중치의 고유값 종류를 줄이고
    
    1. 그 제한된 상수 집합에 대해
    
    1. MCM 그래프(CSE 공유)를 생성
    
    ### 4.1 Depthwise Conv용 “커널 코드북” 예시
    
    • 각 채널의 3×3 커널 (K_c)를
    
    • 코드북 원소 (B_m) 중 하나로 양자화:
    
    [
    K_c \approx B_{\pi(c)},\quad \pi(c)\in{1..M}
    ]
    
    • 그러면 하드웨어는
    
    • 코드북 M개 커널만 MCM 최적화해 구현하고
    
    • 채널마다 “어느 커널을 쓰는지”만 선택하면 됨
    
    👉 이게 되면 CNN에서도 **웨이블릿 필터뱅크처럼 “상수곱 집합” 구조**가 만들어집니다. (MCM이 본업으로 뛰는 지점)
    
    # 5) 하드웨어(칩)에서의 실행 매핑
    
    ## 5.1 가장 깔끔한 매핑
    
    • MCM 엔진(shift/add + CSE 재사용)
    
    • MCM-ISP 블록 전체
    
    • CNN의 depthwise(코드북화 된 앞단 레이어)
    
    • MAC/Matrix 엔진(INT8 GEMM)
    
    • pointwise 1×1
    
    • 후단 conv/FC
    
    • SE의 FC 부분
    
    • Vector 엔진
    
    • activation, BN/scale, LUT, reduce
    
    ## 5.2 데이터플로우(스트리밍)
    
    MCM-ISP와 depthwise는
    
    • 라인버퍼 + 슬라이딩 윈도우로 스트리밍 처리 가능
    
    • 중간 feature map 저장을 줄일 수 있어 “진짜 저전력”이 됩니다.
    
    # 6) ViT 대비 CNN 선택의 “논문화 포인트”가 더 좋아지는 이유
    
    • ISP는 원래 지역적 처리 → CNN이 구조적으로 더 aligned
    
    • depthwise/코드북 → MCM을 위한 “상수 집합”을 만들기 쉽다
    
    • SESR(SR)나 always-on 인식(QuickSeNet)은 엣지 칩 응용과 바로 연결
    
    즉,
    **“ISP 대체 + MCM 가속 + 초경량 CNN”**은 제품/칩 스토리가 매우 강합니다.
    
    # 7) 가장 현실적인 1차 실험 세트(추천)
    
    ### 실험 A: MCM-ISP + QuickSeNet (분류)
    
    • 데이터: CIFAR-100 → ImageNet-subset
    
    • 비교:
    
    1. baseline QuickSeNet
    
    1. depthwise kernel codebook + MCM 오프로딩
    
    ### 실험 B: MCM-ISP + SESR (SR)
    
    • 데이터: DIV2K / Set5 / Set14
    
    • 지표: PSNR/SSIM + latency/energy proxy
    
    **측정(당신 시뮬레이터에 맞게)**
    
    • MAC count vs ADD/SHIFT count
    
    • 코드북 크기 M에 따른 MCM DAG 노드 수
    
    • SRAM traffic(라인버퍼 vs feature map 저장) 추정
    
    • end-to-end latency
    
    ## 다음 단계로, 내가 바로 만들어줄 수 있는 산출물
    
    1. MCM-ISP 블록 스펙
    
    1. QuickSeNet/SESR용 “kernel codebook + QAT” 학습 레시피
    
- **ChatGPT: 가능하고, **ViT보다 CNN이 “... (Part 2/2)**
    
    1. MCM 그래프 생성 규칙
    
    1. 칩 아키텍처/스케줄링 문서
    
    원하는 방향은 어느 쪽이 더 가깝나요?
    
    • A) always-on 인식(QuickSeNet)
    
    • B) SR/이미지 품질(SESR)