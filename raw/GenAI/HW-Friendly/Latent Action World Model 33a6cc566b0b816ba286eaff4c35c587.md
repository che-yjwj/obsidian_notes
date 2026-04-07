# Latent Action World Model

ChatTime: 2026년 1월 15일 15:39 (GMT+9)
Created time: 2026년 4월 6일 오후 11:09
ProjectName: WorldModel
URL: https://chatgpt.com/c/69688b58-5e38-832e-92c2-11802b6fd861

- **You: Learning Latent Acti...**
    
    Learning Latent Action World Models in the Wild
    
- **ChatGPT: 아래에서는 **「Learning La...**
    
    아래에서는 **「Learning Latent Action World Models in the Wild」**의 핵심 아이디어를 **월드모델(World Model)·잠재행동(Latent Action)·시스템 관점**에서 구조적으로 정리하고, **NPU/시스템 아키텍트** 관점에서의 시사점까지 연결해 설명합니다.
    
    image_group {"query":["latent action world model diagram","world model latent action inference","unsupervised action learning world model","offline reinforcement learning world model"]}
    
    ## 1. 문제의식: *행동(action)이 없는 로그 데이터*
    
    현실 데이터(웹 비디오, 로봇 로그, 인간 행동 영상)는 **상태(state)만 있고 행동(action)이 관측되지 않는 경우**가 많습니다. 기존 World Model/Model-based RL은 `(s, a, s')`가 필요해 **실환경 데이터 활용에 한계**가 있었죠.
    
    **이 논문은**
    
    > 
    
    *관측된 상태 전이만으로도* 내부적으로 **잠재 행동(latent action)**을 학습하여 월드모델을 구축하자
    
    는 문제를 정면으로 다룹니다.
    
    ## 2. 핵심 아이디어 요약
    
    ### 2.1 Latent Action World Model
    
    • 실제 행동 
    
    • 전이 모델:
    
    • 추론 모델(인코더):
    
    즉, **상태 쌍 (sₜ, sₜ₊₁) → “그럴듯한 행동” zₜ를 역추론**합니다.
    
    ### 2.2 행동의 의미를 *사후적으로* 부여
    
    • z
    
    • 그러나 
    
    • “밀기 / 회전 / 이동”과 같은 
    
    • 결과적으로 
    
    ## 3. 학습 구조 (VAE 계열)
    
    전체 구조는 **World Model + VAE** 조합으로 이해하면 정확합니다.
    
    ```
    (s_t, s_{t+1})
            │
            ▼
       Inference Net
          q(z | s_t, s_{t+1})
            │
            ▼
       Latent Action z_t
            │
            ▼
       Dynamics Model
       p(s_{t+1} | s_t, z_t)
    ```
    
    **Loss 구성**
    
    • Reconstruction / Prediction loss
    
    • KL regularization (latent action의 정보량 제어)
    
    • 경우에 따라 
    
    ## 4. “in the Wild”의 의미
    
    이 논문의 중요한 포인트는 **통제된 RL 환경이 아니라**:
    
    • 인터넷 비디오
    
    • 로봇 로그
    
    • 인간 시연 데이터
    
    처럼 **노이즈 많고 비정형적인 데이터**에서도
    
    > 
    
    *Action-free World Model이 작동한다*
    
    는 점을 실험적으로 보였다는 것입니다.
    
    ## 5. 기존 접근과의 차별점
    
    | 구분 | 기존 World Model | Latent Action WM |
    | --- | --- | --- |
    | 행동 입력 | 실제 a 필요 | ❌ 필요 없음 |
    | 데이터 | RL 환경 | 오프라인·비디오 |
    | 행동 의미 | 명시적 | **암묵적(latent)** |
    | 활용 | RL/Planning | Planning + Imitation |
    
    ## 6. 시스템 / NPU 관점의 해석 (중요)
    
    이 논문은 **알고리즘 논문이지만**, 시스템 설계에 매우 강한 시사점을 줍니다.
    
    ### 6.1 Action-free = HW 친화성
    
    • 실제 행동 공간은 보통:
    
    • 불연속
    
    • 고차원
    
    • sparse
    
    • 반면 
    
    • 저차원 연속 벡터
    
    • 정규화 가능
    
    • Tensor-friendly
    
    ➡️ **NPU에서 훨씬 다루기 쉬운 제어 인터페이스**
    
    ### 6.2 Latent Action = 내부 마이크로-ISA
    
    시스템적으로 보면:
    
    • z
    
    • dynamics model은 
    
    이 구조는 다음과 같이 해석 가능합니다:
    
    • z
    
    • World Model ≈ 
    
    • Planning ≈ 
    
    👉 **“행동을 ISA처럼 학습한다”**는 관점이 성립
    
    ### 6.3 타일 기반 / 스트리밍 NPU와의 궁합
    
    • z
    
    • (s_t → s_{t+1})
    
    • Planning roll-out은 
    
    ➡️ **LLM inference NPU와 거의 동일한 데이터플로우 패턴**
    
    ## 7. 연구 확장 아이디어 (창훈님 관점)
    
    다음은 현재 진행 중이신 연구들과의 **자연스러운 결합 포인트**입니다.
    
    ### 7.1 Latent Action Quantization
    
    • z
    
    • KL 항에 
    
    • → 
    
    ### 7.2 Chaos / Stochastic Dynamics와 결합
    
    • latent action을 deterministic이 아닌
    [
    z_{t+1} = f(z_t) + \epsilon
    ]
    
    • ChaosNet / stochastic planning
    
    ### 7.3 RISC-V / NPU ISA 확장 아이디어
    
    • LATENT_ACT
    
    • World Model rollout을 
    
    • 실패 시 rollback (ROB 개념과 유사)
    
    ## 8. 한 줄 요약
    
    > 
    
    **이 논문은 “행동이 없는 세계에서도, 세계는 배울 수 있다”는 것을 보였고,시스템 관점에서는 ‘행동을 ISA처럼 학습하는 월드모델’이라는 매우 강력한 설계 패러다임을 제시한다.**
    
    원하시면 다음 단계로:
    
    • Tiny latent-action world model 구현
    
    • NPU-friendly IR 관점 재정의
    
    • LLM prefill/decode와의 구조적 유사성 비교
    
    • Chaos/entropy 기반 확장 설계 문서
    
    중 어떤 방향으로 확장할지 바로 이어서 설계해드릴 수 있습니다.