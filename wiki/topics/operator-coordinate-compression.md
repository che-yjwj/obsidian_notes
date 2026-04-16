---
title: Operator-Coordinate Compression
type: topic
status: canonical
last_compiled: 2026-04-14
---

# Operator-Coordinate Compression

*last_compiled: 2026-04-14 | sources: 17*

---

## Summary [coverage: high -- 6 sources]

`operator-coordinate-compression`은 LLM 압축을 값(value) 중심 문제가 아니라 **연산자(operator) 파라미터를 어떤 좌표계로 표현할 것인가**의 문제로 다시 세우는 연구 축이다. 이 자료군은 기존 outlier 보호, clipping, mixed-precision heuristic을 직접 부정하기보다, 왜 직교 회전이나 공격적인 저비트 양자화가 종종 잘 동작하는지를 더 깊은 기하학 언어로 설명하려고 한다.

핵심 주장은 세 가지다. 첫째, 학습된 가중치는 독립 스칼라 집합이 아니라 비선형 연산자를 매개변수화한다. 둘째, outlier는 본질 신호가 아니라 좌표계-상대적 인공물일 수 있다. 셋째, random rotation이 분포를 평탄화할 수는 있어도, 진짜 압축 이득은 manifold-aligned coordinate가 만드는 coefficient concentration에서 나온다. 이 프레임은 이론 노트, 실험 스펙, 논문 초안, 특허 초안으로 이미 확장되어 있어 독립 topic으로 유지하는 편이 맞다.

## Core Concepts [coverage: high -- 6 sources]

- **Operator view**: 가중치 텐서는 데이터 값이 아니라 연산자 계수다. 따라서 압축은 삭제보다 재매개변수화에 가깝다.
- **Coordinate-relative outliers**: 큰 값은 선택된 축과 접공간이 맞지 않아서 생기는 projection artifact일 수 있다.
- **Flattening vs concentration**: Hadamard/rotation은 outlier를 줄이고 양자화 강인성을 높일 수 있지만, coefficient concentration을 보장하지는 않는다.
- **Geometry-aware rate-distortion**: 좋은 압축은 단순 값 분포가 아니라 연산자 왜곡과 코드 길이를 함께 최적화해야 한다.

## Architecture [coverage: medium -- 4 sources]

문서 구조는 `README/overview`가 개념 프레이밍을 담당하고, `docs/theory/*`가 operator view, manifold alignment, outlier 재해석, rotation 해석을 세분화한다. 그 위에 `docs/paper/*`는 외부 커뮤니케이션용 서사를 만들고, `docs/patent/*`는 신규성과 청구항 후보를 정리한다. `experiments/*`는 구현보다 앞서 “무엇을 검증할지”를 고정하는 명세 레이어다.

실험 프로토콜은 `toy_basis_vs_mlp`와 `tinyllama_validation` 두 축으로 나뉜다. 전자는 basis choice와 concentration 직관을 확인하는 통제 실험이고, 후자는 실제 TinyLLaMA/SLM 체크포인트에서 outlier 지표, top-k energy ratio, entropy proxy, reconstruction error를 비교하는 실험 설계다.

## Key Findings [coverage: high -- 5 sources]

- 현재 프레임워크의 strongest claim은 “outlier는 coordinate artifact”라는 재프레이밍이다.
- TurboQuant는 최종 해답이라기보다 random-rotation baseline 계열로 위치 지어진다.
- 다음 연구 분기점은 weight manifold만이 아니라 **joint weight-activation manifold**까지 포함해 설명력을 넓히는 것이다.
- patent 방향에서는 `rotation + alignment + functional/residual decomposition + entropy coding` 조합이 핵심 신규성 축으로 정리돼 있다.

## Connections [coverage: medium -- 3 sources]

- [[llm-quantization-compression]]: quantization 기법 모음에서 설명하지 못하는 기하학적 설명 층을 제공한다.
- [[patent-mcm]]: parameter structure를 하드웨어 친화적으로 다시 표현한다는 점에서 연결된다.
- [[../concepts/mixed-precision-policy]]: 실제 시스템 정책으로 연결되려면 bitwidth/memory/runtime 모델과 만나야 한다.

## Open Questions [coverage: medium -- 4 sources]

- manifold hypothesis를 empirical하게 어떻게 측정할 것인가?
- SpinQuant의 learned rotation을 partial alignment로 해석할 수 있는가?
- activation-space error를 설명하는 joint formulation을 얼마나 빨리 추가할 수 있는가?
- concentration 개선이 실제 entropy coding gain으로 이어지는지 end-to-end evidence를 어떻게 만들 것인가?

## Sources

- [[../../raw/Research/operator-coordinate-compression/README]]
- [[../../raw/Research/operator-coordinate-compression/roadmap]]
- [[../../raw/Research/operator-coordinate-compression/docs/index]]
- [[../../raw/Research/operator-coordinate-compression/docs/overview]]
- [[../../raw/Research/operator-coordinate-compression/docs/research_direction_review]]
- [[../../raw/Research/operator-coordinate-compression/docs/theory/operator_view]]
- [[../../raw/Research/operator-coordinate-compression/docs/theory/coordinate_relative_outliers]]
- [[../../raw/Research/operator-coordinate-compression/docs/theory/rotation_vs_alignment]]
- [[../../raw/Research/operator-coordinate-compression/docs/theory/manifold_alignment]]
- [[../../raw/Research/operator-coordinate-compression/docs/theory/turboquant_llm_quantization_architecture]]
- [[../../raw/Research/operator-coordinate-compression/docs/paper/paper_draft]]
- [[../../raw/Research/operator-coordinate-compression/docs/paper/related_work]]
- [[../../raw/Research/operator-coordinate-compression/docs/patent/invention_summary]]
- [[../../raw/Research/operator-coordinate-compression/docs/patent/prior_art_mapping]]
- [[../../raw/Research/operator-coordinate-compression/docs/patent/claim_candidates]]
- [[../../raw/Research/operator-coordinate-compression/experiments/toy_basis_vs_mlp/spec]]
- [[../../raw/Research/operator-coordinate-compression/experiments/tinyllama_validation/spec]]
