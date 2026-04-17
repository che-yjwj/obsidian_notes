---
concept: Mixed Precision as a System Policy
last_compiled: 2026-04-14
topics_connected: [llm-quantization-compression, operator-coordinate-compression, riscv-npu-soc-sim, npu-architecture, trace-visualization]
status: active
---

# Mixed Precision as a System Policy

## Pattern

mixed precision은 개별 quantization 기법의 선택이 아니라 시스템 전 계층에 전파되는 정책이다. `LLM Quantization & Compression`에서는 weight, activation, KV를 따로 다루는 이유와 기법들이 정리되고, `RISC-V NPU SoC Simulator`에서는 qbits가 IR annotation, memory mapping, DMA bytes, engine latency, CMDQ 필드까지 영향을 미친다. `Trace & Visualization`은 이 정책의 효과를 latency, traffic, utilization으로 비교하는 관측층이고, `NPU Architecture`는 이러한 정책을 하드웨어 설계 제약과 연결한다.

## Instances

- **2026-04** in [[../topics/llm-quantization-compression]]: PTQ, KV cache quantization, OCEAN, TurboQuant 등 다양한 quantization 전략이 비교된다.
- **2026-04** in [[../topics/operator-coordinate-compression]]: coordinate choice가 outlier, concentration, entropy coding 효율을 바꾼다는 이론이 mixed-precision threshold와 bit allocation 설계의 상위 설명 층을 제공한다.
- **2026-04** in [[../topics/riscv-npu-soc-sim]]: `qbits_weight`, `qbits_activation`, `qbits_kv`가 IR과 CMDQ를 따라 흐르는 독립 필드로 정의된다.
- **2026-04** in [[../topics/trace-visualization]]: quantization impact plot이 필수 visualization 항목으로 명시된다.
- **2026-04** in [[../topics/npu-architecture]]: mixed-precision transport와 reconstruction이 아키텍처 수준 tradeoff로 등장한다.

## What This Means

이 concept는 quantization 문서를 simulator 구현 문서와 다시 연결해 준다. 앞으로 compiler가 이 주제를 더 정교하게 다루려면 mixed precision을 topic의 부속 절이 아니라 cross-cutting policy로 계속 유지해야 한다.

## Sources

- [[../topics/llm-quantization-compression]]
- [[../topics/operator-coordinate-compression]]
- [[../topics/riscv-npu-soc-sim]]
- [[../topics/npu-architecture]]
- [[../topics/trace-visualization]]
