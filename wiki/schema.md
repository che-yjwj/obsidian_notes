# Wiki Schema — AI Hardware & GenAI Research

This file is the source of truth for wiki structure. Edit it to rename topics, merge them, or change conventions. The compiler respects these changes on the next run.

---

## Topics

| Slug | Title | Aliases / Keywords | Description |
|---|---|---|---|
| `npu-architecture` | NPU Architecture | NPU, accelerator, chip, AMBA, NoC, Tesla FSD, AMD Versal, Xilinx FINN, TPU, systolic | NPU chip design, AMBA/NoC interconnects, memory hierarchy, vendor chip analysis |
| `llm-quantization-compression` | LLM Quantization & Compression | quantization, PTQ, QAT, TurboQuant, OCEAN, OliVe, DC-LLM, mxINT8, INT4, INT8, outlier | Post-training quantization, KV cache compression, entropy-aware techniques, coordinate-system theory |
| `hw-friendly-model-design` | HW-Friendly Model Design | KV-cache, normalization-free, SNFT, DyT, MSA, RAG, Gemma, on-device, edge AI | Model architectures co-designed with NPU hardware constraints |
| `npu-simulator-compiler` | NPU Simulator & Compiler | simulator, compiler, supernode, HyperAccel, LPU, Coral, MTIA, Groq, Inferentia, tiling | NPU compiler IR, supernode abstraction, real-world simulator/compiler platforms |
| `riscv-npu-soc-sim` | RISC-V NPU SoC Simulator | RISCV_NPU_SoC_SIM, CMDQ, SDD, NPU IR, TileGraph, LayerIR, static scheduler, SPM | The RISCV_NPU_SoC_SIM project: offline compiler + resource-accurate simulator for LLM inference |
| `paper-reviews` | Paper Reviews | STEM, Dr Zero, chaos theory, PaperDebugger, PaperBanana, Tim Palmer, ensemble prediction | Critical analyses of research papers across transformer architecture, agents, and tooling |
| `patent-mcm` | Patent & MCM Research | MCM, multiplierless, DWT, Taalas, AgentHub, TinyLoRA, 13 parameters, Voronenko, shift-add | MCM-based multiplierless VLSI design, AI infrastructure patents, small-parameter reasoning |
| `soc-spec-english` | SoC Specification Negotiation English | SoC spec, negotiation, technical English, architect vocabulary | English language patterns for SoC specification negotiation in professional settings |
| `operator-coordinate-compression` | Operator-Coordinate Compression | coordinate system, manifold alignment, TurboQuant, operator view, rotation, rate-distortion, basis | Geometry-aware compression theory and experiments centered on operator-coordinate representations |
| `trace-visualization` | Trace & Visualization | trace, gantt, heatmap, utilization, timeline, visualization, profiler, trace schema | Trace schema, visualization requirements, and analysis workflow for simulator outputs |
| `npu-doc-process` | NPU Documentation Process | SDD, roadmap, milestone, naming, review, contribution, changelog, workflow | Documentation, review, roadmap, and spec-driven process for the RISCV_NPU_SoC_SIM project |
| `npu-architecture-spec` | NPU Architecture & Execution Spec | NPU IR, CMDQ, ISA, tile semantics, quantization, KV semantics, scheduling semantics | Normative execution semantics and compiler-simulator contract for RISCV_NPU_SoC_SIM |
| `ir-cmdq-contract` | IR & CMDQ Contract | NPU IR, TensorIR, LayerIR, CMDQ, opcode, deps, deterministic lowering | IR schema and CMDQ execution contract between offline compiler and simulator |
| `tile-semantics-quantization` | Tile Semantics & Quantization Contract | tile lifecycle, KV semantics, mixed precision, qbits, prefill/decode, scheduling semantics | Tile lifecycle, quantization policy, KV semantics, and phase-aware execution contract |
| `npu-timing-memory-model` | NPU Timing & Memory Model | DMA timing, TE timing, VE timing, SPM, Bus, NoC, bandwidth, contention, latency model | Cycle timing, memory hierarchy, Bus/NoC contention, and engine latency modeling |
| `simulation-validation` | Simulation Validation & Golden Traces | golden trace, integration test, unit test, perf validation, regression, reference metrics | Validation protocols, golden artifacts, and acceptance criteria for simulator correctness and usefulness |

---

## Concepts

| Slug | Title | Connects |
|---|---|---|
| `memory-bandwidth-bottleneck` | Memory Bandwidth as Primary Bottleneck | llm-quantization-compression, npu-architecture, hw-friendly-model-design, npu-simulator-compiler, riscv-npu-soc-sim |
| `static-scheduling-determinism` | Static Scheduling and Deterministic Execution | npu-architecture, npu-simulator-compiler, riscv-npu-soc-sim, hw-friendly-model-design, patent-mcm |
| `prefill-decode-duality` | Prefill / Decode Duality | npu-simulator-compiler, riscv-npu-soc-sim, hw-friendly-model-design, llm-quantization-compression |
| `tile-semantics-contract` | Tile Semantics and Contract Boundaries | riscv-npu-soc-sim, npu-simulator-compiler, trace-visualization, npu-doc-process |
| `mixed-precision-policy` | Mixed Precision as a System Policy | llm-quantization-compression, riscv-npu-soc-sim, npu-architecture, trace-visualization |
| `kv-cache-dram-residency` | KV Cache as DRAM-Resident State | hw-friendly-model-design, llm-quantization-compression, riscv-npu-soc-sim, trace-visualization |
| `trace-first-design` | Trace-First Design | riscv-npu-soc-sim, trace-visualization, npu-doc-process, npu-simulator-compiler |

---

## Naming Conventions

- Topic slugs: `lowercase-kebab-case`
- Concept slugs: `lowercase-kebab-case`, named after the pattern (not after topics)
- Obsidian links: `[[path/to/file]]` without `.md` extension
- Source paths: relative from `wiki/topics/` → `../../raw/...`

---

## Document Roles

- `wiki/topics/*`
  Canonical topic hubs maintained by the wiki compiler. Prefer concise synthesis, explicit source coverage, and strong cross-links.
- `wiki/concepts/*`
  Cross-topic patterns that connect two or more topic families. Avoid turning these into project diaries or literature summaries.
- `wiki/AI-Hardware/*`, `wiki/GenAI/*`, `wiki/Research/*`, `wiki/Misc/*`
  Curated deep dives, essays, benchmark notes, strategy memos, or review documents. These can be longer and more opinionated than canonical topic pages.
- In process-heavy source trees, distinguish reusable control-plane docs from operational artifacts. Workflow/roadmap/governance documents can justify a canonical topic; milestone trackers, review summaries, and temporary task lists should normally remain source inputs, not standalone topic candidates.

Promotion rule:
- Promote a deep-dive page into `wiki/topics/*` only when it has a stable source family, repeated graph community support, and clear value as a reusable navigation hub.

Split rule:
- Split a topic when it becomes a mega-hub that mixes architecture, process, validation, and roadmap content in one page.

Small-topic rule:
- A topic may remain canonical even with a small source count if it serves a distinct user intent, has low overlap with nearby topics, and works as a stable navigation hub. Size alone is not sufficient reason to demote it.

Scope note:
- `wiki/.compile-state.json` tracks the curated raw-source corpus used by the wiki compiler.
- `graphify-out/GRAPH_REPORT.md` reports the wider vault graph across raw notes, wiki pages, and supporting documents.
- Topic counts and file counts across those two systems are expected to differ.

---

## Evolution Log

- **2026-04-12**: Initial schema generated from 8 topics, 3 concepts (first compile of 123 source files)
- **2026-04-14**: Expanded schema to 11 topics and 7 concepts to absorb newly connected graphify communities before the next wiki compile
- **2026-04-16**: Added explicit document-role rules to separate canonical topic pages from curated deep-dive essays and strategy memos
- **2026-04-16**: Split the overloaded `riscv-npu-soc-sim` umbrella into architecture-spec, timing-memory, and simulation-validation subtopics while keeping process and trace as separate canonical layers
- **2026-04-17**: Clarified that process control-plane docs may feed `npu-doc-process`, while milestone/review/task documents stay operational artifacts rather than future standalone topics
- **2026-04-17**: Split `npu-architecture-spec` into `ir-cmdq-contract` and `tile-semantics-quantization`, leaving the original page as an umbrella for execution-spec sublayers
- **2026-04-17**: Added the small-topic rule and explicit compiler-scope vs graph-scope note so maintenance decisions are not driven by raw file count alone
