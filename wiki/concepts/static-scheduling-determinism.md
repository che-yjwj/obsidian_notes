---
concept: Static Scheduling and Deterministic Execution
last_compiled: 2026-04-12
topics_connected: [npu-architecture, npu-simulator-compiler, riscv-npu-soc-sim, hw-friendly-model-design, patent-mcm]
status: active
---

# Static Scheduling and Deterministic Execution

## Pattern

A recurring architectural philosophy across chip design, compiler design, simulator design, and model architecture is the preference for **static, compile-time determinism over dynamic, runtime flexibility**. Each source arrives at this preference for different reasons — verification cost, energy predictability, performance headroom — but they all move in the same direction: push decisions from runtime to compile time, eliminate dynamic state from the execution path, and make worst-case behavior provable rather than measured.

The pattern runs from the silicon level (NPU ISA design) through the compiler level (offline command queue generation) to the model level (normalization-free fixed-scale transformers), suggesting this is not a microarchitectural preference but a systems-level philosophy with implications at every design layer.

## Instances

- **2026-04** in [[../topics/npu-architecture]]: Tesla FSD AI5 uses "정적 스케줄 + micro-code: GPU dynamic warp scheduling의 반대 방향. 컴파일러가 타이밍을 결정." AMD Versal AIE uses "deterministic pipeline, 2D Mesh interconnect, 컴파일 타임 정적 라우팅." The rationale: "아키텍처가 단순할수록(고정 ISA, 정적 스케줄, SRAM scratchpad, 제한된 상태공간) 검증 범위가 좁아지고 tape-out 주기가 단축된다" — simple static architecture means 9-month tape-out cycles.

- **2026-04** in [[../topics/npu-simulator-compiler]]: Groq LPU is described as "컴파일러가 모든 cycle을 고정 (deterministic inference)." The supernode concept is fundamentally a static execution contract — all tiling, scheduling, and DMA timing are resolved at compile time, not runtime. Prefill/Decode supernodes are explicitly typed at IR level to enable static schedule specialization per phase.

- **2026-04** in [[../topics/riscv-npu-soc-sim]]: The CMDQ is "오프라인에서 생성되며 런타임에 NPU 내부에는 복잡한 스케줄러가 필요 없다" — the simulator's only input is a pre-compiled command queue. Static scheduler invariant rules include determinism as a hard constraint. Synchronization in the IR is explicit (Tag + Wait), not implicit via hardware coherence.

- **2026-04** in [[../topics/hw-friendly-model-design]]: SNFT's residual scaling constant `c_l` is a compile-time scalar — "folds into the preceding multiply-accumulate at compile time." This is explicitly contrasted with DyT's token-wise dynamic gate which "makes static scheduling and INT8 quantization difficult on NPUs." Static-friendly models are preferred for ASIC targets.

- **2026-04** in [[../topics/patent-mcm]]: Taalas's "model-as-silicon ASIC" paradigm takes static execution to its logical extreme — inference weights become hardwired constants in shift/add networks, eliminating the memory-fetch step entirely. MCM's Voronenko A-graph is computed offline; the resulting DAG is a fully static circuit.

## What This Means

The vault's preference for static scheduling is not merely a microarchitectural preference — it is a **verification and tape-out strategy**. Dynamic dispatch requires exhaustive runtime testing; static dispatch requires only compile-time formal analysis. For hardware teams with 9-month release cycles, reducing the state space that needs verification is not a nice-to-have but a delivery constraint.

The implication for model design is underappreciated: **model architectures that introduce runtime control flow (MoE routing, dynamic early exit, token-wise gates) impose a hidden tax on NPU efficiency** that shows up not in peak TOPS but in verification cost, pipeline bubble rate, and scheduler overhead. SNFT vs. DyT is not just an accuracy tradeoff but an NPU-targeting strategy.

The open question this pattern raises: as LLM batch sizes and sequence lengths become dynamic at the system level (continuous batching, speculative decoding), can fully static NPU execution models adapt — or does the industry ultimately need a two-level architecture (static inner loops, dynamic outer scheduler)?

## Sources

- [[../topics/npu-architecture]]
- [[../topics/npu-simulator-compiler]]
- [[../topics/riscv-npu-soc-sim]]
- [[../topics/hw-friendly-model-design]]
- [[../topics/patent-mcm]]
