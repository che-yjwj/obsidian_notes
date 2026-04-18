# AI Hardware & GenAI Research — Knowledge Base

Last compiled: 2026-04-17
Last curated: 2026-04-18
Canonical topics: 16 | Canonical concepts: 7 | Sources scanned in last compile: 123

Compile scope note: the wiki compiler count reflects the curated raw-source corpus in `wiki/.compile-state.json`, while graphify reports the wider vault graph across raw notes, wiki pages, and supporting docs. The two counts are expected to differ.

---

## Canonical Topics

| Topic | Aliases / Keywords | Sources | Last Updated | Status |
|---|---|---|---|---|
| [[topics/npu-architecture]] | NPU, accelerator, chip, AMBA, NoC, Tesla FSD, AMD Versal, Xilinx FINN, TPU, systolic, sparse core | 8 | 2026-04-18 | active |
| [[topics/llm-quantization-compression]] | quantization, PTQ, QAT, TurboQuant, OCEAN, OliVe, DC-LLM, mxINT8, INT4, INT8, outlier, KV compression | 16 | 2026-04-18 | active |
| [[topics/hw-friendly-model-design]] | KV-cache, normalization-free, SNFT, DyT, MSA, RAG, Gemma, on-device, edge AI, MatFormer | 4 | 2026-04-12 | active |
| [[topics/npu-simulator-compiler]] | simulator, compiler, supernode, HyperAccel LPU, Coral NPU, MTIA, Groq, Inferentia, tiling, IR | 8 | 2026-04-18 | active |
| [[topics/riscv-npu-soc-sim]] | RISCV_NPU_SoC_SIM, CMDQ, SDD, NPU IR, TileGraph, LayerIR, static scheduler, SPM, TE, VE | 70+ | 2026-04-12 | active |
| [[topics/paper-reviews]] | STEM, Dr Zero, chaos theory, PaperDebugger, PaperBanana, Tim Palmer, ensemble prediction | 6 | 2026-04-18 | active |
| [[topics/patent-mcm]] | MCM, multiplierless, DWT, Taalas, AgentHub, TinyLoRA, 13 parameters, Voronenko, shift-add | 7 | 2026-04-12 | active |
| [[topics/soc-spec-english]] | SoC spec, negotiation, technical English, architect vocabulary, cheat sheet | 2 | 2026-04-12 | small-stable |
| [[topics/operator-coordinate-compression]] | coordinate system, manifold alignment, operator view, TurboQuant, rate-distortion, basis | 17 | 2026-04-18 | active |
| [[topics/trace-visualization]] | trace, gantt, heatmap, utilization, timeline, golden trace, profiler | 8 | 2026-04-17 | active |
| [[topics/npu-doc-process]] | SDD, roadmap, milestone, naming, review, contribution, changelog, workflow | 12 | 2026-04-14 | active |
| [[topics/npu-architecture-spec]] | NPU IR, CMDQ, ISA, tile semantics, quantization, scheduling semantics | 20+ | 2026-04-16 | active |
| [[topics/ir-cmdq-contract]] | IR, TensorIR, LayerIR, CMDQ, opcode, deps, deterministic lowering | 7+ | 2026-04-17 | active |
| [[topics/tile-semantics-quantization]] | tile lifecycle, KV semantics, mixed precision, qbits, prefill/decode | 9+ | 2026-04-17 | active |
| [[topics/npu-timing-memory-model]] | timing, DMA, TE, VE, SPM, DRAM, Bus, NoC, contention, bandwidth | 10+ | 2026-04-16 | active |
| [[topics/simulation-validation]] | golden trace, unit test, integration test, perf validation, regression | 8+ | 2026-04-17 | active |

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
| [[AI-Hardware/Project-Helios-Edge-Physical-AI-Custom-SoC-Platform-Strategy]] | strategy memo / platform thesis | keep as a deep dive; extract subtopics later only if fabric/runtime/compute layers grow into distinct source-backed families |
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
- Inside process-heavy source families, roadmap/workflow/governance docs can feed a canonical process topic, but milestone trackers, review summaries, and task lists should stay operational artifacts rather than becoming separate semantic hubs.
- A deep-dive page should be promoted into `wiki/topics/*` only when it accumulates a distinct source family, repeated graph community, and stable outbound links.
- When a topic becomes a mega-hub, split it before adding more prose. Topic size is a maintenance smell, not a success metric.

---

## Immediate Refactor Targets

- Rebalance `riscv-npu-soc-sim` again after one more graph pass so the umbrella stays narrow and project-level
- Keep `npu-doc-process` as the single process hub and avoid letting `documentation_review_summary` / `milestone_plan` become parallel semantic entry points
- Verify that `npu-architecture-spec` now stays umbrella-only and does not regrow detailed IR/CMDQ or quantization prose
- Keep `operator-coordinate-compression` as an umbrella for now; split only when theory and application/validation become separate source-backed navigation hubs
- Keep `AI-Hardware/NPU-Architecture` and `topics/npu-architecture` aligned so the overview page does not silently drift from the canonical topic page
- Keep `Project-Helios-Edge-Physical-AI-Custom-SoC-Platform-Strategy` as a strategy memo for now; only extract `fabric-presets`, `compute-hierarchy`, and `runtime-strategy` if they become source-backed navigation hubs

---

## Topic Watchlist

- `topics/soc-spec-english`
  Keep as a small canonical topic for now. It has a distinct practical use case and low overlap with the AI-hardware topic tree, so size alone is not a demotion signal.
- `AI-Hardware/Project-Helios-Edge-Physical-AI-Custom-SoC-Platform-Strategy`
  Keep as a deep-dive memo for now. Promote only if fabric presets, compute hierarchy, or runtime strategy each grow a real source family, form repeated graph communities, or become reusable navigation hubs beyond this single memo.
- `topics/trace-visualization` and `topics/simulation-validation`
  Boundary is now healthy, but example ownership should continue to stay split between walkthrough-style trace reading and approval-grade validation artifacts.
- `topics/operator-coordinate-compression`
  Keep as a canonical umbrella for now. Split later only if theory and application/validation each form repeated graph communities and distinct entry-point value.
- `topics/paper-reviews` and `Research/AI-Assisted-Research-Workflow`
  Boundary is now explicit: `paper-reviews` owns the review corpus and interpretation layer, while `AI-Assisted-Research-Workflow` owns execution model, agent collaboration, and verification policy.
- `topics/npu-simulator-compiler` and `AI-Hardware/Simulator-and-Implementation-Tools`
  Boundary is now explicit: the canonical topic owns reusable compiler/simulator abstractions, while the deep-dive survey owns vendor/platform comparison and implementation intuition.
- `topics/npu-architecture`
  Canonical ownership is now explicit: reusable architecture patterns stay here, while vendor narrative, memory essays, and strategy memos stay in the deep-dive layer.
- `topics/llm-quantization-compression`
  Canonical ownership is now explicit: reusable compression axes and evaluation frames stay here, while method survey, OCEAN deep dive, and outlier-method comparison stay in the deep-dive layer.

---

## Recent Changes

- **2026-04-12**: First full compilation — 8 topics and 3 concepts created from 123 source files across 7 directories
- **2026-04-14**: Incremental compile added 3 topics and 4 concepts from newly separated graph clusters: operator-coordinate-compression, trace-visualization, npu-doc-process
- **2026-04-16**: Curation pass clarified canonical topics vs deep-dive essays and added explicit operating rules for wiki maintenance
- **2026-04-16**: Structural split extracted `npu-architecture-spec`, `npu-timing-memory-model`, and `simulation-validation` from the overloaded `riscv-npu-soc-sim` umbrella
- **2026-04-17**: Extended the deep-dive operating model to `wiki/GenAI/*` and `wiki/Research/*` so these pages point back to canonical topics instead of drifting into parallel topic trees
- **2026-04-17**: Tightened the process-topic rule so roadmap/workflow docs remain the canonical process hub while milestone and review summaries are treated as operational artifacts
- **2026-04-17**: Split the execution-spec layer again so `ir-cmdq-contract` and `tile-semantics-quantization` can grow independently without turning `npu-architecture-spec` back into a mega-hub
- **2026-04-17**: Synced topic freshness metadata with the latest manual curation passes, documented compiler-scope vs graph-scope counting, and added a watchlist for small-but-distinct topics and deep-dive promotion candidates
- **2026-04-18**: Resolved the `Project-Helios-Edge-Physical-AI-Custom-SoC-Platform-Strategy` watchlist item by keeping it as a deep-dive memo and defining explicit extraction gates for future `fabric-presets`, `compute-hierarchy`, and `runtime-strategy` subtopics
- **2026-04-18**: Reviewed `operator-coordinate-compression` and kept it as a canonical umbrella; split is deferred until theory and application/validation become separate source-backed navigation hubs
- **2026-04-18**: Clarified the boundary between `topics/paper-reviews` and `Research/AI-Assisted-Research-Workflow` so review artifacts stay in the canonical review hub while execution model and agent-collaboration guidance stay in the workflow deep dive
- **2026-04-18**: Re-aligned `topics/npu-simulator-compiler` with `AI-Hardware/Simulator-and-Implementation-Tools` so the canonical page keeps reusable abstractions while the survey page keeps vendor/platform narrative
- **2026-04-18**: Reframed `topics/npu-architecture` as the reusable architecture-pattern hub and pushed vendor comparison, memory essay material, and strategy narrative back into the deep-dive layer
- **2026-04-18**: Refreshed `topics/llm-quantization-compression` as the reusable compression-pattern hub and pushed method-survey narrative back into the deep-dive layer
