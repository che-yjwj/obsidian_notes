---
title: LLM Quantization & Compression
type: topic
status: canonical
last_compiled: 2026-04-18
---

# LLM Quantization & Compression

*last_compiled: 2026-04-18 | sources: 16*

---

## Summary [coverage: high -- 16 sources]

LLM quantization and compression address a fundamental bottleneck in large language model inference: autoregressive decoding is dominated not by compute but by **memory bandwidth and capacity**. The canonical topic is therefore not just a catalog of methods, but a reusable map of the compression design space.

이 페이지에서 canonical하게 유지해야 할 핵심 축은 다음과 같다.

1. **where compression acts** — weights, activations, KV cache, metadata, bitstream
2. **what objective it preserves** — MSE, inner-product geometry, entropy, deployment simplicity
3. **what system cost it changes** — DRAM traffic, on-chip SRAM pressure, kernel complexity, reconstruction overhead
4. **what representation it assumes** — value-centric quantization, coordinate-aware transform, generative basis reconstruction, mixed-precision transport
5. **what hardware contract it requires** — standard INT GEMM path, extra correction path, entropy decode path, or polynomial/log-domain support

개별 기법(GPTQ, AWQ, SmoothQuant, OliVe, TurboQuant, DC-LLM, OCEAN, mxINT8 등)은 이 canonical map을 설명하는 reference methods다. detailed method walkthrough, comparison prose, and theory-specific argumentation belong in the deep-dive layer.

## Role in This Wiki [coverage: high -- 8 sources]

`llm-quantization-compression`은 이 vault의 canonical compression hub다. reusable compression vocabulary, system-level tradeoff, and cross-method comparison frame stay here.

## Boundary [coverage: medium -- 6 sources]

이 topic이 직접 유지해야 할 범위는 다음과 같다.

- PTQ/QAT, weight-only/WA, KV quantization, entropy coding, mixed precision 같은 reusable compression axes
- memory-bandwidth, effective bpc, reconstruction path, inner-product preservation 같은 system-facing evaluation frame
- coordinate-aware compression이 왜 quantization topic 전체를 다시 묶는지에 대한 canonical synthesis

이 topic이 직접 흡수하지 말아야 할 범위는 다음과 같다.

- OCEAN 방법론의 장문 전개와 실험 가설 상세
- outlier mitigation 기법 간 세부 비교 에세이
- 개별 기법의 구현 메모, 구현체 리뷰, 혹은 단일 paper deep dive

그 내용은 [[../GenAI/LLM-Quantization-and-Compression]], [[../GenAI/OCEAN-Compression-Deep-Dive]], [[../GenAI/Outlier-Mitigation-Methods-Comparison]], [[operator-coordinate-compression]]에 남기고, 이 페이지는 canonical map과 system-level synthesis를 유지하는 것이 맞다.

## Reference Methods [coverage: high -- 10 sources]

아래 기법들은 canonical ownership 대상이 아니라, design-space pattern을 설명하는 reference set이다.

- **GPTQ / AWQ / SmoothQuant**: PTQ와 activation-aware scaling의 실무 baseline
- **OliVe**: outlier를 memory-format 문제로 재해석하는 HW-friendly path
- **TurboQuant**: KV cache에서 MSE보다 inner-product geometry가 중요함을 보여주는 decode-specific path
- **DC-LLM**: explicit weight storage 대신 generative basis reconstruction을 쓰는 path
- **OCEAN**: entropy-aware, coordinate-aware compression frame
- **mxINT8 / Tesla mixed-precision bridge**: block-scale 또는 transport/reconstruction 관점의 mixed-precision path

---

## Core Concepts [coverage: high -- 16 sources]

### PTQ vs QAT

- **PTQ (Post-Training Quantization)**: applied after training, requires only calibration data. Main risk is that without calibration on the target distribution, real-service quality can degrade unexpectedly.
- **QAT (Quantization-Aware Training)**: fine-tunes with fake quantization in the forward pass; higher quality ceiling but higher cost and complexity.

### Weight-Only vs. W/A Quantization

- **Weight-only quantization**: compresses stored weights (DRAM traffic ↓), but activations remain FP16/BF16. Easiest to apply; bottleneck stays on memory-bandwidth.
- **W8A8, W4A8, etc.**: compress both weights and activations; requires specialized integer compute kernels; unlocks compute-side savings but demands hardware support.

### KV Cache Quantization

KV cache memory for a decoder-only model scales as `2 × L × d × bytes` per sequence. At 128K context and FP16, this easily dominates on-chip and near-chip memory. KV quantization targets the decode bottleneck specifically, where every generated token reads the entire KV history.

Key metric for honest comparison: **bpc (bits per channel)** — the effective bits including scale, zero-point, per-block normalization constants, and layout padding. Reporting only the nominal bit-width overstates compression.

### Outlier Problem

In uniform per-tensor quantization, a single large-magnitude value forces the scale `s = max|w| / Q`, collapsing all normal values to zero. For example, with 4-bit (Q=7) and values `[1.5, 2.6, −5.2, −98]`, the outlier −98 sets `s = 14`, quantizing 1.5, 2.6, and −5.2 all to 0. Existing solutions include mixed-precision (FP path for outliers), per-channel/group scaling, GPTQ (Hessian-based correction), AWQ (activation-aware scaling), and SmoothQuant (weight-activation re-scaling).

### OliVe: Outlier-Victim Pair Quantization

OliVe reframes outlier handling as a **memory-format problem**, not an algorithm problem. Instead of routing outliers to a separate FP stream, it pairs each outlier with a nearby low-magnitude weight ("victim"), encodes both in the same byte-aligned unit, and sets the victim to zero — freeing that slot for a better representation of the outlier. The result is a uniform INT GEMM with no special datapath, no coordinate lists, and no additional metadata overhead. The hardware sees a standard packed INT4/INT8 stream; the outlier is simply better represented within the same memory footprint.

### TurboQuant: 2-Stage KV Cache VQ

TurboQuant (Google ICLR 2026) is a **2-stage online vector quantization** algorithm:

1. **Stage 1 — PolarQuant (distribution shaping + scalar quantization)**:
   - Apply a random orthogonal rotation `k' = Rk` to the key vector. In high dimensions, random rotation causes each coordinate to follow a near-Beta distribution — energy spreads uniformly. This eliminates coordinate-specific outliers.
   - Convert rotated pairs to polar coordinates `(r, θ)`. After rotation, θ values cluster tightly, so θ requires very few bits; r carries the primary information. Result: ~6× compression at 3-bit equivalent.
   - Use Lloyd-Max scalar quantizer per coordinate — simple and hardware-friendly.

2. **Stage 2 — QJL (1-bit residual correction)**:
   - MSE-optimal quantization introduces a systematic **inner-product bias** that accumulates over long contexts in attention scoring.
   - Apply a Johnson-Lindenstrauss projection to the residual `r = k - k̂` and store only `sign(Pr) ∈ {+1, −1}` at 1 bit per projected dimension.
   - This produces an **unbiased inner-product estimator**, preserving attention ranking without full residual reconstruction.

Key design philosophy: the goal is not to reconstruct the original KV vector accurately (low MSE), but to preserve the **geometry of attention dot products** — ranking, relative scores, and top-k correctness. TurboQuant reports 3–3.5 bit/channel with near-zero quality loss, ~6× memory reduction, and attention-logit kernel speedups up to 8× on H100 (attention kernel scope, not end-to-end).

### DC-LLM: LFSR-Based Weight Compression

DC-LLM (ICLR 2026) replaces explicit weight storage with a generative representation: each weight block is stored as a **random seed + sparse coefficient vector**. The LFSR (Linear Feedback Shift Register) re-generates basis vectors deterministically from the seed; the weight block is reconstructed as a linear combination of those bases.

Key design choices:
- **Adaptive k (basis count per block)**: defined by "Explained Energy Ratio" `R_k = 1 - ||T - T̂^(k)||_F² / ||T||_F²`. Blocks vary in complexity; k is chosen per-block offline to meet a threshold `R_th`.
- **DSE via Bayesian Optimization**: the configuration tuple `⟨B, S, G, R_th⟩` (block size, seed length, group size, energy threshold) is explored with a GP surrogate and EIPV acquisition.
- **Dedicated accelerator**: a Weight Generator + Systolic Array in SystemVerilog, synthesized at 7nm. Generator pre-expands LFSR states into a LUT; systolic PEs do MAC. Area-fair comparison reduces PE count to account for generator overhead.

Reported results: ~4× latency reduction vs. uncompressed; energy savings from reduced DRAM/SRAM access dominate.

### Microscaling (mxINT8) vs. Mixed-Precision Bridge

**mxINT8 (Microscaling INT8)**: the OCP MX format standard. Each block of activations shares a single exponent-level scale factor stored in a compact format. The formula is `x ≈ s_b · q` where `q ∈ INT8` and `s_b` is a block/tile-level scale. This fits cleanly into standard GEMM datapaths (matmul, FFN, Conv) with minimal modification.

Limitation: RoPE (Rotary Positional Encoding) and other trigonometric operations accumulate phase drift under INT8, requiring the block to carry FP anyway for sin/cos computation.

**Tesla Mixed-Precision Bridge (US20260017019A1)**: a pipeline-level precision architecture for edge NPU. Rather than reducing precision uniformly:
- Trigonometric values are encoded as 8-bit log-like approximations and transported through narrow paths.
- At the destination, Horner's method (iterative polynomial evaluation: `a₀ + x(a₁ + x(a₂ + …))`) reconstructs the sin/cos approximation using MAC units alone — no dedicated sin/cos hardware.
- FP32 precision is reserved only for phase-critical accumulation intervals and attention sink normalization.

The two approaches are complementary, not competitive: mxINT8 handles GEMM-class workloads; the Tesla approach handles RoPE/phase-sensitive operations. The optimal combination is **mxINT8 + RoPE-specific polynomial path**.

### OCEAN: Outlier-Compensated Entropy-Aware Numerical Coding

OCEAN is a proposed methodology (developed through the ChatGPT sessions, not yet published as a paper) that decomposes weight blocks as:

```
W = s·Q (low-bit base) + R (low-entropy residual) + E (sparse exceptions/outliers)
```

Design principles:
- Pair selection is driven by entropy minimization `ΔH(Q) + α·ΔH(R) + β·#exceptions`, not purely by accuracy.
- Channel permutation exploits the mathematical equivalence `W ← PW; W_next ← W_next P^{-1}` to cluster outlier-heavy channels together, improving RLE efficiency on exception indices.
- Residual is decomposed into `sign + magnitude`, stored as separate bitplanes; magnitude encoded with Golomb/Rice or ANS.
- Hardware bitstream is a 3-stream layout: base stream (INT4/INT3 + Huffman/ANS), residual stream (bitmask + small-alphabet ANS), exception stream (delta-coded indices + INT8/FP16 values).

### Coordinate-System Theory of Quantization (OCEAN Sessions 5–9)

The OCEAN conversation series develops a theoretical reframing of quantization:

**Claim**: LLM weights are not independent scalar data. They parameterize **high-dimensional nonlinear operators** whose learned values lie near a low-dimensional manifold `M ⊂ R^N`. Outliers arise when coordinate axes are misaligned with the manifold's local tangent space — they are **coordinate projection artifacts**, not intrinsic information.

**Key distinction — Flattening vs. Concentration**:
- Random/Hadamard rotation achieves **flattening**: redistributes energy across coordinates, reduces L∞ and kurtosis, eliminates visible outliers. L2 norm is preserved; information is not lost.
- **Concentration** (manifold-aligned coordinates): energy condenses into a small number of axes. This is what actually improves compression. Flattening is a prerequisite but not sufficient.

**Hadamard vs. DCT/FFT**: Hadamard is a geometric coordinate rotation (preserves L2, changes L∞/kurtosis), not a frequency decomposition. It has no "low-frequency = structure, high-frequency = noise" interpretation in the LLM weight context because weight axes have no inherent ordering or adjacency. The correct label is "axis rotation", not "frequency transform".

**Sensitivity vs. magnitude**: the importance of a parameter direction is `Sensitivity(v) ≈ ||∂f/∂W · v||`, not `|w_i|`. A large-magnitude weight in a low-sensitivity direction can be safely quantized; a small weight in a high-sensitivity direction cannot.

### Key Terminology

| Term | Meaning and Pitfall |
|---|---|
| PTQ | Post-training; calibration data required, not optional |
| bpc (bit per channel) | True compression rate including all metadata |
| MSE distortion | Reconstruction error; does not capture attention bias |
| Inner-product distortion | Directly affects attention scores; distinct from MSE |
| Unbiased estimator | Reduces systematic dot-product bias, but increases variance |
| QJL | 1-bit sketch for residual correction (sign of projected residual) |
| Random rotation | Flattening (outlier suppression), not compression itself |
| Hadamard rotation | Cheap orthogonal rotation via ±1 entries; O(n log n) |
| Incoherence | Degree of energy redistribution across coordinates post-rotation |
| Flattening vs. Concentration | The critical distinction: rotation gives flattening; manifold alignment gives concentration |
| Manifold-aligned coords | Coordinate system aligned with weight manifold tangent space — compression-optimal |
| Online quantization | Applied token-by-token without dataset-dependent codebooks |
| Paged KV cache | vLLM-style KV memory management; quant block sizes must align |
| Kernel fusion | Fusing dequant + matmul + attention reduces memory round-trips |
| HBM bottleneck | LLM decode is memory-bandwidth limited, not compute-limited |

---

## Architecture [coverage: high -- 16 sources]

### Memory System Impact of Quantization

Autoregressive decode is **memory-bound**: each token step reads the full weight matrix (weight-only quant) or the full KV cache (KV quant) from DRAM. Compression directly reduces:
- DRAM bandwidth demand (weights read per step)
- DRAM capacity requirement (KV cache footprint)
- SRAM hit rate (more data fits in on-chip buffers per tile)

TurboQuant's 6× KV reduction can shift decode from memory-bound to compute-bound on H100, explaining the large attention-kernel speedups reported.

### NPU Datapath Implications

**For weight-only quantization (OliVe, INT4/INT8)**:
- Uniform INT GEMM arrays are unchanged if quantization is pure per-block symmetric.
- Channel permutation (OCEAN) and outlier-victim pairing (OliVe) can be absorbed into the weight layout without modifying the PE array or ISA.
- Scale/zero-point metadata must be loaded alongside weight tiles; group size determines the ratio of scale overhead to useful payload.

**For KV cache quantization (TurboQuant)**:
- Requires new compute primitives: random rotation kernel (`R @ x`), Lloyd-Max scalar quantizer, QJL projection (`sign(P @ r)`), and QJL-corrected inner-product estimator.
- DRAM → compressed KV → on-the-fly reconstruction → MAC. Adds compute before MAC but greatly reduces DRAM load.
- Must co-design with paged KV block sizes; a TurboQuant block must align to the paging granularity.

**For DC-LLM (LFSR weight generation)**:
- Requires a dedicated **Weight Generator** block that pre-expands LFSR seed states into a LUT, then performs MAC with stored coefficients to reconstruct weights on-chip.
- New metadata layout: compressed stream `(seed, k, coeffs, scale)` per block. DMA and prefetch schedules must account for variable `k` per block.
- Area-fair comparison: the generator's silicon area is counted against available PE count.

**For microscaling (mxINT8)**:
- Fits cleanly into existing NPU GEMM datapaths; minimal ISA extension required.
- Scale granularity (block size 16 or 32 elements) trades off precision for metadata overhead.
- Accumulator must be FP to avoid overflow; mixed-precision accumulation is standard.

**For Tesla-style polynomial path**:
- Eliminates dedicated sin/cos hardware; replaces with MAC-based Horner polynomial evaluation.
- Requires new pipeline stages for phase-sensitive operations (RoPE, positional encodings).
- Demands co-design of ISA, compiler, and runtime — not a drop-in replacement.

### SRAM/DRAM Layout Considerations

- **Group size**: smaller groups = better dynamic range per block, but more scale overhead. A 32-element group with FP16 scale adds 0.5 bits/element overhead; a 128-element group adds 0.125 bits/element.
- **OliVe pairing**: pair-aligned layout ensures outlier and victim are always in the same cache line / SRAM bank, avoiding extra fetch cycles.
- **OCEAN 3-stream layout**: base stream is random-accessible (per-block offset table); residual and exception streams are sequential with optional pre-decode into SRAM for hot paths.
- **TurboQuant**: rotation matrix `R` is data-independent and can be stored in SRAM or generated on-the-fly via fast Hadamard; the QJL projection matrix `P` is similarly fixed.

### Decode vs. Prefill Distinction

KV quantization benefits are concentrated in the **decode phase**. Prefill is usually compute-bound (large batch matrix multiplies); decode is memory-bound (sequential KV reads). Conflating TPS (tokens per second) across phases misrepresents the gain. Additionally, recent tokens in KV may warrant keeping at FP16 (they are accessed most frequently in the "hot" KV window) while cold tokens use lower precision.

---

## Key Findings [coverage: high -- 16 sources]

### Quantization is a Coordinate Problem, Not a Value Problem

The most theoretically significant finding across the OCEAN conversation series is the reframing: outliers are **coordinate-dependent statistics**, not intrinsic importance markers. Orthogonal transformations (Hadamard, random rotation) can eliminate outliers without altering model accuracy, which directly contradicts the "magnitude = importance" assumption underlying most outlier-aware methods. The implication is that "outlier mitigation" methods (SmoothQuant, mixed-precision) are heuristic workarounds, while **coordinate-aware reparameterization** is the principled solution.

### Random Rotation Achieves Flattening, Not Optimal Compression

Rotation-based methods (QuaRot, SpinQuant) achieve flattening — they reduce L∞ and kurtosis, making uniform quantization viable. However, in a toy experiment comparing Fourier vs. MLP representation of sinusoidal signals, Hadamard rotation of the MLP parameters reduced outliers but did **not** recover Fourier-level coefficient concentration. True compression optimality requires **manifold-aligned coordinates** that concentrate energy into a small number of axes. Rotation is a prerequisite step, not the final answer.

### OliVe: Hardware Simplicity Without Accuracy Loss

OliVe achieves competitive accuracy with GPTQ/AWQ while imposing near-zero hardware overhead. The paired format keeps uniform INT GEMM on the critical path; no FP bypass, no coordinate lists, no extra ISA opcodes. The key trade-off is that the victim weight is set to zero, which is acceptable because victims are statistically low-magnitude and low-sensitivity. The paper demonstrates this holds for LLaMA and OPT series at INT8/INT6/INT4.

### TurboQuant: Inner-Product Preservation Outperforms MSE Minimization

TurboQuant's core finding is that for KV cache compression, **MSE is the wrong objective**. What matters is inner-product preservation (attention ranking and score magnitude). A quantizer that minimizes MSE can still introduce systematic inner-product bias that accumulates over long contexts. The QJL 1-bit residual correction specifically addresses this bias, producing an unbiased inner-product estimator. Reported results: 3–3.5 bit/channel, ~6× memory reduction, attention-logit kernels ~8× faster on H100.

### DC-LLM: LFSR Generative Compression Beats 4-bit Quantization

DC-LLM claims ~4× latency improvement over uncompressed inference on its dedicated accelerator, outperforming even 4-bit quantization designs in end-to-end runtime. The energy savings are dominated by reduced DRAM/SRAM access. The critical implementation risks are: LUT size and bandwidth on-chip, aligning the variable-k GEMM schedule with systolic array tiling, and building a production compiler toolchain for the new weight stream format.

### mxINT8 vs. Tesla Polynomial Path: Complementary Scopes

mxINT8 handles GEMM-class operations (FFN projections, attention linear layers) efficiently and is standards-based (OCP MX format). It cannot adequately handle RoPE or softmax-class operations where phase drift accumulates. Tesla's polynomial path handles these specific operations using low-order Horner polynomial approximation over MAC units. Neither approach covers the full model; the optimal edge NPU combines both in a mixed-precision pipeline.

### Entropy Coding Requires Distribution Shaping First

The OCEAN framework's central insight for entropy coding is that Huffman/ANS can only compress if the symbol distribution is non-uniform. Standard INT4 quantization produces a roughly uniform symbol histogram, yielding near-zero entropy coding gain. OCEAN's entropy-aware quantization deliberately concentrates symbols toward zero (conservative step size + residual + exception decomposition), creating the skewed distributions that Golomb/Rice and ANS can exploit. Channel permutation further consolidates outlier-heavy channels, making exception index sequences more compressible via delta + RLE.

### Practical System Checklist

- Metadata overhead (scale, zero-point, block headers) must be included in effective bpc comparisons.
- Prefill and decode performance must be measured separately; KV quant helps decode, not prefill.
- Short-sequence benchmarks may not reveal reasoning degradation that only appears in long contexts.
- MSE and perplexity metrics can be stable while attention ranking and chain-of-thought reasoning silently degrade.
- Paged KV systems (vLLM) require quant block size to align with page granularity to avoid misaligned memory access.

---

## Connections [coverage: medium -- 4 sources]

The following related topics exist or are implied within this vault:

- [[wiki/topics/npu-accelerator-architecture]] — DC-LLM's Weight Generator + Systolic Array design, OliVe's impact on PE array and tile layout, TurboQuant's requirement for rotation/QJL primitives, and the Tesla patent's MAC-centric RoPE path all connect to NPU datapath and ISA design.
- [[wiki/topics/kv-cache-management]] — TurboQuant, paged KV alignment, prefill/decode bottleneck analysis, and the mixed-precision "recent tokens FP16" policy are directly relevant to KV cache system design.
- [[operator-coordinate-compression]] — OCEAN 계열 대화와 coordinate-relative outlier 해석을 독립 topic으로 분리한 이론 층이다. 이 문서가 기법 catalog라면 해당 topic은 왜 회전/정렬/집중화가 필요한지 설명하는 geometry layer다.
- [[wiki/topics/hw-friendly-model-design]] — The coordinate-system reframing (OCEAN sessions), manifold-aligned representations, and PolarQuant's "representation learning without training" framing connect to hardware-friendly architecture search.
- [[wiki/topics/attention-mechanism]] — TurboQuant's inner-product preservation goal, attention sink (Tesla patent), and the "Attention Is Not What You Need" paper's subspace-flow interpretation referenced in OCEAN Part 8 all connect to attention mechanism analysis.

---

## Open Questions [coverage: high -- 10 sources]

1. **Manifold alignment in practice**: The OCEAN theory posits that manifold-aligned coordinates yield concentration superior to random rotation. The toy experiment (Fourier vs. MLP) supports this directionally, but no practical algorithm for finding manifold-aligned coordinates for real LLM weights has been validated. What is the computational cost of alignment vs. the compression benefit?

2. **TurboQuant production performance**: TurboQuant's reported speedups are for the attention-logit kernel scope on H100. End-to-end tokens/second in a production setup with paged KV, variable batch sizes, and prefill/decode interleaving has not been reported. Does QJL reconstruction add meaningful latency per token?

3. **DC-LLM compiler toolchain**: The paper demonstrates RTL feasibility but not a production compiler that handles variable-k weight blocks, arbitrary model architectures, and tiling/scheduling across a general systolic array. How would this be expressed in a standard NPU IR (e.g., TVM, MLIR)?

4. **OCEAN entropy-aware pairing — validation**: The OCEAN methodology was developed in conversation but has not been empirically validated at scale. The core hypothesis (entropy-minimizing victim selection outperforms accuracy-minimizing selection) needs experimental confirmation on LLaMA-class models.

5. **Hadamard flattening + manifold concentration — combining both**: Rotation-based methods (QuaRot, TurboQuant Stage 1) achieve flattening. Manifold alignment achieves concentration. Is there a practical two-step pipeline that first rotates (cheap) then fine-tunes the coordinate system toward the manifold (expensive once, amortized)?

6. **MSE vs. inner-product distortion in weight quantization**: TurboQuant demonstrates that inner-product preservation matters for KV quantization. Does the same principle apply to weight quantization? Are there weight directions where MSE-optimal quantization introduces systematic sensitivity distortion?

7. **Tesla patent applicability to general NPU design**: The Tesla Mixed-Precision Bridge is a proprietary pipeline designed for edge inference of RoPE-heavy models. Generalizing this to other positional encodings (ALiBi, sinusoidal, learned) and other model families is an open design question.

8. **Coordination preservation as compression objective**: OCEAN Part 8 argues that compression failures (reasoning collapse under INT4) are better explained by disruption of subspace flow continuity than by perplexity degradation. Validating this with the proposed experimental setup (subspace flow metrics, chain-of-thought stability tests) is an active TODO.

9. **OliVe at INT3 and below**: OliVe's pairing mechanism is designed for the INT4/INT8 regime. At INT3 or INT2, the victim-zeroing trade-off may become too costly. Extensions involving "1 outlier + N victims" groups or entropy-aware pairing (OCEAN hybrid) have been proposed but not evaluated.

10. **Production calibration and regression gates**: All PTQ methods depend on calibration data. What is the minimum calibration set size to avoid real-service quality collapse? How should regression gates be structured to catch silent quality degradation in long-context reasoning that perplexity benchmarks miss?

---

## Sources [coverage: high -- 16 sources]

1. [[../../raw/GenAI/Compression/DC-LLM Paper Summary 33a6cc566b0b810abe58cd3c651ad3ad]]
2. [[../../raw/GenAI/Compression/OliVe Paper Summary 33a6cc566b0b81419e7bfc22f35bdf25]]
3. [[../../raw/GenAI/Compression/TurboQuant PyTorch Implementation 33a6cc566b0b8163a307f12187edcc91]]
4. [[../../raw/GenAI/Compression/LLM Quantization Architecture 33a6cc566b0b8181a2d9e4bd12d3d4c8]]
5. [[../../raw/GenAI/Compression/Microscaling vs Mixed-Precision 33a6cc566b0b81798986d497a0b25f67]]
6. [[../../raw/GenAI/Compression/TurboQuant Concept Summary/TurboQuant Concept Summary (Part 1 of 2) 33a6cc566b0b81ebbb8afb6f34807867]]
7. [[../../raw/GenAI/Compression/TurboQuant Concept Summary/TurboQuant Concept Summary (Part 2 of 2) 33a6cc566b0b81bd9472e15a23617554]]
8. [[../../raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 1 of 9) 33a6cc566b0b813bb973c15abd5d814a]]
9. [[../../raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 2 of 9) 33a6cc566b0b81e99238dcafa97547ed]]
10. [[../../raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 3 of 9) 33a6cc566b0b8143b946d2a146d08503]]
11. [[../../raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 4 of 9) 33a6cc566b0b81d5b11df9b158e134de]]
12. [[../../raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 5 of 9) 33a6cc566b0b81339ca2c063e67abe20]]
13. [[../../raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 6 of 9) 33a6cc566b0b81c79bcbc45860000c15]]
14. [[../../raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 7 of 9) 33a6cc566b0b81c79b84c74dbb899aa2]]
15. [[../../raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 8 of 9) 33a6cc566b0b814a9495d3ee2214499d]]
16. [[../../raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 9 of 9) 33a6cc566b0b81d89f96cfedd5ba2b85]]
