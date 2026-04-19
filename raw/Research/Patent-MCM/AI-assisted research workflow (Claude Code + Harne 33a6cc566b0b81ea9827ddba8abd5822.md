# AI-assisted research workflow (Claude Code + Harness)

> Editorial note (2026-04-19): Treat this file as a workflow/operating-model source, not as a reviewed-paper source.
> Primary semantic role: agent-assisted research execution, verification ownership, and output-production workflow.
> If a section reads like a paper/tool review, it should not become the canonical review hub; that belongs under `raw/Research/Paper-Review/*` and `wiki/topics/paper-reviews`.

ChatTime: 2026년 3월 5일 14:55 (GMT+9)
Created time: 2026년 4월 6일 오후 11:00
ProjectName: [Patent-Paper] MCM
URL: https://chatgpt.com/c/69a8bcc0-3cf4-83a8-ae47-cd659a88ab37

- **You: https://www.facebook...**
    
    [https://www.facebook.com/share/p/1aECqGtXot/?mibextid=wwXIfr](https://www.facebook.com/share/p/1aECqGtXot/?mibextid=wwXIfr)
    
- **ChatGPT: 보내신 링크는 **Facebook의 ...**
    
    보내신 링크는 **Facebook의 “공유(share) 링크”** 형식입니다.다만 ChatGPT에서는 **로그인이 필요한 Facebook 게시물 내용 자체를 직접 열어 볼 수 없기 때문에**, 해당 링크에 어떤 글이나 영상이 있는지는 확인할 수 없습니다.
    
    ### 1️⃣ 이 링크의 의미
    
    `facebook.com/share/...` 형식은 **특정 Facebook 게시물을 다른 사람에게 전달하기 위한 공유 URL**입니다.
    
    • 누군가 올린 게시물을
    
    • 메시지나 외부 링크로 보내기 위해 만들어지는 주소입니다.
    
    • 클릭하면 해당 게시물 페이지로 이동합니다.
    
    Facebook에서 **Share 기능은 다른 사람의 게시물(사진, 영상, 링크 등)을 자신의 타임라인, 그룹, 메신저 등으로 다시 게시하는 기능**입니다.   12 turn0search4
    
    즉 구조적으로는
    
    ```
    facebook.com/share/p/{post-id}
    ```
    
    ➡ 특정 게시물 ID를 가리키는 링크입니다.
    
    ### 2️⃣ 안전성 관점
    
    이런 링크 자체는 **정상적인 Facebook 링크**일 가능성이 높습니다.하지만 주의할 점이 있습니다.
    
    1. 로그인 유도 + 외부 사이트 이동
    
    1. 투자/코인/상품 홍보
    
    1. 개인정보 입력 요구
    
    이런 경우는 **피싱 또는 스팸**일 수 있습니다. 실제로 Facebook 링크 구조를 이용한 **사기 페이지 유도 사례도 존재합니다.**   15
    
    ### 3️⃣ 확인 방법 (가장 안전)
    
    다음 방법으로 판단하면 됩니다.
    
    1. 링크를 
    
    1. 게시한 
    
    1. 게시물 내용이
    
    • 투자 권유
    
    • 무료 코인
    
    • 링크 클릭 유도
    
    • 개인정보 요청
    
    이면 **무시하는 것이 좋습니다.**
    
    ✅ **제가 정확히 분석해 줄 수도 있습니다.**
    
    가능하면 아래 중 하나를 보내 주세요.
    
    • 그 게시물 
    
    • 게시물 
    
    • 게시한 
    
    그러면
    
    • 사기 여부
    
    • 내용 요약
    
    • 왜 퍼지고 있는지
    
    까지 분석해 드리겠습니다.
    
- **You: Claude Code + Harnes...**
    
    Claude Code + Harness 구성으로 논문 작성 테스트를 해보았는데 신세계입니다.
    대시보드와 논문 작성에 딱 4번의 프롬프트 요청. 총 20분 소요.
    
    결과물로 확인하시죠.
    
    • 대시보드 : 
    
    • 논문 : 
    
    "NVIDIA A100에서 Rubin R100까지 5년간 에너지 효율이 26.7배 향상된 실증 데이터를 바탕으로, 본 연구는 향후 5–7년간 세대당 가속 배율이 3.5–5.0배에 도달하는 '초가속(Super-Acceleration)' 체제의 형성을 전망한다."
    
- **ChatGPT: 사용자가 보낸 글은 **AI-assi... (Part 1/2)**
    
    사용자가 보낸 글은 **AI-assisted research workflow (Claude Code + Harness)**로 **20분 만에 대시보드 + 논문을 생성했다는 사례**입니다. 핵심 주장과 기술적 의미를 **AI 시스템/컴퓨팅 아키텍처 관점에서 분석**해 보겠습니다.
    
    # 1. 글의 핵심 주장
    
    요약하면 다음입니다.
    
    **Workflow**
    
    • Claude Code + Harness 사용
    
    • 프롬프트 4개
    
    • 20분
    
    • 결과
    
    • 대시보드
    
    • 논문(PDF)
    
    **연구 주장**
    
    > 
    
    NVIDIA A100 → Rubin R100 (약 5년)에너지 효율 26.7× 증가
    
    → 향후 5–7년 동안**세대당 3.5–5× 가속**
    
    이를 **Super-Acceleration**이라고 정의.
    
    # 2. 실제 하드웨어 발전과 비교
    
    ## NVIDIA GPU 세대 흐름
    
    image_group {"aspect_ratio":"1:1","query":["NVIDIA A100 GPU board","NVIDIA H100 Hopper GPU","NVIDIA Blackwell B100 GPU architecture diagram","NVIDIA Rubin GPU roadmap illustration"],"num_per_query":1}
    
    대략적인 AI GPU 세대는 다음과 같습니다.
    
    | 세대 | 연도 | 특징 |
    | --- | --- | --- |
    | **A100 (Ampere)** | 2020 | TF32, sparse tensor |
    | **H100 (Hopper)** | 2022 | FP8 Transformer Engine |
    | **B100/B200 (Blackwell)** | 2024 | FP4 inference |
    | **Rubin R100 (예정)** | ~2026 | NVL144 scale |
    
    이 발전은 단순 FLOPS가 아니라
    
    **3가지 축에서 이루어집니다**
    
    1️⃣ 아키텍처2️⃣ 공정3️⃣ 시스템 규모
    
    # 3. 26.7× 효율 증가의 실제 원인
    
    GPU 자체만 보면 **26×는 과장에 가까울 가능성이 큽니다.**
    
    대부분 다음 요소가 포함됩니다.
    
    ## 1. Precision 축소
    
    | 세대 | datatype |
    | --- | --- |
    | A100 | FP16 / TF32 |
    | H100 | FP8 |
    | Blackwell | FP4 |
    
    연산 효율 증가
    
    ```
    FP16 → FP8 → FP4
    ```
    
    대략
    
    ```
    2× → 2×
    ```
    
    ## 2. Sparse acceleration
    
    Ampere부터
    
    ```
    2:4 sparsity
    ```
    
    이 적용됨.
    
    ```
    2× theoretical speedup
    ```
    
    ## 3. 시스템 스케일링
    
    AI 컴퓨팅 성능은
    
    ```
    GPU × GPU 연결
    ```
    
    이 핵심입니다.
    
    예
    
    | 시스템 | GPU 수 |
    | --- | --- |
    | DGX A100 | 8 |
    | DGX H100 | 8 |
    | GB200 NVL72 | 72 |
    
    여기서 성능 증가가 큼.
    
    ## 4. HBM 대역폭
    
    | 세대 | bandwidth |
    | --- | --- |
    | A100 | 1.6 TB/s |
    | H100 | 3.35 TB/s |
    | Blackwell | ~8 TB/s |
    
    AI workload는
    
    ```
    memory bound
    ```
    
    이기 때문에
    
    **대역폭 증가가 핵심**
    
    # 4. "Super-Acceleration" 가설 분석
    
    글의 핵심 주장
    
    ```
    세대당 3.5–5× 가속
    ```
    
    이건 **부분적으로 맞고 부분적으로 틀립니다.**
    
    ## GPU 단일칩 기준
    
    현실적인 성장률
    
    ```
    1.6× ~ 2.2×
    ```
    
    예
    
    ```
    A100 → H100
    ≈ 2×
    ```
    
    ## 시스템 기준
    
    Cluster scale에서는
    
    ```
    3× ~ 5×
    ```
    
    가능합니다.
    
    예
    
    ```
    DGX → NVL72
    ```
    
    ## 따라서
    
    논문의 주장
    
    ```
    Super-Acceleration
    ```
    
    은 실제로는
    
    ```
    system scaling acceleration
    ```
    
    일 가능성이 큽니다.
    
    # 5. AI 시대의 새로운 성능 법칙
    
    최근 연구에서 제안되는 법칙
    
    ## Post-Moore acceleration
    
    성능 증가
    
    ```
    Compute scaling
    +
    Cluster scaling
    +
    Algorithm efficiency
    ```
    
    대표 사례
    
    ### 1️⃣ FlashAttention
    
    ```
    2× ~ 4× speedup
    ```
    
    ### 2️⃣ MoE
    
    ```
    10× parameter scale
    ```
    
    ### 3️⃣ FP8 / FP4
    
    ```
    2× ~ 4× efficiency
    ```
    
    # 6. Claude Code로 논문 작성이 "신세계"인 이유
    
    이 글의 진짜 의미는 **AI 연구 workflow 변화**입니다.
    
    예전
    
    ```
    연구
    → 코드
    → 데이터
    → 논문 작성
    → 그림 제작
    ```
    
    지금
    
    ```
    프롬프트
    → 코드 생성
    → 실험
    → 대시보드
    → 논문
    ```
    
    즉
    
    **Research Copilot**
    
    # 7. 이 워크플로우 구조
    
    Claude Code + Harness 구조는 대략 이렇습니다.
    
    ```
    Prompt
       ↓
    LLM
       ↓
    Code generation
       ↓
    Data analysis
       ↓
    Visualization
       ↓
    Paper generation
    ```
    
    대시보드는 보통
    
    ```
    Next.js
    +
    Vercel
    +
    Plotly
    ```
    
    로 생성됩니다.
    
    # 8. System Architect 관점에서 중요한 점
    
    사용자처럼 **NPU/SoC 아키텍처 연구하는 입장**에서는 이 흐름이 매우 중요합니다.
    
    이제 연구 workflow가
    
- **ChatGPT: 사용자가 보낸 글은 **AI-assi... (Part 2/2)**
    
    ```
    Architecture idea
       ↓
    LLM simulation code
       ↓
    performance modeling
       ↓
    paper auto draft
    ```
    
    로 바뀌고 있습니다.
    
    즉
    
    **AI-assisted architecture research**
    
    # 9. 앞으로 더 강력한 연구 방법
    
    지금 단계는 사실 **초기 수준**입니다.
    
    앞으로는
    
    ### AI + Simulator
    
    ```
    LLM
      ↓
    Architecture exploration
      ↓
    Simulator auto run
      ↓
    Pareto frontier search
    ```
    
    예
    
    ```
    NPU tile size
    SRAM size
    bandwidth
    precision
    ```
    
    자동 탐색.
    
    # 10. 개인적으로 중요한 질문
    
    사용자 연구 분야와 연결하면 다음이 가능합니다.
    
    현재 연구
    
    ```
    RISC-V NPU simulator
    ```
    
    여기에 AI agent를 붙이면
    
    ```
    architecture search AI
    ```
    
    가 됩니다.
    
    예
    
    ```
    LLM
      ↓
    modify NPU config
      ↓
    run simulator
      ↓
    collect metrics
      ↓
    optimize architecture
    ```
    
    # 결론
    
    이 글의 핵심은 두 가지입니다.
    
    ### 1️⃣ AI 하드웨어
    
    ```
    GPU 성능 증가
    ≈ 2× / generation
    ```
    
    하지만 시스템 기준
    
    ```
    3× ~ 5×
    ```
    
    가능.
    
    ### 2️⃣ AI 연구 workflow
    
    20분 논문은
    
    ```
    AI assisted research
    ```
    
    시대의 시작입니다.
    
    원하시면 제가 **이 사례를 분석해서**
    
    **“AI 시대의 SoC 아키텍처 연구 방법 (2026)”**
    
    이라는 관점에서
    
    1️⃣ Claude Code 연구 workflow2️⃣ AI 기반 NPU architecture exploration3️⃣ 자동 논문 생성 pipeline
    
    까지 **실제 연구 시스템 설계**로 정리해 드리겠습니다.
    
    이건 지금 AI 하드웨어 연구에서 **굉장히 중요한 방향**입니다.
