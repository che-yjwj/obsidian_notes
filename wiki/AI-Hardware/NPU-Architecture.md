---
title: NPU Architecture Overview
type: topic
sources:
  - raw/AI-Hardware/Architecture/Tesla AI Chip Roadmap 33a6cc566b0b8129a0f3eef1ddc4dfa3.md
  - raw/AI-Hardware/Architecture/Tesla Edge AI Innovation 33a6cc566b0b81c8aa60dc6f39c11a1c.md
  - raw/AI-Hardware/Architecture/AMD Versal ACAP Overview 33a6cc566b0b81d6b8d8ca55e7b2ba1d.md
  - raw/AI-Hardware/Architecture/Xilinx FINN Overview 33a6cc566b0b81068908d40fc473eaef.md
  - raw/AI-Hardware/Architecture/NPU Performance-based AMBA Recommendation 33a6cc566b0b81d1a6f8fd7a1f0b2ffe.md
  - raw/AI-Hardware/Architecture/TPU Sparse Core Explanation 33a6cc566b0b81f7a2a5f1e85f847ccb.md
  - raw/AI-Hardware/Architecture/Synopsys Platform Architect Introduction 33a6cc566b0b81c78ed7c21c89862ab6.md
tags: [NPU, TPU, Tesla, AMD, Xilinx, AMBA, architecture]
updated: 2026-04-07
---

# NPU Architecture Overview

## Tesla AI 칩 로드맵

Tesla는 FSD(Full Self-Driving) 전용 AI 칩을 자체 설계하여 NVIDIA GPU 의존도를 낮추는 방향으로 발전해왔다.
- **FSD HW3** → HW4 → Dojo 훈련 칩으로 이어지는 세대별 성능 향상
- 엣지 AI(차량 내 실시간 추론)에 최적화된 저전력 설계 강조

## AMD Versal ACAP

Versal은 FPGA + AI Engine + ARM CPU를 하나의 칩에 통합한 Adaptive Compute Acceleration Platform이다.
- AI Engine 배열: VLIW + SIMD 기반 벡터 프로세서 타일
- 프로그래머블 로직(PL)과 AI Engine을 함께 활용하는 이기종 설계
- 고대역폭 NoC(Network-on-Chip) 내장

## Xilinx FINN

FINN은 Xilinx(AMD)의 FPGA 기반 신경망 추론 프레임워크다.
- 양자화(BNN/QNN)된 모델을 FPGA 스트리밍 파이프라인으로 변환
- 초저지연, 초저전력 엣지 추론에 강점
- HLS 코드 생성 → Vivado 합성까지 자동화

## Google TPU Sparse Core

- Sparse Core는 임베딩 테이블 연산(추천 시스템, NLP 임베딩)에 특화된 TPU 서브코어
- Dense Core(행렬 연산)와 분리하여 sparse workload를 병렬 처리
- 메모리 접근 패턴이 불규칙한 sparse 연산의 효율을 크게 향상

## NPU 성능별 AMBA 버스 인프라 권장

| 티어 | 성능 범위 | Control-plane | Data-plane | 비고 |
|---|---|---|---|---|
| A (소형 엣지) | 1–10 TOPS | APB / AHB | AXI4 (단일) | 저면적, QoS 최소 |
| B (모바일 중급) | 10–50 TOPS | AHB | AXI4 멀티레벨 | L0 내부 크로스바 + L1 SoC 버스 |
| C (고성능 서버/차량) | 50 TOPS↑ | AHB | CHI / AXI5 + NoC | 일관성 도메인, 가상화(SMMU) |

**핵심 원칙:**
- Control-plane과 Data-plane은 반드시 분리
- NPU 내부는 전용 로컬 크로스바로 짧게 처리
- CPU-NPU 공유 텐서가 없으면 coherent 버스 불필요

## Synopsys Platform Architect

- SoC 초기 아키텍처 탐색을 위한 시스템 수준 시뮬레이터
- 버스 구성, 메모리 레이아웃, 전력 분석을 빠르게 평가
- RTL 이전 단계에서 병목 사전 탐지
