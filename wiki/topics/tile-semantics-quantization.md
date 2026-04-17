---
title: Tile Semantics & Quantization Contract
type: topic
status: canonical
last_compiled: 2026-04-17
---

# Tile Semantics & Quantization Contract

*last_compiled: 2026-04-17 | source families: spec/architecture, spec/quantization, spec/scheduling*

---

## Summary [coverage: tile / quantization contract layer]

이 topic은 `RISCV_NPU_SoC_SIM`에서 **타일이 무엇인지**와 **mixed precision / KV cache 정책이 그 타일 계약에 어떻게 실리는지**를 다룬다. 즉, tile lifecycle, SPM residency, KV append/update, prefill/decode semantics, `qbits` propagation이 어디에서 정의되고 어떤 invariant를 가져야 하는지를 정리하는 축이다.

핵심 질문은 다음과 같다.

- tile은 최소 실행 단위인지, 최소 메모리 전달 단위인지, 둘 다인지
- `tile_contract_spec`과 `tile_semantics_spec`이 어떤 lifecycle을 고정하는가
- mixed precision과 KV cache quantization은 IR/CMDQ를 어떻게 통과하는가
- prefill/decode, scheduling metadata, bitwidth-memory mapping은 어떤 시스템 정책으로 읽어야 하는가

---

## Included Source Families

- `docs/spec/architecture/*`
- `docs/spec/quantization/*`
- `docs/spec/scheduling/*`

---

## Core Axes

### Tile lifecycle as correctness boundary

tile은 단순 쪼갠 텐서 조각이 아니다. allocation, fill, compute, handoff, retire 과정이 정해진 contract를 가져야 하고, 이 contract는 simulator correctness와 trace 해석 기준까지 함께 결정한다.

### Quantization as system policy

weight / activation / KV bitwidth는 accuracy 옵션이 아니라 DMA bytes, SPM occupancy, DRAM spill, timing model 입력을 동시에 바꾸는 시스템 정책이다. 따라서 mixed precision은 compiler annotation이 아니라 architecture-semantic layer에서 정의돼야 한다.

### KV semantics and phase-aware execution

KV cache는 attention 부속 메모리가 아니라 decode path를 지배하는 장기 상태다. `prefill_decode_workload_semantics_spec.md`와 `kv_cache_semantics_spec.md`는 phase boundary와 KV lifecycle을 정의하고, quantization spec은 그 위에 bitwidth 정책을 올린다.

---

## What Stays Here

- `tile_contract_spec.md`, `tile_semantics_spec.md`, `stb_adoption_rfc.md`
- `mixed_precision_policy.md`, `bitwidth_memory_mapping.md`, `kv_cache_quantization_spec.md`, `quantization_model_overview.md`
- `prefill_decode_workload_semantics_spec.md`, `static_scheduler_semantics_spec.md`

## What Moved Out

- IR schema and CMDQ field contract
  -> [[topics/ir-cmdq-contract]]
- cycle timing, DMA/TE/VE latency, Bus/NoC contention
  -> [[topics/npu-timing-memory-model]]
- test, golden trace, regression criteria
  -> [[topics/simulation-validation]]

---

## Related Topics and Concepts

- Parent umbrella: [[topics/npu-architecture-spec]]
- Related topics: [[topics/ir-cmdq-contract]], [[topics/npu-timing-memory-model]]
- Related concepts: [[concepts/tile-semantics-contract]], [[concepts/mixed-precision-policy]], [[concepts/kv-cache-dram-residency]], [[concepts/prefill-decode-duality]]
