---
title: Research Tooling Reviews
type: topic
sources:
  - raw/Research/Paper-Review/PaperDebugger Research Productivity Innovation 33a6cc566b0b81438d17de8e443da8de.md
  - raw/Research/Paper-Review/PaperBanana AI Research 33a6cc566b0b81f5b652f4c9128db47b.md
tags: [paper-review, research-tools, automation, PaperDebugger, PaperBanana]
updated: 2026-04-09
---

# Research Tooling Reviews

논문을 더 빨리 읽고, 재현하고, 시각화하고, 연관 자료를 찾는 도구 계열 리뷰를 묶는다. 이 페이지는 연구 생산성 도구를 `debugging`과 `discovery/visualization` 축으로 정리한다.

## PaperDebugger: 실행 가능한 논문

PaperDebugger의 핵심은 논문을 정적 PDF가 아니라 `가설 → 코드 → 실험 → 결과`로 이어지는 디버깅 가능한 시스템으로 재정의하는 것이다.

- 논문과 코드의 mismatch를 추적하는 alignment debugging 관점이 강하다
- 재현 실패를 실패가 아니라 숨은 가정과 구현 의존성을 드러내는 정보로 해석한다
- 연구 워크플로를 spec-to-implementation 검증 과정으로 바꾸는 점이 중요하다

## PaperBanana: 시각화와 발견의 자동화

PaperBanana는 논문용 이미지 생성과 시각 구성 자동화에 초점을 둔 다중 에이전트 프레임워크다.

- planning, retrieval, styling, visualization, critique를 역할별 agent로 분리한다
- 이미지 생성과 코드 기반 시각화를 분리해 환각을 줄이려는 전략이 특징이다
- 연구 산출물의 표현층을 자동화한다는 점에서 PaperDebugger와 보완적이다

## 두 도구를 함께 읽는 기준

| 도구 | 주된 관심사 | 시스템 관점 해석 |
|---|---|---|
| PaperDebugger | 재현성과 차이 원인 추적 | 연구 artifact debugging |
| PaperBanana | 시각 표현과 발견 자동화 | multi-agent content generation |

## 왜 별도 축으로 보는가

이 둘은 reasoning 이론보다 연구자의 생산성 루프를 직접 겨냥한다.

- PaperDebugger는 실험과 구현의 정합성 검증 도구에 가깝다
- PaperBanana는 결과 표현과 논문 생산 보조 도구에 가깝다
- 둘 다 [[wiki/Research/AI-Assisted-Research-Workflow]]와 연결되지만, 초점은 cognitive theory가 아니라 workflow acceleration이다

## 관련 문서

- [[wiki/Research/Paper-Reviews]]: STEM, Dr Zero, Chaos Theory 등 reasoning/theory 축
- [[wiki/Research/AI-Assisted-Research-Workflow]]: 연구 자동화 파이프라인 전체 관점
