---
title: Trace & Visualization
type: topic
status: canonical
last_compiled: 2026-04-16
---

# Trace & Visualization

*last_compiled: 2026-04-16 | sources: trace-spec centered*

---

## Summary [coverage: trace-spec layer]

이 토픽은 RISCV_NPU_SoC_SIM에서 trace를 단순 로그가 아니라 **관찰 가능성(observability) 인터페이스**로 다루는 문서군을 묶는다. `trace_format_spec.md`는 simulator와 viewer가 공유할 단일 포맷을 정의하고, `visualization_requirements.md`는 Gantt timeline, bandwidth heatmap, engine utilization, quantization impact plot을 필수 뷰로 요구한다.

핵심 메시지는 trace가 구현 후에 붙는 부가 산출물이 아니라는 점이다. 하지만 이 topic의 책임은 **trace를 어떻게 기록하고 보여줄 것인가**까지다. trace를 golden artifact로 승인하거나 regression-safe한 acceptance 기준으로 쓰는 문제는 [[topics/simulation-validation]]로 넘긴다.

## Core Concepts [coverage: trace format + visualization]

- **Trace event schema**: `run_metadata`, `config_snapshot`, `timeline_events`, `bandwidth_samples`, `summary_metrics`를 기본 구조로 한다.
- **Visualization suite**: Gantt timeline, bandwidth heatmap, utilization dashboard, quantization impact plot이 최소 필수 뷰다.
- **Trace as contract**: simulator output과 viewer input 사이의 단일 interchange surface다.
- **Trace-first interpretation**: prefill/decode, bandwidth saturation, engine stall을 같은 분석 언어로 읽게 해준다.

## Scope Boundary [coverage: explicit ownership]

이 topic이 소유하는 것:

- `spec/trace/*`의 schema와 visualization requirement
- timeline, heatmap, utilization 같은 시각화 surface
- trace를 사람이 읽을 수 있는 공통 분석 언어로 만드는 일

이 topic이 직접 소유하지 않는 것:

- golden trace 승인 절차
- unit/integration/performance validation protocol
- 허용 오차와 regression gate

이 항목들은 [[topics/simulation-validation]]의 책임이다.

## Key Findings [coverage: trace-spec focused]

- trace format은 JSON 기반 단일 구조를 기본으로 하며, quantization/timing metadata와 직접 연결된다.
- Gantt timeline, bandwidth heatmap, utilization dashboard는 서로 다른 뷰가 아니라 같은 trace를 다른 투영으로 읽는 방식이다.
- trace schema는 quantization/timing metadata와 연결돼 있어서 단순 logging format이 아니라 profiling contract 역할을 한다.
- tutorial 성격의 예제는 trace를 읽는 법을 설명하는 교육 자료로 유효하지만, validation ground truth와는 분리해서 관리하는 편이 더 명확하다.

## Connections [coverage: medium -- 4 sources]

- [[riscv-npu-soc-sim]]: trace는 프로젝트의 실행 결과와 검증 루프를 묶는 핵심 출력이다.
- [[npu-simulator-compiler]]: compiler 산출물인 CMDQ와 simulator 결과물인 trace가 이 topic에서 만난다.
- [[simulation-validation]]: trace가 검증 artifact로 승격되는 지점은 validation topic이 담당한다.
- [[../concepts/trace-first-design]]: trace를 설계 초기에 고정해야 한다는 process 원리와 이어진다.
- [[../concepts/kv-cache-dram-residency]]: decode bottleneck과 KV-cache 정책은 trace에서 가장 직접적으로 드러난다.

## Open Questions [coverage: medium -- 3 sources]

- 1M events 이상에서 level-of-detail 렌더링을 어떻게 구현할지 아직 구체 구현이 없다.
- CMDQ snapshot, IR metadata, trace visualization을 한 번에 비교하는 batch workflow가 아직 문서화 단계에 머물러 있다.
- tutorial/example trace를 visualization topic에 남길지, validation이나 onboarding 쪽으로 다시 재배치할지 기준이 더 필요하다.

## Sources

- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/trace_format_spec]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/gantt_timeline_spec]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/bandwidth_heatmap_spec]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/tile_semantics_validation_checklist]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/visualization_requirements]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/test/examples/tutorial_minimal_llama_to_tile_npu]]
