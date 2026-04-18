---
title: AI-Assisted Research Workflow
type: deep-dive
sources:
  - raw/Research/Patent-MCM/AgentHub and Collaboration Model Change 33a6cc566b0b815ab9faec6381cdfa23.md
  - raw/Research/Patent-MCM/AI-assisted research workflow (Claude Code + Harne 33a6cc566b0b81ea9827ddba8abd5822.md
  - raw/Research/Patent-MCM/Reasoning in 13M Parameters 33a6cc566b0b816480b1d91f9b8363c9.md
tags: [research-workflow, AgentHub, automation, reasoning, Claude-Code]
updated: 2026-04-18
---

# AI-Assisted Research Workflow

Canonical topics: [[topics/paper-reviews]], [[topics/patent-mcm]]

## Role in This Wiki

- This page is a workflow deep dive about how research gets executed with agents, tools, and low-cost reasoning units.
- It is not the canonical home for reviewed papers or tooling summaries; those belong under [[topics/paper-reviews]] and its deep-dive companions.
- Its value is in connecting collaboration substrate, execution workflow, and reasoning unit economics into one operating model.

## Boundary

This page should focus on:
- agent collaboration structure
- research execution pipeline and verification ownership
- the cost/throughput logic behind scaling many small reasoning units

This page should not become the main home for:
- paper-by-paper review summaries
- research-tooling feature comparisons as standalone reviews
- canonical hardware interpretation of reviewed papers

Those belong in [[topics/paper-reviews]], [[wiki/Research/Paper-Reviews]], and [[wiki/Research/Research-Tooling-Reviews]].

AI를 보조 도구가 아니라 연구 파이프라인의 실행 주체로 배치하는 흐름을 정리한다. 이 페이지는 AgentHub식 협업 구조, Claude Code 기반 연구 자동화, tiny reasoning 실험을 한 축으로 묶는다.

## AgentHub: 저장소가 아니라 탐색 그래프

AgentHub의 핵심 아이디어는 사람 중심의 `branch → review → merge` 절차를 줄이고, 에이전트가 병렬로 가설과 결과를 남기는 DAG형 탐색 구조를 채택하는 것이다.

- merge와 linear history보다 exploration throughput을 우선한다
- commit은 최종본보다 실험 흔적과 가설 검증 단위에 가깝다
- 인간 역할은 코드 작성자보다 instruction designer, system architect 쪽으로 이동한다

## Claude Code + Harness: 연구 산출물 생성 파이프라인

raw 사례의 포인트는 적은 수의 프롬프트로 대시보드와 논문 초안까지 빠르게 생성했다는 점이다.

- literature scan, 정리, 코드 수정, 결과 해석, 문서 초안 작성이 하나의 흐름으로 묶인다
- 핵심 병목은 작성 속도보다 검증과 가정 관리로 이동한다
- 따라서 좋은 workflow는 "무엇을 자동화할지"보다 "어떤 가정을 고정하고 무엇을 사람이 검토할지"를 먼저 정해야 한다

## Tiny reasoning 연구가 왜 같이 묶이는가

`Reasoning in 13M Parameters`는 주제가 다소 달라 보이지만, 연구 자동화 관점에서는 비용이 낮은 reasoning unit을 대량 배치할 수 있는지와 연결된다.

- reasoning 성능이 거대 모델만의 전유물이 아니라면 agent swarm의 단가가 크게 내려간다
- 작은 reasoning 모듈은 대규모 병렬 탐색, 후보 필터링, 자동 평가에 더 잘 맞는다
- 즉, tiny reasoning은 workflow 자동화의 알고리즘 측 기반이라고 볼 수 있다

## 실무적으로 보는 3단 분해

| 층위 | 질문 | 대표 source |
|---|---|---|
| Collaboration substrate | 에이전트가 어떻게 병렬로 일할 것인가 | AgentHub |
| Execution workflow | 실제 논문/대시보드/실험을 어떻게 돌릴 것인가 | Claude Code + Harness |
| Cognitive unit economics | reasoning을 얼마나 싸게 많이 배치할 수 있는가 | Reasoning in 13M Parameters |

## 이 vault에서의 의미

이 vault 자체가 raw source 보존, wiki 합성, graphify 기반 연결 추적을 통해 작은 연구 자동화 실험장 역할을 한다.

- raw는 실험 로그와 외부 입력 저장소 역할을 한다
- wiki는 사람이 다시 사용할 수 있는 정제된 지식층이다
- graphify는 문서 간 연결과 고립 노드를 드러내는 메타 레이어다

## 관련 문서

- [[wiki/Research/Patent-MCM]]: 하드웨어 특수화와 모델-실리콘 매핑 관점
- [[wiki/Misc/SoC-Specification-Negotiation-English]]: 시스템 가정과 ownership를 구조적으로 설명하는 표현 관점
