---
concept: Tile Semantics and Contract Boundaries
last_compiled: 2026-04-14
topics_connected: [riscv-npu-soc-sim, npu-simulator-compiler, trace-visualization, npu-doc-process]
status: active
---

# Tile Semantics and Contract Boundaries

## Pattern

타일은 이 vault에서 단순한 실행 단위가 아니다. `RISC-V NPU SoC Simulator`에서는 IR, SPM, STB handoff, tile lifecycle의 의미를 고정하는 계약 단위이고, `Trace & Visualization`에서는 golden trace와 validation checklist가 같은 계약을 추적한다. `NPU Documentation Process`는 이 의미를 spec/design/test 문서에 일관되게 유지하도록 강제한다.

즉, tile semantics는 한 문서의 세부 구현이 아니라 compiler, simulator, trace, process를 잇는 공통 boundary condition이다.

## Instances

- **2026-04** in [[../topics/riscv-npu-soc-sim]]: 타일은 atomic execution unit이며 payload는 SPM에만 존재하고 DRAM은 중간 전달 경로가 아니다.
- **2026-04** in [[../topics/npu-simulator-compiler]]: supernode와 static schedule은 결국 tile-level execution contract를 compile time에 고정하는 방식으로 설명된다.
- **2026-04** in [[../topics/trace-visualization]]: `tile_semantics_validation_checklist`와 golden trace가 tile lifecycle, phase, engine handoff를 회귀 기준으로 삼는다.
- **2026-04** in [[../topics/npu-doc-process]]: 문서 우선 workflow는 tile semantics 같은 contract를 spec/design/test 전 계층에서 유지하도록 요구한다.

## What This Means

이 패턴은 tile semantics가 기술 세부가 아니라 시스템 경계라는 뜻이다. 향후 wiki compiler를 돌릴 때도 tile 관련 문서는 한 topic 내부에만 가두면 안 되고, cross-topic concept로 남겨야 문서 구조가 안정된다.

## Sources

- [[../topics/riscv-npu-soc-sim]]
- [[../topics/npu-simulator-compiler]]
- [[../topics/trace-visualization]]
- [[../topics/npu-doc-process]]
