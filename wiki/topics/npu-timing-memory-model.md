---
title: NPU Timing & Memory Model
type: topic
status: canonical
updated: 2026-04-16
---

# NPU Timing & Memory Model

*last_curated: 2026-04-16 | source families: spec/timing, overview/memory_noc, design/dma-spm-engine timing*

---

## Summary [coverage: timing/memory layer]

이 topic은 `RISCV_NPU_SoC_SIM`에서 가장 hardware-like한 부분을 모은다. 관심사는 tile이나 IR의 의미론이 아니라, **DMA가 몇 cycle 걸리는지**, **Bus/NoC contention을 어떻게 모델링하는지**, **SPM과 DRAM 사이의 병목이 어떻게 드러나는지**, **TE/VE/utilization/stall을 어떤 규칙으로 계산하는지**다.

프로젝트의 신뢰성은 이 계층에서 결정된다. 이유는 간단하다. architecture spec이 아무리 잘 서 있어도, timing과 memory model이 불안정하면 simulator output이 architecture decision에 쓸 수 있는 수준으로 올라오지 못한다.

---

## Included Source Families

- `docs/spec/timing/*`
- `docs/overview/memory_noc_overview.md`
- `docs/design/dma_engine_design.md`
- `docs/design/te_engine_design.md`
- `docs/design/ve_engine_design.md`
- `docs/design/tile_rt_analysis.md`
- `docs/design/npu_simulator_core_design.md`

---

## Core Axes

### Bus / NoC contention as first-class model

Bus와 NoC는 백그라운드 인프라가 아니라 latency 결과를 결정하는 핵심 제약이다. arbitration policy, tie-break rule, queue depth, weighted RR 같은 규칙은 모두 결정론적 시뮬레이션을 위한 일부다.

### DMA and SPM as throughput bottleneck

실제 stall은 대부분 compute array보다 DMA scheduling, bank conflict, outstanding queue saturation에서 나온다. 이 topic은 TE/VE peak보다 **bytes per cycle**, **effective bandwidth**, **stall propagation**을 우선시한다.

### Multi-engine overlap and issue timing

ControlFSM, DMA cluster, TE cluster, VE cluster가 같은 global cycle loop 안에서 어떻게 상호작용하는지가 핵심이다. 이 계층은 “무엇을 계산하는가”가 아니라 “언제 issue되고 언제 완료되는가”를 모델링한다.

---

## What Moved Out

- IR, ISA, quantization, tile contract
  -> [[topics/npu-architecture-spec]]
- trace schema와 visualization surface
  -> [[topics/trace-visualization]]
- validation and golden trace acceptance
  -> [[topics/simulation-validation]]

---

## Related Topics and Concepts

- Parent umbrella: [[topics/riscv-npu-soc-sim]]
- Related concepts: [[concepts/memory-bandwidth-bottleneck]], [[concepts/static-scheduling-determinism]], [[concepts/trace-first-design]]

