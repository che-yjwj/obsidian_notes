# HW-Friendly Model Design

*last_compiled: 2026-04-12 | sources: 4*

---

## Summary [coverage: medium -- 4 sources]

HW-friendly model design is the discipline of co-designing neural network architectures with the physical constraints of the hardware that runs them — SRAM capacity, DRAM bandwidth, NPU dataflow, quantization precision, and control-flow regularity. Rather than training a large model and compressing it post-hoc, HW-friendly design treats power budget, memory footprint, and scheduling predictability as first-class training objectives.

The motivation is sharpest at the edge: on-device and mobile SoCs have fixed SRAM (often a few MB), shared memory bandwidth across CPU/NPU/DSP, no HBM, and strict thermal envelopes. A model that ignores these realities will be DRAM-bandwidth-bound, suffer KV-cache explosion during long-context decode, or stall pipelines with irregular control flow. Four interconnected strategies address this: (1) reducing and structuring KV-cache memory access, (2) eliminating or restructuring normalization layers, (3) introducing learnable sparse memory hierarchies to replace brute-force full attention, and (4) designing latent representations that are inherently tensor-friendly and amenable to quantization.

---

## Core Concepts [coverage: medium -- 4 sources]

### KV-Cache Memory Access Patterns

During autoregressive decode, every transformer layer must read cached K and V tensors for all previous tokens. The total footprint scales as:

```
KV_total ≈ 2 · L · T · H · d_head · bytes_per_elem
```

where L = number of layers, T = sequence length, H = number of heads, d_head = head dimension. Both T and L create linear growth; on-device DRAM bandwidth is the hard wall.

**Interleaved Local/Global Attention (Gemma 3 / Gemma 3n):** The 5-to-1 interleaved pattern uses 5 sliding-window local layers (window W = 1024 tokens) for every 1 full-context global layer. Local layers only need to store W tokens of KV, not the full T. Because 5/6 of layers are local, the DRAM-bound regime is reached far later, and the local KV can potentially reside in on-chip SRAM or SLC while only global layers touch DRAM.

**KV Cache Sharing (Gemma 3n):** Middle-layer K and V are shared by upper layers, cutting the number of independent KV writes during prefill and reducing Time-To-First-Token (TTFT) by approximately 2x.

**Memory Sparse Attention (MSA):** Takes the bandwidth argument further by making KV access selective at the architectural level. During offline preprocessing, each 64-token document chunk is encoded into a KV cache block plus a low-dimensional Router Key. At query time, only the Top-K most similar chunks (via cosine similarity on Router Keys) are fetched into GPU/NPU, so active compute context stays in the thousands of tokens even when the indexed corpus is 100 M tokens. Complexity drops from O(n²) to O(n·k).

### Normalization-Free Transformers

LayerNorm (LN) and RMSNorm require per-token reduction, division, and learned scale/bias parameters. These operations are unfriendly on NPUs because they (a) introduce data-dependent activation ranges that complicate INT8 calibration, (b) add reduction latency to every layer boundary, and (c) break pipeline regularity.

**SNFT (Stronger Normalization-Free Transformer)** replaces normalization with a statically designed residual scaling:

```
x_{l+1} = x_l + c_l · f_l(x_l)
```

where `c_l` is a compile-time constant determined by the joint initialization/scaling design, not a runtime computation. The approach provides theoretical bounds on both forward activation variance and backward gradient variance across arbitrary depth, making training as stable as LN-based models. Because `c_l` is a static scalar, it folds into the preceding multiply-accumulate at compile time — the ISA-level operation becomes `ADD x, alpha_l * y`, directly fusable on a Tensor Engine / Vector Engine pipeline.

**DyT (Dynamic Transformer) contrast:** DyT uses a learned, token-wise gate `α_l(x_l)` that can skip or emphasize layers per token. While this reduces average FLOPs, it introduces irregular control flow (token-wise branching), making static scheduling and INT8 quantization difficult on NPUs. SNFT is the preferred approach for ASIC/NPU targets; DyT is useful as an offline analysis tool for identifying layer importance and guiding static pruning.

### Conditional / Sparse Computation

**MatFormer (Matryoshka Transformer, used in Gemma 3n):** Nested multi-granularity parameter activation enables the model to expose sub-models of different capacities within a single weight set. Only the parameters relevant to the current task and modality are loaded and activated.

**Per-Layer Embedding (PLE) Caching:** Embeddings are cached per layer rather than recomputed, reducing runtime memory pressure and allowing the model to operate within mobile SRAM budgets.

**Conditional Parameter Loading:** Vision and audio encoder parameters are loaded only when the corresponding modality is present in the input, reducing baseline memory footprint for text-only queries.

### RAG Internalization and Learnable Memory Hierarchies

Traditional RAG uses an external retriever (vector DB) decoupled from the generator LLM, with no gradient flow between the two components. MSA internalizes retrieval: the Router Key function is trained end-to-end with the attention module, so gradient flows from attention loss back into retrieval, tightening query-document alignment. The architecture resembles a CPU memory hierarchy:

```
L0: active token window (SRAM, few thousand tokens)
L1: selected KV chunks (HBM / fast DRAM, Top-K × 64 tokens)
L2: compressed memory bank (DRAM, full corpus KV)
L3: full corpus storage (SSD / persistent memory)
```

### Latent Action Representations (World Models)

World models that learn Latent Actions from state-transition pairs `(s_t, s_{t+1})` produce low-dimensional continuous latent vectors z_t that are inherently tensor-friendly (regular shape, normalizable distribution, quantizable via KL regularization). This stands in contrast to raw action spaces that are often discrete, high-dimensional, or sparse. The dynamics model `p(s_{t+1} | s_t, z_t)` operates as an autoregressive decoder with the same prefill/decode dataflow as LLM inference, making it directly runnable on LLM-inference NPUs.

---

## Architecture [coverage: medium -- 4 sources]

### Memory Bandwidth Reduction

The central HW constraint across all sources is DRAM bandwidth. Strategies are hierarchical:

1. **Reduce total KV stored:** Interleaved local attention caps per-layer KV at W = 1024 rather than T = 32K. KV Cache Sharing reduces total unique KV writes per prefill.
2. **Make access patterns regular:** Local attention layers produce streaming, cache-local access (read/write the last W tokens only). This allows DMA prefetch and tile-based NPU scheduling. Global attention layers access the full T but can be isolated and treated differently.
3. **Make access selective:** MSA fetches only Top-K KV chunks, transforming a uniform HBM scan into a router-guided sparse fetch. The router key is a low-dimensional embedding, fast to compute; the similarity search (cosine over all chunk keys) becomes a hardware-acceleratable SIMD kernel.

### SRAM Utilization

On a mobile/edge NPU, on-chip SRAM (typically 2–16 MB) is the fast scratchpad. HW-friendly design tries to keep hot data in SRAM:

- Local attention KV (W = 1024 tokens × heads × d_head × INT8) is more likely to fit in SRAM than the full context; global layers spill to DRAM.
- SNFT's static scaling constants are folded into the compiled weight tensor — no extra SRAM budget for normalization parameters.
- With Gemma 3n's sequence-length discipline (32K maximum, structured 5:1 interleaving), SRAM KV budgets can be computed at design time and used to drive SRAM size specifications during chip architecture.

### Compute Scheduling

- **SNFT** produces a fully static compute graph: every layer has fixed shape, fixed scaling factors, no data-dependent branches. Compile-time scheduling, loop tiling, and constant folding apply without dynamic dispatch.
- **MSA** introduces a two-stage schedule: router stage (similarity search + Top-K selection) followed by attention stage (sparse KV load + attention kernel). The router stage is a new hardware primitive (similarity engine + top-K engine); the attention stage is standard but operates on a sparse, pre-fetched KV tile.
- **Latent Action World Model rollout** is a sequential `s_t → z_t → s_{t+1}` loop structurally identical to LLM token-by-token decoding: the same tile-based streaming NPU dataflow applies.

### Quantization Friendliness

- LN removal (SNFT) makes activation dynamic ranges bounded and predictable, enabling stable INT8 / mxINT8 / microscaling quantization without per-token calibration artifacts.
- MatFormer and conditional loading reduce peak activation memory, leaving more headroom for INT8 accumulation buffers.
- Latent action vectors with KL regularization have controlled range and can be quantized to INT8 with minimal loss.
- MSA's independent per-chunk RoPE application means each attention computation starts with position 0 within its chunk, eliminating long-range positional extrapolation failures under quantization.

---

## Key Findings [coverage: medium -- 4 sources]

### From KV-cache Optimization (Gemma 3 / Gemma 3n analysis)

- The 5-to-1 interleaved attention structure is not merely an accuracy-performance tradeoff but a deliberate memory hierarchy design: 5/6 of layers operate with predictable, SRAM-resident local KV, while only 1/6 require full-T DRAM access.
- KV Cache Sharing in Gemma 3n targets prefill latency (TTFT) specifically, not decode memory — a distinction important when modeling prefill vs. decode separately on NPU.
- On-device context length is a design parameter, not a benchmark target: 32K with structured access is more useful than 128K with flat full-attention.
- Gemma 3n qualifies as a reference model for edge NPU architecture validation because its multimodal encoder, interleaved attention, and conditional parameter loading map directly onto NPU Tensor Engine / Vector Engine / DSP heterogeneous execution.

### From MSA and RAG Analysis

- MSA is correctly characterized as "attention-based memory hierarchy," not a context-length extension. Active compute context remains a few thousand tokens regardless of indexed corpus size.
- The true differentiator of MSA over RAG is end-to-end gradient flow between the retrieval router and the attention computation, which produces approximately 16% accuracy improvement on retrieval-grounded tasks.
- Independent per-chunk RoPE solves the positional extrapolation problem without fine-tuning on long sequences: each 64-token chunk resets to position 0, making 100 M-token indexing feasible from a 64 K-trained model.
- The NPU hardware implications of MSA require three new primitives: a similarity search engine (cosine accelerator), a Top-K selection engine (parallel partial sort), and a KV prefetch engine (DMA with address list from Top-K output).
- Context window scaling does not eliminate the need for RAG; it raises the token threshold at which RAG becomes necessary, but enterprise-scale knowledge bases (billions of tokens, real-time updates) always exceed any practical context budget.

### From Stronger Normalization-Free Transformers (SNFT)

- SNFT provides theoretical guarantees (bounded forward activation variance and backward gradient variance) that prior NF methods (FixUp, ScaleNorm, DeepNorm) only provided heuristically.
- At deep network scale (hundreds of layers), SNFT matches or improves over LN Transformer in perplexity while being fully normalization-free.
- The compile-time constant nature of SNFT scaling (`c_l`) is the critical property for NPU deployment: no runtime reduction, no division, no dynamic parameter load. Full constant folding and ISA-level fusion are enabled.
- DyT (Dynamic Transformer) is unsuitable as a primary NPU architecture because its token-wise gating produces irregular control flow and variable activation ranges; however, the gate statistics learned by DyT can inform static layer pruning or depth reweighting for SNFT-based designs.
- The combination SNFT + mxINT8 + KV Cache Sharing + interleaved attention constitutes a practically realizable end-to-end HW-friendly LLM block.

### From Latent Action World Model

- Unsupervised latent action learning from raw state-transition pairs produces latent vectors that are structurally more hardware-friendly than raw action spaces: low-dimensional, continuous, normalizable.
- The dynamics model rollout is isomorphic to LLM autoregressive decoding, enabling direct reuse of LLM-inference NPU pipelines for world model planning.
- Latent action quantization (INT8 z_t under KL constraint) is identified as a viable research direction, reducing the bandwidth cost of world model rollout.
- The architecture can be extended with a RISC-V / NPU ISA custom instruction `LATENT_ACT` that executes the inference net and dynamics model as a hardware-accelerated primitive, with ROB-style rollback for failed predictions.

---

## Connections [coverage: medium -- 4 sources]

- [[wiki/topics/kv-cache-optimization]] — KV-cache memory modeling, interleaved attention, prefill/decode split; directly informed by Gemma 3n analysis in this topic.
- [[wiki/topics/quantization]] — SNFT's normalization-free structure enables stable INT8 / mxINT8 quantization; latent action vectors are quantizable under KL constraint.
- [[wiki/topics/npu-architecture]] — NPU hardware implications: Tensor Engine / Vector Engine pipelines, SRAM tier design, static scheduling requirements, and new primitives (similarity engine, Top-K engine, KV prefetch engine).
- [[wiki/topics/transformer-architecture]] — SNFT, interleaved attention, and MatFormer are all transformer architectural variants; normalization comparison (LN vs RMSNorm vs NF) belongs here.
- [[wiki/topics/world-models]] — Latent Action World Model is the primary source for the world-model section; the NPU rollout dataflow analogy connects to LLM inference design.
- [[wiki/topics/rag-and-retrieval]] — MSA internalized retrieval, RAG vs learned memory hierarchy, gradient flow alignment between retriever and generator.

---

## Open Questions [coverage: medium -- 4 sources]

1. **SRAM budget estimation for interleaved attention:** What is the precise SRAM requirement for the local-layer KV cache (W = 1024, H heads, d_head, INT8) at Gemma 3n parameter scales? At what SRAM size does the local KV fit fully on-chip?

2. **MSA similarity search hardware cost:** The Router Key similarity search (cosine similarity over all chunk keys) must itself be efficient. At 100 M tokens / 64 tokens per chunk = 1.5 M chunks, even a simple dot-product scan is non-trivial. What is the latency and area cost of a hardware similarity engine for this scale?

3. **SNFT scaling law at depth:** SNFT's theoretical stability bounds are proven but the empirical behavior at very large depth (1000+ layers) and sub-8-bit quantization remains unpublished. Does activation range remain bounded under INT4 / FP8 microscaling?

4. **DyT-to-SNFT static distillation:** Is it possible to learn DyT gate statistics, extract a per-layer importance ranking, and use it to prune or depth-reweight an SNFT-based model — recovering FLOPs reduction without dynamic control flow? No published result exists.

5. **Latent action dimensionality vs quantization:** How small can z_t be (dimensionality reduction) before world model planning quality degrades? Is INT8 z_t sufficient for Atari/robot benchmarks, or does it require FP16?

6. **MSA training stability with independent RoPE:** Independent per-chunk position encoding disables cross-chunk positional reasoning. In tasks requiring ordered multi-document reasoning (e.g., temporal event ordering across chunks), does MSA degrade compared to full-context models?

7. **KV Cache Sharing interaction with quantization:** Gemma 3n's shared KV (upper layers reuse middle-layer KV) may create quantization sensitivity if the shared KV spans layers with different activation scale profiles. Has joint SNFT + KV sharing + INT8 been validated end-to-end?

---

## Sources [coverage: medium -- 4 sources]

- [[../../raw/GenAI/HW-Friendly/KV-cache Optimization Explanation 33a6cc566b0b811890a0df4bc615b114]]
- [[../../raw/GenAI/HW-Friendly/Latent Action World Model 33a6cc566b0b816ba286eaff4c35c587]]
- [[../../raw/GenAI/HW-Friendly/Stronger Normalization-Free Transformers 33a6cc566b0b81648fc4ec9576dc68a0]]
- [[../../raw/GenAI/HW-Friendly/MSA and RAG Analysis 33a6cc566b0b81b6b40ec9ead1726cf4]]
