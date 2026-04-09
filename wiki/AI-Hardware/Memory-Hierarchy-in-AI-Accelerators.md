---
title: Memory Hierarchy in AI Accelerators
type: topic
sources:
  - raw/AI-Hardware/Architecture/Memory Hierarchy in AI 33a6cc566b0b81f4b111e6d0e5d21553.md
tags: [memory, HBM, SRAM, NPU, GPU]
updated: 2026-04-09
---

# Memory Hierarchy in AI Accelerators

AI 가속기에서 메모리 계층 구조는 성능의 핵심 결정 요인이다. 연산–데이터 이동–전력/면적 트레이드오프 관점에서 주요 플랫폼을 비교한다.

이 페이지는 [[wiki/AI-Hardware/NPU-Architecture]]의 상위 아키텍처 선택을 메모리 관점에서 구체화하고, [[wiki/AI-Hardware/Simulator-and-Implementation-Tools]]에 등장하는 구현체들이 왜 서로 다른 메모리 전략을 택하는지 설명하는 허브 역할을 한다.

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

## 왜 이 구조가 아키텍처 선택을 가른다

[[wiki/AI-Hardware/NPU-Architecture]]에서 다룬 플랫폼들은 연산 유닛보다 먼저 메모리 병목을 어떤 방식으로 처리할지부터 갈린다.

- **Tesla AI 칩 계열**: 차량 내 실시간 추론과 전력 제약이 강해서 데이터 재사용과 on-chip/local memory 활용이 우선이다. HBM 중심 서버 GPU와 달리 edge NPU는 SRAM과 짧은 데이터 경로 확보가 더 중요하다.
- **AMD Versal / FINN 계열**: 대형 공유 캐시보다 스트리밍 데이터플로와 로컬 버퍼 조합이 핵심이다. 이는 낮은 정밀도 모델과 deterministic pipeline에 잘 맞는다.
- **TPU Sparse Core / 추천계 워크로드**: 연산량보다 irregular memory access를 얼마나 흡수하느냐가 성능을 좌우한다. 이 경우 메모리 계층 설계가 곧 아키텍처 특화 포인트다.
- **AMBA/CHI 선택**: coherent interconnect는 CPU와 NPU가 같은 텐서를 자주 공유할 때만 정당화된다. 그렇지 않으면 비일관 AXI와 로컬 SRAM 계층이 더 단순하고 효율적이다.

## 구현 사례와의 연결

[[wiki/AI-Hardware/Simulator-and-Implementation-Tools]]에서 소개한 구현체들도 같은 축으로 읽을 수 있다.

| 구현체 | 메모리 전략 | 이 페이지와의 연결 포인트 |
|---|---|---|
| HyperAccel LPU | LPDDR5X + on-chip reuse 극대화 | HBM 없이도 토큰 지연을 줄이려면 off-chip 왕복보다 데이터플로 설계가 우선임을 보여준다 |
| Coral NPU | 작은 모델을 SRAM 안에 최대한 유지 | edge 추론에서는 온칩 적재 가능성이 곧 성능 상한을 결정한다 |
| MTIA / TPU 계열 | workload별 전용 메모리 접근 최적화 | dense/sparse 접근 패턴 분리가 메모리 계층 분화로 이어진다 |
| Transformer C 구현 | 계층 없는 단순 메모리 접근 노출 | 실제 가속기에서 왜 scratchpad, shared SRAM, tiling이 필요한지 기준선 역할을 한다 |
| NPU 슈퍼노드 | 여러 코어 간 데이터 분산·집계 필요 | 단일 코어 hierarchy보다 interconnect와 shared buffer 설계가 중요해진다 |

## Tesla AI5/AI6, LPU, NPU를 함께 읽는 기준

graphify가 추론한 연결은 메모리 병목 관점에서는 타당하다.

- **Tesla AI5/AI6 roadmap과의 연결**: 차량용 NPU는 배치 처리보다 실시간 응답과 전력 예산이 중요하므로, 대규모 HBM 확장보다 local memory, scheduler, 데이터 이동 최소화가 더 직접적인 설계 변수다.
- **HyperAccel LPU와의 연결**: LPU 역시 대역폭 절대치보다 토큰 단위 지연과 off-chip traffic 감축을 우선한다. 즉, 서로 다른 시장이지만 "memory-balanced compute"라는 공통 해석 축이 있다.
- **NPU architecture 전반과의 연결**: compute array, NoC, bus, coherence 같은 구조 선택은 결국 어떤 데이터가 어느 계층에 머무를지에 대한 결정의 결과다.

## NPU 설계 체크포인트

메모리 계층은 별도 하위 블록이 아니라 아키텍처 결정 표를 읽는 기준이어야 한다.

| 항목 | 권장 방향 |
|---|---|
| Tensor Engine / Vector Engine | Register에 가장 많은 locality 확보 |
| L2급 공유 SRAM | 타일링 단위에 맞춘 크기 설계 |
| Off-chip | KV-cache 크기가 SRAM을 초과하면 HBM 필요 |
| 일관성 | CPU-NPU 공유 시에만 coherent AXI 고려 |
