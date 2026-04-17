---
title: NPU/LPU Simulator and Implementation Tools
type: deep-dive
sources:
  - raw/AI-Hardware/Simulator/HyperAccel LPU Explanation/HyperAccel LPU Explanation (Part 1 of 3) 33a6cc566b0b81349b96fd84f1659f77.md
  - raw/AI-Hardware/Simulator/HyperAccel LPU Explanation/HyperAccel LPU Explanation (Part 2 of 3) 33a6cc566b0b8184b3e7c80f64b99235.md
  - raw/AI-Hardware/Simulator/HyperAccel LPU Explanation/HyperAccel LPU Explanation (Part 3 of 3) 33a6cc566b0b81dbb100c4737c051345.md
  - raw/AI-Hardware/Simulator/coral npu/coral npu (Part 1 2) 33a6cc566b0b813d98cee31bdda3eac4.md
  - raw/AI-Hardware/Simulator/coral npu/coral npu (Part 2 2) 33a6cc566b0b81d8a159d87e7e1b114c.md
  - raw/AI-Hardware/Simulator/MTIA TPU Paper Analysis 33a6cc566b0b816ebdbfedfedbb11fb6.md
  - raw/AI-Hardware/Simulator/Transformer C Implementation Analysis 33a6cc566b0b818585ddf76217de5112.md
  - raw/AI-Hardware/Simulator/NPU Supernode Concept 33a6cc566b0b81aca099f1e6a9a8061e.md
tags: [LPU, NPU, HyperAccel, Coral, MTIA, simulator]
updated: 2026-04-17
---

# NPU/LPU Simulator and Implementation Tools

Canonical topic: [[topics/npu-simulator-compiler]]
Related concepts: [[concepts/static-scheduling-determinism]], [[concepts/trace-first-design]]

## Role in This Wiki

- This page is a curated survey of external platforms, implementation styles, and simulator-facing reference systems.
- The canonical reusable hub for compiler/simulator structure is [[topics/npu-simulator-compiler]].
- When material becomes a stable navigation point rather than a survey note, it should move into the canonical topic layer.

## Boundary

This page should focus on:
- vendor/platform comparisons such as HyperAccel, Coral, MTIA, and Groq-like execution styles
- implementation intuition and benchmark-style interpretation
- educational references such as low-level Transformer implementations

This page should not become the main home for:
- RISCV_NPU_SoC_SIM execution-spec ownership
- trace schema or validation rules
- project process documents
- detailed IR/CMDQ contract text

Those belong in the canonical topic chain around [[topics/riscv-npu-soc-sim]], [[topics/ir-cmdq-contract]], [[topics/trace-visualization]], and [[topics/simulation-validation]].

## HyperAccel LPU (국내 AI 반도체 스타트업)

**LPU(Latency Processing Unit)**: LLM 추론 전용 도메인 특화 가속기.

### 핵심 설계 철학
- GPU는 수천 개의 소형 코어로 병렬 연산 → LPU는 소수의 대형 특화 코어로 **데이터 흐름 우선**
- **메모리 대역폭 병목** 해결이 핵심: LPDDR5X 기반, streamlined dataflow
- off-chip ↔ on-chip 전송 최소화

### 아키텍처 특성
- LLM 연산 플로우(Attention, FFN, Norm) 전 과정을 추론 레벨에서 처리
- 다중 LPU 연동으로 확장 가능한 파이프라인 구성
- Latency-optimized + Scalable + Memory-balanced Compute

### GPU 대비 차별점
| 항목 | GPU | HyperAccel LPU |
|---|---|---|
| 코어 구성 | 수천 개 소형 코어 | 소수 대형 특화 코어 |
| 최적화 대상 | Throughput (배치) | Latency (토큰 단위) |
| 메모리 | HBM (고대역폭) | LPDDR5X (저전력) |

## Google Coral NPU (Edge TPU)

- Google의 엣지 AI 가속기; USB/PCIe/M.2 폼팩터
- 4 TOPS @ 0.5W의 초저전력 설계
- TensorFlow Lite 모델만 지원, 모델 크기 제한 있음
- 온-칩 SRAM에 모델 전체가 올라와야 최고 성능 발휘

## Meta MTIA vs Google TPU

- MTIA(Meta Training and Inference Accelerator): 추천 시스템 workload 특화
- TPU와 달리 sparse embedding 접근 패턴에 최적화
- 두 설계 모두 범용 GPU 대비 특정 workload 집중으로 PPA 우위

## Transformer C 구현 분석

- 순전파(forward pass)를 순수 C로 구현한 교육용/분석용 코드
- BLAS 없이 행렬 곱, Softmax, LayerNorm을 직접 구현하여 동작 원리 이해에 유용
- 시뮬레이터 또는 NPU 펌웨어 레퍼런스로 활용 가능

## NPU 슈퍼노드 개념

- 여러 NPU 코어를 묶어 하나의 논리 연산 단위(슈퍼노드)로 추상화
- 타일링·스케줄링 복잡도를 줄이고 대형 모델을 분산 처리하는 구조
