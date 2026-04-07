---
title: HW-Friendly GenAI Model Design
type: topic
sources:
  - raw/GenAI/HW-Friendly/KV-cache Optimization Explanation 33a6cc566b0b811890a0df4bc615b114.md
  - raw/GenAI/HW-Friendly/Latent Action World Model 33a6cc566b0b816ba286eaff4c35c587.md
  - raw/GenAI/HW-Friendly/Stronger Normalization-Free Transformers 33a6cc566b0b81648fc4ec9576dc68a0.md
  - raw/GenAI/HW-Friendly/MSA and RAG Analysis 33a6cc566b0b81b6b40ec9ead1726cf4.md
tags: [KV-cache, normalization-free, world-model, RAG, MSA, Gemma]
updated: 2026-04-07
---

# HW-Friendly GenAI Model Design

## On-Device AI: Gemma 3n과 KV-cache 최적화

Gemma 3n은 Google의 모바일 우선 멀티모달 모델로, 온디바이스 제약 환경에서 KV-cache 및 파라미터 로딩 최적화를 잘 보여준다.

### MatFormer (Matryoshka Transformer)
- 중첩 구조: 요청마다 활성화할 파라미터 서브셋을 동적으로 선택
- 하나의 모델 안에 여러 크기의 서브모델 내장 → 엣지 기기 메모리에 맞게 조절

### PLE (Per-Layer Embedding) 캐싱
- 레이어별 임베딩을 캐싱하여 런타임 메모리 대폭 절감
- KV-cache와 연계하여 연속 토큰 생성 시 중복 계산 제거

### 조건부 파라미터 로드
- 텍스트만 처리 시 시각/오디오 파라미터 미로드 → 메모리·전력 절감

## Normalization-Free Transformers

- LayerNorm/BatchNorm 제거로 연산 단순화
- 대규모 배치나 긴 시퀀스에서 정규화 레이어가 병목이 될 수 있음
- Adaptive Gradient Clipping(AGC) 등으로 안정성 보완
- NPU에서 Norm 연산 처리 비용을 줄여 HW 활용도 향상

## Latent Action World Model

- 에이전트가 환경 모델(World Model)을 내재화하여 계획 수립
- Latent space에서 행동을 추론 → 고해상도 시뮬레이션 없이 추론 가능
- 온디바이스 로봇/자율주행 등 실시간 추론 요구 환경에 적합
- 메모리·연산 효율 관점: latent 표현이 raw 센서 데이터보다 압축됨

## MSA(Multi-head Self-Attention)와 RAG 연계

- MSA: Transformer의 핵심 연산 — 시퀀스 길이 O(n²) 복잡도가 HW 설계 제약
- RAG(Retrieval-Augmented Generation): 검색으로 컨텍스트 길이 부담을 줄이는 접근
- MSA + RAG 조합 분석: 긴 문서 처리 시 검색으로 입력 토큰을 줄이고 attention 연산 절감
- HW 설계 관점: KV-cache 크기와 attention window 크기가 on-chip SRAM 설계에 직접 영향
