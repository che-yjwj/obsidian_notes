---
title: NPU Documentation Process
type: topic
status: canonical
last_compiled: 2026-04-17
---

# NPU Documentation Process

*last_compiled: 2026-04-17 | sources: 12*

---

## Summary [coverage: high -- 6 sources]

이 토픽은 RISCV_NPU_SoC_SIM 프로젝트에서 문서가 구현을 선행하고 통제하는 방식을 다룬다. 핵심은 `spec_driven_development_workflow.md`, `project_roadmap.md`, `vibe_coding_sdd_guide.md` 같은 **process control plane** 문서가 작업 순서와 ownership를 정의하고, 나머지 plan/review 문서는 그 위에서 실행 상태를 기록하는 **operational artifact**로 동작한다는 점이다.

즉, 이 topic은 기술 스펙 자체가 아니라 **문서 아키텍처와 작업 순서**를 정의한다. graphify에서도 process 문서군이 architecture/spec와 별도 커뮤니티를 이루고 있지만, `documentation_review_summary.md`나 `milestone_plan.md` 같은 운영 문서를 별도 지식 허브로 읽으면 과대해석이 생긴다. canonical layer에서는 이 topic이 process 문서군의 단일 진입점으로 남고, review/plan 문서는 실행 artifact로 취급하는 편이 맞다.

## Core Concepts [coverage: high -- 6 sources]

- **Document first**: 코드 전에 spec, design, test 문서를 갱신한다.
- **Single source of truth**: IR, CMDQ, timing, trace의 의미는 대응 스펙 문서가 주도한다.
- **Roadmap staging**: Foundation → Timing/Engine Modeling → Full Operator Support → LLM Specialization → DSE Platform 순으로 확장한다.
- **Traceability**: naming, changelog, review guide를 통해 문서와 코드 사이 연결을 남긴다.
- **Operational artifact demotion**: review summary, milestone tracker, task list는 중요하지만 reusable semantic hub로 취급하지 않는다.

## Architecture [coverage: medium -- 5 sources]

process 문서 계층은 세 부분으로 읽는 편이 안전하다. 첫째, `spec_driven_development_workflow.md`, `project_roadmap.md`, `vibe_coding_sdd_guide.md`는 작업 루프와 개발 원칙을 정의하는 **control-plane docs**다. 둘째, `versioning_and_changelog_guide.md`, `contribution_and_review_guide.md`, `naming_convention.md`는 이 흐름을 협업 규칙으로 고정하는 **governance docs**다. 셋째, `milestone_plan.md`, `doc_study_milestone_plan.md`, `doc_improvement_tasks.md`, `documentation_review_summary.md`는 특정 시점의 상태와 TODO를 담는 **operational artifacts**다.

이 구조는 기술 문서의 품질 자체를 관리하는 문서적 control plane이라고 볼 수 있다. 그래서 architecture topic과는 다르게 “어떤 순서로 문서를 만들고 검증하느냐”가 핵심이 된다. 다만 graph 운영 관점에서는 control-plane/gov 문서와 operational artifact를 한 덩어리로 취급하면 `Documentation Review Summary` 같은 집계 문서가 과한 허브가 되므로, canonical 위키에서는 이 경계를 의식적으로 분리해야 한다.

## Document Classes [coverage: medium -- 5 sources]

### Control-plane docs

- `spec_driven_development_workflow.md`
- `project_roadmap.md`
- `vibe_coding_sdd_guide.md`

이 셋은 process topic의 핵심 축이다. 순서, phase, AI-assisted coding usage rule을 정의하므로 reusable process knowledge로 남는다.

### Governance docs

- `versioning_and_changelog_guide.md`
- `contribution_and_review_guide.md`
- `naming_convention.md`

이 문서들은 spec/code/review 관계를 안정화하는 규칙층이다. process topic 안에서는 control plane을 보조하는 규칙 레이어로 읽는 편이 맞다.

### Operational artifacts

- `milestone_plan.md`
- `doc_study_milestone_plan.md`
- `doc_improvement_tasks.md`
- `documentation_review_summary.md`

이 문서들은 중요하지만 재사용 가능한 개념 허브는 아니다. 특정 시점의 리뷰 결과, TODO, phase 상태를 담는 execution artifact이므로, canonical topic에서 별도 semantic 중심축으로 승격시키지 않는 것이 구조상 안전하다.

## Key Findings [coverage: high -- 6 sources]

- 프로젝트는 이미 문서 중심 코드베이스를 전제로 하며, 실제 구현은 milestone plan에 따라 추후 채워지는 구조다.
- `Vertical-first`, `MatMul-only E2E`, `Trace-first design`, `Test-first for core logic`이 process 문서 전반에서 반복된다.
- Codex 사용도 자유형 구현이 아니라 Phase/Task/Spec/Design context를 상단에 고정하는 방식으로 제한한다.
- versioning 정책은 spec이 코드를 주도하는 `spec-first versioning`을 명시한다.
- roadmap와 workflow는 process control plane으로 유지하되, review summary와 milestone tracker는 운영 artifact로 내려 읽는 편이 graph 허브 왜곡을 줄인다.

## Connections [coverage: medium -- 4 sources]

- [[riscv-npu-soc-sim]]: 실제 프로젝트 내용과 연결되는 process control plane이다.
- [[trace-visualization]]: trace-first design과 golden trace 운영 규칙이 process와 직접 연결된다.
- [[../concepts/trace-first-design]]: process 문서와 simulator/test 문서가 공유하는 설계 원리다.
- [[../concepts/tile-semantics-contract]]: 문서 간 contract를 맞추는 프로세스가 결국 tile semantics 정합성으로 수렴한다.
- [[simulation-validation]]: process layer가 정의한 review/acceptance 흐름이 golden trace와 regression 운영 기준으로 이어진다.

## Open Questions [coverage: medium -- 4 sources]

- roadmap와 workflow는 현재 process topic 안에 두는 쪽으로 정리했지만, 장기적으로 DSE/benchmark 운영 문서가 늘면 project-management topic 분리가 다시 필요할 수 있다.
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
