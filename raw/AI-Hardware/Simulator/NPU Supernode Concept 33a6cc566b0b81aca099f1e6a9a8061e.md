# NPU 슈퍼노드 개념

ChatTime: 2026년 1월 12일 16:34 (GMT+9)
Created time: 2026년 4월 6일 오후 11:09
ProjectName: NPU Simulator
URL: https://chatgpt.com/c/6964a2a8-1060-8321-af05-4d50735148cb

- **You: Npu컴파일러에서 슈퍼노드 개념 알려...**
    
    Npu컴파일러에서 슈퍼노드 개념 알려줘
    
- **ChatGPT: NPU 컴파일러에서 **슈퍼노드(Su... (Part 1/2)**
    
    NPU 컴파일러에서 **슈퍼노드(Supernode)** 는**그래프 레벨 최적화와 하드웨어 친화적 실행 단위를 연결하는 핵심 추상화**입니다.단순한 “연산 묶음”이 아니라, **메모리·스케줄·ISA·마이크로아키텍처 제약을 동시에 만족하도록 설계된 실행 단위**라고 보는 것이 정확합니다.
    
    아래에서 개념 → 필요성 → 정의 → 생성 과정 → 하드웨어 관점 → 연구/설계 확장 포인트 순으로 정리하겠습니다.
    
    ## 1. 왜 슈퍼노드가 필요한가 (문제의식)
    
    일반적인 DNN 그래프는 다음과 같은 형태입니다.
    
    ```
    MatMul → Add → Bias → LayerNorm → GELU → Mul
    ```
    
    그래프 관점에서는 **여러 개의 작은 operator 노드**이지만,하드웨어 관점에서는 다음 문제가 발생합니다.
    
    • 각 op마다
    
    • DRAM ↔ SRAM 왕복
    
    • 커널 런치 / 명령 디스패치
    
    • 파이프라인 flush
    
    • 실제 연산보다 
    
    ➡️ **“그래프 단위 최적화”만으로는 NPU 성능 한계가 명확**
    
    ## 2. 슈퍼노드의 정의 (정확한 의미)
    
    > 
    
    **Supernode = 하나의 하드웨어 실행 단위로 묶여 실행되는 연산자 부분 그래프**
    
    조금 더 엄밀히 말하면:
    
    • 연속된 operator subgraph
    
    • 중간 결과를 외부 메모리에 쓰지 않음
    
    • NPU 내부 로컬 메모리(SPM / Tile SRAM)에 유지
    
    • 하나의 커맨드/마이크로커널로 실행
    
    즉,
    
    ```
    [Graph Level Node 묶음]
            ↓
    [Single HW Execution Region]
    ```
    
    ## 3. 슈퍼노드 vs 일반 노드 vs Fusion
    
    | 개념 | 의미 | 한계 |
    | --- | --- | --- |
    | Operator | 단일 연산 | 오버헤드 큼 |
    | Fusion | 단순 연산 결합 | HW 제약 고려 부족 |
    | **Supernode** | HW-aware 실행 블록 | ✔ 메모리·파이프라인·ISA 반영 |
    
    👉 **슈퍼노드는 “하드웨어 의미를 가진 fusion”**
    
    ## 4. 슈퍼노드 내부 구조 (개념도)
    
    image_group {"query":["NPU supernode execution diagram","neural network operator fusion pipeline","NPU tile based execution block diagram","MLIR fusion hardware execution diagram"]}
    
    ```
    Supernode
     ├─ Input Tile Load (DMA)
     ├─ Tensor Engine (MatMul)
     ├─ Vector Engine (Add / Bias / Norm)
     ├─ Activation Unit (GELU / ReLU)
     ├─ Optional Reduce / Scale
     └─ Output Tile Store (DMA)
    ```
    
    • 중간 tensor는 외부로 나오지 않음
    
    • 타일 단위로 
    
    • TE/VE/Scalar 유닛이 파이프라인으로 연결됨
    
    ## 5. 슈퍼노드 생성 과정 (컴파일러 관점)
    
    ### ① 그래프 분석
    
    • Topological order
    
    • Data dependency graph
    
    • Shape / stride / broadcast 분석
    
    ### ② Fusion 후보 탐색
    
    • Producer–consumer chain
    
    • Elementwise + reduction 패턴
    
    • Layout compatible 여부
    
    ### ③ HW 제약 필터링 (핵심)
    
    • SPM 용량 초과?
    
    • Tile 크기 유지 가능?
    
    • TE ↔ VE 파이프라인 연결 가능?
    
    • Precision / quantization 호환?
    
    ➡️ **여기서 살아남은 subgraph만 슈퍼노드**
    
    ## 6. 슈퍼노드와 타일 기반 NPU의 관계 (중요)
    
    창훈님이 설계 중인 **Tile-based NPU 구조**에서는:
    
    > 
    
    **슈퍼노드 = “타일 라이프사이클을 공유하는 연산 묶음”**
    
    ```
    Tile Load
      ↓
    [Supernode Execution]
      ↓
    Tile Store
    ```
    
    • 타일은 슈퍼노드 경계를 넘지 않음
    
    • 슈퍼노드 내부에서만:
    
    • double buffering
    
    • partial sum accumulation
    
    • reuse 극대화
    
    👉 **슈퍼노드 경계 = 메모리 트래픽 경계**
    
    ## 7. 슈퍼노드와 ISA / IR 연결
    
    슈퍼노드는 보통 다음 레벨에서 명시됩니다.
    
    ### IR 레벨
    
    • MLIR / NPU-IR에서:
    
    • npu.supernode { ... }
    
    • region
    
    • 내부에 여러 op 포함
    
    ### ISA 레벨
    
    • 하나의 
    
    • 또는:
    
    • setup + loop + compute + drain
    
    • VCIX / custom opcode와 잘 맞음
    
    ## 8. LLM에서의 대표적인 슈퍼노드 예시
    
    ### (1) FFN Block
    
    ```
    MatMul → Bias → GELU → MatMul → Bias
    ```
    
    → **하나의 슈퍼노드**
    
    • intermediate hidden vector 외부 저장 ❌
    
    • activation fusion 필수
    
    ### (2) Attention Tile
    
    ```
    QK^T → Softmax → V
    ```
    
    • FlashAttention 스타일
    
    • softmax state를 tile-local로 유지
    
    • 슈퍼노드 없이는 불가능
    
    ## 9. 연구/설계 관점에서 중요한 확장 포인트
    
    ### ① 슈퍼노드 자동 분할 (Adaptive)
    
    • SPM pressure 기반 split
    
    • decode vs prefill 다른 전략
    
    ### ② 슈퍼노드 성능 모델링
    
    • compute-bound / memory-bound 판별
    
    • energy per supernode
    
    ### ③ 슈퍼노드 = scheduling unit
    
    • OOO dispatch
    
    • ROB retire 단위
    
    • Gantt chart 시각화에 최적
    
    ## 10. 핵심 요약 (한 문장)
    
    > 
    
    **NPU 컴파일러에서 슈퍼노드는 “그래프와 하드웨어 사이의 실질적인 실행 계약(execution contract)”이다.**
    
    • 그래프 최적화 ❌
    
    • 커널 fusion ❌
    
    • HW-aware execution abstraction ✔
    
    다음 단계로 추천할 수 있는 주제는:
    
    • 슈퍼노드 정의를 NPU-IR spec으로 문서화
    
    • 슈퍼노드 단위 성능/에너지 모델
    
- **ChatGPT: NPU 컴파일러에서 **슈퍼노드(Su... (Part 2/2)**
    
    • Prefill / Decode용 슈퍼노드 분리 전략
    
    • VCIX 기반 슈퍼노드 lowering 예제
    
    원하시면 **NPU Simulator용 `supernode.md` 스펙 초안**으로 바로 정리해드릴 수 있습니다.
    
- **ChatGPT: 아래에서는 **“Prefill / D...**
    
    아래에서는 **“Prefill / Decode용 슈퍼노드 분리 전략”**을**NPU 컴파일러 + 타일 기반 아키텍처 관점**에서 **왜 분리해야 하는지 → 어떻게 분리하는지 → IR/스케줄/하드웨어에 어떤 차이가 생기는지** 순서로 정리합니다.
    
    ## 1. 왜 Prefill / Decode를 같은 슈퍼노드로 두면 안 되는가
    
    LLM 추론은 본질적으로 **두 개의 전혀 다른 워크로드**입니다.
    
    | 구분 | Prefill | Decode |
    | --- | --- | --- |
    | 시퀀스 길이 | 김 (L ≫ 1) | 1 |
    | 주요 연산 | GEMM (MxM) | GEMV (MxV) |
    | 병렬성 | 높음 (token × head × dim) | 낮음 |
    | 병목 | Compute / SRAM | Memory / KV access |
    | 최적화 목표 | Throughput | Latency |
    
    ➡️ **동일한 슈퍼노드 구조를 쓰면 둘 중 하나는 반드시 손해**입니다.
    
    ## 2. 핵심 아이디어 (요약)
    
    > 
    
    **슈퍼노드는 “연산 묶음”이 아니라 “실행 전략 단위”이므로,
    
    Prefill과 Decode는 서로 다른 슈퍼노드 타입을 가져야 한다.**
    
    즉,
    
    • Prefill Supernode = Compute-dense tile pipeline
    
    • Decode Supernode = KV-cache–aware streaming micro-kernel
    
    ## 3. Prefill용 슈퍼노드 전략
    
    ### 3.1 구조적 특징
    
    image_group {"query":["LLM prefill attention tile pipeline","NPU prefill supernode tile execution","FlashAttention prefill block diagram"]}
    
    ```
    [Tile Load Q/K/V]
          ↓
    [QK^T  (TE)]
          ↓
    [Softmax (VE, tile-local)]
          ↓
    [Attn × V (TE)]
          ↓
    [Tile Store]
    ```
    
    ### 3.2 설계 포인트
    
    • 타일 중심
    
    • (Tq × Tk) block
    
    • head 병렬
    
    • 슈퍼노드 크기 큼
    
    • 여러 op를 하나의 region으로 묶음
    
    • SPM 재사용 극대화
    
    • Q/K/V, softmax state 모두 on-chip
    
    • FlashAttention 계열과 정합
    
    ### 3.3 컴파일러 관점
    
    • 슈퍼노드 생성 기준:
    
    • MatMul → Softmax → MatMul
    
    • Cost model:
    
    • FLOPs / tile
    
    • 목표:
    
    • Throughput 최적화
    
    • DRAM access 최소화
    
    ## 4. Decode용 슈퍼노드 전략 (가장 중요)
    
    Decode는 **Prefill과 완전히 다른 사고방식**이 필요합니다.
    
    ### 4.1 구조적 특징
    
    image_group {"query":["LLM decode KV cache access pattern","NPU decode attention streaming diagram","KV cache decode attention block"]}
    
    ```
    [Load Q (1 token)]
          ↓
    [Stream K/V from KV-cache]
          ↓
    [Incremental QK^T + Softmax Update]
          ↓
    [Accumulate Attn × V]
          ↓
    [Output token]
    ```
    
    ### 4.2 설계 포인트
    
    • 타일 개념이 약함
    
    • Q는 1×D
    
    • K/V는 time-accumulated
    
    • KV cache access가 지배적
    
    • DRAM / LLC / SLC latency
    
    • 슈퍼노드는 작고 반복적
    
    • loop-based micro-supernode
    
    • 상태 유지 중요
    
    • softmax 분모/최대값 유지
    
    ### 4.3 컴파일러 관점
    
    • 슈퍼노드 분해:
    
    • “Attention 전체” ❌
    
    • “1-step attention update” ✔
    
    • Cost model:
    
    • memory latency / token
    
    • 목표:
    
    • Token latency 최소화
    
    ## 5. Prefill / Decode 슈퍼노드의 결정적 차이
    
    | 항목 | Prefill Supernode | Decode Supernode |
    | --- | --- | --- |
    | 크기 | 큼 (coarse-grained) | 작음 (fine-grained) |
    | 반복성 | 낮음 | 매우 높음 |
    | 병렬성 | Token-level | Head-level |
    | 주 연산 | GEMM | GEMV |
    | 병목 | Compute | Memory |
    | SPM 사용 | Aggressive | Minimal |
    | Scheduler | Static | Dynamic |
    
    ## 6. IR 레벨에서의 분리 전략 (권장)
    
    ### 6.1 Supernode Type 명시
    
    ```vbnet
    npu.supernode @attn_prefill {
      mode = "prefill"
      tile = (Tq, Tk)
    }
    ```
    
    ```vbnet
    npu.supernode @attn_decode {
      mode = "decode"
      streaming = true
    }
    ```
    
    → **컴파일러가 lowering / scheduling을 다르게 적용**
    
    ## 7. 스케줄러 & 런타임 영향
    
    ### Prefill
    
    • static scheduling
    
    • large tile loop
    
    • DMA overlap 적극 활용
    
    ### Decode
    
    • token-driven dispatch
    
    • KV cache locality 기반 head ordering
    
    • speculative prefetch 가능
    
    ## 8. 하드웨어 설계에 주는 시사점 (중요)
    
    • 같은 NPU라도
    
    • Prefill: TE utilization이 핵심
    
    • Decode: DMA + cache hierarchy가 핵심
    
    • 슈퍼노드 분리는:
    
    • 컴파일러 최적화 ❌
    
    • SoC memory 설계까지 연결되는 결정
    
    ## 9. 한 문장 요약
    
    > 
    
    **Prefill/Decode 슈퍼노드 분리는 “LLM을 하나의 모델이 아니라, 두 개의 전혀 다른 하드웨어 워크로드로 재정의하는 작업”이다.**
    
    다음으로 이어서 매우 잘 맞는 주제는:
    
    • Decode 슈퍼노드를 위한 KV-cache layout (GQA/MQA, interleaved heads)
    
    • 슈퍼노드 단위 성능 모델 (latency vs throughput)
    
    • Prefill/Decode dual-path NPU 스케줄러 설계
    
    원하시면 이 내용을 **NPU Simulator용 설계 문서(`supernode_prefill_decode.md`)** 형태로 바로 고정 문서화해드릴 수 있습니다.