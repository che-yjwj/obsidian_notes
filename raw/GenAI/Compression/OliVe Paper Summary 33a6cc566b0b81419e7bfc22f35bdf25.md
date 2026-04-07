# OliVe 논문 요약

ChatTime: 2026년 1월 12일 1:03 (GMT+9)
Created time: 2026년 4월 6일 오후 10:51
ProjectName: GenAI.Compression
URL: https://chatgpt.com/c/69639adf-6c10-8327-96f0-35691c092079

- **You: OliVe: Accelerating ...**
    
    OliVe: Accelerating Large Language Models via Hardware-friendly Outlier-Victim Pair Quantization
    해당 논문의 핵심 요약해서 정리해줘.
    
- **ChatGPT: 아래는 **OliVe: Acceler... (Part 1/2)**
    
    아래는 **OliVe: Accelerating Large Language Models via Hardware-friendly Outlier-Victim Pair Quantization** 논문의 핵심을 **하드웨어·시스템 관점(NPU/SoC, 데이터패스, 컴파일러)**에서 정리한 요약이다.(연구 의도 → 핵심 아이디어 → 알고리즘 구조 → 하드웨어 친화성 → 기존 기법 대비 차별점 → 적용 시사점 순)
    
    ## 📌 논문 한 줄 요약
    
    **OliVe는 LLM 양자화에서 문제였던 outlier를 ‘제거(prune)’하지 않고, 일반 weight(victim)와 짝(pair)으로 묶어 동일한 스케일에서 처리함으로써, 정확도 손실 없이 하드웨어 구현이 매우 쉬운 정적 저비트 양자화를 달성한다.**
    
    ## 1. 문제의식 (왜 이 논문이 나왔나)
    
    LLM 양자화의 고질적 문제는 다음이다.
    
    • Weight/Activation 분포에 
    
    • 이 outlier 때문에
    
    • 스케일이 커짐 → 
    
    • 해결책으로 등장한 기존 방식들:
    
    - Outlier를 FP16/FP32로 분리 (Mixed Precision)
    
    - 채널별/그룹별 스케일 (fine-grained scaling)
    
    - GPTQ / AWQ / SmoothQuant
    
    👉 **하지만**
    
    • Mixed precision → 하드웨어 복잡도 폭증
    
    • Channel-wise scaling → 디코더/스케일 로딩/파이프라인 부담
    
    • GPTQ/AWQ → 오프라인 비용 큼, HW 매핑 복잡
    
    **OliVe의 질문**
    
    > 
    
    “Outlier를 특별 대우하지 말고, 그냥 *같이* 처리할 수는 없을까?”
    
    ## 2. 핵심 아이디어: Outlier–Victim Pair Quantization
    
    ### 🔹 핵심 관찰
    
    • Outlier는 
    
    • Outlier 하나가 스케일을 망가뜨리는 이유는 
    
    ### 🔹 OliVe의 발상
    
    > 
    
    **Outlier 하나를, 크기가 작은 weight(victim) 하나와 ‘쌍(pair)’으로 묶어 동일 스케일에서 양자화하자**
    
    즉,
    
    • 하나의 quantization unit = 
    
    • 두 값의 합성 통계로 스케일 결정
    
    ## 3. 알고리즘 구조 (개념적)
    
    ### 3.1 Pair 구성
    
    1. Weight tensor에서 outlier 후보 탐색
    
    1. 각 outlier에 대해
    
    • 분포상 적절한 일반 weight(victim) 선택
    
    1. (outlier, victim) → 하나의 pair
    
    ### 3.2 Quantization
    
    • Pair 단위로:
    
    • 단일 scale / zero-point
    
    • 저비트(INT8/INT6/INT4) 양자화
    
    • 별도 FP 처리 없음
    
    • 추가 메타데이터 거의 없음
    
    ### 3.3 추론 시
    
    • GEMM/MVM 연산은 
    
    • pair 구조는 
    
    • 런타임은 pair 인지 불필요
    
    ## 4. 왜 “Hardware-friendly” 인가
    
    이 논문의 가장 중요한 포인트는 **HW 관점에서의 단순성**이다.
    
    ### ✅ 제거된 복잡성
    
    | 기존 방식 | 문제 |
    | --- | --- |
    | Mixed precision | FP + INT datapath 필요 |
    | Channel-wise scale | 스케일 메모리/로드 증가 |
    | Outlier bypass | 분기, 특수 경로 필요 |
    
    ### ✅ OliVe의 특징
    
    • 정적 양자화 (static quant)
    
    • Uniform integer GEMM
    
    • 스케일 granularity 증가 없음
    
    • 추가 연산 없음
    
    👉 **NPU 관점**
    
    • 기존 INT8/INT4 MAC array 그대로 사용
    
    • 스케일은 기존 per-tensor / per-block 수준 유지
    
    • DMA, SPM, PE 모두 수정 최소
    
    ## 5. 정확도와 성능
    
    논문 결과 요약:
    
    • LLaMA 계열, OPT 계열
    
    • INT8 / INT6 / INT4에서
    
    • GPTQ / AWQ 대비 
    
    • 특히
    
    • Activation outlier에 민감한 layer
    
    중요한 점:
    
    > 
    
    정확도를 유지하면서도 **추론 latency와 energy는 pure INT 연산 수준**
    
    ## 6. 기존 기법과의 구조적 차이
    
    | 기법 | 핵심 전략 | HW 복잡도 |
    | --- | --- | --- |
    | GPTQ | Hessian 기반 보정 | 높음 (오프라인) |
    | AWQ | activation-aware scaling | 중 |
    | SmoothQuant | weight/act 재스케일 | 중 |
    | **OliVe** | **Outlier–Victim pairing** | **매우 낮음** |
    
    👉 OliVe는 **“양자화 알고리즘이 아니라 데이터 재배치 문제”**에 가깝다.
    
    ## 7. NPU / 시스템 아키텍트 관점 시사점 (중요)
    
    이 논문은 특히 **당신이 고민 중인 구조**와 잘 맞는다.
    
    ### 🔹 1) Tile 기반 NPU
    
    • Pairing은 
    
    • Tile/SRAM 배치 시
    
    • pair를 같은 tile에 배치
    
    • 실행 시 추가 제어 없음
    
    ### 🔹 2) VCIX / Custom ISA
    
    • 별도 opcode 불필요
    
    • 일반 INT GEMM lowering 그대로
    
    ### 🔹 3) 스트리밍 / 데이터플로우 아키텍처
    
    • outlier bypass 경로 제거 가능
    
    • 파이프라인 균질화
    
    ### 🔹 4) Compression + Quantization 결합
    
    • Pair 구조는
    
    • entropy coding (Golomb/Huffman)과도 친화적
    
    • victim weight가 통계적 완충 역할
    
    ## 8. 한 단계 더 나아간 아이디어 (확장 제안)
    
    연구적으로 매우 좋은 확장 포인트:
    
    1. Pair → Group (1 outlier + N victim)
    
    1. Pair 기반 
    
    1. Pair 단위 entropy-aware encoding
    
    1. Pair-aware tiling (NPU SRAM bank conflict 최소화)
    
    ## ✍️ 결론
    
    > 
    
    **OliVe는 “정확도를 위한 알고리즘”이라기보다“하드웨어가 싫어하는 예외(outlier)를 없애는 설계 철학”에 가깝다.**
    
    • NPU/SoC 설계자에게:
    
    • 가장 덜 귀찮은 양자화
    
    • 컴파일러 관점:
    
- **ChatGPT: 아래는 **OliVe: Acceler... (Part 2/2)**
    
    • IR/Lowering에 부담 거의 없음
    
    • 시스템 관점:
    
    • 정적·균질·예측 가능
    
    원하시면 다음도 이어서 정리해줄 수 있다:
    
    • OliVe를 
    
    • KV cache
    
    • Tile 기반 NPU IR
    
    • 엔트로피 코딩 + OliVe
    
    어디까지 확장할지 말씀해줘.
    
- **You: 정확하게 이해가 안돼. 예시와 수식을...**
    
    정확하게 이해가 안돼. 예시와 수식을 이용해서 좀 더 알기 쉽게 자세하게 알려줘.
    
- **ChatGPT: 좋아. “Outlier–Victim ...**
    
    좋아. “Outlier–Victim Pair(OVP)”가 헷갈리는 이유는, 이게 **“양자화 스케일을 잘 잡는 테크닉”**이라기보다 **“메모리 포맷/인코딩을 바꿔서 outlier를 같은 스트림 안에 끼워 넣는 기법”**이기 때문이야.
    
    논문에서 말하는 핵심은 딱 이거야:
    
    > 
    
    **Outlier는 중요하지만, 그 옆(같은 메모리 pair에 묶일) normal 값은 중요하지 않은 경우가 많다.그래서 그 normal 값을 0으로 “희생(victim)”시키고, 그 자리를 outlier를 더 잘 표현하는 데 쓴다.**  cite turn1view0
    
    아래는 수식+숫자 예시로 완전히 풀어쓴 설명이야.
    
    # 1) 왜 Outlier가 “한 개만 있어도” 4-bit 양자화를 망가뜨리나 (기본 수식)
    
    가장 단순한 **대칭 uniform quantization**을 생각해보자.
    
    • b-bit signed quant의 정수 범위:
    
    • 스케일 (s) (per-tensor 단순 버전):
    [
    s = \frac{\max |w|}{Q}
    ]
    
    • 양자화/복원:
    [
    q=	ext{round}(w/s),\quad \hat{w}=s\cdot q
    ]
    
    ## 숫자 예시: outlier 하나가 스케일을 폭발시키는 케이스
    
    어떤 블록(또는 텐서)에 값들이 이렇다고 하자:
    
    [
    [1.5,; 2.6,; -5.2,; -98]
    ]
    
    4-bit라면 (Q=7), (\max|w|=98) 이므로
    
    [
    s=\frac{98}{7}=14
    ]
    
    이제 작은 값들을 양자화하면
    
    • (1.5/14=0.107 \Rightarrow q=0 \Rightarrow \hat{w}=0)
    
    • (2.6/14=0.186 \Rightarrow q=0 \Rightarrow \hat{w}=0)
    
    • (-5.2/14=-0.371 \Rightarrow q=0 \Rightarrow \hat{w}=0)
    
    즉, **outlier 하나(-98) 때문에 나머지가 전부 0이 되어버림**.이게 Transformer에서 4-bit가 어려운 이유의 “정석적인” 그림이야.  cite turn1view0
    
    # 2) 기존(outlier-aware sparsity) 방식은 왜 HW가 싫어하나
    
    기존 방법(GOBO류)은 outlier를 따로 떼서
    
    • dense normal matrix (4-bit)
    
    • sparse outlier list (좌표+값)
    
    를 따로 저장/처리하는데, 이러면
    
    • 좌표 리스트 decode/merge 로직 필요
    
    • global coordination(어디에 outlier가 있는지) 필요
    
    • 메모리 access가 
    
    논문도 이걸 “하드웨어 비효율”의 근본으로 짚어.   0 turn1view0
    
    # 3) OliVe의 OVP 핵심: “outlier 옆 normal 하나를 0으로 만들고, 그 자리에 outlier를 넣는다”
    
    논문 표현 그대로 쓰면:
    
    • outlier 옆의 normal은 
    
    • victim을 
    
    여기서 “옆”은 **메모리/포맷 상에서 같은 pair로 묶이는 이웃**을 의미해.
    
    # 4) OVP를 “바이트(8-bit) 정렬 pair 포맷”으로 이해하면 제일 쉽다
    
    가장 직관적인 모델(그림 1b의 핵심)을 말로 풀면 이거야:  cite turn1view0
    
    • 가중치를 
    
    • 정상(normal-normal) pair는:
    
    • (4-bit, 4-bit) → 
    
    • outlier가 섞인(outlier-victim) pair는:
    
    • victim을 0으로 만들고,
    
    • 그 1바이트를 “outlier를 위한 더 좋은 표현”에 사용
    
    • 그래도 1바이트 정렬 유지(memory-aligned)
    
    즉, outlier를 위해 “추가 메모리”나 “좌표 리스트”를 만들지 않고,
    **같은 바이트 안에서 해결**한다는 게 OVP의 본질이야.  cite turn1view0
    
    # 5) 숫자 예시로 OVP 동작을 단계별로 보자
    
    아까 벡터를 다시 쓰자:
    
    [
    [1.5,; 2.6,; -5.2,; -98]
    ]
    
    이를 (1.5, 2.6), (-5.2, -98) 처럼 **pair로 묶는다고 하자.**
    
    ## (A) normal-normal pair: (1.5, 2.6)
    
    outlier가 없으니 평범하게 4-bit로 가면 된다.
    
    이 pair의 “normal 스케일”은 outlier 없이 잡을 수 있어서 예를 들어
    
    [
    s_n \approx \frac{\max(|1.5|,|2.6|)}{7}=\frac{2.6}{7}=0.371
    ]
    
    • (q_{1}=round(1.5/0.371)=round(4.04)=4 \Rightarrow \hat{w}=1.48)
    
    • (q_{2}=round(2.6/0.371)=round(7.01)=7 \Rightarrow \hat{w}=2.60)
    
    → 정상 값들은 4-bit에서도 꽤 잘 살아남지.
    
    ## (B) outlier-victim pair: (-5.2, -98)
    
    여기서 OVP가 하는 일은:
    
    1. outlier 판정: (|-98| > T) 이면 outlier
    
    1. 같은 pair 안의 다른 값 (-5.2)를 
    
    1. victim을 
    
    1. outlier (-98)는 
    
    중요 포인트는,
    
    • 기존 방식은 outlier를 저장하려면 “(좌표, 값)”을 별도로 둬야 했는데,
    
    • OVP는 
    
    # 6) “victim을 0으로 만들어도 왜 정확도가 유지되나?” (수식 관점)
    
    Transformer의 한 출력(한 뉴런)을 단순화하면:
    
    [
    y=\sum_{i} w_i x_i
    ]
    
    여기서 outlier 하나 (w_o)가 매우 크고,
    그 옆의 victim (w_v)는 작다고 하자.
    
    OVP는 (w_v)를 0으로 바꾸는 것이므로 출력 변화는
    
    [
    \Delta y = (w_v - 0),x_v = w_v x_v
    ]
    
    반면 outlier를 제대로 표현하지 못하면 (예: 4-bit에서 saturate/큰 에러)
    
    [
    \Delta y \approx (w_o - \hat{w}_o),x_o
    ]
    
    인데 (|w_o|\gg |w_v|)이면 일반적으로
    [
    |\Delta y_{	ext{outlier error}}| \gg |\Delta y_{	ext{victim prune}}|
    ]
    
    즉, **작은 값 하나를 버리고(outlier 옆 normal), 큰 값을 제대로 살리는 게 전체 오차를 줄인다**는 구조야.논문도 “outlier는 중요하지만 adjacent normal은 중요하지 않다”는 실험적 관찰을 핵심 인사이트로 둔다.  cite turn1view0 turn0search0
    
    # 7) OVP가 “하드웨어 친화적”인 이유를 한 문장으로
    
    > 
    
    outlier를 따로 빼서 sparse 리스트로 처리하지 않고,**같은 메모리 정렬 단위(예: 1바이트 pair) 안에서 local decode로 끝내서**,기존 텐서코어/시스톨릭의 dense dataflow를 깨지 않는다.  cite turn1view0 turn0search0
    
    # 8) 이해 확인용: OVP를 당신 NPU 관점으로 번역하면
    
    • 기존(outlier-sparse)은 “IR/스케줄러가 outlier 스트림을 따로 orchestrate” 해야 함
    
    • OVP는 “
    
    즉, ISA/컴파일러 입장에선 **특수 케이스를 줄이고**,메모리 시스템 입장에선 **정렬/연속 access를 유지**하는 구조로 읽으면 된다.
    
    원하면 다음으로 더 직관적인 “한 장짜리” 예시도 만들어줄게:
    
    1. 4×4 weight 블록
    
    1. pair 재배치(어떤 값을 victim으로 고르는지)
    
    1. 바이트 단위 인코딩이 어떻게 되는지(비트 필드 느낌으로)
    
    1. 그 상태로 GEMM에서 어떻게 읽혀서 PE로 들어가는지
    
    이 중에서 특히 헷갈리는 지점이 “victim 선택(어떤 normal을 희생시키나)”인지, 아니면 “바이트 안에 outlier를 어떻게 넣나(포맷)”인지 어느 쪽이야?
    
- **You: Victim처리한후 같은 1바이트 표...**
    
    Victim처리한후 같은 1바이트 표현에서 이것과 b인 outlier-victim인지 어떻게 구별할수 있어?
    
- **ChatGPT: 아주 핵심을 정확히 찔렀어. **“같... (Part 1/2)**
    
    아주 핵심을 정확히 찔렀어.**“같은 1바이트를 쓰는데, 이게 (normal, normal)인지 (outlier, victim)인지 어떻게 구별하느냐?”**→ 이걸 이해하면 OliVe가 완전히 정리된다.
    
    결론부터 말하면:
    
    > 
    
    **OliVe는 ‘같은 1바이트’가 아니라👉 ‘같은 *메모리 정렬 단위*’를 유지하면서👉 내부 비트 패턴 또는 별도의 *경량 메타데이터*로 구별한다.**
    
    아래를 차근차근 보자.
    
    ## 1️⃣ 오해부터 바로잡기: “완전히 동일한 8비트 포맷”은 아니다
    
    많이들 이렇게 오해한다:
    
    > 
    
    “normal-normal도 1바이트, outlier-victim도 1바이트 →그럼 비트 패턴만 보고 어떻게 구별하지?”
    
    ❌ **아니다.**
    
    OliVe의 핵심은:
    
    • 메모리 접근 단위(alignment, bandwidth)는 동일
    
    • 비트 해석 방식은 다를 수 있음
    
    즉,
    
    • 같은 1B를 읽지만
    
    • 디코딩 규칙이 다르다
    
    ## 2️⃣ 구별 방법은 크게 3가지 가능성 (논문 + HW 관점)
    
    논문에서 직접/간접적으로 허용하는 방식은 아래 3가지 범주로 정리된다.
    
    ## ✅ 방법 1. Pair-level metadata (가장 현실적, 추천)
    
    ### 개념
    
    • weight를 
    
    • 각 pair에 대해 
    
    ```vbnet
    [ Pair0 ][ Pair1 ][ Pair2 ] ...
       |        |        |
      flag0    flag1    flag2
    ```
    
    • flag = 0
    
    • flag = 1
    
    ### 메모리 관점
    
    • flag는
    
    • 별도의 
    
    • 또는 
    
    • 오버헤드:
    
    • 1 bit / 2 weights = 
    
    • 기존 sparse outlier list 대비 
    
    ### 디코딩 흐름 (NPU)
    
    ```vbnet
    load 1B (pair payload)
    load 1bit (flag)
    
    if flag == 0:
        decode as (int4, int4)
    else:
        decode as (outlier_representation)
    ```
    
    👉 **중요**
    
    • 분기 있음 ❌ → 걱정할 필요 없음
    
    • 이 분기는 
    
    • MAC array에는 이미 정규화된 값만 들어감
    
    📌 **하드웨어적으로 가장 깔끔한 방식**
    → 당신이 만드는 NPU에도 제일 잘 맞음
    
    ## ✅ 방법 2. Reserved bit-pattern (비트 패턴 자체로 구별)
    
    ### 개념
    
    • normal 4-bit 값에서 
    
    • 예:
    
    • 4-bit signed: 
    
    • -8 (1000)
    
    ### 예시
    
    ```vbnet
    [ 1000 | xxxx ]  → outlier-victim pair
    [ aaaa | bbbb ]  → normal-normal
    ```
    
    • upper nibble = 
    
    • lower nibble = outlier payload 일부
    
    ### 장점
    
    • flag 메모리 불필요
    
    • payload-only decode
    
    ### 단점 (중요)
    
    • 표현 범위 감소
    
    • 양자화 자유도 감소
    
    • 논문에서는 
    
    📌 **ASIC에선 가능**,📌 **범용성/정확도 측면에선 metadata 방식이 더 안전**
    
    ## ✅ 방법 3. Block-wise mode (논문이 암시한 방식)
    
    ### 개념
    
    • 작은 block (예: 32 or 64 weights) 단위로
    
    • “이 블록에는 outlier pair가 있음/없음”
    
    • block header에 mode 저장
    
    ```vbnet
    [ block header: has_outlier = 1 ]
    [ pair0 ][ pair1 ][ pair2 ] ...
    ```
    
    • outlier 없는 블록 → 전부 normal-normal
    
    • outlier 있는 블록 → 일부 pair만 outlier-victim
    
    ### 특징
    
    • flag density 감소
    
    • decoding control 단순화
    
    📌 **Tile-based NPU와 매우 궁합이 좋음**
    
    ## 3️⃣ 중요한 점: victim은 “표현되지 않으므로” ambiguity가 없음
    
    핵심 논리 하나 더.
    
    > 
    
    “victim이 0이면,그 4-bit 자리에 normal 값이 들어갈 이유 자체가 없다”
    
    즉,
    
    • outlier-victim pair에서는
    
    • victim 값은 의미적으로 항상 0
    
    • 따라서
    
    • “이게 진짜 normal 0이냐, victim 0이냐?”는
    
    • 계산 결과상 구분할 필요가 없음
    
    구별해야 하는 건 **디코딩 방식**이지,
    **수치 의미는 이미 분리돼 있다.**
    
    ## 4️⃣ 수식으로 다시 연결해보자 (정확성 관점)
    
    pair (p=(w_1, w_2)) 에 대해:
    
    • normal-normal:
    
    [
    \hat{w}_1 = s \cdot q_1,\quad \hat{w}_2 = s \cdot q_2
    ]
    
    • outlier-victim:
    
    [
    \hat{w}_v = 0,\quad \hat{w}*o = f*{	ext{outlier}}(	ext{payload})
    ]
    
    여기서 중요한 건:
    
    • 같은 pair라도 reconstruction 함수 (f)가 다르다
    
    • 그 선택 신호가 바로 
    
    ## 5️⃣ 당신이 NPU 설계자라면 이렇게 구현하는 게 정답
    
    현실적인 추천 아키텍처:
    
    ```vbnet
    DMA → SRAM
            |
            v
       Pair Decoder
       ├─ load payload (1B)
       ├─ load flag (1bit)
       ├─ if normal:
       │     unpack int4,int4
       └─ if outlier:
             reconstruct outlier
             set victim=0
            |
            v
          MAC Array (완전 dense)
    ```
    
    • MAC array는 
    
    • 복잡성은 
    
    ## 6️⃣ 한 문장 요약 (가장 중요)
    
    > 
    
    **OliVe는 “같은 1바이트를 쓰는 것”이 목적이 아니라,“같은 *메모리 정렬·로드·파이프라인*을 유지하면서디코딩 규칙만 바꾸는 것”이 핵심이다.**
    
    다음으로 가장 많이 나오는 질문은 보통 이거야:
    
    • ❓ 
    
    • ❓ 
    
- **ChatGPT: 아주 핵심을 정확히 찔렀어. **“같... (Part 2/2)**
    
    • ❓ 
    
    어디까지 더 파고들까?👉 **비트 포맷**, **pair 선택 알고리즘**, **NPU 디코더 RTL 관점** 중 하나 골라줘.
    
- **You: Pair가 아웃라이어-아웃라이어이면?**
    
    Pair가 아웃라이어-아웃라이어이면?
    
- **ChatGPT: 이 질문은 **OliVe를 제대로 이...**
    
    이 질문은 **OliVe를 제대로 이해했을 때만 나올 수 있는 핵심 반례 질문**이야.결론부터 명확히 말할게.
    
    ## 결론 한 줄
    
    > 
    
    **OliVe에서는 “outlier–outlier pair”가 되지 않도록 설계한다.
    
    만약 발생한다면, 그 pair는 OVP 대상에서 제외되어 다른 처리 경로로 간다.**
    
    즉, **outlier–outlier는 허용 상태가 아니라 “회피해야 할 상태”**다.
    
    아래에서 **왜 그런지 → 어떻게 회피하는지 → 그래도 발생하면 어떻게 하는지**를 순서대로 설명할게.
    
    ## 1️⃣ 왜 outlier–outlier pair는 OVP의 전제가 아닌가
    
    OliVe의 핵심 가정은 다음이다.
    
    ### 🔹 핵심 가정 (논문의 암묵적 전제)
    
    • outlier는 
    
    • outlier 하나가 
    
    • normal 하나(victim)를 0으로 만들어도 정확도 손실은 미미하다
    
    이 가정이 성립하려면 반드시:
    
    > 
    
    **pair 안에는 “희생 가능한 값(victim)”이 최소 1개 있어야 한다**
    
    즉,
    
    | pair 형태 | 가능 여부 | 이유 |
    | --- | --- | --- |
    | normal–normal | ✅ | 일반 양자화 |
    | outlier–normal | ✅ | normal → victim |
    | **outlier–outlier** | ❌ | 희생할 대상이 없음 |
    
    outlier–outlier에서 하나를 victim으로 만들면→ **중요한 값 하나를 그냥 날리는 것**이 되기 때문에OVP의 수식적/통계적 근거가 붕괴된다.
    
    ## 2️⃣ 그럼 실제로 outlier–outlier pair가 생길 수 있나?
    
    ### 이론적으로는 가능
    
    예:
    
    • 같은 row/column에 큰 weight가 몰린 경우
    
    • attention head 일부에서 분산이 비정상적으로 큰 경우
    
    ### 하지만 실험적으로는:
    
    • LLM weight 분포에서
    
    • outlier 밀도는 매우 낮고
    
    • outlier끼리 인접할 확률은 극히 낮음
    
    👉 그래서 논문에서는**pair 재배치를 통해 outlier–outlier를 회피할 수 있다고 본다.**
    
    ## 3️⃣ OVP에서 실제로 사용하는 해결 전략 (중요)
    
    ### ✅ 전략 1. Pair 재배치 (re-pairing) – 기본 전략
    
    pair는 **물리적 인접(weight index)** 이 아니라**논리적으로 재구성**된다.
    
    예를 들어 원래 순서가:
    
    ```
    [w0, w1, w2, w3, w4, w5]
            ↑      ↑
         outlier  outlier
    ```
    
    이면, 그대로 (w2, w3), (w4, w5)로 묶지 않고:
    
    ```
    (outlier w2, normal w1)
    (outlier w4, normal w5)
    ```
    
    처럼 **outlier를 서로 다른 pair로 흩뿌린다**.
    
    📌 중요:
    
    • 이 재배치는 
    
    • GEMM 수학적 의미는 그대로 유지
    
    • 단지 
    
    👉 당신이 관심 있는 **NPU tile / SRAM layout 최적화와 완전히 같은 문제**다.
    
    ### ✅ 전략 2. Outlier–outlier pair는 “normal quantization에서 제외”
    
    재배치로도 안 되는 경우 (아주 드묾):
    
    • 해당 pair는
    
    • OVP 미적용
    
    • 일반적인 outlier 처리 경로 사용
    
    예:
    
    • higher-bit quant (INT8)
    
    • block-wise scaling
    
    • 또는 소수의 sparse outlier list
    
    📌 핵심은:
    
    > 
    
    **OVP는 “모든 weight를 커버해야 하는 의무”가 없다**outlier–outlier는 전체 weight 중 극소수이기 때문
    
    ### ✅ 전략 3. Pair size 확장 (논문 외 확장 아이디어, 하지만 매우 자연스러움)
    
    논문의 pair는 개념적 단위이고, 실제 구현에선:
    
    • 1 outlier + 
    
    • 즉, 
    
    이렇게 되면:
    
    • outlier–outlier 발생 확률 → 거의 0
    
    • victim 후보 pool 증가
    
    이건 당신이 이전에 말한
    
    > 
    
    “multiple constant multiplication / 구조적 묶음”
    
    이랑 정확히 같은 방향이다.
    
    ## 4️⃣ 수식으로 보면 왜 outlier–outlier는 안 되는지
    
    출력 변화량을 다시 쓰자:
    
    [
    y=\sum_i w_i x_i
    ]
    
    outlier–outlier pair에서 하나를 victim으로 만들면:
    
    [
    \Delta y = w_{o2} \cdot x_{o2}
    ]
    
    여기서 (w_{o2})는 **outlier**이므로:
    
    [
    |w_{o2}| \gg |w_v|
    ]
    
    즉,
    
    • OVP가 의도한
    
    👉 이게 이론적으로 OVP가 outlier–outlier를 배제하는 이유다.
    
    ## 5️⃣ NPU / 컴파일러 관점에서의 “정답 구현”
    
    당신이 만든다면 이렇게 해야 한다:
    
    ### Compile-time
    
    1. outlier 탐지
    
    1. outlier 리스트 생성
    
    1. outlier를 서로 다른 group(pair)에 분산
    
    1. normal을 victim 후보로 매칭
    
    1. 실패한 outlier–outlier group만 별도 태깅
    
    ### Runtime (NPU)
    
    • decoder는
    
    • normal pair
    
    • outlier-victim pair
    
    • (아주 드물게) fallback pair
    
    만 처리
    
    MAC array는 **끝까지 oblivious**.
    
    ## 6️⃣ 핵심 요약 (이 문장만 기억해도 됨)
    
    > 
    
    **OliVe에서 pair는 “물리적 이웃”이 아니라 “논리적 매칭”이다.
    
    outlier–outlier가 생기지 않도록 compile-time에 재배치하며,만약 생긴다면 OVP의 적용 대상이 아니다.**
    
    다음으로 자연스럽게 이어지는 질문은 이거야:
    
    • ❓ 
    
    • ❓ 
    
    • ❓ 
    
    이 중에서 **NPU 구조랑 가장 맞닿은 질문**부터 파볼까?