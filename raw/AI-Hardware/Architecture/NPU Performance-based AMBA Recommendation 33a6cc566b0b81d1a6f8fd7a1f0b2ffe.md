# NPU 성능별 AMBA 추천

ChatTime: 2026년 1월 18일 23:16 (GMT+9)
Created time: 2026년 4월 6일 오후 11:04
ProjectName: NPU Architecture
URL: https://chatgpt.com/c/696cea84-58f8-832f-95b5-40886959dc16

- **You: Npu 성능별 적절한 amba bus...**
    
    Npu 성능별 적절한 amba bus intrastructure 추천. 멀티레벨 버스도 고려
    
- **ChatGPT: 아래는 **NPU 성능(TOPS/대역...**
    
    아래는 **NPU 성능(TOPS/대역폭/동시성)**에 따라 “현실적으로 잘 굴러가는” **AMBA 기반 버스/NoC 인프라 조합**을 단계별로 추천한 것입니다. 핵심은 **(1) Control-plane과 Data-plane 분리**, **(2) NPU 근처는 로컬/클러스터 인터커넥트로 짧고 굵게**, **(3) DRAM/캐시 일관성은 필요한 범위만**입니다.
    
    ## 0) 공통 전제: AMBA 구성의 기본 “정답 패턴”
    
    • Control-plane
    
    • 레지스터 접근, 디버그, 인터럽트, 성능 카운터
    
    • High-throughput Data-plane
    
    • 가중치/활성/중간 feature, DMA, 스케줄러 큐, completion ring
    
    • 일관성(coherency)
    
    • “CPU가 캐시로 들고 있는 텐서를 NPU가 바로 읽는다” 같은 유스케이스가 아니면, 
    
    • 메모리 보호/가상화
    
    • 멀티프로세스/보안 격리/컨테이너/안드로이드 HAL 시나리오에서 필수에 가까움
    
    ## 1) 성능 티어별 권장 AMBA Bus Infrastructure
    
    ### Tier A) 소형 Edge NPU (대략 1–10 TOPS / DDR 대역폭 수~수십 GB/s)
    
    **목표**: 저전력·저면적, 통합 쉬움, 충분한 QoS
    
    **추천**
    
    • Control: 
    
    • Data: 
    
    • 로컬 SRAM이 있으면: NPU 내부는 전용 버스(로컬 크로스바)로 해결하고 
    
    **포인트**
    
    • SoC 크로스바에서 
    
    • AXI 폭은 보통 
    
    ### Tier B) 모바일/임베디드 중급 (대략 10–50 TOPS / DDR 50–200 GB/s급 체감 목표)
    
    **목표**: 멀티 클라이언트 경합, 지연/대역폭 균형, 멀티레벨 인터커넥트 필요
    
    **추천(멀티레벨 버스)**
    
    • L0 (NPU 내부): 
    
    • TE/VE, DMA, SRAM 뱅크 간 충돌 최소화 (banked SRAM + multicast 지원 고려)
    
    • L1 (NPU 섬/서브시스템): 
    
    • 여기서 
    
    • L2 (SoC 메인): 
    
    **일관성 선택지**
    
    • 대부분은 Non-coherent + SMMU + explicit cache maintenance
    
    • CPU와 공유 버퍼를 “자주” 핸드오프하면 
    
    **포인트**
    
    • “NPU 성능”보다 실제론 
    
    • QoS를 2레벨로
    
    • L1에서 NPU 내부 우선순위(예: weight/read > writeback)
    
    • L2에서 시스템 우선순위(예: display underrun 방지 > NPU)
    
    ### Tier C) 고성능 SoC NPU (대략 50–200 TOPS / HBM/고대역 DDR, 멀티 NPU 클러스터)
    
    **목표**: 크로스바로는 한계 → NoC 중심 설계, 다중 메모리 포트/파티션
    
    **추천**
    
    • SoC Top: 
    
    • NPU는 
    
    • 예: 
    
    • 메모리: DRAM 컨트롤러도 
    
    • NPU 트래픽을 “한 채널에 몰아넣지 않게” 주소 매핑/인터리빙 정책이 중요
    
    **일관성**
    
    • CPU SLC/L3와 강하게 얽히면 
    
    • 단, NPU의 bulk DMA가 캐시를 오염시키는 부작용이 있어 
    
    ### Tier D) 서버/가속기급 (200 TOPS+ / 멀티 다이/칩렛/PCIe-CXL 연동)
    
    이 영역은 “AMBA만”으로 끝나기보다, **온패키지 NoC/다이다이 인터커넥트** + 호스트 인터페이스(PCIe/CXL)로 넘어갑니다.다만 내부 설계 원리는 Tier C를 확장한 형태이며, AMBA는 **다이 내부 로컬 패브릭**으로 남는 경우가 많습니다.
    
    ## 2) 멀티레벨(계층형) 버스 토폴로지 “권장 템플릿”
    
    아래 템플릿이 실제로 가장 디버그/확장/성능 튜닝이 쉽습니다.
    
    ```
    [CPU Cluster]--(ACE/CHI optional)--[SLC/L3]--+
                                                 |
                                          +------v------+
                                          |   SoC NoC   |  (L2)
                                          +---+-----+---+
                                              |     |
                               +--------------+     +----------------+
                               |                                     |
                        +------v------+                       +------v------+
                        |  NPU Subsys | (L1)                  | Multimedia   |
                        | AXI Switch  |                       | ISP/Display  |
                        +--+---+---+--+                       +-------------+
                           |   |   |
            (L0)           |   |   |
       +-------------------v+  |  +-------------------+
       |   NPU Cluster Bus  |  |  |   NPU Cluster Bus  |
       | (TE/VE/SRAM banks) |  |  | (optional 2nd cl.) |
       +--------------------+  |  +--------------------+
            |      |          |
         [DMA-R] [DMA-W]   [SMMU optional]
    ```
    
    **해석**
    
    • L0: NPU 내부는 “연산/로컬 SRAM 중심”으로 최적화 (SoC 공유자원 영향 최소화)
    
    • L1: NPU 밖으로 나가는 포트에서 트래픽을 정리(Shaping/Reorder/QoS/계측)
    
    • L2: SoC 전체는 NoC가 책임 (다수 마스터/슬레이브 확장성)
    
    ## 3) 버스 선택 체크리스트 (성능을 좌우하는 진짜 항목들)
    
    ### (1) AXI 포트 수 분리
    
    • Read/Write 분리
    
    • 가능하면 트래픽 성격별 분리:
    
    • Weight(대용량 순차 read)
    
    • Activation/KV(혼합 read/write, reuse 존재)
    
    • Writeback(버퍼링 가능, 지연 허용 가능)
    
    ### (2) QoS를 “정책”으로 설계
    
    • 단순 우선순위보다 
    
    • Display/ISP 같은 실시간 클라이언트가 있으면, NPU는 
    
    ### (3) Outstanding / Reorder / ID 폭
    
    • NPU는 메모리 레이턴시 숨기려면 
    
    • 대신 reorder가 커지면 디버그가 어려워지므로 
    
    ### (4) Coherency는 “정확히 필요한 곳만”
    
    • 무턱대고 full coherent(ACE/CHI)로 가면 전력·복잡도·검증 비용이 크게 증가합니다.
    
    • 일반적으로 추천 순서:
    
    1. non-coherent DMA + flush/invalidate
    
    1. 필요 구간만 ACE-Lite(또는 I/O coherent)
    
    1. 정말 필요하면 CHI coherent NoC
    
    ## 4) 빠른 결론: “성능별 한 줄 추천”
    
    • ~10 TOPS
    
    • 10~50 TOPS
    
    • 50~200 TOPS
    
    • 200 TOPS+
    
    원하면, 지금 구상 중인 NPU(TE/VE 구조, 온칩 SRAM 크기/뱅킹, 목표 DDR/HBM, prefill/decode 비중)를 기준으로 **“포트 구성(몇 개/폭/클럭) + QoS 룰 + 주소 인터리빙 + SMMU/일관성 정책”**을 한 장짜리 사양으로 뽑아줄게요.