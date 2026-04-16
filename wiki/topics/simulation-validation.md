---
title: Simulation Validation & Golden Traces
type: topic
status: canonical
updated: 2026-04-16
---

# Simulation Validation & Golden Traces

*last_curated: 2026-04-16 | source families: docs/test, golden trace examples, validation protocol*

---

## Summary [coverage: validation layer]

이 topic은 `RISCV_NPU_SoC_SIM`의 시뮬레이터가 “돌아간다”를 넘어서 “믿을 수 있다”로 올라가기 위한 검증 층을 다룬다. integration test, unit test, performance validation protocol, golden trace example은 모두 같은 질문에 답한다.

> 이 시뮬레이터의 출력은 regression-safe하며, architecture decision에 다시 써도 되는가?

따라서 이 topic은 trace visualization 자체보다 **acceptance criteria**, **reference comparison**, **error tolerance**, **golden artifact 관리 방식**에 초점을 둔다.

---

## Included Source Families

- `docs/test/unit_test_spec.md`
- `docs/test/integration_test_spec.md`
- `docs/test/test_plan.md`
- `docs/test/performance_validation_protocol.md`
- `docs/test/golden_trace_examples.md`
- `docs/test/examples/*`

---

## Core Axes

### Golden traces as regression anchors

golden trace는 단순 예시가 아니라 deterministic execution이 유지되는지 보는 고정점이다. same CMDQ + same config에서 trace shape가 흔들리면 simulator가 architecture tool이 아니라 moving target이 된다.

### Performance validation as model trust boundary

이 프로젝트는 cycle-accurate가 아니라 resource-accurate simulator를 지향한다. 그래서 validation 핵심은 exact cycle matching이 아니라, latency/bandwidth/utilization 예측이 허용 오차 안에 들어오는지 검증하는 데 있다.

### Test hierarchy

unit test는 엔진/모듈 correctness, integration test는 pipeline coherence, performance validation은 architecture-model usefulness를 본다. 셋은 같은 테스트가 아니라 다른 실패 모드를 막는 서로 다른 계층이다.

---

## What Moved Out

- trace schema, visualization UX, heatmap/timeline semantics
  -> [[topics/trace-visualization]]
- SDD workflow, roadmap, review process
  -> [[topics/npu-doc-process]]
- timing model 자체
  -> [[topics/npu-timing-memory-model]]

---

## Related Topics and Concepts

- Parent umbrella: [[topics/riscv-npu-soc-sim]]
- Related concepts: [[concepts/trace-first-design]], [[concepts/static-scheduling-determinism]]
