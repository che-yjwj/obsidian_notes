---
title: Paper Reviews
type: topic
sources:
  - raw/Research/Paper-Review/STEM Structure and Scalability 33a6cc566b0b8134bba4f04b6ccf5786.md
  - raw/Research/Paper-Review/PaperDebugger Research Productivity Innovation 33a6cc566b0b81438d17de8e443da8de.md
  - raw/Research/Paper-Review/Dr Zero Concept Summary 33a6cc566b0b81dea270edb325090317.md
  - raw/Research/Paper-Review/PaperBanana AI Research 33a6cc566b0b81f5b652f4c9128db47b.md
  - raw/Research/Paper-Review/Chaos Theory and Prediction/Chaos Theory and Prediction (Part 1 of 2) 33a6cc566b0b81b7aeabd969fc648765.md
  - raw/Research/Paper-Review/Chaos Theory and Prediction/Chaos Theory and Prediction (Part 2 of 2) 33a6cc566b0b8185b540df4617ca05a5.md
tags: [paper-review, STEM, transformer, chaos-theory, research-tools]
updated: 2026-04-07
---

# Paper Reviews

## STEM: Scaling Transformers with Embedding Modules

**핵심 아이디어**: 기존 Transformer의 고정 차원 임베딩을 모듈화하여 확장성 확보.

- 모든 토큰을 동일한 고정 차원으로 처리하는 구조를 탈피
- 입력·중간 표현·출력 임베딩 자체를 모듈화
- **확장성 방향**: 모델 파라미터 수를 늘리지 않고 임베딩 모듈 교체·조합으로 성능 향상
- 효율적인 파라미터 활용 → 큰 모델과 유사한 성능을 작은 연산량으로 달성

## PaperDebugger: 연구 생산성 혁신

- AI 도구를 활용하여 논문 읽기·분석·디버깅 워크플로를 자동화
- 논문 내 실험 재현, 오류 탐지, 코드-논문 일치 검증에 초점
- 연구자의 반복적 수작업(실험 설정 추적, 결과 정리)을 AI가 대체

## PaperBanana: AI 연구 도구

- AI 기반 논문 탐색·요약·연관 논문 발견 도구
- PaperDebugger와 유사한 생산성 향상 방향이지만 탐색(discovery) 중심
- 키워드 없이도 의미 기반 탐색 가능

## Dr Zero: 개념 정리

- Zero-shot 또는 Zero-cost 관련 추론/학습 기법 (상세 내용은 raw 소스 참고)
- → ingest 시 상세 합성 필요

## 카오스 이론과 예측

- 카오스 이론(Chaos Theory): 초기 조건 민감성, 장기 예측 불가능성
- LLM의 예측 능력과 카오스 시스템 사이의 관계 분석
- 시계열 예측, 동적 시스템 모델링에서 LLM의 한계와 가능성
- 예측 지평선(prediction horizon) 개념과 AI 모델 정확도의 연관성
