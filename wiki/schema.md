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

---

## Concepts

| Slug | Title | Connects |
|---|---|---|
| `memory-bandwidth-bottleneck` | Memory Bandwidth as Primary Bottleneck | llm-quantization-compression, npu-architecture, hw-friendly-model-design, npu-simulator-compiler, riscv-npu-soc-sim |
| `static-scheduling-determinism` | Static Scheduling and Deterministic Execution | npu-architecture, npu-simulator-compiler, riscv-npu-soc-sim, hw-friendly-model-design, patent-mcm |
| `prefill-decode-duality` | Prefill / Decode Duality | npu-simulator-compiler, riscv-npu-soc-sim, hw-friendly-model-design, llm-quantization-compression |

---

## Naming Conventions

- Topic slugs: `lowercase-kebab-case`
- Concept slugs: `lowercase-kebab-case`, named after the pattern (not after topics)
- Obsidian links: `[[path/to/file]]` without `.md` extension
- Source paths: relative from `wiki/topics/` → `../../raw/...`

---

## Evolution Log

- **2026-04-12**: Initial schema generated from 8 topics, 3 concepts (first compile of 123 source files)
