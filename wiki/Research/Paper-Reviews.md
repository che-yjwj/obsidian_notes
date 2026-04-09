---
title: Paper Reviews
type: topic
sources:
  - raw/Research/Paper-Review/STEM Structure and Scalability 33a6cc566b0b8134bba4f04b6ccf5786.md
  - raw/Research/Paper-Review/Dr Zero Concept Summary 33a6cc566b0b81dea270edb325090317.md
  - raw/Research/Paper-Review/Chaos Theory and Prediction/Chaos Theory and Prediction (Part 1 of 2) 33a6cc566b0b81b7aeabd969fc648765.md
  - raw/Research/Paper-Review/Chaos Theory and Prediction/Chaos Theory and Prediction (Part 2 of 2) 33a6cc566b0b8185b540df4617ca05a5.md
tags: [paper-review, STEM, transformer, chaos-theory, reasoning]
updated: 2026-04-09
---

# Paper Reviews

이 페이지는 개별 리뷰를 나열하는 문서가 아니라 `research tooling`과 `reasoning/theory` 두 축을 묶는 허브다. 생산성 도구 계열은 [[wiki/Research/Research-Tooling-Reviews]]로 분리했다.

## 축 1. Reasoning and Theory

이 축은 "지능을 어떻게 구성하고 해석할 것인가"에 가까운 리뷰를 묶는다.

### STEM: Scaling Transformers with Embedding Modules

**핵심 아이디어**: 기존 Transformer의 고정 차원 임베딩을 모듈화하여 확장성 확보.

- 모든 토큰을 동일한 고정 차원으로 처리하는 구조를 탈피
- 입력·중간 표현·출력 임베딩 자체를 모듈화
- **확장성 방향**: 모델 파라미터 수를 늘리지 않고 임베딩 모듈 교체·조합으로 성능 향상
- 효율적인 파라미터 활용 → 큰 모델과 유사한 성능을 작은 연산량으로 달성

### Dr Zero: search-centric reasoning

- 학습 데이터 없이도 문제 생성, 탐색, 자기검증, refinement를 반복하는 self-evolving agent 프레임워크
- 핵심 성능 향상 메커니즘은 gradient가 아니라 `Search-Verify-Refine` 루프
- 시스템 관점에서는 trace cache, rollback, branch-heavy execution 같은 제어 중심 구조와 잘 맞음
- Dense GEMM 중심 GPU보다 CPU+NPU 하이브리드나 search-centric accelerator 관점에서 해석 가치가 큼

### 카오스 이론과 예측

- 단일 실행 경로보다 분포와 ensemble이 더 본질적이라는 관점을 제공
- deterministic system도 long-horizon에서는 chaotic하게 diverge할 수 있다는 점을 강조
- quantization, compression, performance modeling에서 작은 오차가 큰 결과 차이를 낳는 이유를 해석하는 데 유용

## 축 2. Why This Matters

이 세 문서는 서로 다른 레벨에서 같은 질문을 다룬다.

| 문서 | 질문 | 시스템 관점 해석 |
|---|---|---|
| STEM | 모델 표현을 어떻게 더 효율적으로 확장할 것인가 | parameter-efficient scaling |
| Dr Zero | 학습 없이도 reasoning을 어떻게 끌어올릴 것인가 | search-centric execution |
| Chaos Theory | 예측 한계를 어떻게 다룰 것인가 | distribution-aware evaluation |

## 관련 문서

- [[wiki/Research/Research-Tooling-Reviews]]: PaperDebugger, PaperBanana 등 연구 생산성/자동화 도구 축
- [[wiki/Research/AI-Assisted-Research-Workflow]]: multi-agent workflow와 연구 자동화의 실행 구조
