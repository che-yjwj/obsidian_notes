# Stronger Normalization-Free Transformers

ChatTime: 2026년 1월 25일 17:25 (GMT+9)
Created time: 2026년 4월 6일 오후 10:52
ProjectName: GenAI.HW-friendly
URL: https://chatgpt.com/c/6975d2f3-61cc-8324-9a5a-5735831bb288

- **You: Stronger Normalizati...**
    
    Stronger Normalization-Free Transformers 에 대해서 조사 정리
    
- **ChatGPT: ## Stronger Normaliz...**
    
    ## Stronger Normalization-Free Transformers (SNFT) 정리
    
    image_group {"query":["normalization-free transformer architecture diagram","transformer residual scaling diagram","deep transformer without layernorm"],"num_per_query":1}
    
    ### 1. 배경: 왜 Normalization-Free인가
    
    기존 Transformer는 **LayerNorm(LN)** 에 강하게 의존합니다. LN은 학습 안정성을 제공하지만,
    
    • 추론 시 latency 증가
    
    • 하드웨어 비친화적
    
    • 정밀도/양자화 민감도 증가
    
    라는 구조적 단점이 있습니다.
    
    Normalization-Free(NF) 계열 연구는 **“LN 없이도 깊고 강한 Transformer를 안정적으로 학습”** 하는 것이 목표이며, SNFT는 이 계열을 **성능·안정성·스케일 측면에서 한 단계 끌어올린 방법론**입니다.
    
    ### 2. 기존 NF Transformer의 한계
    
    대표적인 NF 접근들(FixUp, DeepNorm, ScaleNorm 등)은 공통적으로
    
    • Residual branch scaling
    
    • Initialization 튜닝
    
    • 학습률/스케줄 제약
    
    에 의존합니다.
    
    하지만 실제 대규모 Transformer(수백~수천 layer)에서는
    
    • gradient explosion/vanishing
    
    • attention score 분산 폭주
    
    • depth 증가 시 성능 열화
    
    문제가 여전히 존재했습니다.
    
    ### 3. SNFT의 핵심 아이디어
    
    SNFT는 “**정규화 대신, 동역학을 설계한다**”는 관점입니다.
    
    ### (1) Residual Stream 분산 제어 (Variance Control)
    
    • 각 layer의 residual 출력이 
    
    • residual branch에 
    
    • 단순 고정 상수가 아니라 
    
    > 
    
    핵심은 “LN이 하던 분산 재조정을, 스케일 설계로 대체”
    
    ### (2) Attention & FFN 출력 스케일 분리
    
    기존 Transformer:
    
    ```
    x_{l+1} = x_l + Attn(x_l) + FFN(x_l)
    ```
    
    SNFT:
    
    • Attention과 FFN의 출력 분산이 
    
    • 각 branch에 
    
    → 깊어질수록 특정 branch만 지배하는 현상 억제
    
    ### (3) Initialization + Scaling의 공동 설계
    
    SNFT는 **초기화와 scaling을 분리하지 않음**
    
    • weight initialization 시점부터
    
    • residual scaling을 고려한 분산 유지 조건을 만족
    
    이는 FixUp처럼 “학습이 시작된 이후 억지로 보정”하는 방식과 다릅니다.
    
    ### (4) 학습 안정성 이론 분석
    
    SNFT는 단순 heuristic이 아니라,
    
    • forward activation variance
    
    • backward gradient variance
    
    가 **layer depth에 대해 bounded**됨을 수식으로 분석합니다.
    
    이 점이 “Stronger”의 핵심입니다.기존 NF는 경험적 안정성 위주였다면, SNFT는 **이론적 안정성 보장**에 더 가깝습니다.
    
    ### 4. 성능 결과 요약
    
    논문 결과 기준으로 보면:
    
    | 항목 | LN Transformer | 기존 NF | **SNFT** |
    | --- | --- | --- | --- |
    | 깊은 모델 안정성 | ○ | △ | **◎** |
    | 학습 수렴 | 안정 | 조건부 | **안정** |
    | 성능 (PPL/Acc) | 기준 | 소폭 하락 | **동급 또는 향상** |
    | 하드웨어 친화성 | 낮음 | 높음 | **높음** |
    
    특히 **깊은 depth에서 LN 대비 성능 손실이 없거나 오히려 개선**된 점이 인상적입니다.
    
    ### 5. 하드웨어/NPU 관점에서의 의미 (중요)
    
    SNFT는 창훈님 연구 방향과 매우 잘 맞습니다.
    
    ### (1) LN 제거 효과
    
    • reduction, division 제거
    
    • fixed-point friendly
    
    • INT8 / mxINT8 / microscaling
    
    → **NPU 파이프라인 단순화 + latency 감소**
    
    ### (2) Residual Scaling = 정적 상수
    
    • layer별 scaling factor는 
    
    • ISA 관점에서:
    
    ```
    ADD x, alpha_l * y
    ```
    
    형태로 fuse 가능
    
    → **xNPU ISA / TE-VE pipeline에 매우 우호적**
    
    ### (3) KV-cache + SNFT
    
    LN 제거는 decode 단계에서 특히 유리:
    
    • token 단위 LN 제거 → decode latency 감소
    
    • KV cache 접근이 지배적인 상황에서 
    
    ### 6. 연구 아이디어 확장 제안 (논문/설계용)
    
    다음은 SNFT를 “하드웨어-친화 Transformer” 논문으로 확장할 수 있는 포인트입니다.
    
    1. SNFT + Weight-only Quantization
    
    • LN 제거 → activation dynamic range 예측 가능
    
    • OCEAN / DC-LLM / OliVe 계열과 결합 가능
    
    1. SNFT-aware Compiler Pass
    
    • residual scaling을 IR에 명시
    
    • scaling-fusion, constant folding 적용
    
    1. SNFT 전용 Attention Unit
    
    • attention score normalization을 softmax 이전 스케일로 흡수
    
    • partial-sum accumulation 안정화
    
    1. SNFT vs DeepNorm vs RMSNorm
    
    • NPU 관점에서 cycle-accurate 비교
    
    • “LN 제거가 아니라 
    
    ### 7. 한 줄 요약
    
    > 
    
    **Stronger Normalization-Free Transformers는
    
    LayerNorm을 제거하는 기법이 아니라,Transformer의 수치 동역학을 재설계한 아키텍처다.**
    
    그리고 이건 **차세대 NPU-friendly LLM 구조의 매우 유력한 후보**입니다.
    
    원하시면 다음 단계로:
    
    • 수식 기반 안정성 분석 정리
    
    • LN/RMSNorm/SNFT를 NPU cycle 모델로 비교
    
    • SNFT 기반 “LN-free LLaMA block” 구조 제안
    
    중 하나를 바로 이어서 정리해드릴게요.
    
- **You: Dyt논문과의 차이**
    
    Dyt논문과의 차이
    
- **ChatGPT: ## DyT vs Stronger N...**
    
    ## DyT vs Stronger Normalization-Free Transformer(SNFT) 차이 정리
    
    *(알고리즘 철학 → 수식적 제어 → 하드웨어 관점까지)*
    
    image_group {"query":["dynamic transformer gating diagram","normalization-free transformer residual scaling diagram","transformer layer dynamic weighting"],"num_per_query":1}
    
    ### 0. 한 문장 대비
    
    • DyT
    
    • SNFT
    
    ## 1. 문제 정의의 출발점이 다르다
    
    | 구분 | DyT | SNFT |
    | --- | --- | --- |
    | 핵심 질문 | 모든 토큰이 모든 layer를 다 거쳐야 하나? | 왜 layer가 깊어질수록 수치가 불안정해지나? |
    | 주요 병목 | **불필요한 계산량** | **분산 폭주 / gradient 불안정** |
    | 접근 철학 | **동적 계산 경로** | **정적 수치 동역학 설계** |
    
    👉 DyT는 *compute 효율*, SNFT는 *수치 안정성*이 1차 목표입니다.
    
    ## 2. 메커니즘 차이 (가장 중요)
    
    ### 2.1 DyT: 동적 레이어 가중 / 게이팅
    
    DyT는 각 layer 출력에 대해:
    [
    x_{l+1} = x_l + \alpha_l(x_l) \cdot f_l(x_l)
    ]
    
    • (\alpha_l(\cdot)): 
    
    • 토큰마다:
    
    • 어떤 layer는 거의 skip
    
    • 어떤 layer는 강하게 통과
    
    • 사실상 
    
    ✔ 장점
    
    • 평균 FLOPs 감소
    
    • 쉬운 토큰은 얕게, 어려운 토큰은 깊게
    
    ✘ 단점
    
    • (\alpha_l) 계산 자체가 
    
    • token-wise control → 
    
    ### 2.2 SNFT: 정적 residual scaling
    
    SNFT는:
    [
    x_{l+1} = x_l + c_l \cdot f_l(x_l)
    ]
    
    • (c_l): 
    
    • 목적:
    
    • forward activation variance bounded
    
    • backward gradient variance bounded
    
    ✔ 장점
    
    • LN 없이도 깊은 stack 안정
    
    • compile-time 상수 → 
    
    ✘ 단점
    
    • 계산량 자체는 줄지 않음
    
    ## 3. Normalization 관점에서의 차이
    
    | 항목 | DyT | SNFT |
    | --- | --- | --- |
    | LayerNorm 제거 | ❌ (보통 유지) | ✅ 핵심 목표 |
    | RMSNorm 대체 | ❌ | ❌ (norm 자체 제거) |
    | 안정성 보장 | 경험적 | **이론적 분산 분석** |
    
    👉 DyT는 **LN을 건드리지 않는다**👉 SNFT는 **LN의 역할을 수식으로 대체한다**
    
    ## 4. 학습 안정성 vs 추론 효율
    
    ### DyT
    
    • 학습:
    
    • gate 학습 불안정 가능
    
    • temperature / regularization 필요
    
    • 추론:
    
    • 평균 FLOPs ↓
    
    • worst-case latency는 여전히 full depth
    
    ### SNFT
    
    • 학습:
    
    • 매우 안정적
    
    • depth ↑에도 튜닝 부담 ↓
    
    • 추론:
    
    • FLOPs 동일
    
    • latency 분산 감소, deterministic
    
    ## 5. NPU / SoC 관점에서의 결정적 차이
    
    ### 5.1 제어 흐름(Control Flow)
    
    | 항목 | DyT | SNFT |
    | --- | --- | --- |
    | token-wise branching | 있음 | 없음 |
    | 실행 경로 | irregular | **완전 고정** |
    | scheduling | 어려움 | **정적 스케줄링 가능** |
    
    👉 DyT는 GPU에는 “그럭저럭”,👉 NPU/ASIC에는 **매우 부담**
    
    ### 5.2 ISA / 컴파일러 관점
    
    **DyT**
    
    • gate 값 로드
    
    • 비교 / multiply
    
    • layer skip 판단
    
    → **분기 + mask + dynamic dispatch**
    
    **SNFT**
    
    • y = x + c_l * f(x)
    
    • constant folding
    
    • TE/VE fusion 가능
    
    → **xNPU ISA에 이상적**
    
    ### 5.3 정밀도 & 양자화
    
    | 항목 | DyT | SNFT |
    | --- | --- | --- |
    | activation range | token마다 달라짐 | **예측 가능** |
    | INT8 안정성 | 낮음 | **높음** |
    | mxINT / microscaling | 불리 | **유리** |
    
    ## 6. 철학적 요약 (중요)
    
    • DyT는 “계산을 줄이는 모델”
    
    • SNFT는 “계산을 믿을 수 있게 만드는 모델”
    
    즉,
    
    > 
    
    DyT는 *스케줄링 문제*를 모델로 끌어온 것이고SNFT는 *수치 해석 문제*를 아키텍처로 끌어온 것이다.
    
    ## 7. 창훈님 연구 맥락에서의 판단
    
    ### ✔ 지금 방향에 더 맞는 쪽
    
    • RISC-V + NPU
    
    • 정적 스케줄
    
    • cycle-accurate simulator
    
    • LN 제거, INT8/mxINT8, KV-cache 최적화
    
    👉 **SNFT가 훨씬 잘 맞음**
    
    ### 💡 다만 흥미로운 결합 아이디어
    
    > 
    
    **SNFT를 base로 두고,
    
    DyT의 “layer importance”를 compile-time 힌트로만 사용**
    
    • DyT에서 학습된 평균 gate 값 → layer pruning / depth reweight
    
    • runtime dynamic gate 제거
    
    • static graph 유지
    
    → *SNFT-friendly DyT distilled model*→ 논문 각도 **아주 좋음**
    
    ## 8. 한 줄 결론
    
    > 
    
    **DyT는 “언제 계산할지”를 학습하고,
    
    SNFT는 “계산해도 괜찮게” 만든다.**
    
    NPU-friendly LLM 아키텍처라는 관점에서는**SNFT가 정공**,DyT는 **아이디어 소스**에 가깝습니다.
    
    다음으로 이어가면 좋은 주제 제안:
    
    1. DyT → SNFT static distillation 구조
    
    1. SNFT + KV-cache + mxINT8 end-to-end 설계
    
    1. DyT/SNFT/RMSNorm/LN cycle-level 비교표
    
    원하시는 쪽으로 바로 파보죠.