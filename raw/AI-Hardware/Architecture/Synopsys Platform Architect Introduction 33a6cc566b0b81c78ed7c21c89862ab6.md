# Synopsys Platform Architect 소개

ChatTime: 2026년 1월 20일 23:38 (GMT+9)
Created time: 2026년 4월 6일 오후 11:02
ProjectName: NPU Architecture
URL: https://chatgpt.com/c/696f92fe-359c-8332-ac21-237c9a2e8590

- **You: synopsis platform ar...**
    
    synopsis platform architect 툴에 대해서 조사해서 알려줘.
    
- **ChatGPT: ## entity ["company"...**
    
    ## entity ["company","Synopsys","eda company santa clara ca us"]  Platform Architect 개요
    
    image_group {"query":["Synopsys Platform Architect UI","Synopsys Platform Architect system modeling","Synopsys virtual prototyping Platform Architect"]}
    
    **Synopsys Platform Architect**는 SoC/시스템 수준에서 **아키텍처 탐색(Architecture Exploration)**과 **성능·전력·대역폭 모델링**을 수행하는 **System-Level Design & Analysis(SLDA)** 도구입니다. RTL 이전 단계에서 다양한 설계 대안을 빠르게 비교·검증하는 것이 핵심 가치이며, CPU·NPU·GPU·DSP·메모리·인터커넥트(AXI/NoC)까지 포함한 **이종 컴퓨팅 시스템**을 **트랜잭션/행위 수준**으로 모델링합니다.
    
    ## 1. 핵심 목적과 철학
    
    • RTL 이전 의사결정 가속
    
    • 정량적 지표 중심
    
    • Executable Spec
    
    ## 2. 주요 기능 블록
    
    ### 2.1 시스템 모델링
    
    • 컴퓨트
    
    • 메모리
    
    • 인터커넥트
    
    • 전력
    
    ### 2.2 워크로드 & 트래픽
    
    • 워크로드 주입
    
    • AI 관점
    
    ### 2.3 분석 & 리포팅
    
    • 타이밍/대역폭/병목
    
    • 파라미터 스윕
    
    • 시각화
    
    ## 3. NPU/AI SoC 관점에서의 활용 시나리오
    
    ### 3.1 NPU 아키텍처 탐색
    
    • TE/VE 분리 구조
    
    • Prefill vs Decode
    
    • DMA vs NoC
    
    ### 3.2 메모리 서브시스템
    
    • 온칩 SRAM 분할/분산
    
    • LLC 크기 변화
    
    • 압축/양자화
    
    ### 3.3 버스/NoC 설계
    
    • AXI 멀티레벨, NoC 토폴로지·VC·QoS 정책
    
    • AI Burst 트래픽
    
    ## 4. Synopsys 생태계 내 위치
    
    | 단계 | 도구 | 역할 |
    | --- | --- | --- |
    | System/Arch | **Platform Architect** | RTL 이전 탐색·의사결정 |
    | Virtual Prototype | Virtualizer | SW 조기 검증/부팅 |
    | RTL Design | Design Compiler / Fusion Compiler | 논리합성 |
    | Verification | VCS / Verdi | 검증 |
    | PPA Signoff | PrimeTime / IC Compiler II | 타이밍·물리 |
    
    → Platform Architect 결과는 **IP 구성/인터커넥트/클럭 계획의 기준선**을 제공해 이후 단계의 재작업 리스크를 줄입니다.
    
    ## 5. 강점과 한계 (실무 관점)
    
    ### 강점
    
    • 초기 병목 제거
    
    • 의사소통 도구
    
    • 대규모 파라미터 스윕
    
    ### 한계
    
    • Cycle-accurate 아님
    
    • 모델 정확도 의존
    
    • 학습 비용
    
    ## 6. 당신의 연구/개발과의 연결 (권장 활용 전략)
    
    • Platform Architect = 상위 시스템 탐색
    
    • Python/Numba/Py-V 시뮬레이터 = 하위 정밀 검증
    
    권장 워크플로우:
    
    1. Platform Architect로 
    
    1. 후보 아키텍처를 Py-V 기반 
    
    1. 결과를 다시 상위 모델에 피드백하여 
    
    ## 7. 언제 도입 가치가 큰가?
    
    • Custom SoC/고객 파생 설계
    
    • AI 트래픽이 SoC 병목의 주범
    
    • 컴파일러·런타임 정책
    
    ### 요약
    
    **Synopsys Platform Architect**는 “아키텍처를 코드처럼 실행해보는” 도구입니다. NPU 중심 SoC에서 메모리·NoC·워크로드의 상호작용을 **초기 단계에서 정량화**해 주며, 당신이 구축 중인 **정밀 NPU 시뮬레이터와 상보적으로 결합**할 때 가장 큰 효과를 냅니다.