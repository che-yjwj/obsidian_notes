---
title: NPU Documentation Process
type: topic
status: canonical
last_compiled: 2026-04-14
---

# NPU Documentation Process

*last_compiled: 2026-04-14 | sources: 12*

---

## Summary [coverage: high -- 6 sources]

이 토픽은 RISCV_NPU_SoC_SIM 프로젝트에서 문서가 구현을 선행하고 통제하는 방식을 다룬다. `spec_driven_development_workflow.md`는 “spec → design → test → implementation → review” 순서를 강제하고, `project_roadmap.md`와 `milestone_plan.md`는 이 흐름을 Stage/Phase 단위 구현 계획으로 번역한다. `versioning_and_changelog_guide.md`, `naming_convention.md`, `contribution_and_review_guide.md`, `vibe_coding_sdd_guide.md`는 이 프로세스를 실제 협업과 AI-assisted coding 방식까지 확장한다.

즉, 이 topic은 기술 스펙 자체가 아니라 **문서 아키텍처와 작업 순서**를 정의한다. graphify에서도 process 문서군이 architecture/spec와 별도 커뮤니티를 이루고 있으므로, 이제는 `riscv-npu-soc-sim`의 부록이 아니라 독립 topic으로 유지하는 편이 맞다.

## Core Concepts [coverage: high -- 6 sources]

- **Document first**: 코드 전에 spec, design, test 문서를 갱신한다.
- **Single source of truth**: IR, CMDQ, timing, trace의 의미는 대응 스펙 문서가 주도한다.
- **Roadmap staging**: Foundation → Timing/Engine Modeling → Full Operator Support → LLM Specialization → DSE Platform 순으로 확장한다.
- **Traceability**: naming, changelog, review guide를 통해 문서와 코드 사이 연결을 남긴다.

## Architecture [coverage: medium -- 5 sources]

process 문서 계층은 세 부분으로 읽을 수 있다. 첫째, `spec_driven_development_workflow.md`와 `vibe_coding_sdd_guide.md`가 작업 루프와 AI coding usage rule을 정의한다. 둘째, `project_roadmap.md`, `milestone_plan.md`, `doc_study_milestone_plan.md`가 단계별 구현 계획을 잡는다. 셋째, `documentation_review_summary.md`, `doc_improvement_tasks.md`, `versioning_and_changelog_guide.md`, `contribution_and_review_guide.md`, `naming_convention.md`가 품질 관리와 유지보수 규칙을 담당한다.

이 구조는 기술 문서의 품질 자체를 관리하는 문서적 control plane이라고 볼 수 있다. 그래서 architecture topic과는 다르게 “어떤 순서로 문서를 만들고 검증하느냐”가 핵심이 된다.

## Key Findings [coverage: high -- 6 sources]

- 프로젝트는 이미 문서 중심 코드베이스를 전제로 하며, 실제 구현은 milestone plan에 따라 추후 채워지는 구조다.
- `Vertical-first`, `MatMul-only E2E`, `Trace-first design`, `Test-first for core logic`이 process 문서 전반에서 반복된다.
- Codex 사용도 자유형 구현이 아니라 Phase/Task/Spec/Design context를 상단에 고정하는 방식으로 제한한다.
- versioning 정책은 spec이 코드를 주도하는 `spec-first versioning`을 명시한다.

## Connections [coverage: medium -- 4 sources]

- [[riscv-npu-soc-sim]]: 실제 프로젝트 내용과 연결되는 process control plane이다.
- [[trace-visualization]]: trace-first design과 golden trace 운영 규칙이 process와 직접 연결된다.
- [[../concepts/trace-first-design]]: process 문서와 simulator/test 문서가 공유하는 설계 원리다.
- [[../concepts/tile-semantics-contract]]: 문서 간 contract를 맞추는 프로세스가 결국 tile semantics 정합성으로 수렴한다.

## Open Questions [coverage: medium -- 4 sources]

- roadmap와 changelog를 process topic 안에 둘지 project-management용 별도 topic으로 분리할지 아직 확정되지 않았다.
- `docs/process/archive/*`를 활성 process 문서와 어떻게 구분해 보여줄지 정리 필요가 있다.
- CI에서 문서 상태와 cross-reference를 자동 검증하는 실제 스크립트는 아직 문서 수준 계획에 머물러 있다.

## Sources

- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/spec_driven_development_workflow]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/project_roadmap]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/milestone_plan]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/versioning_and_changelog_guide]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/documentation_review_summary]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/doc_improvement_tasks]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/doc_study_milestone_plan]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/contribution_and_review_guide]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/naming_convention]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/tile_based_integration_mapping]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/process/vibe_coding_sdd_guide]]
- [[../../raw/AI-Hardware/Simulator/RISCV_NPU_SoC_SIM/docs/overview/sdd_devflow_overview]]
