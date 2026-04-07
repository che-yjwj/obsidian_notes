---
title: LLM Quantization and Compression Methods
type: topic
sources:
  - raw/GenAI/Compression/DC-LLM Paper Summary 33a6cc566b0b810abe58cd3c651ad3ad.md
  - raw/GenAI/Compression/OliVe Paper Summary 33a6cc566b0b81419e7bfc22f35bdf25.md
  - raw/GenAI/Compression/TurboQuant Concept Summary/TurboQuant Concept Summary (Part 1 of 2) 33a6cc566b0b81ebbb8afb6f34807867.md
  - raw/GenAI/Compression/TurboQuant Concept Summary/TurboQuant Concept Summary (Part 2 of 2) 33a6cc566b0b81bd9472e15a23617554.md
  - raw/GenAI/Compression/TurboQuant PyTorch Implementation 33a6cc566b0b8163a307f12187edcc91.md
  - raw/GenAI/Compression/LLM Quantization Architecture 33a6cc566b0b8181a2d9e4bd12d3d4c8.md
  - raw/GenAI/Compression/Microscaling vs Mixed-Precision 33a6cc566b0b81798986d497a0b25f67.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 1 of 9) 33a6cc566b0b813bb973c15abd5d814a.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 2 of 9) 33a6cc566b0b81e99238dcafa97547ed.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 3 of 9) 33a6cc566b0b8143b946d2a146d08503.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 4 of 9) 33a6cc566b0b81d5b11df9b158e134de.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 5 of 9) 33a6cc566b0b81339ca2c063e67abe20.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 6 of 9) 33a6cc566b0b81c79bcbc45860000c15.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 7 of 9) 33a6cc566b0b81c79b84c74dbb899aa2.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 8 of 9) 33a6cc566b0b814a9495d3ee2214499d.md
  - raw/GenAI/Compression/OCEAN-based LLM Compression/OCEAN-based LLM Compression (Part 9 of 9) 33a6cc566b0b81d89f96cfedd5ba2b85.md
tags: [quantization, compression, LLM, DC-LLM, OliVe, TurboQuant, OCEAN, microscaling]
updated: 2026-04-07
---

# LLM Quantization and Compression Methods

LLM 추론은 **메모리-바운드** 워크로드다. 가중치 전송 비용이 병목이므로, 압축의 핵심 목표는 모델 크기·메모리 대역폭 절감이다.

## DC-LLM (ICLR 2026)

**Dynamic Linear Combination** 기반 hardware-friendly 가중치 압축.

- **아이디어**: 가중치 블록을 `seed + basis(난수 생성) + 계수 벡터`로 표현 → 저장량 대폭 감소
- **복원**: LFSR(shift+XOR) 기반 하드웨어 생성기 → HW 비용 낮음
- **적응형 설계**: 블록마다 basis 개수(k)를 다르게 하여 난이도 차이 대응
- weight-only 압축, autoregressive 추론에 특히 효과적

## OliVe

**Outlier-Victim Pair** 기반 양자화.

- LLM 가중치/활성화의 이상치(outlier)가 양자화 정확도를 크게 저하시키는 문제 해결
- outlier를 인접 victim 채널과 쌍으로 묶어 공동 표현 → 정밀도 손실 최소화
- HW 친화적: 쌍별 처리로 특수 연산 없이 기존 MAC 활용 가능

## TurboQuant

- 고속 PTQ(Post-Training Quantization) 기법
- 보정 데이터셋 없이도 낮은 비트폭에서 정확도 유지
- PyTorch 구현 제공 → 실제 LLM에 적용 용이

## Microscaling (MX) vs Mixed-Precision

| 항목 | Microscaling (MX) | Mixed-Precision |
|---|---|---|
| 단위 | 미세 블록(e.g. 16개 원소)별 스케일 | 레이어/텐서 단위 |
| 오버헤드 | 스케일 팩터 저장 추가 필요 | 낮음 |
| 정확도 | 높음 (세밀한 표현) | 레이어 민감도에 따라 다름 |
| HW 지원 | OCP MX 표준(FP8 MX 등) | 기존 FP16/INT8 혼용 |

## OCEAN 기반 LLM 압축 (9-part 심층 분석)

- OCEAN(Orthogonal Compression with Efficient Approximation Network) 프레임워크
- 직교 분해 기반 가중치 압축 + 경량 복원 네트워크 결합
- 9개 파트에 걸친 심층 분석 → 이론·구현·실험 결과 포함
- 상세 정리: [[wiki/GenAI/OCEAN-Compression-Deep-Dive]]

## LLM 양자화 아키텍처 전반

- PTQ vs QAT 트레이드오프
- W4A8, W8A8, FP8 등 비트폭 조합별 HW 친화도
- KV-cache 양자화(KV8, KV4)의 정확도·메모리 영향
- Activation outlier 처리 전략 (SmoothQuant, AWQ 등)
