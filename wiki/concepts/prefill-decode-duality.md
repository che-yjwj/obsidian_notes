---
concept: Prefill / Decode Duality
last_compiled: 2026-04-12
topics_connected: [npu-simulator-compiler, riscv-npu-soc-sim, hw-friendly-model-design, llm-quantization-compression]
status: active
---

# Prefill / Decode Duality

## Pattern

LLM inference contains two structurally distinct compute phases that share the same model weights but have opposite bottlenecks, opposite memory-access patterns, and optimal execution strategies that are mutually incompatible if forced through the same pipeline. The vault surfaces this duality independently across compiler design, simulator specification, model architecture, and quantization algorithm design — in each domain, the same bifurcation appears and demands a split treatment.

| Property | Prefill | Decode |
|---|---|---|
| Input shape | Full prompt (sequence T tokens) | Single next token |
| Core operation | GEMM (matrix × matrix) | GEMV (matrix × vector) |
| Bottleneck | Compute (FLOPS) | Memory (KV-cache read BW) |
| KV traffic | Write-heavy (populate cache) | Read-heavy (scan full history) |
| Latency metric | Time To First Token (TTFT) | Per-token latency |
| Ideal NPU mode | Coarse-grained tiles, throughput | Fine-grained tiles, low latency |

## Instances

- **2026-04** in [[../topics/npu-simulator-compiler]]: The supernode abstraction explicitly splits into Prefill Supernode (coarse-grained, GEMM, aggressive SPM, static schedule, throughput target) and Decode Supernode (fine-grained, GEMV, minimal SPM, dynamic/token-driven, latency target). The Prefill/Decode distinction is embedded in the IR: `npu.supernode @attn_prefill { mode = "prefill" }` vs `npu.supernode @attn_decode { streaming = true }`. HyperAccel LPU is specifically optimized for the decode side.

- **2026-04** in [[../topics/riscv-npu-soc-sim]]: The RISCV_NPU_SoC_SIM spec explicitly models "LLM Prefill vs Decode performance characteristics" as distinct scenarios. The `KV_UPDATE` and `KV_CACHE_RESIZE` LayerIR opcodes exist specifically for the decode-phase incremental KV growth. The project targets "Mixed-precision quantization, KV-cache, Prefill/Decode 분리 시나리오 지원" as first-class requirements.

- **2026-04** in [[../topics/hw-friendly-model-design]]: Gemma 3's 5-to-1 local/global attention interleaving is a model-level response to decode-phase KV bandwidth pressure. The optimization directly trades prefill compute cost (extra attention computation over local window) for decode memory savings (only W=1024 tokens of KV per local layer vs. full T). KV Cache Sharing in Gemma 3n reduces TTFT (prefill metric) by ~2×, independently of decode optimization.

- **2026-04** in [[../topics/llm-quantization-compression]]: "KV cache quantization targets the decode bottleneck specifically, where every generated token reads the entire KV history." TurboQuant is explicitly a decode-phase algorithm — its 2-stage online VQ runs per-token during the decode phase. The "honest comparison" metric (bpc) exists because naive KV quantization reporting conflates prefill and decode costs.

## What This Means

The prefill/decode duality is not just a performance engineering detail — it is a **fundamental bifurcation** in the LLM inference problem that propagates up and down the stack. A hardware design that treats inference as a single uniform workload will be suboptimal on both sides: too coarse for decode latency, too fine-grained for prefill throughput.

The consistent vault-wide finding is that the right response is **explicit bifurcation at every layer**: separate IR node types (riscv-npu-soc-sim, npu-simulator-compiler), separate quantization algorithms per phase (llm-quantization-compression), and separate model-level attention patterns per layer position (hw-friendly-model-design).

The practical implication for design decisions: when evaluating a new NPU architecture or compiler pass, ask separately "what is the prefill throughput?" and "what is the per-token decode latency?" — not a single end-to-end benchmark. The two metrics require fundamentally different optimizations and will often trade off against each other.

## Sources

- [[../topics/npu-simulator-compiler]]
- [[../topics/riscv-npu-soc-sim]]
- [[../topics/hw-friendly-model-design]]
- [[../topics/llm-quantization-compression]]
