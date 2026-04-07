---
title: Patent Research — MCM and AI Infrastructure
type: topic
sources:
  - raw/Research/Patent-MCM/Taalas AI Infrastructure Innovation 33a6cc566b0b81378bc9f4ae1054ac16.md
  - raw/Research/Patent-MCM/AgentHub and Collaboration Model Change 33a6cc566b0b815ab9faec6381cdfa23.md
  - raw/Research/Patent-MCM/Reasoning in 13M Parameters 33a6cc566b0b816480b1d91f9b8363c9.md
  - raw/Research/Patent-MCM/AI-assisted research workflow (Claude Code + Harne 33a6cc566b0b81ea9827ddba8abd5822.md
  - raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 1 of 3) 33a6cc566b0b81028b7bdfa2171ef078.md
  - raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 2 of 3) 33a6cc566b0b81e1a9a9f08c25a9f5d0.md
  - raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 3 of 3) 33a6cc566b0b81c9a32eca2323221258.md
tags: [MCM, patent, multiplierless, DWT, AI-infrastructure, AgentHub]
updated: 2026-04-07
---

# Patent Research — MCM and AI Infrastructure

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

## Taalas AI 인프라 혁신

- AI 서비스 인프라를 혁신하는 Taalas 플랫폼 사례 분석
- 대규모 AI 워크로드를 위한 효율적 인프라 설계 방향

## AgentHub와 협업 모델 변화

- AI 에이전트 허브(AgentHub)가 조직 내 협업 방식에 미치는 영향 분석
- 멀티 에이전트 시스템에서 역할 분담, 태스크 라우팅, 권한 관리

## Reasoning in 13M Parameters

- 13M 파라미터라는 소형 모델에서 reasoning 능력을 최대화하는 방법 탐구
- 효율적인 파라미터 활용, 지식 증류, 특화 학습 방향

## AI-assisted Research Workflow (Claude Code + Harness)

- Claude Code + 커스텀 하네스를 활용한 AI 보조 연구 워크플로우
- 논문 분석, 코드 생성, 실험 추적을 AI가 지원하는 구체적 파이프라인
- 이 vault 자체가 해당 워크플로우의 실험 사례
