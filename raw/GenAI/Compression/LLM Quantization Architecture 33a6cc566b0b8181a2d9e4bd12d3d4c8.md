# LLM 양자화 아키텍처

ChatTime: 2026년 3월 30일 23:45 (GMT+9)
Created time: 2026년 4월 6일 오후 10:45
ProjectName: GenAI.Compression
URL: https://chatgpt.com/c/69ca86e3-6c50-83a5-a1c8-e7de258d725c

- **You: # LLM 양자화·압축 아키텍처 노트... (Part 1/2)**
    
    # LLM 양자화·압축 아키텍처 노트 (TurboQuant 중심)
    
    ## 0) 이 문서의 목적
    
    이 문서는 두 가지를 동시에 수행한다.
    
    1. 현재 리포지토리(
    
    1. Google Research의 
    
    > 
    
    핵심 관점: “양자화는 값(value) 손실 최적화 문제가 아니라, 연산자(operator)와 좌표계(coordinate system) 선택 문제”라는 이 리포의 주장을 실제 KV-cache 압축 알고리즘(TurboQuant)까지 연결한다.
    
    ## 1) 현재 코드베이스(문서베이스)의 아이디어 맵
    
    ### 1.1 프로젝트의 North Star
    
    리포는 LLM 압축을 다음처럼 재정의한다.
    
    • 가중치는 독립 스칼라 집합이 아니라 
    
    • 학습된 파라미터는 고차원 공간 전체가 아니라 
    
    • outlier는 본질 신호라기보다 
    
    • Hadamard류는 “주파수 변환”이라기보다 **축 변환(회전)**이다.
    
    • 랜덤 회전은 flattening(평탄화)은 잘 하지만, 진짜 압축 이득은 manifold-aligned coordinate에서 오는 concentration(집중)에서 나온다.
    
    즉 이 리포의 주제는 “low-bit 트릭 모음집”이 아니라, **표현좌표(표기법)의 기하학적 선택**이다.
    
    ### 1.2 실무적으로 중요한 해석
    
    • 일반적인 양자화 운영은 보통 
    
    • 이 리포 관점은 그보다 상위 레벨에서 “무슨 좌표축에서 scale을 잡는가?”를 먼저 본다.
    
    • 이 프레이밍은 KV cache, weight-only quant, activation quant, vector DB PQ를 하나의 공통 언어로 묶는다.
    
    ### 1.3 코드베이스 상태 요약
    
    • 현재는 구현보다 
    
    • 토이 실험 spec(기저에 따른 압축성 차이)과 실제 LLM 검증 spec이 분리되어 있다.
    
    • 논문/특허 관점 문서까지 이미 배치되어 있어, 연구→제품화로 이어질 문맥이 명확하다.
    
    ## 2) TurboQuant 상세 정리
    
    ### 2.1 TurboQuant의 문제 정의
    
    TurboQuant는 고차원 벡터를 온라인(데이터 의존적 학습 없이)으로 저비트 압축하면서,
    
    • MSE 왜곡
    
    • inner-product 왜곡
    
    을 동시에 낮추는 VQ 계열 알고리즘이다.
    
    LLM에서는 특히 **KV cache quantization**에 적용되며, 긴 컨텍스트에서 메모리/대역폭 병목을 풀기 위한 목적이 강하다.
    
    ### 2.2 핵심 구성: 2-stage
    
    TurboQuant는 크게 두 단계다.
    
    1. MSE 중심 stage (PolarQuant 계열 아이디어 포함)
    
    • 벡터에 랜덤 회전을 적용
    
    • 회전 후 좌표 분포 성질(Beta 집중/고차원 근사 독립성)을 활용
    
    • 좌표별 scalar quantizer(Lloyd-Max) 적용
    
    1. Bias 교정 stage (QJL 1-bit residual)
    
    • 1단계 잔차(residual)에 대해 1-bit QJL 적용
    
    • MSE-최적 양자화가 만드는 inner-product bias를 보정
    
    • attention logit/유사도 계산에서 편향 없는 추정기에 가깝게 구성
    
    요약하면:
    
    • stage1 = 에너지 대부분을 고효율로 압축
    
    • stage2 = 작은 비트(보통 1bit/channel budget)를 써서 “잘못된 방향의 오차”를 잡음
    
    ### 2.3 왜 “회전 + 좌표별 양자화”가 먹히는가
    
    실무자가 놓치기 쉬운 포인트:
    
    • “좌표별 독립 quantizer는 구식”이라는 인식이 강하지만,
    
    • 고차원 랜덤 회전 이후 분포가 잘 섞이면 좌표 간 결합정보가 약해지고,
    
    • 이때 per-coordinate quantizer가 계산/배치/벡터화 측면에서 매우 강해진다.
    
    즉 TurboQuant는 복잡한 codebook search를 피하고,
    
    • 온라인성(학습 불필요)
    
    • 구현 단순성
    
    • 하드웨어 친화성
    
    을 이론보장과 같이 가져가려는 설계다.
    
    ### 2.4 PolarQuant 역할
    
    PolarQuant는 좌표계를 Cartesian→Polar 계열로 재표현해,
    
    • 분포 예측 가능성 증가
    
    • 정규화 메타데이터(quant constants) 부담 감소
    
    를 노린다.
    
    현업 의미:
    
    • “값 자체 양자화”보다 “기하 파라미터(반지름/각도)의 통계 구조”를 양자화 대상으로 삼아 메타 오버헤드를 줄이는 전략이다.
    
    ### 2.5 QJL residual 보정이 중요한 이유
    
    실무에서 흔한 함정:
    
    • MSE가 낮으면 attention 품질도 자동으로 좋을 거라 가정한다.
    
    • 하지만 attention은 dot-product 편향에 민감하고, 작은 systematic bias가 길은 문맥에서 누적된다.
    
    TurboQuant는 residual의 sign sketch(QJL)를 추가해 이 편향을 줄인다.
    
    ### 2.6 보고된 성능 포인트(공식 공개 기준: 2025-04-28 논문, 2026-03-24 블로그)
    
    공개된 논문/Google Research 블로그 기준으로 TurboQuant는 다음 메시지를 강조한다.
    
    • KV cache를 3~3.5bit/channel 수준으로 압축하며 품질 손실 최소화/중립 사례 보고
    
    • long-context 벤치(Needle, LongBench 등)에서 강한 유지 성능
    
    • H100에서 attention-logit 계산 기준 큰 폭의 가속 사례 보고
    
    주의:
    
    • 특정 speedup 숫자는 “attention kernel 구간” 기준인지 “end-to-end tokens/s” 기준인지 반드시 분리해야 한다.
    
    • 프로덕션에서는 scheduler, paged KV, batch shape, prefill/decode mix 때문에 체감 성능이 크게 달라진다.
    
    ## 3) 실무 암묵지: Senior Engineer/System Architect 관점 체크리스트
    
    ### 3.1 알고리즘-시스템 경계
    
    1. 알고리즘 승리 ≠ 서비스 승리
    
    • 논문은 왜곡률/리콜/task score를 본다.
    
    • 서비스는 p99 latency, tail OOM, 배치 안정성, 장애 복구를 본다.
    
    1. decode 단계와 prefill 단계를 분리해 측정
    
    • KV quant는 decode 메모리 병목 완화가 본진이다.
    
    • prefill이 긴 workload에서는 기대 이득이 희석될 수 있다.
    
    1. 메타데이터 오버헤드 회계 분리
    
    • “n-bit”만 보고 비교하면 함정.
    
    • scale/zero-point, per-block norm, layout padding까지 합친 실제 bpc(bit per channel)를 봐야 한다.
    
    1. 정확도 metric의 지연 붕괴(drift) 감시
    
    • 짧은 시퀀스는 버티는데 긴 체인-of-thought에서 갑자기 무너지는 경우가 있다.
    
    • multi-turn eval, retrieval stress를 함께 둬야 한다.
    
    ### 3.2 배치/런타임 설계 팁
    
    • quant/dequant를 attention kernel과 최대한 fusion해 메모리 왕복을 줄인다.
    
    • paged KV(vLLM류)와 block size를 맞춰 misaligned access를 피한다.
    
    • mixed precision 정책(예: recent tokens FP16 유지)을 두어 tail-risk를 제어한다.
    
    • fallback path(특정 헤드/레이어만 고정밀도)를 준비해 incident 대응 시간을 줄인다.
    
    ### 3.3 품질 관리 팁
    
    • MSE
    
    • needle retrieval + summarization + code completion을 분리 리포팅한다.
    
    • 모델별(아키텍처별) 민감도가 크게 다르므로 universal one-size를 가정하지 않는다.
    
    ## 4) 핵심 용어 사전 (실무 약어/관용어/의사표준 포함)
    
    아래 표는 실무에서 자주 쓰는 표현을 포함해 정리했다. 각 항목마다 **왜 중요한지 / 언제 쓰는지 / 오해 리스크**를 명시한다.
    
    | 용어 | 왜 중요한가 | 언제 쓰는가 | 잘못 이해하면 생기는 문제 |
    | --- | --- | --- | --- |
    | Quantization | 메모리·대역폭·연산량 절감의 기본 축 | 모델 서빙 최적화, 엣지 배포 | 비트수만 낮추면 무조건 이득이라 오판 |
    | PTQ (Post-Training Quantization) | 재학습 없이 적용 가능 | 모델 배포 직전 | calibration 생략 가능하다고 착각 |
    | QAT (Quantization-Aware Training) | 품질 보존 여지 큼 | 대규모 재학습 가능 조직 | 비용/복잡도 과소평가 |
    | Weight-only quant | 적용 난이도 낮음 | LLM 추론 가속 초기 단계 | activation 병목을 놓침 |
    | W/A quant (W8A8 등) | 연산량/메모리 동시 최적화 | 전용 커널 보유 시 | 커널 미지원으로 실제 성능 저하 |
    | KV cache quant | long-context 비용 핵심 절감 | decode 병목 워크로드 | prefill/decode 구분 없이 기대치 과대 |
    | VQ (Vector Quantization) | 고차원 구조 보존에 유리 | KV, retrieval index | codebook 오버헤드 누락 |
    | PQ (Product Quantization) | 검색 시스템 표준 압축 | vector DB/ANN | 오프라인 학습비용 무시 |
    | OPQ | 회전+PQ로 왜곡 감소 | 검색 품질 개선 | 회전비용/온라인성 제약 간과 |
    | RQ (Residual Quantization) | 단계적 오차 보정 | 초저비트 설계 | 지연 증가를 과소평가 |
    | Scalar quant | 구현·벡터화 용이 | 실시간 추론 | 고차 상관구조 손실 무시 |
    | Lloyd-Max | 분포 적합 scalar codebook | 사전 코드북 생성 | 데이터 분포 이동에 취약 |
    | Codebook | 양자화 중심 표현 | VQ/PQ 공통 | 메모리/캐시 locality 손실 |
    | Quant constants | scale/offset/norm 메타값 | block/group quant | 실제 압축률 과대평가 |
    | bpc (bit per channel) | 공정 비교 지표 | KV quant 비교 | 메타비트 제외한 허수 비교 |
    | Distortion-rate | 이론적 효율 척도 | 알고리즘 비교 | 실제 시스템비용 반영 누락 |
    | MSE distortion | 재구성 오차 기본지표 | quant 설계/튜닝 | attention 편향 문제 은폐 |
    | Inner-product distortion | attention/retrieval 품질과 직접 연관 | KV/ANN 평가 | MSE만으로 대체 가능하다고 착각 |
    | Unbiased estimator | 누적편향 완화 | 긴 시퀀스 attention | 분산 증가 trade-off 무시 |
    | QJL | 1-bit 스케치 기반 보정 | residual 보정 | 원신호 대체 알고리즘으로 오해 |
    | Residual | 1차 quant의 남은 오차 | 2-stage quant | residual scale 폭주 관리 실패 |
    | Random rotation | outlier 평탄화·분포 혼합 | TurboQuant/OPQ류 | “주파수 변환”으로 잘못 해석 |
    | Hadamard rotation | 저비용 직교변환 | 고속 전처리 | 회전 자체가 압축이라고 오해 |
    | Incoherence | 좌표 편중 완화 지표 | 회전 기반 quant 논의 | 해석 없이 수치만 추종 |
    | Outlier channel | 저비트 파괴의 주범 | activation/KV 분석 | 무조건 clipping으로 해결 시도 |
    | Clipping | tail 제거 | scalar quant 기본 | 과도 clipping으로 정보 손실 |
    | Per-channel quant | 채널 이질성 대응 | activation/KV | 구현복잡도·메타비트 급증 |
    | Per-token quant | 시점별 분포 대응 | KV value quant | 커널 비효율/메타 증가 |
    | Asymmetric quant | zero-point로 bias 보정 | 비대칭 분포 | dequant 비용 증가 간과 |
    | Symmetric quant | 커널 단순성 | GPU 고속경로 | 편향 데이터에서 품질 손실 |
    | Group size | 메타비트/정밀도 trade-off 핵심 | block/group quant | 기본값 고정으로 성능 미스 |
    | Block quant | locality 단위 압축 | KV pages/tiles | block 경계 인공물 방치 |
    | Calibration | scale 결정 신뢰성 | PTQ 워크플로우 | 샘플 편향으로 실서비스 붕괴 |
    | Data-oblivious | 온라인성·즉시성 | 실시간 KV | 특정 분포에서 성능 과신 |
    | Data-dependent | 품질 잠재력 높음 | 오프라인 인덱싱 | 인덱스 재생성 비용 폭증 |
    | Online quantization | 스트리밍 적용 가능 | token-by-token decode | 상태 누수/지연 누적 문제 |
    | Streaming quantization | 생성 중 지속 압축 | 긴 대화형 추론 | warm-up 구간만 보고 판단 |
    | Prefill vs Decode | 병목 위치가 다름 | 성능 프로파일링 | 전체 TPS 해석 오류 |
    | Paged KV cache | 메모리 단편화 완화 | vLLM류 런타임 | quant block과 불일치 |
    | HBM bottleneck | 대역폭 지배 병목 | GPU 추론 | FLOPs 최적화만 집착 |
    | SRAM reuse | 실제 속도 좌우 | fused kernel 최적화 | 메모리 계층 무시한 설계 |
    | Kernel fusion | 메모리 왕복 절감 | quant+matmul+attention | 디버깅/유지보수 난이도 급증 |
    | Memory-bound | KV/attention 특성 | 시스템 병목 진단 | compute-bound 가정으로 오판 |
    | Roofline | 이론상 한계 분석 | 커널 최적화 | 실제 runtime overhead 무시 |
    | Tail latency (p99) | 서비스 품질 핵심 | SLO 운영 | 평균지표만 보고 배포 |
    | Quality neutrality | “무손실에 가까움” 커뮤니케이션 키워드 | 제품 보고서/릴리즈 | 테스트 커버리지 부족 은폐 |
    | Needle-in-a-haystack | long-context 회수력 테스트 | KV quant 벤치 | 단일 벤치 과적합 |
    | LongBench | 멀티태스크 장문 벤치 | 모델 비교 | 실트래픽 대표성 과대해석 |
    | RULER / L-Eval / ZeroSCROLLS | 장문 추론 보완 벤치 | 일반화 검증 | 데이터 중복/누수 간과 |
    | Drift | 장시간 품질 저하 | 멀티턴/에이전트 | 초기구간 OK로 안정성 착각 |
    | Regression gate | 배포 품질 관문 | CI/CD inference | metric 선정 부실 |
    | Rollback-safe | 장애시 복귀 가능성 | 프로덕션 배포 | one-way schema 변경으로 장애 장기화 |
    | Canary | 점진 배포 | quant 정책 전환 | 표본 편향으로 오판 |
    | Blast radius | 장애 영향 범위 | 아키텍처 설계 | 범위 격리 실패 |
    | SLA/SLO | 운영 목표 | 서비스 품질 관리 | 연구지표만으로 의사결정 |
    | Manifold-aligned coords | 이 리포의 핵심 개념 | 차세대 압축 설계 | 랜덤 회전만으로 충분하다고 오해 |
    | Flattening vs Concentration | 회전의 한계와 정렬의 목표 구분 | 알고리즘 비교 프레임 | 둘을 동일개념으로 혼동 |
    | Functional + residual decomposition | 구조/잔차 비대칭 부호화 가능 | 고압축 설계 | 잔차만 줄이면 된다고 오판 |
    | RD (Rate-Distortion) design | 비트배분 최적화 프레임 | 알고리즘·시스템 동시설계 | kernel/metadata 제외한 이론최적 집착 |
- **You: # LLM 양자화·압축 아키텍처 노트... (Part 2/2)**
    
    ## 5) TurboQuant를 이 리포의 이론과 연결하기
    
    ### 5.1 연결점
    
    • 이 리포: outlier는 좌표계 인공물, 회전은 flattening.
    
    • TurboQuant: 랜덤 회전 후 per-coordinate quantizer로 near-optimal distortion을 노림.
    
    즉 TurboQuant는 이 리포의 “axis transform 관점”과 정합적이다.
    
    ### 5.2 차이점(중요)
    
    • 이 리포는 
    
    • TurboQuant의 핵심은 랜덤 회전 기반 온라인 보장.
    
    해석:
    
    • TurboQuant는 “강력한 범용 baseline + 이론 보장”으로 훌륭하다.
    
    • 그러나 리포의 장기 비전(매니폴드 정렬형 표현)에서는 추가 개선 여지가 남아 있다.
    
    ## 6) 실전 적용 권고안 (아키텍트 관점)
    
    1. 1차 도입
    
    1. 2차 최적화
    
    1. 3차 연구
    
    1. 운영 안정화
    
    ## 7) 참고 자료
    
    • TurboQuant arXiv: 
    
    • Google Research Blog (TurboQuant): 
    
    • QJL(arXiv 링크는 TurboQuant 논문 참고문헌/블로그 참조)
    
    • PolarQuant(arXiv 링크는 Google Research 블로그 참조)
    
    ## 8) 사실/추론 경계와 1차 출처(Primary Sources)
    
    이 문서는 아래 원칙으로 읽어야 한다.
    
    • 사실(fact)
    
    • 추론(inference)
    
    ### 8.1 1차 출처
    
    1. TurboQuant 논문 (arXiv, 제출일 2025-04-28)
    
    • 제목: 
    
    • 핵심: 랜덤 회전 + 좌표별 스칼라 양자화 + residual에 대한 1-bit QJL 보정이라는 2-stage 구성
    
    1. Google Research 공식 블로그 (게시일 2026-03-24)
    
    • 제목: 
    
    • 핵심: 제품/시스템 관점 메시지(3-bit KV, long-context 벤치, H100 attention-logit 구간 가속)를 대외 커뮤니케이션
    
    1. PolarQuant (arXiv/Google Research publications)
    
    • TurboQuant의 stage-1 직관을 이해하는 데 필요한 배경(랜덤 preconditioning + polar transform + quant constants 오버헤드 절감 논점)
    
    ### 8.2 실무 적용 시 주의(암묵지 보강)
    
    • 블로그 수치(예: 최대 8x)는 
    
    • “무손실” 표현은 벤치/모델/워크로드 의존이다.
    
    • 운영에서는 반드시 
    
    • KV quant 성능은 런타임(Paged KV, block size, prefill/decode mix)에 의해 크게 흔들린다.
    
    • 즉 알고리즘 설명만 맞아도 운영 성과가 자동으로 보장되지는 않는다.
    
- **ChatGPT: 이 문서는 이미 “연구 → 시스템 →... (Part 1/2)**
    
    이 문서는 이미 “연구 → 시스템 → 제품”까지 연결된 매우 좋은 구조를 가지고 있습니다.여기서 한 단계 더 나아가, **아키텍처 설계 관점에서 actionable insight**와 **연구 확장 방향**을 명확히 드리겠습니다.
    
    # 1. 핵심 요약 (System Architect 관점 재정의)
    
    당신의 문서를 한 줄로 압축하면:
    
    > 
    
    **Quantization = (Value compression) 문제가 아니라→ (Coordinate system + Operator decomposition) 설계 문제**
    
    TurboQuant는 이 프레임에서 보면:
    
    • 좌표계 선택
    
    • 연산자 분해
    
    • main signal → scalar quant (MSE optimal)
    
    • bias correction → QJL residual (inner-product unbiased)
    
    즉,
    
    ```
    x  →  R x  →  Q_scalar  +  Q_residual(QJL)
    ```
    
    이 구조 자체가 이미 “operator decomposition”이다.
    
    # 2. 당신 리포 vs TurboQuant: deeper alignment
    
    ## 2.1 동일한 철학 (중요)
    
    둘 다 사실 같은 문제를 다른 depth에서 본 것
    
    | 관점 | 리포 | TurboQuant |
    | --- | --- | --- |
    | Outlier 해석 | 좌표계 artifact | rotation으로 flatten |
    | 핵심 목표 | manifold-aligned concentration | incoherence 확보 |
    | quant 방식 | coordinate choice 먼저 | coordinate 고정 후 scalar quant |
    | 구조 | operator-level abstraction | 2-stage decomposition |
    
    👉 결론:
    
    > 
    
    TurboQuant = “random coordinate 기반 baseline 최적화”
    
    당신 리포 = “optimal coordinate 찾는 문제”
    
    ## 2.2 본질적인 차이 (가장 중요)
    
    TurboQuant의 한계:
    
    > 
    
    **"good average coordinate"는 찾지만
    
    "optimal task-aware coordinate"는 아니다**
    
    즉:
    
    • random rotation → distribution 평균화
    
    • 하지만 attention/semantic structure는 고려 안 함
    
    # 3. 새로운 해석: TurboQuant의 진짜 본질
    
    TurboQuant를 더 깊게 보면 사실 이것이다:
    
    ## 3.1 Functional decomposition 구조
    
    ```
    x ≈ Q_main(x) + Q_residual(x)
    ```
    
    여기서
    
    • Q_main → energy preserving (MSE)
    
    • Q_residual → bias correcting (inner-product)
    
    👉 이건 classical signal processing 관점:
    
    > 
    
    **Orthogonal decomposition + correction term**
    
    ## 3.2 더 강한 재정의
    
    TurboQuant는 사실:
    
    > 
    
    **“dot-product preserving compression” 문제를 풀고 있음**
    
    왜냐하면:
    
    • attention = softmax(q·k)
    
    • retrieval = similarity(q, x)
    
    즉 objective는:
    
    ```
    minimize distortion of xᵀy
    ```
    
    👉 이건 MSE 문제가 아니라:
    
    > 
    
    **bilinear form preservation 문제**
    
    # 4. 여기서 나오는 새로운 연구 방향 (핵심)
    
    이 문서에서 가장 중요한 부분입니다.
    
    ## 4.1 방향 1: Random → Task-aware coordinate
    
    현재:
    
    ```
    R_random x
    ```
    
    당신이 해야 할 것:
    
    ```
    R_task x
    ```
    
    예:
    
    • attention covariance 기반 rotation
    
    • KV subspace PCA
    
    • head-wise basis
    
    👉 효과:
    
    • random보다 훨씬 강한 concentration
    
    • bit allocation 효율 증가
    
    ## 4.2 방향 2: Operator-aware quantization
    
    현재 TurboQuant:
    
    ```
    uniform scalar quant per coordinate
    ```
    
    개선 방향:
    
    ```
    operator-dependent quantizer
    ```
    
    예:
    
    • key vs value 다른 quantizer
    
    • attention-sensitive dimension 더 높은 precision
    
    • layer별 다른 bit allocation
    
    👉 이건 사실:
    
    > 
    
    **RD (Rate-Distortion) + importance weighting 문제**
    
    ## 4.3 방향 3: Residual 구조 확장
    
    현재:
    
    ```
    1-bit QJL residual
    ```
    
    확장:
    
    ```
    multi-level residual hierarchy
    ```
    
    예:
    
    ```
    x ≈ Q1(x) + Q2(residual) + Q3(residual²)
    ```
    
    👉 이건:
    
    • RQ (Residual Quantization)
    
    • wavelet compression
    
    • scalable codec 구조
    
    와 동일
    
    ## 4.4 방향 4: Manifold-aligned coordinate (핵심 연구)
    
    당신 리포의 진짜 핵심
    
    ```
    x ≈ f(z),   z ∈ low-d manifold
    ```
    
    즉:
    
    • 실제 데이터는 full space에 없음
    
    • low-rank / nonlinear manifold 위에 있음
    
    👉 따라서:
    
    > 
    
    optimal quantization은**space quantization이 아니라 manifold parameter quantization**
    
    # 5. 시스템 아키텍처 관점 확장
    
    이건 당신의 NPU/SoC 관점에서 매우 중요합니다.
    
- **ChatGPT: 이 문서는 이미 “연구 → 시스템 →... (Part 2/2)**
    
    ## 5.1 현재 KV quant pipeline
    
    ```
    HBM → KV load → dequant → attention → writeback
    ```
    
    문제:
    
    • memory-bound
    
    • dequant overhead
    
    • bandwidth waste
    
    ## 5.2 TurboQuant-friendly HW 구조
    
    핵심 아이디어:
    
    > 
    
    **"dequant을 하지 않는다"**
    
    즉:
    
    ```
    quantized domain에서 바로 dot-product 수행
    ```
    
    ## 5.3 필요한 HW primitive
    
    ### (1) rotated-domain matmul
    
    ```
    (q · k) ≈ (q · R⁻¹)(Rk)
    ```
    
    → rotation-aware kernel 필요
    
    ### (2) bit-serial dot-product
    
    • low-bit KV
    
    • mixed precision accumulation
    
    ### (3) residual correction unit
    
    • QJL sign bit 처리
    
    • cheap correction
    
    👉 결론:
    
    > 
    
    TurboQuant는 알고리즘이 아니라**HW-friendly ISA primitive 후보**
    
    # 6. 당신 연구와 직접 연결
    
    지금 당신이 하고 있는 것과 정확히 맞물립니다:
    
    • ONNX → IR → NPU mapping
    
    • KV cache modeling
    
    • tile-based execution
    
    • memory bandwidth modeling
    
    👉 연결:
    
    ## 6.1 새로운 ISA 제안
    
    예:
    
    ```
    KV_LOAD_Q
    KV_DOT_Q
    KV_RESIDUAL_CORR
    ```
    
    ## 6.2 Simulator 확장 포인트
    
    당신 Py-V 시뮬레이터에서:
    
    • KV quant latency 모델링
    
    • bandwidth reduction 효과
    
    • residual correction cost
    
    👉 이건 바로 논문 + 제품 연결 포인트
    
    # 7. 가장 중요한 인사이트 (핵심 정리)
    
    이 문서의 가장 중요한 결론:
    
    ## Insight 1
    
    > 
    
    Quantization의 본질은**"좌표계 선택 문제"가 맞다**
    
    TurboQuant는 그 증거
    
    ## Insight 2
    
    > 
    
    하지만 random rotation은 **끝이 아니라 baseline**
    
    ## Insight 3
    
    > 
    
    진짜 frontier는:
    
    ```
    Random coordinate  →  Task-aware coordinate  →  Manifold coordinate
    ```
    
    ## Insight 4
    
    > 
    
    미래 KV compression은:
    
    ```
    Scalar quant → VQ → Operator-aware → Manifold-aware
    ```
    
    # 8. 다음 단계 (당신에게 추천)
    
    당신 수준에서는 여기로 가야 합니다:
    
    ## Step 1 (빠른 실험)
    
    • TurboQuant baseline 구현
    
    • KV cache에 적용
    
    • prefill vs decode 분리 측정
    
    ## Step 2 (차별화)
    
    • head-wise PCA rotation
    
    • random vs learned rotation 비교
    
    ## Step 3 (논문급 아이디어)
    
    • manifold-aligned KV compression
    
    • residual hierarchy 확장
    
    • operator-aware bit allocation
    
    ## Step 4 (아키텍처)
    
    • NPU ISA extension 설계
    
    • simulator + performance model
    
    # 9. 마지막 한 줄
    
    > 
    
    TurboQuant는 “정답”이 아니라
    
    **“coordinate-aware compression 시대의 시작점”이다**
    
    원하면 다음 단계로:
    
    • TurboQuant + PCA rotation 결합 실험 코드 (PyTorch + Numba)
    
    • KV cache quantization용 ISA + simulator spec (PRD/TDD 형태)
    
    • manifold-aligned quantization 수식화 (논문 수준)
    
    까지 바로 이어서 설계해드리겠습니다.
    
- **You: manifold-aligned qua...**
    
    manifold-aligned quantization 수식화 (논문 수준)
    
- **ChatGPT: 좋습니다. 아래는 **manifold... (Part 1/3)**
    
    좋습니다.아래는 **manifold-aligned quantization**을 LLM/KV-cache/weight compression까지 포괄할 수 있도록, **논문 본문에 바로 들어갈 수준**으로 수식화한 버전입니다.
    
    의도는 단순한 아이디어 메모가 아니라, 다음 3가지를 동시에 만족하는 것입니다.
    
    1. 왜 기존 scalar / random-rotation quantization보다 상위 개념인지
    
    1. TurboQuant류와의 관계
    
    1. 이후 
    
    # Manifold-Aligned Quantization: 논문 수준 수식화
    
    ## 1. 문제 설정
    
    고차원 벡터 (x \in \mathbb{R}^d)를 저비트 표현으로 압축한다고 하자.기존 양자화는 보통 다음 문제로 쓴다.
    
    [
    \min_{Q \in \mathcal{Q}} \mathbb{E}_{x \sim P_X}\big[|x - Q(x)|_2^2\big]
    ]
    
    여기서 (Q)는 스칼라 양자화, 벡터 양자화, 혹은 회전 후 스칼라 양자화일 수 있다.
    
    하지만 이 formulation은 두 가지를 숨긴다.
    
    1. 데이터 분포 (P_X)가 실제로는 (\mathbb{R}^d) 전체를 쓰지 않는다는 점
    
    1. task loss가 단순 reconstruction loss가 아니라, attention/logit/similarity 같은 **연산자 보존(operator preservation)**이라는 점
    
    따라서 우리는 (x)가 어떤 저차원 latent 좌표 (z \in \mathbb{R}^m), (m \ll d), 위의 smooth manifold (\mathcal{M}) 근처에 존재한다고 가정한다.
    
    [
    x = f(z) + \varepsilon, \qquad z \sim P_Z,\quad \varepsilon \sim P_\varepsilon
    ]
    
    여기서
    
    • (f: \mathbb{R}^m 	o \mathbb{R}^d) 는 immersion 또는 embedding
    
    • (\mathcal{M} = {f(z): z \in \mathcal{Z}}\subset \mathbb{R}^d)
    
    • (\varepsilon)는 manifold 밖의 잔차 또는 노이즈
    
    즉 실제 압축 문제는 (x) 전체가 아니라,**latent coordinate (z)** 와 **normal residual (\varepsilon)** 를 어떻게 다르게 다룰 것인가의 문제다.
    
    ## 2. 핵심 주장: 양자화는 좌표 선택 문제다
    
    기존 random rotation 기반 양자화는 어떤 직교변환 (R \in O(d))를 도입하여
    
    [
    y = Rx
    ]
    
    후에 각 좌표를 독립적으로 양자화한다.
    
    [
    Q_{	ext{rot}}(x) = R^	op \hat y,\qquad \hat y_i = q_i(y_i)
    ]
    
    이 구조는 좌표축 간 energy concentration을 flattening하여, 좌표별 스칼라 양자화를 쉽게 만든다.
    
    그러나 manifold 관점에서는 더 적절한 좌표계가 존재한다.즉, 좋은 양자화기는 “값을 직접 양자화”하는 것이 아니라,**manifold에 정렬된 intrinsic coordinate**를 먼저 찾고 그 위에서 양자화해야 한다.
    
    이를 일반화된 chart (\phi)로 쓰면,
    
    [
    u = \phi(x), \qquad \hat u = Q_u(u), \qquad \hat x = \phi^{-1}(\hat u)
    ]
    
    이며, 여기서 (\phi)가 random rotation이면 TurboQuant류,(\phi)가 manifold chart이면 manifold-aligned quantization이다.
    
    ## 3. Local manifold model
    
    매니폴드 상의 한 점 (x_0 = f(z_0)) 근방에서, (x)는 tangent-normal decomposition으로 근사할 수 있다.
    
    [
    x \approx x_0 + J_f(z_0)(z-z_0) + r(z;z_0) + \varepsilon
    ]
    
    여기서
    
    • (J_f(z_0) \in \mathbb{R}^{d 	imes m}): Jacobian
    
    • (r(z;z_0)): 2차 이상 곡률 항
    
    • tangent space (T_{x_0}\mathcal{M} = \mathrm{span}(J_f(z_0)))
    
    정규직교 기저를 잡아
    
    [
    U_{x_0} \in \mathbb{R}^{d 	imes m}, \qquad V_{x_0} \in \mathbb{R}^{d 	imes (d-m)}
    ]
    
    라 두면, (U_{x_0})는 tangent basis, (V_{x_0})는 normal basis다.
    
    그러면 (x)는 locally
    
    [
    x = x_0 + U_{x_0} a + V_{x_0} b + \xi
    ]
    
    로 쓸 수 있다. 여기서
    
    • (a \in \mathbb{R}^m): intrinsic coordinate
    
    • (b \in \mathbb{R}^{d-m}): manifold 이탈(normal residual)
    
    • (\xi): 고차 비선형 remainder
    
    이제 핵심 아이디어는:
    
    > 
    
    (a)와 (b)를 같은 방식으로 양자화하면 비효율적이다.
    
    왜냐하면 보통
    
    [
    \mathbb{E}|a|^2 \gg \mathbb{E}|b|^2
    ]
    
    이고, task-relevant 정보는 대부분 tangent 방향에 있다.
    
    ## 4. Manifold-aligned quantizer 정의
    
    ### 4.1 기본 정의
    
    manifold-aligned quantizer (Q_{\mathcal{M}})를 다음과 같이 정의한다.
    
    1. projection / encoding
    
    [
    (a,b) = \Psi(x)
    ]
    
    1. anisotropic quantization
    
    [
    \hat a = Q_T(a), \qquad \hat b = Q_N(b)
    ]
    
    1. reconstruction
    
    [
    \hat x = \Phi(\hat a,\hat b)
    ]
    
    여기서
    
    • (\Psi: \mathbb{R}^d 	o \mathbb{R}^m 	imes \mathbb{R}^{d-m})
    
    • (\Phi: \mathbb{R}^m 	imes \mathbb{R}^{d-m} 	o \mathbb{R}^d)
    
    이며 locally는
    
    [
    \Psi(x) = \big(U_x^	op(x-\pi_{\mathcal M}(x)),, V_x^	op(x-\pi_{\mathcal M}(x))\big)
    ]
    
    혹은 더 실용적으로
    
    [
    \Psi(x)=\big(z, n\big), \quad x=f(z)+n
    ]
    
    으로 둘 수 있다.
    
    그 결과 quantizer는
    
    [
    Q_{\mathcal M}(x)=\Phi\big(Q_T(\Psi_T(x)),, Q_N(\Psi_N(x))\big)
    ]
    
    가 된다.
    
    ### 4.2 Bit allocation이 다른 이유
    
    전체 rate budget (R) 하에서,
    
    [
    R = R_T + R_N
    ]
    
    이고 tangent와 normal에 대해 별도 bit allocation을 한다.
    
    [
    Q_T: \mathbb{R}^m 	o \mathcal{C}_T,\qquad Q_N: \mathbb{R}^{d-m} 	o \mathcal{C}_N
    ]
    
    일반적으로 (R_T)는 tangent 정보 보존을 위해 더 많이, (R_N)은 더 적게 배정된다.
    
    이것은 classical anisotropic transform coding의 manifold generalization으로 해석할 수 있다.
    
    ## 5. 왜 유리한가: rate-distortion 관점
    
    ## 5.1 유클리드 전공간 양자화 vs intrinsic 양자화
    
    전공간에서 (d)-차원 양자화를 하면 high-rate regime에서 distortion은 대략
    
    [
    D_{	ext{ambient}}(R) \asymp C_d \cdot 2^{-2R/d}
    ]
    
    형태를 가진다.
    
    반면 데이터가 실제로 (m)-차원 manifold에 집중되어 있고, normal noise가 작으면 intrinsic coordinate만 잘 양자화해도
    
    [
    D_{	ext{intrinsic}}(R_T) \asymp C_m \cdot 2^{-2R_T/m} + D_N(R_N) + D_{	ext{curv}}
    ]
    
    가 된다.
    
    여기서
    
    • (D_N(R_N)): normal residual 양자화 오차
    
    • (D_{	ext{curv}}): local linearization 혹은 chart mismatch에 의한 curvature error
    
    (m \ll d)이면 동일 rate에서 지수의 분모가 작아져 훨씬 빠른 distortion 감소를 기대할 수 있다.
    
    이게 manifold-aligned quantization의 핵심 이론적 근거다.
    
    ## 5.2 Proposition 스타일 정리
    
    ### Proposition 1 (Intrinsic rate advantage)
    
    데이터 (x)가 compact (m)-dimensional smooth manifold (\mathcal M \subset \mathbb R^d)의 (\sigma)-tubular neighborhood에 집중되어 있고,normal energy가 (\mathbb E|b|^2 \le \sigma^2)라 하자.또한 chart distortion과 curvature remainder가 충분히 작다고 하자.
    
    그러면 manifold-aligned quantizer (Q_{\mathcal M})에 대해 적절한 constants (C_1, C_2 > 0)가 존재하여
    
    [
    \mathbb E|x-Q_{\mathcal M}(x)|^2
    \le
    C_1 2^{-2R_T/m} + C_2 2^{-2R_N/(d-m)} + O(\sigma^2 + \kappa^2 \delta^4)
    ]
    
    를 만족한다. 여기서 (\kappa)는 local curvature bound, (\delta)는 chart radius다.
    
    반면 ambient (d)-dimensional isotropic quantization은 일반적으로
    
    [
    \mathbb E|x-Q_{	ext{amb}}(x)|^2 \ge 	ilde C 2^{-2R/d}
    ]
    
- **ChatGPT: 좋습니다. 아래는 **manifold... (Part 2/3)**
    
    스케일을 가진다.
    
    따라서 (m \ll d)이고 (\sigma,\kappa)가 충분히 작을 때,동일 total rate에서 manifold-aligned quantization이 더 낮은 distortion을 달성할 수 있다.
    
    ## 6. 단순 MSE가 아니라 operator-preserving quantization
    
    LLM에서 중요한 것은 reconstruction이 아니라 attention/retrieval/logit 보존이다.즉 관심 대상은 어떤 bilinear operator
    
    [
    \mathcal O(x,y) = x^	op A y
    ]
    
    혹은 attention의 경우 (A=I)인 dot product다.
    
    양자화 후 오차는
    
    # [
    \Delta_{\mathcal O}(x,y)
    
    \mathcal O(x,y)-\mathcal O(\hat x,\hat y)
    ]
    
    로 정의된다.
    
    [
    = x^	op A y - \hat x^	op A \hat y
    ]
    
    이를 전개하면
    
    # [
    \Delta_{\mathcal O}(x,y)
    
    e_x^	op A y + x^	op A e_y - e_x^	op A e_y
    ]
    
    여기서
    
    [
    e_x = x-\hat x,\qquad e_y = y-\hat y
    ]
    
    이제 manifold-aligned 구조에서는 (e_x)가 tangent/normal로 분해된다.
    
    [
    e_x = U_x e_{x,T} + V_x e_{x,N} + \xi_x
    ]
    
    따라서 operator distortion은
    
    [
    \Delta_{\mathcal O}(x,y)
    \approx
    e_{x,T}^	op U_x^	op A y
    +
    e_{x,N}^	op V_x^	op A y
    +
    x^	op A U_y e_{y,T}
    +
    x^	op A V_y e_{y,N}
    ]
    
    이 된다.
    
    여기서 중요한 점은, task operator (A)가 tangent subspace에 더 민감할 수 있다는 것이다.예를 들어 attention에서는 semantic signal이 특정 low-dimensional subspace에 집중되어 있으면, normal error는 MSE에는 나타나지만 task error에는 덜 민감할 수 있다.
    
    그래서 최적 objective는 단순 MSE가 아니라 다음과 같이 써야 한다.
    
    [
    \min_{Q_{\mathcal M}}
    \mathbb E\Big[
    |x-\hat x|^2
    +
    \lambda \cdot
    \mathbb E_{y \sim P_Y}\big[
    (\mathcal O(x,y)-\mathcal O(\hat x,y))^2
    \mid x
    \big]
    \Big]
    ]
    
    또는 symmetric하게
    
    [
    \min_{Q_{\mathcal M}}
    \mathbb E_{x,y}\Big[
    \alpha |x-\hat x|^2 + \beta |y-\hat y|^2
    
    • \lambda (\mathcal O(x,y)-\mathcal O(\hat x,\hat y))^2
    
    \Big]
    ]
    
    ## 7. TurboQuant를 포함하는 일반화된 형태
    
    TurboQuant는 사실 아래 special case로 볼 수 있다.
    
    ### Case A: Random-coordinate quantization
    
    [
    \phi(x)=Rx,\quad R\in O(d)
    ]
    [
    Q(x)=R^	op q(Rx)
    ]
    
    이는 (\phi)가 manifold chart가 아니라 전역 직교좌표라는 점에서,manifold-aligned quantization의 특수한 “coordinate transform quantization”이다.
    
    ### Case B: Residual-corrected quantization
    
    [
    \hat x = Q_{	ext{main}}(x) + Q_{	ext{res}}(x-Q_{	ext{main}}(x))
    ]
    
    이 구조도 다음의 분해형으로 일반화된다.
    
    [
    x = f(z) + n
    ]
    [
    \hat x = \underbrace{\hat f(\hat z)}_{	ext{functional / manifold part}}
    
    • \underbrace{\hat n}_{	ext{residual / normal part}}
    
    ]
    
    즉 TurboQuant의 stage-1 / stage-2는,manifold 관점에서는
    
    • stage-1: functional approximation
    
    • stage-2: residual correction
    
    으로 해석 가능하다.
    
    ## 8. 실제 알고리즘을 위한 surrogate formulation
    
    실제 학습 또는 offline fitting을 위해서는 true manifold (\mathcal M)를 모르므로 encoder-decoder를 둔다.
    
    [
    z = g_	heta(x), \qquad 	ilde x = f_\phi(z)
    ]
    
    그리고 residual:
    
    [
    n = x - f_\phi(g_	heta(x))
    ]
    
    양자화는 latent와 residual에 대해 separately 수행한다.
    
    [
    \hat z = Q_z(z), \qquad \hat n = Q_n(n)
    ]
    
    재구성은
    
    [
    \hat x = f_\phi(\hat z) + \hat n
    ]
    
    이때 loss는 다음처럼 둘 수 있다.
    
    # [
    \mathcal L
    
    \underbrace{\mathbb E|x-\hat x|^2}*{	ext{reconstruction}}
    +
    \lambda*{	ext{op}}
    \underbrace{\mathbb E_{x,y}[
    (\mathcal O(x,y)-\mathcal O(\hat x,\hat y))^2]}*{	ext{operator preservation}}
    +
    \lambda*{	ext{rate}}
    \underbrace{(R_z + R_n)}*{	ext{bit budget}}
    +
    \lambda*{	ext{jac}}
    \underbrace{\mathbb E |J_{f_\phi}(z)|**}*{	ext{smoothness / low intrinsic rank}}
    ]
    
    여기서 마지막 항은 manifold complexity regularization 역할을 한다.
    
    보다 정보이론적으로는 rate를 엔트로피 surrogate로 대체할 수 있다.
    
    # [
    \mathcal L
    
    \mathbb E|x-\hat x|^2
    +
    \lambda_{	ext{op}}
    \mathbb E(\Delta_{\mathcal O}^2)
    +
    \beta_z H(\hat z)
    +
    \beta_n H(\hat n)
    ]
    
    ## 9. LLM KV-cache에 특화한 formulation
    
    KV cache에서는 토큰 (t), head (h), layer (\ell)별로 key/value 벡터가 있다.
    
    [
    k_{\ell,h,t} \in \mathbb R^{d_h}, \qquad v_{\ell,h,t} \in \mathbb R^{d_h}
    ]
    
    이들이 각 head/layer마다 서로 다른 manifold 근처에 놓인다고 가정하자.
    
    [
    k_{\ell,h,t} = f^{(K)}*{\ell,h}(z^{(K)}*{\ell,h,t}) + n^{(K)}*{\ell,h,t}
    ]
    [
    v*{\ell,h,t} = f^{(V)}*{\ell,h}(z^{(V)}*{\ell,h,t}) + n^{(V)}_{\ell,h,t}
    ]
    
    그러면 양자화도 head/layer-specific하게 설계할 수 있다.
    
    # [
    \hat k_{\ell,h,t}
    
    f^{(K)}*{\ell,h}(Q_z(z^{(K)}*{\ell,h,t}))
    +
    Q_n(n^{(K)}_{\ell,h,t})
    ]
    
    # [
    \hat v_{\ell,h,t}
    
    f^{(V)}*{\ell,h}(Q_z(z^{(V)}*{\ell,h,t}))
    +
    Q_n(n^{(V)}_{\ell,h,t})
    ]
    
    attention logit 보존 objective는
    
    # [
    \mathcal L_{	ext{attn}}
    
    ## \mathbb E\Big[
    \big(q_{\ell,h,t}^	op k_{\ell,h,s}
    
    q_{\ell,h,t}^	op \hat k_{\ell,h,s}\big)^2
    \Big]
    ]
    
    value path까지 포함하면
    
    # [
    \mathcal L_{	ext{ctx}}
    
    ## \mathbb E\Big[
    |
    \mathrm{Attn}(Q,K,V)
    
    \mathrm{Attn}(Q,\hat K,\hat V)
    |^2
    \Big]
    ]
    
    실무적으로는 이걸 direct하게 쓰기 무거우므로, 보통 surrogate로
    
    • key: dot-product preservation
    
    • value: context/output MSE
    
    • 둘 다: layer output drift
    
    를 사용하면 된다.
    
    ## 10. Bit allocation 최적화 문제
    
    manifold-aligned quantization의 진짜 강점은**bit allocation이 구조적으로 정당화**된다는 점이다.
    
    예를 들어 latent dimension별 importance weight (\omega_i), residual dimension별 weight (
    u_j)가 있을 때
    
    [
    \min_{{r_i},{s_j}}
    \sum_{i=1}^m \omega_i D_i(r_i)
    +
    \sum_{j=1}^{d-m}
    u_j E_j(s_j)
    ]
    
    subject to
    
    [
    \sum_{i=1}^m r_i + \sum_{j=1}^{d-m} s_j \le R
    ]
    
    로 둘 수 있다.
    
    여기서
    
    • (r_i): tangent latent bit allocation
    
    • (s_j): residual bit allocation
    
    • (D_i, E_j): 각 성분 distortion-rate function
    
    Lagrangian은
    
    # [
    \mathcal J
    
    \sum_i \omega_i D_i(r_i)
    +
    \sum_j
    u_j E_j(s_j)
    +
    \lambda
    \left(
    \sum_i r_i + \sum_j s_j - R
    \right)
    ]
    
    이고 water-filling 유사 구조가 나온다.
    
    즉,
    
    > 
    
    “어떤 좌표에 몇 bit를 줄 것인가”가ad hoc tuning이 아니라 RD 최적화로 정식화된다.
    
- **ChatGPT: 좋습니다. 아래는 **manifold... (Part 3/3)**
    
    ## 11. 정리된 theorem / conjecture 형태
    
    ### Theorem-style Claim A
    
    데이터가 낮은 곡률을 가진 (m)-차원 manifold의 tubular neighborhood에 집중되어 있고, task operator가 주로 tangent subspace에서 변동한다면,manifold-aligned quantizer는 동일 bit budget 하에서 isotropic ambient quantizer보다 더 낮은 operator distortion을 달성할 수 있다.
    
    ### Claim B
    
    Random rotation 기반 quantization은 coordinate incoherence를 높여 scalar quantization을 쉽게 만드는 범용 baseline이지만,task-aware manifold coordinate를 명시적으로 이용하면 rate-distortion 및 operator-preservation 측면에서 추가 이득이 가능하다.
    
    ### Claim C
    
    Residual correction stage는 manifold 관점에서 normal residual 또는 model mismatch residual을 보정하는 단계로 재해석할 수 있으며,이는 multi-stage residual codec으로 자연스럽게 일반화된다.
    
    ## 12. 논문 본문용 서술 초안
    
    아래 문단은 거의 그대로 논문 초안에 들어갈 수 있습니다.
    
    ### Abstract-level phrasing
    
    We formulate quantization as a coordinate selection problem rather than a value discretization problem. Instead of directly quantizing vectors in the ambient space (\mathbb R^d), we assume that model states lie near a low-dimensional manifold and propose to quantize their intrinsic coordinates and normal residuals separately. This yields a manifold-aligned quantization framework that unifies transform coding, residual quantization, and operator-aware compression under a single geometric view.
    
    ### Method-level phrasing
    
    Given (x \in \mathbb R^d), we decompose it as (x = f(z) + n), where (z \in \mathbb R^m) is the intrinsic latent coordinate and (n) is the normal residual. We then quantize (z) and (n) with separate bit budgets, obtaining (\hat x = f(Q_z(z)) + Q_n(n)). Unlike isotropic quantization in the ambient space, this formulation exploits the low intrinsic dimensionality of the data distribution and enables operator-aware rate allocation.
    
    ### Relation-to-prior-work phrasing
    
    Random-rotation methods can be understood as a special case of coordinate-aware quantization where the coordinate map is chosen from a distribution over orthogonal transforms. Our framework generalizes this perspective by allowing the coordinate map to be manifold-adaptive and task-aware.
    
    ## 13. 실험 섹션으로 연결되는 검증 가설
    
    이 수식화는 다음 실험 가설로 바로 이어집니다.
    
    ### Hypothesis 1
    
    동일 bpc에서ambient scalar quant < random-rotation quant < manifold-aligned quant
    
    ### Hypothesis 2
    
    MSE 기준 개선보다 operator distortion 기준 개선이 더 크게 나타난다.
    
    ### Hypothesis 3
    
    manifold-aligned quantization은 key와 value에서 이득 패턴이 다르다.보통 key는 dot-product 민감, value는 output reconstruction 민감.
    
    ### Hypothesis 4
    
    head-wise 또는 layer-wise local manifold를 쓰면 global manifold보다 더 안정적이다.
    
    ## 14. 한 단계 더 밀어붙인 연구 버전
    
    더 공격적으로 쓰면, 이 프레임은 단순 quantization이 아니라 다음의 일반화로 갈 수 있습니다.
    
    [
    x \approx \sum_{r=1}^{R} \alpha_r , \psi_r(z_r) + n
    ]
    
    즉 single manifold가 아니라 union-of-submanifolds 혹은 mixture-of-charts 구조입니다.
    
    LLM에서는 이것이 자연스럽습니다.
    
    • layer마다 다른 geometry
    
    • head마다 다른 semantic subspace
    
    • token regime마다 다른 local chart
    
    이 경우 quantization은
    
    [
    \hat x =
    \sum_{r=1}^{R}
    Q_\alpha(\alpha_r),
    \psi_r(Q_{z_r}(z_r))
    +
    Q_n(n)
    ]
    
    로 확장될 수 있습니다.
    
    이건 사실상
    
    • low-rank
    
    • dictionary model
    
    • mixture-of-experts
    
    • local chart atlas
    
    를 모두 포괄하는 상위 프레임이 됩니다.
    
    # 결론
    
    manifold-aligned quantization의 핵심 수식은 결국 이것입니다.
    
    [
    x = f(z) + n
    ]
    [
    \hat x = f(Q_z(z)) + Q_n(n)
    ]
    
    그리고 최적화 대상은 단순 reconstruction이 아니라
    
    [
    \min ;
    \mathbb E|x-\hat x|^2
    +
    \lambda ,\mathbb E\big[(\mathcal O(x,y)-\mathcal O(\hat x,\hat y))^2\big]
    +
    \beta_z H(\hat z)
    +
    \beta_n H(\hat n)
    ]
    
    입니다.
    
    이 formulation의 의미는 명확합니다.
    
    • 기존 quantization
    
    • TurboQuant류
    
    • manifold-aligned quantization
    
    이제 이걸 다음 단계로 이어갈 수 있습니다.원하시면 바로 이어서
    
    1. 관련연구 섹션 형태로 변환
    
    1. 정리/정의/정리명(Definition, Proposition, Theorem) 포맷으로 재구성
    
    1. KV-cache용 알고리즘 박스(Algorithm 1)와 pseudo-code 작성
    
    1. 논문 초안 전체 섹션 구조(Abstract, Intro, Method, Experiments)로 확장
    
    중 하나로 바로 전개하겠습니다.