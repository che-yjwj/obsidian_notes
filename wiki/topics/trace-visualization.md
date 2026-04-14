# Trace & Visualization

*last_compiled: 2026-04-14 | sources: 8*

---

## Summary [coverage: high -- 5 sources]

이 토픽은 RISCV_NPU_SoC_SIM에서 trace를 단순 로그가 아니라 **설계 검증 인터페이스**로 다루는 문서군을 묶는다. `trace_format_spec.md`는 simulator와 viewer가 공유할 단일 포맷을 정의하고, `visualization_requirements.md`는 Gantt timeline, bandwidth heatmap, engine utilization, quantization impact plot을 필수 뷰로 요구한다. `golden_trace_examples.md`와 tutorial 문서는 이 스펙을 실제 regression과 해석 절차로 연결한다.

핵심 메시지는 trace가 구현 후에 붙는 부가 산출물이 아니라는 점이다. trace schema, golden trace, performance validation, tutorial이 모두 이미 존재하므로, 향후 compiler와 simulator 변경은 trace를 기준으로 비교 가능해야 한다. 이런 이유로 trace는 `riscv-npu-soc-sim` 내부 섹션이면서도 별도 topic으로 다루는 편이 맞다.

## Core Concepts [coverage: high -- 5 sources]

- **Trace event schema**: `run_metadata`, `config_snapshot`, `timeline_events`, `bandwidth_samples`, `summary_metrics`를 기본 구조로 한다.
- **Visualization suite**: Gantt timeline, bandwidth heatmap, utilization dashboard, quantization impact plot이 최소 필수 뷰다.
- **Golden trace regression**: 대표 workload의 trace를 승인된 참조값으로 저장해 regression test에 사용한다.
- **Trace-first validation**: latency, bandwidth, token event, phase boundary를 trace 기준으로 검증한다.

## Architecture [coverage: medium -- 4 sources]

trace 문서군은 세 층으로 나뉜다. 첫째, `spec/trace/*`는 포맷과 뷰 요구사항을 정의한다. 둘째, `docs/test/golden_trace_examples.md`와 `performance_validation_protocol.md`는 그 포맷을 회귀/성능 검증 규칙으로 바꾼다. 셋째, `tutorial_minimal_llama_to_tile_npu.md`는 LLaMA attention을 tile-based NPU 관점에서 읽는 방법을 설명하며 trace 해석의 교육 자료 역할을 한다.

이 구조 덕분에 trace는 simulator output과 visualizer input 사이의 contract가 되고, 동시에 prefill/decode, KV-cache, mixed precision, memory contention을 설명하는 공통 분석 언어가 된다.

## Key Findings [coverage: high -- 5 sources]

- trace format은 JSON 기반 단일 구조를 기본으로 하며, quantization/timing metadata와 직접 연결된다.
- golden trace는 `GT-MLP-01`, `GT-CONV-01`, `GT-ATTN-01`, `GT-LLM-01`처럼 representative workload 중심으로 관리된다.
- performance validation은 latency accuracy뿐 아니라 DRAM/Bus bandwidth pattern realism까지 검증 대상으로 둔다.
- tutorial은 prefill은 compute-heavy, decode는 KV-cache DRAM traffic-heavy라는 병목 구분을 trace 해석 언어로 설명한다.

## Connections [coverage: medium -- 4 sources]

- [[riscv-npu-soc-sim]]: trace는 프로젝트의 실행 결과와 검증 루프를 묶는 핵심 출력이다.
- [[npu-simulator-compiler]]: compiler 산출물인 CMDQ와 simulator 결과물인 trace가 이 topic에서 만난다.
- [[../concepts/trace-first-design]]: trace를 설계 초기에 고정해야 한다는 process 원리와 이어진다.
- [[../concepts/kv-cache-dram-residency]]: decode bottleneck과 KV-cache 정책은 trace에서 가장 직접적으로 드러난다.

## Open Questions [coverage: medium -- 3 sources]

- trace topic에 `performance validation`과 `golden trace`를 계속 같이 둘지, test topic으로 분리할지 결정이 필요하다.
- 1M events 이상에서 level-of-detail 렌더링을 어떻게 구현할지 아직 구체 구현이 없다.
- CMDQ snapshot, IR metadata, trace visualization을 한 번에 비교하는 batch workflow가 아직 문서화 단계에 머물러 있다.

## Sources

- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/trace_format_spec]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/gantt_timeline_spec]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/bandwidth_heatmap_spec]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/tile_semantics_validation_checklist]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/spec/trace/visualization_requirements]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/test/golden_trace_examples]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/test/examples/tutorial_minimal_llama_to_tile_npu]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/test/performance_validation_protocol]]
