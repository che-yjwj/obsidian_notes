# Log

append-only 활동 기록. 형식: `## [YYYY-MM-DD] <type> | <title>`
type: `ingest` · `query` · `lint` · `restructure`

---

## [2026-04-07] restructure | LLM Wiki 패턴으로 vault 초기 셋업

- raw/ 폴더 생성 후 AI-Hardware, GenAI, Misc, Research 이동
- index.md, log.md 생성
- CLAUDE.md를 schema 문서로 재작성
- 기존 노트(ChatGPT 대화 export)를 raw source 레이어로 분류

## [2026-04-07] ingest | 초기 합성 wiki 페이지 백필

- 기존 raw source를 바탕으로 생성된 합성 페이지 8개를 기록 계층에 반영
- AI-Hardware, GenAI, Research, Misc 상위 요약 페이지를 wiki 레이어의 canonical 페이지로 정리
- 이후 index.md를 raw-only 목록이 아니라 wiki 우선 카탈로그로 재구성하기 위한 기준 마련

## [2026-04-07] restructure | index 카탈로그 재구성

- index.md를 synthesized wiki pages + raw source inventory 구조로 전면 재편
- 파트 분할 raw 문서는 대표 주제 단위로 묶어 inventory에 반영
- wiki 생성물과 raw source를 분리해서 탐색 가능한 카탈로그 구조 확정

## [2026-04-07] ingest | OCEAN 상세 페이지 및 lint 체크리스트 추가

- wiki/GenAI/OCEAN-Compression-Deep-Dive.md 생성
- wiki/Misc/Vault-Lint-Checklist.md 생성
- wiki/GenAI/LLM-Quantization-and-Compression.md에 OCEAN 9개 파트 source coverage와 상세 페이지 링크 반영

## [2026-04-07] query | 다음 단계 작업 메모

- lint 체크리스트 기준으로 orphan raw, broken link, source coverage를 실제 점검
- QuaRot, SpinQuant, SmoothQuant 비교용 wiki 상세 페이지 후보 검토
- README.md를 vault 운영 방식과 카탈로그 구조에 맞게 최소 설명으로 정리

## [2026-04-09] lint | vault 정합성 점검

- `index.md` raw inventory와 `raw/` 전체 파일 목록을 대조했고 orphan raw는 없음을 확인
- `wiki/` frontmatter `sources:`와 내부 `[[...]]` 링크를 점검했고 깨진 source/link는 없음을 확인
- `wiki/Research/Paper-Reviews.md`의 TODO성 문구를 원문 기반 요약으로 치환해 content drift를 줄임

## [2026-04-09] restructure | README 정리

- `README.md`를 vault 목적, 디렉토리 구조, 작업 규칙, graphify 사용 방식 기준으로 최소 설명 형태로 정리

## [2026-04-09] restructure | index 소개 문구 정렬

- `index.md` 상단 설명을 `README.md`와 같은 어휘로 조정해 vault 운영 문서 톤을 맞춤

## [2026-04-09] ingest | outlier 완화 기법 비교 페이지 추가

- `wiki/GenAI/Outlier-Mitigation-Methods-Comparison.md` 생성
- `SmoothQuant`, `QuaRot`, `SpinQuant` 비교와 OCEAN 상위 프레임 차이를 정리
- `wiki/GenAI/LLM-Quantization-and-Compression.md`와 `index.md`에 링크 반영

## [2026-04-09] restructure | Memory Hierarchy 허브화

- `wiki/AI-Hardware/Memory-Hierarchy-in-AI-Accelerators.md`에 `NPU-Architecture`, `Simulator-and-Implementation-Tools`와의 연결 문맥 추가
- Tesla AI5/AI6, HyperAccel LPU, edge/server 메모리 전략 차이를 메모리 병목 관점으로 명시
- 단순 비교 페이지가 아니라 AI-Hardware 축의 허브 문서로 역할을 확장

## [2026-04-09] restructure | SoC 협의 영어 문서 실무화

- `wiki/Misc/SoC-Specification-Negotiation-English.md`를 표현 모음에서 실무 reference로 확장
- `NPU-Architecture`, `Memory-Hierarchy-in-AI-Accelerators`, `Simulator-and-Implementation-Tools`와 연결되는 협상 포인트를 추가
- requirement, assumption, trade-off, ownership 기준의 재사용 가능한 문장 묶음을 정리

## [2026-04-09] restructure | Patent-MCM 주제 분리

- `wiki/Research/Patent-MCM.md`를 MCM과 Taalas 중심 문서로 축소 정리
- `wiki/Research/AI-Assisted-Research-Workflow.md`를 새로 추가해 AgentHub, Claude Code 기반 연구 자동화, tiny reasoning을 별도 축으로 분리
- `index.md`의 synthesized wiki pages 설명을 새 구조에 맞게 갱신

## [2026-04-09] restructure | Paper Reviews 재클러스터링

- `wiki/Research/Paper-Reviews.md`를 reasoning/theory 축의 허브 문서로 재구성
- `wiki/Research/Research-Tooling-Reviews.md`를 새로 추가해 PaperDebugger와 PaperBanana를 분리
- `index.md`의 Research 섹션 요약을 새 분류에 맞게 조정

## [2026-04-09] restructure | index 탐색 경로 추가

- `index.md` 상단에 `Recommended Reading Paths` 섹션 추가
- AI-Hardware, GenAI compression, Research workflow 기준의 입문 동선을 명시
- 문서 수가 늘어난 상태에서 첫 진입자의 읽기 순서를 짧게 안내하도록 정리

## [2026-04-09] restructure | README 루트 허브화

- `README.md`에 `시작점`과 `허브 문서` 섹션 추가
- `index`, AI-Hardware, GenAI, Research 축의 대표 페이지를 직접 링크
- vault root 문서가 단순 설명이 아니라 실제 탐색 진입점으로 동작하도록 조정

## [2026-04-09] lint | 구조 재정비 후 정합성 재점검

- `README.md`, `index.md`, `wiki/` 내부 `[[...]]` 링크를 점검했고 깨진 wiki 링크는 없음을 확인
- `wiki/` frontmatter `sources:` 경로를 재검증했고 누락되거나 존재하지 않는 raw source는 없음을 확인
- `index.md`의 synthesized wiki table과 실제 `wiki/` 파일 목록을 대조했고 누락/과잉 항목이 없음을 확인
- `raw/` 전체 파일과 `index.md` raw inventory를 다시 비교했고 orphan raw는 없음을 확인
- `wiki/GenAI/LLM-Quantization-and-Compression.md`가 OCEAN 9개 파트를 모두 source로 포함함을 재확인

## [2026-04-09] restructure | lint 체크리스트 최신화

- `wiki/Misc/Vault-Lint-Checklist.md`의 `Current High-Value Checks`를 현재 허브 구조 기준으로 갱신
- `README` 루트 허브, Research 분리 구조, `index`/`log` 동기화, `graphify-out/needs_update` 상태 확인을 상시 점검 항목에 추가

## [2026-04-09] restructure | graphify 갱신 절차 문서화

- `README.md`에 code-only rebuild, `needs_update`, `/graphify --update` 순서의 운영 절차를 추가
- `wiki/Misc/Vault-Lint-Checklist.md`에 stale graph 해석 규칙과 graphify 갱신 기준을 명시

## [2026-04-09] restructure | graphify 업데이트 후 검토 기준 추가

- `wiki/Misc/Vault-Lint-Checklist.md`에 `/graphify --update` 이후 확인할 비교 포인트를 추가
- `README.md`에 `GRAPH_REPORT.md`에서 우선 볼 섹션을 짧게 명시

## [2026-04-09] restructure | graphify python 경로 고정

- 저장소 루트에 `.graphify_python` 파일을 추가해 graphify용 Python 인터프리터 경로를 명시
- `README.md`에 graphify 스크립트/스킬이 이 경로를 기준으로 실행된다는 운영 메모를 추가

## [2026-04-09] restructure | graphify 인터프리터 사용 강제

- `AGENTS.md`에 graphify 관련 Python 실행은 항상 `.graphify_python`을 사용한다는 규칙을 추가
- `scripts/graphify-python` 래퍼 스크립트를 추가해 system `python3` 대신 고정 인터프리터를 쉽게 호출하도록 정리

## [2026-04-09] restructure | graphify 업데이트 검토 템플릿 추가

- `wiki/Misc/Graphify-Post-Update-Review-Template.md`를 생성해 `/graphify --update` 직후 비교 항목을 표준화
- `index.md`, `README.md`에 해당 템플릿 링크를 추가
