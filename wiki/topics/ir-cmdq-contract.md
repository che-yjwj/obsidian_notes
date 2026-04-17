---
title: IR & CMDQ Contract
type: topic
status: canonical
last_compiled: 2026-04-17
---

# IR & CMDQ Contract

*last_compiled: 2026-04-17 | source families: spec/ir, spec/isa, design/npu_ir_core_reference*

---

## Summary [coverage: IR / ISA contract layer]

이 topic은 `RISCV_NPU_SoC_SIM`에서 **IR과 CMDQ가 어떤 계약을 맺는지**를 다룬다. 관심사는 opcode 목록 자체보다, `LayerIR`/`TensorIR`/`QConfig`가 어떤 single source of truth 역할을 하고, 그 결과가 어떤 규칙으로 `CMDQ` 정적 명령 스트림에 내려가며, 시뮬레이터가 무엇을 deterministic하게 해석해야 하는지다.

핵심 질문은 다음과 같다.

- 어떤 IR 필드가 compiler와 simulator가 공유하는 execution contract인가
- `TileGraph`와 `ScheduleDAG`의 결과가 CMDQ에 어떤 불변식으로 반영되는가
- `deps_before`, `engine_type`, `layer_id`, `qbits_*` 같은 필드는 어디까지 의미를 가져야 하는가
- 동일 입력, 동일 config, 동일 초기 메모리 상태에서 CMDQ가 어떤 결정론을 보장해야 하는가

---

## Included Source Families

- `docs/spec/ir/*`
- `docs/spec/isa/*`
- `docs/design/npu_ir_core_reference.md`

---

## Core Axes

### IR as single source of truth

IR은 단순 변환 중간 단계가 아니라 compiler pass와 simulator가 함께 참조하는 규범 계층이다. `TensorIR`, `LayerIR`, `QuantMeta` 같은 필드는 후단 편의용 annotation이 아니라, 타일링, 메모리 배치, 정적 스케줄링, trace attribution이 모두 기대는 공통 의미론이다.

### CMDQ as execution surface

CMDQ는 런타임 스케줄러를 대체하는 정적 실행 인터페이스다. 중요한 것은 “어떤 opcode가 있느냐”보다, compiler가 산출한 dependency와 engine assignment가 simulator에서 동일하게 재현되어야 한다는 점이다.

### Deterministic lowering boundary

IR -> TileGraph -> Schedule -> CMDQ 경로는 자유형 최적화 체인이 아니라, 의미를 보존하는 lowering 경로다. 따라서 각 단계는 execution meaning을 추가로 창조하기보다, 이미 정의된 contract를 더 구체적인 artifact로 전개하는 역할을 맡아야 한다.

---

## What Stays Here

- `npu_ir_spec.md`, `tensor_metadata_spec.md`, `quantization_ir_extension.md`
- `cmdq_overview.md`, `cmdq_format_spec.md`, `opcode_set_definition.md`
- IR/CMDQ field contract, dependency semantics, deterministic execution rule

## What Moved Out

- tile lifecycle, SPM residency, KV semantics, mixed precision policy
  -> [[topics/tile-semantics-quantization]]
- cycle latency, bus arbitration, NoC contention
  -> [[topics/npu-timing-memory-model]]
- golden trace and validation ownership
  -> [[topics/simulation-validation]]

---

## Related Topics and Concepts

- Parent umbrella: [[topics/npu-architecture-spec]]
- Related topics: [[topics/tile-semantics-quantization]], [[topics/npu-timing-memory-model]]
- Related concepts: [[concepts/static-scheduling-determinism]], [[concepts/tile-semantics-contract]]
