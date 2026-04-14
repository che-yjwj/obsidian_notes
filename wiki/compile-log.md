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
