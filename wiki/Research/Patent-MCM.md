---
title: Patent Research — MCM and AI Infrastructure
type: topic
sources:
  - raw/Research/Patent-MCM/Taalas AI Infrastructure Innovation 33a6cc566b0b81378bc9f4ae1054ac16.md
  - raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 1 of 3) 33a6cc566b0b81028b7bdfa2171ef078.md
  - raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 2 of 3) 33a6cc566b0b81e1a9a9f08c25a9f5d0.md
  - raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 3 of 3) 33a6cc566b0b81c9a32eca2323221258.md
tags: [MCM, patent, multiplierless, DWT, AI-infrastructure, model-centric]
updated: 2026-04-09
---

# Patent Research — MCM and AI Infrastructure

이 페이지는 `Multiplierless/MCM`과 `model-centric AI infrastructure`의 접점을 정리하는 상위 문서다. AgentHub, AI-assisted workflow, tiny reasoning 관련 내용은 [[wiki/Research/AI-Assisted-Research-Workflow]]로 분리했다.

## Multiplierless 설계: 패턴 탐색 기법 기반 리프팅 DWT

**논문**: "패턴 탐색 기법을 사용한 Multiplierless 리프팅 기반의 웨이블릿 변환의 설계" (한국멀티미디어학회지, 2010)

### 연구 목적
리프팅 기반 9/7 DWT(이산 웨이블릿 변환) 필터의 VLSI 구현에서 곱셈 연산을 제거하여 면적·전력 절감.

### 핵심 기술
- **Multiplierless Constant Multiplication (MCM)**: 상수 곱셈을 shift+add 연산으로 대체
- **Lefevre 알고리즘**: 패턴 탐색으로 최적 shift+add 조합을 탐색
- 결과: 곱셈기 제거 → 로직 면적 감소, 연산 지연 감소
- JPEG2000 표준 9/7 필터에 적용

### 특허/MCM 연관성
- MCM(Multiple Constant Multiplication) 기법은 NPU 가속기 내 고정 계수 연산 최적화에도 직접 적용 가능
- 웨이블릿 → CNN 가중치 연산으로 개념 확장 시 하드웨어 면적 절감 특허 아이디어 도출 가능

## Taalas AI 인프라 혁신: 모델 중심 특수화

- AI 서비스 인프라를 혁신하는 Taalas 플랫폼 사례 분석
- 대규모 AI 워크로드를 위한 효율적 인프라 설계 방향
- 범용 GPU가 아닌 모델 특화 실리콘으로 inference 비용 구조를 다시 짜려는 시도
- 메모리 병목, 비용, 지연을 줄이기 위해 memory-compute 통합과 구조 단순화를 전면에 둠

## MCM과 Taalas를 함께 읽는 이유

두 주제는 스케일은 다르지만 공통적으로 "고정된 모델/가중치를 어떻게 더 싸고 단순한 하드웨어로 바꿀 것인가"를 다룬다.

- **MCM 관점**: inference에서는 weight가 상수이므로 constant multiplication을 shift+add로 치환해 면적과 전력을 줄일 수 있다.
- **Taalas 관점**: 모델 그래프와 가중치를 실리콘에 더 직접적으로 매핑해 runtime programmability와 외부 메모리 의존을 줄인다.
- **공통 설계 질문**: 어떤 수준까지 가중치와 데이터플로를 고정할 수 있는가, 그 대가로 유연성을 얼마나 포기할 것인가.

## 특허 관점의 정리

이 묶음에서 재사용 가치가 높은 아이디어는 아래 세 가지다.

| 축 | 핵심 질문 | 특허/연구 포인트 |
|---|---|---|
| Constant multiplication | 고정 계수 연산을 얼마나 multiplierless로 치환할 수 있는가 | shift+add 그래프 최적화, 공유 부분식 재사용 |
| Model-to-silicon mapping | 모델 구조를 어느 수준까지 하드와이어할 것인가 | IR→RTL 자동화, layer-specific dataflow |
| Memory-compute integration | 외부 메모리 왕복을 얼마나 줄일 수 있는가 | 온칩 저장-연산 통합, 고정 가중치 저장 구조 |

## 관련 문서

- [[wiki/Research/AI-Assisted-Research-Workflow]]: AgentHub, multi-agent research workflow, tiny reasoning 실험 관점
- [[wiki/AI-Hardware/Memory-Hierarchy-in-AI-Accelerators]]: memory-compute 통합과 병목 해석 관점
