---
concept: KV Cache as DRAM-Resident State
last_compiled: 2026-04-14
topics_connected: [hw-friendly-model-design, llm-quantization-compression, riscv-npu-soc-sim, trace-visualization]
status: active
---

# KV Cache as DRAM-Resident State

## Pattern

KV cache는 단순 intermediate activation이 아니라 decode 단계에서 반복적으로 참조되는 장기 상태다. 이 때문에 `HW-Friendly Model Design`은 KV-cache 정책을 모델 구조 선택과 연결하고, `LLM Quantization & Compression`은 KV bitwidth를 별도 정책으로 다룬다. `RISC-V NPU SoC Simulator`는 KV cache를 DRAM-resident state로 모델링하며, `Trace & Visualization`은 decode latency와 DRAM traffic을 이 상태를 중심으로 해석한다.

## Instances

- **2026-04** in [[../topics/hw-friendly-model-design]]: KV-cache access와 conditional loading이 edge-NPU 친화 설계의 핵심으로 등장한다.
- **2026-04** in [[../topics/llm-quantization-compression]]: KV cache quantization이 weight/activation과 분리된 축으로 정리된다.
- **2026-04** in [[../topics/riscv-npu-soc-sim]]: decode는 `KV_LOAD_TILE`이 선행되고 KV는 DRAM 영속 저장소로 모델링된다.
- **2026-04** in [[../topics/trace-visualization]]: tutorial과 golden trace가 decode 병목을 KV-cache DRAM traffic으로 설명한다.

## What This Means

KV cache는 model design, quantization, runtime scheduling, trace analysis를 하나로 묶는 상태 객체다. 그래서 topic 내부 부록이 아니라 별도 concept로 유지하는 편이 미래 컴파일 품질에 유리하다.

## Sources

- [[../topics/hw-friendly-model-design]]
- [[../topics/llm-quantization-compression]]
- [[../topics/riscv-npu-soc-sim]]
- [[../topics/trace-visualization]]
