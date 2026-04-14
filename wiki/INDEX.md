# AI Hardware & GenAI Research — Knowledge Base

Last compiled: 2026-04-14
Total topics: 11 | Total concepts: 7 | Total sources: 123

---

## Topics

| Topic | Aliases / Keywords | Sources | Last Updated | Status |
|---|---|---|---|---|
| [[topics/npu-architecture]] | NPU, accelerator, chip, AMBA, NoC, Tesla FSD, AMD Versal, Xilinx FINN, TPU, systolic, sparse core | 8 | 2026-04-12 | active |
| [[topics/llm-quantization-compression]] | quantization, PTQ, QAT, TurboQuant, OCEAN, OliVe, DC-LLM, mxINT8, INT4, INT8, outlier, KV compression | 16 | 2026-04-12 | active |
| [[topics/hw-friendly-model-design]] | KV-cache, normalization-free, SNFT, DyT, MSA, RAG, Gemma, on-device, edge AI, MatFormer | 4 | 2026-04-12 | active |
| [[topics/npu-simulator-compiler]] | simulator, compiler, supernode, HyperAccel LPU, Coral NPU, MTIA, Groq, Inferentia, tiling, IR | 8 | 2026-04-12 | active |
| [[topics/riscv-npu-soc-sim]] | RISCV_NPU_SoC_SIM, CMDQ, SDD, NPU IR, TileGraph, LayerIR, static scheduler, SPM, TE, VE | 70+ | 2026-04-12 | active |
| [[topics/paper-reviews]] | STEM, Dr Zero, chaos theory, PaperDebugger, PaperBanana, Tim Palmer, ensemble prediction | 6 | 2026-04-12 | active |
| [[topics/patent-mcm]] | MCM, multiplierless, DWT, Taalas, AgentHub, TinyLoRA, 13 parameters, Voronenko, shift-add | 7 | 2026-04-12 | active |
| [[topics/soc-spec-english]] | SoC spec, negotiation, technical English, architect vocabulary, cheat sheet | 2 | 2026-04-12 | active |
| [[topics/operator-coordinate-compression]] | coordinate system, manifold alignment, operator view, TurboQuant, rate-distortion, basis | 17 | 2026-04-14 | active |
| [[topics/trace-visualization]] | trace, gantt, heatmap, utilization, timeline, golden trace, profiler | 8 | 2026-04-14 | active |
| [[topics/npu-doc-process]] | SDD, roadmap, milestone, naming, review, contribution, changelog, workflow | 12 | 2026-04-14 | active |

---

## Concepts

| Concept | Connects | Last Updated |
|---|---|---|
| [[concepts/memory-bandwidth-bottleneck]] | llm-quantization-compression, npu-architecture, hw-friendly-model-design, npu-simulator-compiler, riscv-npu-soc-sim | 2026-04-12 |
| [[concepts/static-scheduling-determinism]] | npu-architecture, npu-simulator-compiler, riscv-npu-soc-sim, hw-friendly-model-design, patent-mcm | 2026-04-12 |
| [[concepts/prefill-decode-duality]] | npu-simulator-compiler, riscv-npu-soc-sim, hw-friendly-model-design, llm-quantization-compression | 2026-04-12 |
| [[concepts/tile-semantics-contract]] | riscv-npu-soc-sim, npu-simulator-compiler, trace-visualization, npu-doc-process | 2026-04-14 |
| [[concepts/mixed-precision-policy]] | llm-quantization-compression, riscv-npu-soc-sim, npu-architecture, trace-visualization | 2026-04-14 |
| [[concepts/kv-cache-dram-residency]] | hw-friendly-model-design, llm-quantization-compression, riscv-npu-soc-sim, trace-visualization | 2026-04-14 |
| [[concepts/trace-first-design]] | riscv-npu-soc-sim, trace-visualization, npu-doc-process, npu-simulator-compiler | 2026-04-14 |

---

## Next Compile Focus

- Rebalance `riscv-npu-soc-sim` so trace/process-heavy source files are referenced from their new dedicated topics instead of remaining concentrated in one mega-topic
- Decide whether test-oriented trace documents stay under `trace-visualization` or move to a future `simulation-validation` topic
- Evaluate whether `operator-coordinate-compression` should later split into theory and patent/application subtopics

---

## Recent Changes

- **2026-04-12**: First full compilation — 8 topics and 3 concepts created from 123 source files across 7 directories
- **2026-04-14**: Incremental compile added 3 topics and 4 concepts from newly separated graph clusters: operator-coordinate-compression, trace-visualization, npu-doc-process
