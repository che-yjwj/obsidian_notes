# Wiki Compile Log

<!-- append-only log of each /wiki-compile run -->

## 2026-04-12

**Topics updated:** (first run — all new)
**New topics:** npu-architecture, llm-quantization-compression, hw-friendly-model-design, npu-simulator-compiler, riscv-npu-soc-sim, paper-reviews, patent-mcm, soc-spec-english
**New concepts:** memory-bandwidth-bottleneck, static-scheduling-determinism, prefill-decode-duality
**Sources scanned:** 123
**Sources changed:** 123 (first run)

## 2026-04-14

**Run type:** pre-compile schema expansion
**Topics updated:** schema/index only
**New topics:** operator-coordinate-compression, trace-visualization, npu-doc-process
**New concepts:** tile-semantics-contract, mixed-precision-policy, kv-cache-dram-residency, trace-first-design
**Sources scanned:** not recompiled yet
**Sources changed:** graphify review indicates post-2026-04-12 additions are now large enough to justify topic split
**Notes:** next wiki compile should rebalance RISCV_NPU_SoC_SIM sources across spec/design/trace/process-oriented topic boundaries

## 2026-04-14 (run 2)

**Run type:** incremental compile
**Topics updated:** operator-coordinate-compression, trace-visualization, npu-doc-process, INDEX
**New topics:** operator-coordinate-compression, trace-visualization, npu-doc-process
**New concepts:** tile-semantics-contract, mixed-precision-policy, kv-cache-dram-residency, trace-first-design
**Sources scanned:** 123
**Sources changed:** 10 wiki output files + source reclassification for 3 new topic groups
**Notes:** compiled new topic articles from existing source corpus without touching raw files; next run should revisit `riscv-npu-soc-sim` article boundaries using the expanded schema

## 2026-04-16

**Run type:** curation pass
**Topics updated:** INDEX, schema
**New topics:** none
**New concepts:** none
**Sources scanned:** not recompiled
**Sources changed:** canonical/deep-dive boundary clarified for wiki maintenance
**Notes:** added document-role rules so `wiki/topics/*` remains the canonical topic layer while `wiki/AI-Hardware/*` and related folders can host longer synthesis essays and strategy memos without creating topic drift

## 2026-04-16 (run 2)

**Run type:** manual structural split
**Topics updated:** riscv-npu-soc-sim, INDEX, schema, compile-state
**New topics:** npu-architecture-spec, npu-timing-memory-model, simulation-validation
**New concepts:** none
**Sources scanned:** existing RISCV_NPU_SoC_SIM corpus re-grouped, not re-extracted from raw
**Sources changed:** umbrella topic narrowed; architecture/spec, timing-memory, and validation layers promoted to standalone canonical topics
**Notes:** `trace-visualization` and `npu-doc-process` remain canonical satellite topics; next pass should verify whether `simulation-validation` fully owns test-oriented trace material or still shares boundaries with trace visualization

## 2026-04-17

**Run type:** manual structural split
**Topics updated:** npu-architecture-spec, riscv-npu-soc-sim, INDEX, schema, compile-state
**New topics:** ir-cmdq-contract, tile-semantics-quantization
**New concepts:** none
**Sources scanned:** existing RISCV_NPU_SoC_SIM spec corpus re-grouped, not re-extracted from raw
**Sources changed:** execution-spec layer split into IR/CMDQ contract and tile/quantization contract; original `npu-architecture-spec` reduced to umbrella role
**Notes:** `npu-timing-memory-model`, `simulation-validation`, `trace-visualization`, and `npu-doc-process` remain sibling canonical topics; next pass should verify whether any of the new subtopics accumulates enough depth to justify further split

## 2026-04-17 (run 2)

**Run type:** curation / maintenance metadata sync
**Topics updated:** INDEX, schema, compile-state
**New topics:** none
**New concepts:** none
**Sources scanned:** not recompiled
**Sources changed:** no raw-source regrouping; freshness metadata and operating rules updated
**Notes:** documented compiler-scope vs graph-scope counting, marked `soc-spec-english` as a small-but-stable canonical topic, and kept `Project-Helios-Edge-Physical-AI-Custom-SoC-Platform-Strategy` as a deep-dive promotion candidate rather than forcing a premature topic split

## 2026-04-18

**Run type:** curation / topic-promotion decision
**Topics updated:** INDEX, schema
**New topics:** none
**New concepts:** none
**Sources scanned:** not recompiled
**Sources changed:** no raw-source regrouping; clarified the promotion gate for strategy-memo material
**Notes:** kept `Project-Helios-Edge-Physical-AI-Custom-SoC-Platform-Strategy` as a deep-dive because it is still a single synthesis memo rather than a source-backed topic family; future extraction should happen only through subtopics such as `fabric-presets`, `compute-hierarchy`, and `runtime-strategy` once they accumulate distinct source families or repeated graph-community support

## 2026-04-18 (run 2)

**Run type:** curation / split decision
**Topics updated:** operator-coordinate-compression, INDEX, schema
**New topics:** none
**New concepts:** none
**Sources scanned:** not recompiled
**Sources changed:** no raw-source regrouping; clarified that the topic remains an umbrella
**Notes:** reviewed `operator-coordinate-compression` and decided not to split yet because theory, patent framing, paper framing, and validation plans still behave like one tightly coupled source family; future split should happen only if theory and application/validation become separate source-backed navigation hubs
