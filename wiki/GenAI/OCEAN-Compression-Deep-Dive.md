---
title: OCEAN Compression Deep Dive
type: deep-dive
sources:
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 1 of 9) 33a6cc566b0b813bb973c15abd5d814a.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 2 of 9) 33a6cc566b0b81e99238dcafa97547ed.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 3 of 9) 33a6cc566b0b8143b946d2a146d08503.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 4 of 9) 33a6cc566b0b81d5b11df9b158e134de.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 5 of 9) 33a6cc566b0b81339ca2c063e67abe20.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 6 of 9) 33a6cc566b0b81c79bcbc45860000c15.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 7 of 9) 33a6cc566b0b81c79b84c74dbb899aa2.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 8 of 9) 33a6cc566b0b814a9495d3ee2214499d.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 9 of 9) 33a6cc566b0b81d89f96cfedd5ba2b85.md
tags: [OCEAN, quantization, compression, entropy-coding, outlier, rotation, manifold]
updated: 2026-04-17
---

# OCEAN Compression Deep Dive

Canonical topic: [[topics/llm-quantization-compression]]
Related concepts: [[concepts/mixed-precision-policy]]

OCEAN은 LLM weight를 단순히 저비트로 깎는 문제가 아니라, 좌표계와 표현 방식을 바꿔서 더 낮은 엔트로피와 더 나은 rate-distortion을 얻으려는 프레임으로 정리할 수 있다. 핵심은 `값 자체`보다 `구조가 드러나는 표현`을 만드는 데 있다.

## Core Thesis

- outlier는 weight의 본질적 정보라기보다 좌표계에 상대적인 관측 결과일 수 있다
- Hadamard 같은 rotation은 정보를 버리지 않고 outlier를 퍼뜨리는 flattening 단계다
- 진짜 압축 이득은 flattening만으로는 부족하고, 구조가 적은 축에 모이는 concentration이 필요하다
- 따라서 목표는 `value-centric quantization`이 아니라 `coordinate-aware reparameterization`이다

## Proposed Representation

OCEAN의 기본 표현은 아래와 같다.

```text
W = s * Q + R + E
```

- `Q`: 저비트 base quantization
- `R`: 작은 정수 범위에 머무는 low-entropy residual
- `E`: sparse exception 또는 outlier 경로

이 분해의 목적은 outlier를 base 분포 안에 억지로 집어넣지 않고, 전체 symbol 분포를 엔트로피 코딩 친화적으로 바꾸는 것이다.

## Compression Mechanics

### 1. Entropy-aware base

- 블록 단위로 INT4 또는 INT3 base를 만든다
- base symbol이 `0`, `±1` 같은 작은 alphabet에 몰리도록 scale과 dead-zone을 조정한다
- 정확도 손실만이 아니라 symbol entropy 자체를 함께 줄이는 방향으로 설계한다

### 2. Residual as cheap correction

- residual은 `|R| in {0,1,2,3}`처럼 작은 정수 범위로 제한한다
- magnitude와 sign을 분리하면 Golomb, Rice, Huffman, ANS류 코딩에 유리하다
- 하드웨어에서는 작은 LUT와 간단한 누산 경로로 복원 가능하다

### 3. Sparse exception path

- exception은 개수가 적은 대신 value precision을 조금 더 허용한다
- index는 delta coding과 RLE로, value는 INT8 또는 FP16으로 처리할 수 있다
- 핵심은 exception 수를 줄여 index overhead를 억제하는 것이다

## Why Pairing Matters

기존 outlier-victim 계열은 주로 정확도 손실을 줄이기 위해 pair를 만든다. OCEAN 관점에서는 pair 선택 기준이 한 단계 더 확장된다.

- base histogram이 더 뾰족해지는가
- residual이 0 근처로 더 잘 모이는가
- exception 개수가 줄어드는가

즉, pairing은 단순한 오차 보정이 아니라 `entropy shaping` 수단이다.

## Axis Transform and Functional View

OCEAN 논의의 후반부는 압축을 더 큰 좌표계 문제로 재해석한다.

### Rotation is not frequency transform

- Hadamard transform은 DCT처럼 의미 있는 주파수축 분해가 아니다
- 시간축이나 공간축이 명확하지 않은 LLM weight에서는 `주파수`보다 `rotation`이라는 표현이 정확하다
- 역할은 정보 제거가 아니라 에너지 재분배다

### Outliers are coordinate-relative

- 같은 operator라도 좌표계를 바꾸면 kurtosis와 `L_inf`는 크게 달라질 수 있다
- accuracy가 유지된 채 outlier가 사라질 수 있다는 점은, 큰 값이 곧 중요한 정보라는 가정을 약하게 만든다
- 이 관점에서 outlier는 매니폴드와 축이 맞지 않아 curvature가 일부 축에 집중된 결과로 해석된다

### Function + residual decomposition

OCEAN은 결국 weight를 값의 나열로 저장하기보다 구조를 생성하는 규칙과 작은 잔차로 나누려는 방향이다.

- `weight ~= function(axis, coefficients) + residual`
- 좋은 축을 찾으면 function 파트가 더 짧게 설명된다
- residual은 더 공격적으로 양자화하고 코딩할 수 있다

## Hardware-Friendly Bitstream

권장되는 비트스트림은 3개 스트림 구조다.

1. Base stream
   - 블록 단위 INT4, INT3 packed symbols
   - 블록 오프셋 테이블로 랜덤 액세스 유지
2. Residual stream
   - 존재 여부 bitmask
   - small alphabet residual values
   - sign bitplane 분리 가능
3. Exception stream
   - block count
   - delta-coded indices
   - higher precision values

이 구조는 NPU에서 `base fast path + optional correction path`로 구현할 수 있어, 정확도 보호와 런타임 비용을 분리하기 쉽다.

## Validation Roadmap

OCEAN raw 대화에서 가장 실용적인 부분은 실험 가설이 명확하다는 점이다.

### H1. Axis transform hypothesis

- 채널 재정렬이나 rotation을 적용하면 동일 기능 또는 유사 출력을 유지하면서 분포의 구조 집중도가 증가한다

### H2. Functional representation hypothesis

- 축 변환 후 function + residual 분해를 하면 residual entropy와 INT4 이하 quantization distortion이 함께 감소한다

### H3. Entropy-coding hypothesis

- 축 변환과 함수적 표현은 동일 perplexity 또는 유사 정확도에서 bit-rate를 줄인다

### Minimal experiments

1. single layer 또는 single matrix에서 baseline INT4와 transform variants 비교
2. entropy, quantization MSE, top-k energy ratio, spectral flatness 측정
3. TinyLLaMA급 모델에서 original, Hadamard, aligned basis를 비교
4. perplexity뿐 아니라 reasoning stability와 long-context failure를 함께 본다

## Practical Takeaways

- Hadamard rotation만으로도 outlier 완화와 양자화 안정성 향상은 기대할 수 있다
- 하지만 compression-optimal representation을 주장하려면 flattening이 아니라 concentration 근거가 필요하다
- 따라서 OCEAN의 강점은 `rotation + residual + entropy coding`의 조합보다, 이를 `coordinate-aware compression framework`로 묶는 데 있다
- 이 vault에서는 OCEAN을 별도 방법론 페이지로 유지하고, 이후 QuaRot, SpinQuant, SmoothQuant와 비교 페이지로 확장하는 것이 적절하다

## Related Pages

- 상위 개요: [[wiki/GenAI/LLM-Quantization-and-Compression]]
