---
title: Outlier Mitigation Methods Comparison
type: comparison
sources:
  - raw/GenAI/Compression/OliVe Paper Summary 33a6cc566b0b81419e7bfc22f35bdf25.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 2 of 9) 33a6cc566b0b81e99238dcafa97547ed.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 4 of 9) 33a6cc566b0b81d5b11df9b158e134de.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 5 of 9) 33a6cc566b0b81339ca2c063e67abe20.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 6 of 9) 33a6cc566b0b81c79bcbc45860000c15.md
tags: [quantization, outlier, SmoothQuant, QuaRot, SpinQuant, AWQ, OCEAN]
updated: 2026-04-09
---

# Outlier Mitigation Methods Comparison

LLM 저비트 양자화에서 outlier 처리는 크게 `스케일링`, `회전`, `재배치` 계열로 나뉜다. 이 페이지는 `SmoothQuant`, `QuaRot`, `SpinQuant`를 중심에 두고, OCEAN 관점에서 각 방법의 위치를 비교한다.

## One-Line Summary

- `SmoothQuant`: weight와 activation을 재스케일해서 축별 동적 범위를 맞춘다
- `QuaRot`: 고정 orthogonal rotation으로 outlier를 여러 축에 분산시킨다
- `SpinQuant`: 학습된 rotation으로 ultra-low-bit 정확도 손실을 더 줄인다
- `OCEAN`: outlier 완화 자체보다 좌표계, 표현, entropy coding을 함께 최적화하는 상위 프레임이다

## Comparison Table

| 기법 | 핵심 아이디어 | 좌표계 해석 | 장점 | 한계 |
|---|---|---|---|---|
| SmoothQuant | weight/activation 재스케일 | 축별 동적 범위를 맞추는 scaling | 구현 난도와 HW 복잡도가 중간 수준 | 축 재정렬이나 구조 집중까지는 가지 않음 |
| QuaRot | Hadamard 계열 고정 회전 | 의미를 바꾸지 않는 orthogonal axis transform | outlier-free rotated space를 만들어 4-bit W/A/KV에 유리 | 왜 효과가 나는지 기하학적으로 일반화하지 않음 |
| SpinQuant | learned rotation | 회전 행렬 자체를 학습해 더 좋은 좌표계 탐색 | 고정 회전보다 정확도 상한이 높을 수 있음 | 학습 비용과 적용 복잡도가 증가 |
| AWQ | activation-aware scaling | 민감 축을 스케일로 보호 | 실무 적용 사례가 많음 | rotation처럼 좌표계 자체를 바꾸지는 않음 |
| OliVe | outlier-victim pairing | 값 보호보다 재배치 문제에 가까움 | HW 복잡도가 매우 낮음 | 표현/코딩 관점의 상위 프레임은 아님 |
| OCEAN | axis transform + function/residual + coding | 좌표계와 표현을 함께 재설계 | rate-distortion과 entropy coding까지 연결 가능 | 아직 구현·검증 범위가 더 넓고 무거움 |

## 1. SmoothQuant: Scaling 계열의 대표

- 표면적으로는 activation scale 조정과 weight scaling이 핵심이다
- OCEAN raw 관점에서는 이것도 `축별 동적 범위를 맞춰 구조를 더 균질한 좌표계로 옮기는 작업`으로 해석할 수 있다
- 즉, outlier를 절대적 큰 값으로 보기보다 현재 좌표계에서 과도하게 튄 값으로 다룬다는 점에서는 rotation 계열과 방향이 같다
- 다만 permutation이나 orthogonal rotation처럼 좌표계 자체를 크게 바꾸지는 않으므로 구조 집중(concentration)까지 끌고 가기 어렵다

## 2. QuaRot: Fixed Rotation 계열의 기준점

- QuaRot의 핵심은 출력을 바꾸지 않는 방식으로 내부 표현을 회전시키고, 그 결과 큰 값을 여러 축에 퍼뜨리는 것이다
- 이때 Hadamard transform은 주파수 분해가 아니라 `L2 에너지를 보존하는 직교 회전`으로 보는 편이 정확하다
- 실무 장점은 명확하다
  - 고정 회전이라 구현이 단순하다
  - outlier-free rotated space를 만들어 균일 저비트 양자화에 유리하다
  - HW-friendly한 fast transform으로 설명하기 쉽다
- 반면 한계도 분명하다
  - outlier가 왜 좌표계에 상대적인지에 대한 일반화가 약하다
  - rotation 이후의 표현 압축이나 entropy coding까지는 프레임이 확장되지 않는다

## 3. SpinQuant: Learned Rotation으로 상한 성능 추구

- SpinQuant는 `무작위 또는 고정 회전도 편차가 있다`는 문제의식에서 출발한다
- 핵심은 회전 행렬을 학습해 ultra-low-bit 구간에서 정확도 갭을 더 줄이는 것이다
- 따라서 비교 실험에서는 보통 이렇게 해석하면 된다
  - QuaRot: 단순하고 HW-friendly한 baseline
  - SpinQuant: 더 좋은 좌표계를 찾는 learned upper bound
- 이 관점은 OCEAN의 `축 변환 자체가 핵심 변수`라는 가설을 검증할 때도 유용하다

## 4. OCEAN과의 관계

OCEAN 프레임에서 보면 `SmoothQuant`, `QuaRot`, `SpinQuant`는 모두 outlier mitigation의 하위 구현들이다.

- SmoothQuant는 `scaling`
- QuaRot는 `fixed orthogonal transform`
- SpinQuant는 `learned transform`

OCEAN이 더 넓게 잡는 범위는 아래와 같다.

1. outlier를 좌표계-매니폴드 불일치의 결과로 해석한다
2. transform을 단순 preprocessing이 아니라 `axis design` 문제로 본다
3. 최종 목표를 quantization 안정화가 아니라 `function + residual + entropy coding`까지 포함한 rate-distortion 최적화로 둔다

즉, QuaRot/SpinQuant는 강한 선행 실증 사례이고, OCEAN은 그 위에 놓이는 상위 압축 프레임이다.

## 5. 실무적으로 어떻게 쓰면 좋은가

- 빠른 baseline이 필요하면 `SmoothQuant` 또는 `QuaRot`
- ultra-low-bit 정확도를 더 밀어붙이고 싶으면 `SpinQuant`
- NPU 관점에서 제어 복잡도를 낮게 유지하고 싶으면 `SmoothQuant`, `QuaRot`, `OliVe`
- 압축률과 bitstream 구조까지 함께 설계하려면 `OCEAN` 방향이 더 적합하다

## 6. 이 Vault에서의 권장 Positioning

- `LLM Quantization and Compression Methods`는 상위 개요 페이지로 유지
- 이 페이지는 outlier mitigation 계열 비교 페이지로 둔다
- `OCEAN Compression Deep Dive`는 별도 방법론 페이지로 유지한다

## Related Pages

- 상위 개요: [[wiki/GenAI/LLM-Quantization-and-Compression]]
- 방법론 상세: [[wiki/GenAI/OCEAN-Compression-Deep-Dive]]
