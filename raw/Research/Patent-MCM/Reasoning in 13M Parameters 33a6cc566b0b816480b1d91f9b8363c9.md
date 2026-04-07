# Reasoning in 13M Parameters

ChatTime: 2026년 3월 10일 22:46 (GMT+9)
Created time: 2026년 4월 6일 오후 10:59
ProjectName: [Patent-Paper] MCM
URL: https://chatgpt.com/c/69b0200c-f4c8-83a2-94a9-f9243ec7b6ce

- **You: learning to reason i...**
    
    learning to reason in 13 parameters
    
- **ChatGPT: “**Learning to Reaso...**
    
    “**Learning to Reason in 13 Parameters**”는 **매우 작은 모델(약 13M 파라미터)**로도 **추론 능력(reasoning ability)**을 학습할 수 있다는 연구 흐름을 의미합니다. 일반적으로 LLM에서 추론 능력은 **수십억~수천억 파라미터 모델에서만 가능**하다고 여겨졌지만, 최근 연구들은 **적절한 데이터, 학습 방식, 아키텍처 설계**를 사용하면 **매우 작은 모델에서도 reasoning capability를 만들 수 있다**는 점을 보여줍니다.
    
    아래를 구조적으로 설명하겠습니다.
    
    # 1. 문제 배경: 왜 “13M parameters”가 중요한가
    
    기존 관점
    
    | 모델 | 파라미터 |
    | --- | --- |
    | GPT-3 | 175B |
    | LLaMA | 7B ~ 70B |
    | reasoning 모델 (o1 등) | 수십~수백B |
    
    일반적인 인식:
    
    > 
    
    **Reasoning = huge model**
    
    그러나 최근 연구 질문:
    
    > 
    
    **“Reasoning이 정말 large parameter scale 때문인가?”**
    
    즉
    
    **Reasoning = 모델 크기?**아니면
    
    **Reasoning = training method?**
    
    # 2. 핵심 아이디어
    
    “Learning to Reason in 13M Parameters” 연구들의 핵심은 다음입니다.
    
    ### 핵심 가설
    
    작은 모델도 아래 조건을 만족하면 reasoning을 학습할 수 있다.
    
    1️⃣ **synthetic reasoning dataset**2️⃣ **chain-of-thought supervision**3️⃣ **curriculum learning**4️⃣ **iterative reasoning tasks**
    
    # 3. 실험 구조
    
    일반적으로 이런 구조로 실험합니다.
    
    ```
    Synthetic reasoning data generator
            ↓
    Chain-of-thought dataset
            ↓
    Small transformer
    (13M parameters)
            ↓
    Train
            ↓
    Evaluate on reasoning benchmarks
    ```
    
    모델은 대략 다음과 같습니다.
    
    ```
    Transformer
    layers: 6~12
    hidden dim: 256~512
    params: ~13M
    ```
    
    # 4. reasoning dataset 구성
    
    대표적인 reasoning task:
    
    ### Arithmetic
    
    ```
    23 + 48 = ?
    ```
    
    CoT
    
    ```
    23 + 48
    = (20 + 40) + (3 + 8)
    = 60 + 11
    = 71
    ```
    
    ### Symbol manipulation
    
    ```
    reverse the string "ABCDEF"
    ```
    
    ### Logic reasoning
    
    ```
    If A>B and B>C, is A>C?
    ```
    
    ### algorithmic tasks
    
    ```
    sort numbers
    count parentheses
    copy sequence
    ```
    
    이런 task는 **algorithmic reasoning**이라고 합니다
    
- **You: https://arxiv.org/ht...**
    
    [https://arxiv.org/html/2602.04118v1](https://arxiv.org/html/2602.04118v1)
    
- **ChatGPT: 논문: **“Learning to R... (Part 1/2)**
    
    논문: **“Learning to Reason in 13 Parameters”**(2026, Meta FAIR + Cornell + CMU)→ [https://arxiv.org/abs/2602.04118](https://arxiv.org/abs/2602.04118)
    
    이 논문은 LLM 연구에서 상당히 중요한 메시지를 던집니다.
    
    > 
    
    **추론 능력을 학습하는 데 수십억 파라미터를 업데이트할 필요가 없다.단지 13개의 파라미터만 업데이트해도 reasoning 능력을 크게 개선할 수 있다.**
    
    핵심 내용을 **아키텍처 관점 + 학습 알고리즘 관점 + 시스템 설계 관점**에서 정리해 보겠습니다.
    
    # 1. 연구의 핵심 질문
    
    기존 LLM fine-tuning 구조:
    
    ```
    Base model (7B ~ 70B)
            ↓
    LoRA / full finetune
            ↓
    millions of parameters updated
    ```
    
    문제
    
    • LoRA도 
    
    • reasoning training 비용이 큼
    
    연구 질문
    
    ```
    Reasoning capability
    = 정말 많은 파라미터가 필요한가?
    ```
    
    논문의 답:
    
    > 
    
    **아니다. 극단적으로 줄여도 된다.**
    
    # 2. 핵심 아이디어: TinyLoRA
    
    논문에서 제안한 방법
    
    **TinyLoRA**
    
    기존 LoRA
    
    ```
    W' = W + A B
    ```
    
    • A ∈ R(d × r)
    
    • B ∈ R(r × d)
    
    • r = rank
    
    문제
    
    ```
    parameter ≈ 2 * d * r
    ```
    
    예
    
    ```
    d = 4096
    r = 1
    
    → 8192 parameters
    ```
    
    논문 질문
    
    > 
    
    rank=1 LoRA도 필요할까?
    
    그래서 제안
    
    ```
    TinyLoRA
    ```
    
    아이디어
    
    ```
    W' = W + α * U
    ```
    
    여기서
    
    ```
    α = scalar parameter
    U = fixed random matrix
    ```
    
    즉
    
    ```
    trainable parameter = 1
    ```
    
    논문 실험
    
    ```
    13 parameters
    ```
    
    # 3. 결과 (핵심)
    
    Base model
    
    ```
    Qwen2.5-8B
    ```
    
    trainable parameters
    
    ```
    13 parameters
    = 26 bytes (bf16)
    ```
    
    결과
    
    GSM8K
    
    ```
    accuracy ≈ 91%
    ```
    
    또한 reasoning benchmark에서도 성능 유지
    
    • AIME
    
    • AMC
    
    • MATH500
    
    논문 결과
    
    > 
    
    **1000x fewer parameters로 90% 성능 유지**   0 turn0search12
    
    # 4. 중요한 발견
    
    논문에서 매우 중요한 관찰
    
    ### RL이 핵심
    
    SFT (supervised fine-tuning)
    
    ```
    requires 100~1000x more parameters
    ```
    
    하지만
    
    ```
    Reinforcement Learning
    ```
    
    을 사용하면
    
    ```
    13 parameters
    ```
    
    로도 reasoning 학습 가능   0
    
    즉
    
    ```
    Reasoning = parameter scale
    ```
    
    이 아니라
    
    ```
    Reasoning = training signal
    ```
    
    일 가능성.
    
    # 5. 왜 이런 현상이 발생할까
    
    논문이 시사하는 메커니즘
    
    Base model에는 이미
    
    ```
    latent reasoning capability
    ```
    
    가 존재
    
    ```
    pretraining
    → reasoning circuits 존재
    ```
    
    하지만
    
    ```
    activation되지 않음
    ```
    
    TinyLoRA 역할
    
    ```
    small steering signal
    ```
    
    즉
    
    ```
    model behavior steering
    ```
    
    # 6. 해석 (LLM 내부 관점)
    
    LLM 내부 구조
    
    ```
    W = pretrained knowledge
    ```
    
    TinyLoRA
    
    ```
    ΔW = reasoning direction
    ```
    
    즉
    
    ```
    parameter update
    ≠ knowledge learning
    ```
    
    오히려
    
    ```
    behavior steering
    ```
    
    # 7. 시스템 아키텍처 관점 (매우 중요)
    
    이 논문은 **AI hardware / system architecture 관점에서도 의미가 큽니다.**
    
    ### 기존
    
    fine-tuning
    
    ```
    millions parameters
    memory update
    checkpoint
    ```
    
    ### TinyLoRA
    
    ```
    13 parameters
    ```
    
    즉
    
- **ChatGPT: 논문: **“Learning to R... (Part 2/2)**
    
    ```
    26 bytes
    ```
    
    가능한 구조
    
    ```
    base LLM (frozen)
    +
    tiny reasoning adapter
    ```
    
    이건 다음과 같은 시스템을 가능하게 합니다.
    
    ### 1️⃣ ultra-cheap model adaptation
    
    예
    
    ```
    device personalization
    ```
    
    업데이트
    
    ```
    26 bytes
    ```
    
    ### 2️⃣ reasoning plug-in
    
    ```
    base LLM
    + reasoning adapter
    + coding adapter
    + math adapter
    ```
    
    ### 3️⃣ on-device RL
    
    Edge AI에서도 가능
    
    # 8. hardware 관점에서의 흥미로운 포인트
    
    이 논문은 **NPU 설계 관점에서도 interesting**
    
    기존
    
    ```
    fine-tune = memory heavy
    ```
    
    TinyLoRA
    
    ```
    parameter = register 수준
    ```
    
    즉
    
    ```
    reasoning = weight modification
    ```
    
    이 아니라
    
    ```
    activation steering
    ```
    
    가능.
    
    # 9. 앞으로의 연구 방향
    
    이 논문이 던지는 질문
    
    ### 1️⃣ reasoning circuit location
    
    ```
    which layer?
    which attention head?
    ```
    
    ### 2️⃣ minimal parameter learning
    
    가능성
    
    ```
    1 parameter
    ```
    
    ### 3️⃣ neuromodulation style training
    
    ```
    base network
    + tiny control signal
    ```
    
    생물학적 neural system과도 유사.
    
    # 10. 중요한 insight (한 문장 요약)
    
    이 논문 메시지
    
    ```
    Reasoning is not stored in parameters.
    It is activated by them.
    ```
    
    즉
    
    ```
    LLM = reasoning capable system
    ```
    
    하지만
    
    ```
    training = steering
    ```
    
    # 11. 당신의 연구(NPU / AI accelerator)와 연결되는 부분
    
    특히 중요한 부분
    
    **Future inference architecture**
    
    ```
    Base model (frozen SRAM)
    +
    tiny adapter memory
    ```
    
    예
    
    ```
    7B model
    + 32 byte adapter
    ```
    
    즉
    
    ```
    reasoning upgrade
    = firmware patch
    ```
    
    이건 **AI hardware deployment 모델을 바꿀 수 있음.**
    
    원하시면 제가 다음도 설명해 드리겠습니다.
    
    1️⃣ **TinyLoRA 수식과 gradient 흐름 완전 해설**2️⃣ **왜 RL이 reasoning을 잘 만드는지 (policy gradient 관점)**3️⃣ **LLM 내부 reasoning circuit 연구 (Anthropic / DeepMind)**4️⃣ **이 논문이 future AI hardware architecture에 주는 의미**
    
    특히 **3번은 NPU 설계 관점에서 매우 흥미로운 이야기입니다.**