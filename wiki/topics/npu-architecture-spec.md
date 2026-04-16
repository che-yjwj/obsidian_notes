---
title: NPU Architecture & Execution Spec
type: topic
status: canonical
updated: 2026-04-16
---

# NPU Architecture & Execution Spec

*last_curated: 2026-04-16 | source families: spec/architecture, spec/ir, spec/isa, spec/quantization, spec/scheduling*

---

## Summary [coverage: architecture/spec layer]

이 topic은 `RISCV_NPU_SoC_SIM`의 **규범적 execution model**을 다룬다. 관심사는 “프로젝트가 무엇을 만들려는가”가 아니라, **타일이 무엇인지**, **IR이 어떤 의미를 갖는지**, **CMDQ가 어떤 실행 불변식을 보장해야 하는지**, **mixed precision과 KV cache가 어떤 contract 위에서 표현되는지**를 명확히 정의하는 데 있다.

핵심 질문은 다음과 같다.

- NPU IR의 single source of truth는 무엇인가
- tile lifecycle과 HW-SW contract는 어떻게 고정되는가
- ISA/CMDQ는 어떤 deterministic execution model을 강제하는가
- quantization, KV cache, scheduling metadata는 어느 계층에서 결정되는가

---

## Included Source Families

- `docs/spec/architecture/*`
- `docs/spec/ir/*`
- `docs/spec/isa/*`
- `docs/spec/quantization/*`
- `docs/spec/scheduling/*`
- `docs/design/npu_ir_core_reference.md`

---

## Core Axes

### IR as the execution contract

NPU IR은 단순 중간 표현이 아니라 컴파일러와 시뮬레이터가 함께 참조하는 contract layer다. `LayerIR`, `TensorIR`, `QConfig` 같은 구조는 후단 최적화 편의를 위한 메타데이터가 아니라, tile planning, SPM allocation, static scheduling, CMDQ generation이 공유하는 의미론의 중심이다.

### Tile semantics and lifecycle

타일은 최소 실행 단위이자 최소 메모리 전달 단위다. 따라서 tile lifecycle, SPM residency, handoff 규칙, TE-VE 경계는 architecture choice가 아니라 simulator correctness의 일부다.

### CMDQ as deterministic ISA surface

CMDQ는 런타임 스케줄러를 대체하는 정적 명령 스트림이다. 이 topic에서 중요한 것은 opcode 나열보다, **동일 입력이면 동일 실행 결과가 나와야 한다**는 ISA-level determinism이다.

### Quantization as metadata propagation

weight / activation / KV bitwidth는 성능 모델 전체를 관통한다. quantization은 accuracy 보정 부속물이 아니라 DMA bytes, SPM capacity, latency model, DRAM occupancy를 함께 바꾸는 시스템 정책이다.

---

## What Moved Out

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
- Related concepts: [[concepts/static-scheduling-determinism]], [[concepts/tile-semantics-contract]], [[concepts/mixed-precision-policy]], [[concepts/kv-cache-dram-residency]]

