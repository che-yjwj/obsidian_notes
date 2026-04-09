---
title: SoC Specification Negotiation English
type: reference
sources:
  - raw/Misc/SoC Specification Negotiation English/SoC Specification Negotiation English (Part 1 of 2) 33a6cc566b0b8129952df13d553ab241.md
  - raw/Misc/SoC Specification Negotiation English/SoC Specification Negotiation English (Part 2 of 2) 33a6cc566b0b811a9636ed54f24fd862.md
tags: [English, SoC, negotiation, vocabulary]
updated: 2026-04-09
---

# SoC Specification Negotiation English

SoC 사양 협의 시 사용하는 영어 표현 모음. 고객사·파트너사와의 기술 협의, 스펙 리뷰, 설계 변경 요청 상황에 활용한다.

이 페이지는 단순 회화 메모가 아니라 [[wiki/AI-Hardware/NPU-Architecture]], [[wiki/AI-Hardware/Memory-Hierarchy-in-AI-Accelerators]], [[wiki/AI-Hardware/Simulator-and-Implementation-Tools]]를 읽은 뒤 고객 미팅과 spec review에서 바로 꺼내 쓰는 실무 reference를 목표로 한다.

## 주요 표현 분류

> 상세 표현은 raw 소스를 참고하세요.
> → [[raw/Misc/SoC Specification Negotiation English]]

### 스펙 제안·협의
- "We'd like to propose..." / "Our recommendation would be..."
- "Could you clarify the requirement for...?"
- "Is there any flexibility on the...?"

### 설계 변경 요청
- "We need to revisit the specification for..."
- "Due to [constraint], we suggest modifying..."
- "This would impact the [timeline/area/power]..."

### 우선순위 협의
- "Which is higher priority, [A] or [B]?"
- "We can accommodate [X] if we relax [Y]."

### 확인·승인
- "Please confirm the agreed spec by..."
- "This is subject to final sign-off from..."

## 협상에서 자주 부딪히는 기술 포인트

raw source의 핵심은 "영어를 유창하게 말하는 것"보다 가정, 병목, ownership, trade-off를 구조적으로 설명하는 것이다.

### 1. Architecture scope 정렬

[[wiki/AI-Hardware/NPU-Architecture]]를 설명할 때는 블록 소개보다 먼저 시스템 경계를 정리하는 표현이 중요하다.

- "At the SoC level, our focus is on the NPU, memory subsystem, and interconnect."
- "This falls under the SoC architecture scope, while compiler optimization is handled separately."
- "Before we go deeper, I'd like to align on the system boundary and responsibilities."

### 2. Memory bottleneck 협의

[[wiki/AI-Hardware/Memory-Hierarchy-in-AI-Accelerators]]와 연결되는 가장 실전적인 협상 포인트는 bandwidth, locality, coherence다.

- "From a system perspective, memory bandwidth is the primary bottleneck."
- "If latency is the higher priority, we need to reduce off-chip traffic rather than simply increase compute."
- "Coherency support makes sense only if the CPU and NPU are expected to share the same tensor frequently."
- "We can support the requested model size only if the working set fits within the target memory hierarchy."

### 3. Dataflow / implementation 책임 구분

[[wiki/AI-Hardware/Simulator-and-Implementation-Tools]]에서 보는 구현체 차이는 협상 시 ownership 분리에 바로 연결된다.

- "This is achievable at the architecture level, but the final efficiency depends on runtime scheduling."
- "The hardware can provide the buffer structure, but software needs to guarantee the access pattern."
- "Our baseline assumption is a streaming dataflow with limited off-chip round trips."
- "If that assumption changes, the area and power impact will need to be revisited."

## 자주 쓰는 협상 프레임

원문에서 반복되는 패턴은 아래 네 가지다.

| 프레임 | 목적 | 예문 |
|---|---|---|
| Clarify | 모호한 요구사항을 재정의 | "Let me rephrase the requirement to make sure we're aligned." |
| Assumption | 성능 수치의 전제 명시 | "Under the current assumptions, this target is achievable." |
| Trade-off | 불가능을 완곡하게 구조화 | "If we prioritize latency, we may have to relax the area target." |
| Ownership | 책임 범위를 분리 | "This is handled at the SoC level, while the compiler team covers the rest." |

## 문서 리뷰와 미팅에서 바로 쓰는 흐름

원문은 12주 학습 로드맵과 4주 실전 체크리스트까지 포함하지만, vault 관점에서는 아래 순서로 압축해 쓰는 편이 더 낫다.

1. [[wiki/AI-Hardware/NPU-Architecture]]로 블록 구성과 시스템 경계를 설명한다.
2. [[wiki/AI-Hardware/Memory-Hierarchy-in-AI-Accelerators]]로 병목이 compute인지 memory인지 먼저 규정한다.
3. [[wiki/AI-Hardware/Simulator-and-Implementation-Tools]]를 근거로 구현 책임이 hardware, compiler, runtime 중 어디에 있는지 나눈다.
4. 마지막으로 requirement, assumption, open issue, action item을 영어 문장으로 고정한다.

## 바로 가져다 쓸 수 있는 문장 묶음

### 요구사항 재정의

- "So what you're asking is essentially a latency-focused NPU configuration under a fixed power budget."
- "If I understand correctly, the requirement is not peak TOPS itself, but sustained real-time response."

### 제약 설명

- "Technically possible, but it would have a significant impact on memory bandwidth and area."
- "That target is achievable only if we narrow the scope of coherency and reduce external memory traffic."

### 대안 제시

- "One option is to keep the current compute array and increase SRAM for better locality."
- "An alternative approach would be to relax the model-size target and keep the LPDDR-based design."

### 합의 고정

- "Let us freeze the memory assumption first, then we can finalize the interconnect requirement."
- "We'll summarize today's decision as assumptions, open issues, and action items."
