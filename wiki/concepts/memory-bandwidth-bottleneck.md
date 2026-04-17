---
concept: Memory Bandwidth as Primary Bottleneck
last_compiled: 2026-04-12
topics_connected: [llm-quantization-compression, operator-coordinate-compression, npu-architecture, hw-friendly-model-design, npu-simulator-compiler, riscv-npu-soc-sim]
status: active
---

# Memory Bandwidth as Primary Bottleneck

## Pattern

Across every layer of this vault — quantization algorithms, chip architecture, model design, simulator design — the same conclusion recurs independently: **in modern LLM inference, the bottleneck is not arithmetic throughput but memory bandwidth and capacity**. Each topic arrives at this conclusion from a different angle, but they all converge on the same implication: the decisive design choice is how to minimize, stage, and predict data movement, not how to maximize FLOPS.

This pattern appears at five distinct levels of abstraction simultaneously. That simultaneous appearance — algorithm, architecture, model design, compiler, and simulator specification — is what makes it architecturally foundational rather than a local optimization concern.

## Instances

- **2026-04** in [[../topics/llm-quantization-compression]]: Autoregressive decode is "dominated not by compute but by **memory bandwidth and capacity** — loading weight tensors and KV cache from DRAM on every token step." The entire quantization research direction (TurboQuant, OCEAN, OliVe) is motivated by reducing DRAM traffic, not FLOPs.

- **2026-04** in [[../topics/operator-coordinate-compression]]: coordinate alignment, coefficient concentration, and entropy-aware decomposition are all valuable only insofar as they shrink effective bit-rate and DRAM traffic. This topic supplies the geometric explanation for why the bandwidth bottleneck can be attacked upstream at the representation level.

- **2026-04** in [[../topics/npu-architecture]]: The SRAM scratchpad vs. cache debate is framed as "Cache is not a device to improve performance — it is a safety net for uncertainty that SRAM-only designs cannot handle." AMBA bus tiers are designed around separating control-plane from high-throughput data-plane. Memory Hierarchy in AI sources explicitly compare SRAM/HBM/LPDDR tradeoffs across H100, Qualcomm AHPM, and AMD MI300.

- **2026-04** in [[../topics/hw-friendly-model-design]]: The KV-cache memory formula (`2·L·T·H·d_head·bytes`) is derived as the "hard wall." Gemma 3's 5-to-1 local/global attention interleaving and MSA's Top-K retrieval both exist to keep active KV data in SRAM rather than DRAM.

- **2026-04** in [[../topics/npu-simulator-compiler]]: Meta MTIA is described as "memory-centric (DRAM BW first)." HyperAccel LPU is specifically designed around decode GEMV being memory-bound (not GEMM-bound like prefill). The Prefill/Decode supernode split exists precisely because their bottlenecks differ.

- **2026-04** in [[../topics/riscv-npu-soc-sim]]: The RISCV_NPU_SoC_SIM project enforces that "DRAM must not be used as intermediate result transfer between engines." SPM is the sole shared tile payload store. KV cache gets its own `qbits_kv` field separate from weights and activations.

## What This Means

The vault's coverage is almost entirely from the perspective of inference, not training — and inference is memory-bound in a way training is not. This has a structural consequence: **techniques and architectures that optimize FLOPs efficiency without co-optimizing memory traffic will consistently underperform their theoretical peak**.

More concretely: when evaluating a new quantization technique, chip design, or model architecture from this vault's perspective, the first question to ask is not "what is the TOPS?" but "what is the effective memory bandwidth utilization, and what is the working set size at each memory tier?" The answer to those questions predicts real-world inference performance better than roofline arithmetic alone.

The pattern also implies a research gap: most of the sources address the memory bottleneck independently within their domain. There is no source in this vault that co-optimizes across all five levels simultaneously — algorithm × architecture × model design × compiler × simulator.

## Sources

- [[../topics/llm-quantization-compression]]
- [[../topics/operator-coordinate-compression]]
- [[../topics/npu-architecture]]
- [[../topics/hw-friendly-model-design]]
- [[../topics/npu-simulator-compiler]]
- [[../topics/riscv-npu-soc-sim]]
