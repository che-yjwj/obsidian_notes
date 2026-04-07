---
title: Memory Hierarchy in AI Accelerators
type: topic
sources:
  - raw/AI-Hardware/Architecture/Memory Hierarchy in AI 33a6cc566b0b81f4b111e6d0e5d21553.md
tags: [memory, HBM, SRAM, NPU, GPU]
updated: 2026-04-07
---

# Memory Hierarchy in AI Accelerators

AI 가속기에서 메모리 계층 구조는 성능의 핵심 결정 요인이다. 연산–데이터 이동–전력/면적 트레이드오프 관점에서 주요 플랫폼을 비교한다.

## NVIDIA H100 / B100: "HW-managed cache + SW-managed shared memory"

```
Register File (per SM)
  ↓
Shared Memory (SW-managed, scratchpad)
  ↓
L1 Cache
  ↓
L2 Cache (HW-managed, 수십 MB)
  ↓
HBM3 / HBM3e (~3 TB/s)
```

- **설계 철학**: 프로그래머가 locality를 만들면 HW가 증폭
- Shared Memory는 FlashAttention, tiled GEMM, softmax fusion의 핵심
- L2는 KV-cache reuse에 활용
- HBM은 대역폭 극대화이나 전력/비용 부담 큼

## Qualcomm AHPM: 모바일·엣지형 "on-chip SRAM 중심"

- 대형 L2 캐시 대신 크고 빠른 on-chip SRAM 비중을 높임
- LPDDR 기반 메모리 서브시스템
- 저전력 설계가 우선: HBM보다 낮은 대역폭을 SRAM으로 보완

## AMD MI300: CPU+GPU 통합 "통합 메모리 풀"

- HBM3 + CDNA 통합 → 공유 메모리 풀
- CPU와 GPU가 동일 물리 메모리 직접 접근 가능
- 대용량 LLM 추론에 유리 (메모리 복사 없음)

## NPU 설계 시사점

| 항목 | 권장 방향 |
|---|---|
| Tensor Engine / Vector Engine | Register에 가장 많은 locality 확보 |
| L2급 공유 SRAM | 타일링 단위에 맞춘 크기 설계 |
| Off-chip | KV-cache 크기가 SRAM을 초과하면 HBM 필요 |
| 일관성 | CPU-NPU 공유 시에만 coherent AXI 고려 |
