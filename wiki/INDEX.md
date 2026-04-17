# AI Hardware & GenAI Research — Knowledge Base

Last compiled: 2026-04-16
Last curated: 2026-04-17
Canonical topics: 14 | Canonical concepts: 7 | Sources scanned in last compile: 123

---

## Canonical Topics

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
| [[topics/npu-architecture-spec]] | NPU IR, CMDQ, ISA, tile semantics, quantization, scheduling semantics | 20+ | 2026-04-16 | active |
| [[topics/npu-timing-memory-model]] | timing, DMA, TE, VE, SPM, DRAM, Bus, NoC, contention, bandwidth | 10+ | 2026-04-16 | active |
| [[topics/simulation-validation]] | golden trace, unit test, integration test, perf validation, regression | 8+ | 2026-04-16 | active |

---

## Canonical Concepts

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

## Curated Deep Dives

These are intentionally longer synthesis documents. They are useful entry points, but they are not automatically the canonical topic hubs used by the wiki compiler.

| Document | Role | Current Position |
|---|---|---|
| [[AI-Hardware/NPU-Architecture]] | curated overview / vendor benchmark | overlaps with canonical topic `topics/npu-architecture` |
| [[AI-Hardware/Memory-Hierarchy-in-AI-Accelerators]] | deep dive / framing essay | supports `topics/npu-architecture` and memory concepts |
| [[AI-Hardware/Simulator-and-Implementation-Tools]] | curated survey | overlaps with `topics/npu-simulator-compiler` |
| [[AI-Hardware/Flash-Resident-LLM-and-HBF-for-Edge-Inference]] | architecture synthesis essay | bridges memory hierarchy, KV policy, and edge inference |
| [[AI-Hardware/Project-Helios-Edge-Physical-AI-Custom-SoC-Platform-Strategy]] | strategy memo / platform thesis | currently a deep dive; candidate for future topic extraction if related source corpus grows |
| [[GenAI/HW-Friendly-Model-Design]] | domain deep dive / model-design essay | expands `topics/hw-friendly-model-design` with on-device and KV-centric examples |
| [[GenAI/LLM-Quantization-and-Compression]] | domain deep dive / compression survey | expands `topics/llm-quantization-compression` with method-level comparisons |
| [[GenAI/OCEAN-Compression-Deep-Dive]] | method deep dive | specialized essay under `topics/llm-quantization-compression` |
| [[GenAI/Outlier-Mitigation-Methods-Comparison]] | comparative review | compares outlier-mitigation methods under `topics/llm-quantization-compression` |
| [[Research/AI-Assisted-Research-Workflow]] | workflow essay / operating model | bridges `topics/paper-reviews` and `topics/patent-mcm` around research execution |
| [[Research/Paper-Reviews]] | curated review hub | expands `topics/paper-reviews` with a human-readable survey layer |
| [[Research/Patent-MCM]] | patent and hardware-specialization essay | expands `topics/patent-mcm` with multiplierless and model-to-silicon framing |
| [[Research/Research-Tooling-Reviews]] | tooling review essay | companion deep dive to `topics/paper-reviews` for research productivity tools |

---

## Operating Model

- `wiki/topics/*` is the canonical topic layer. These pages should stay compact, source-backed, and link outward.
- `wiki/concepts/*` is the cross-topic abstraction layer. These pages capture reusable patterns, not project-specific summaries.
- `wiki/AI-Hardware/*`, `wiki/GenAI/*`, `wiki/Research/*`, and `wiki/Misc/*` can contain curated deep dives, essays, reviews, and strategy memos.
- A deep-dive page should be promoted into `wiki/topics/*` only when it accumulates a distinct source family, repeated graph community, and stable outbound links.
- When a topic becomes a mega-hub, split it before adding more prose. Topic size is a maintenance smell, not a success metric.

---

## Immediate Refactor Targets

- Rebalance `riscv-npu-soc-sim` again after one more graph pass so the umbrella stays narrow and project-level
- Verify the ownership boundary between `trace-visualization` and `simulation-validation`, especially for test-oriented trace documents
- Evaluate whether `operator-coordinate-compression` should later split into theory and patent/application subtopics
- Keep `AI-Hardware/NPU-Architecture` and `topics/npu-architecture` aligned so the overview page does not silently drift from the canonical topic page
- Decide whether `Project-Helios-Edge-Physical-AI-Custom-SoC-Platform-Strategy` remains a strategy memo or becomes a future topic family with subpages on fabric presets, compute hierarchy, and runtime strategy

---

## Recent Changes

- **2026-04-12**: First full compilation — 8 topics and 3 concepts created from 123 source files across 7 directories
- **2026-04-14**: Incremental compile added 3 topics and 4 concepts from newly separated graph clusters: operator-coordinate-compression, trace-visualization, npu-doc-process
- **2026-04-16**: Curation pass clarified canonical topics vs deep-dive essays and added explicit operating rules for wiki maintenance
- **2026-04-16**: Structural split extracted `npu-architecture-spec`, `npu-timing-memory-model`, and `simulation-validation` from the overloaded `riscv-npu-soc-sim` umbrella
- **2026-04-17**: Extended the deep-dive operating model to `wiki/GenAI/*` and `wiki/Research/*` so these pages point back to canonical topics instead of drifting into parallel topic trees
