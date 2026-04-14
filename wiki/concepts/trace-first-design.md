---
concept: Trace-First Design
last_compiled: 2026-04-14
topics_connected: [riscv-npu-soc-sim, trace-visualization, npu-doc-process, npu-simulator-compiler]
status: active
---

# Trace-First Design

## Pattern

이 vault에서는 trace가 구현 후에 붙는 로그가 아니라 설계 초기부터 고정해야 하는 인터페이스다. `RISC-V NPU SoC Simulator`는 trace를 simulator output의 핵심 계약으로 취급하고, `Trace & Visualization`은 trace schema와 visualization requirement를 미리 정한다. `NPU Documentation Process`는 trace-first design을 roadmap와 review workflow 안에 포함시키고, `NPU Simulator & Compiler`는 CMDQ와 static schedule의 결과가 결국 trace를 통해 검증된다는 점을 드러낸다.

## Instances

- **2026-04** in [[../topics/riscv-npu-soc-sim]]: trace writer와 visualizer가 project 목표의 필수 축으로 포함된다.
- **2026-04** in [[../topics/trace-visualization]]: trace schema, golden trace, performance validation이 이미 독립 문서군으로 존재한다.
- **2026-04** in [[../topics/npu-doc-process]]: roadmap와 documentation review가 trace-first design을 반복해서 강조한다.
- **2026-04** in [[../topics/npu-simulator-compiler]]: static-scheduled artifact는 trace를 통해서만 병목과 correctness를 비교할 수 있다.

## What This Means

trace-first design은 simulator 프로젝트가 “결과를 남기는 도구”가 아니라 “비교 가능한 실행 모델”이 되기 위한 조건이다. compiler는 앞으로 trace 관련 문서를 하나의 spec 폴더로만 보지 말고, 이 개념을 cross-topic principle로 유지해야 한다.

## Sources

- [[../topics/riscv-npu-soc-sim]]
- [[../topics/trace-visualization]]
- [[../topics/npu-doc-process]]
- [[../topics/npu-simulator-compiler]]
