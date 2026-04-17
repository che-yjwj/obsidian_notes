---
title: NPU Architecture & Execution Spec
type: topic
status: canonical
updated: 2026-04-17
---

# NPU Architecture & Execution Spec

*last_curated: 2026-04-17 | umbrella topic for execution-spec sublayers*

---

## Summary [coverage: architecture/spec layer]

이 topic은 `RISCV_NPU_SoC_SIM`의 **규범적 execution model**을 다루는 umbrella page다. 관심사는 여전히 “프로젝트가 무엇을 만들려는가”가 아니라, execution spec 계층을 어떤 하위 canonical topic으로 나눠 읽어야 하는지 명확히 하는 데 있다.

핵심 질문은 다음과 같다.

- IR/CMDQ contract는 어디까지 하나의 의미 계층으로 묶이는가
- tile lifecycle과 quantization policy는 왜 별도 contract layer가 필요한가
- timing, validation, trace와 execution spec의 경계는 어디인가

---

## Canonical Subtopics

| Topic | Owns |
|---|---|
| [[topics/ir-cmdq-contract]] | IR schema, tensor metadata, CMDQ format/opcodes, deterministic lowering contract |
| [[topics/tile-semantics-quantization]] | tile lifecycle, KV semantics, mixed precision, bitwidth-memory mapping, prefill/decode semantics |
| [[topics/npu-timing-memory-model]] | cycle timing, DMA/TE/VE latency, SPM/DRAM/Bus/NoC contention |
| [[topics/simulation-validation]] | golden trace, unit/integration/perf validation |
| [[topics/trace-visualization]] | trace schema and visualization surface |

---

## What Moved Out

- IR schema, tensor metadata, CMDQ field contract
  -> [[topics/ir-cmdq-contract]]
- tile semantics, mixed precision, KV semantics, scheduling policy contract
  -> [[topics/tile-semantics-quantization]]
- cycle latency, bus arbitration, NoC contention, SPM timing
  -> [[topics/npu-timing-memory-model]]
- golden trace, integration test, performance validation
  -> [[topics/simulation-validation]]
- trace schema와 시각화 요구사항
  -> [[topics/trace-visualization]]
- workflow, roadmap, naming, review process
  -> [[topics/npu-doc-process]]

---

## Related Topics and Concepts

- Parent umbrella: [[topics/riscv-npu-soc-sim]]
- Related topics: [[topics/ir-cmdq-contract]], [[topics/tile-semantics-quantization]]
- Related concepts: [[concepts/static-scheduling-determinism]], [[concepts/tile-semantics-contract]], [[concepts/mixed-precision-policy]], [[concepts/kv-cache-dram-residency]]
