# obsidian_notes

AI 하드웨어와 GenAI 주제를 정리하는 vault. 원본 소스는 `raw/`에 보존하고, 합성된 canonical 문서는 `wiki/`에 유지한다.

처음 들어올 때는 [[index]]를 먼저 보고, 아래 허브 문서 중 하나에서 시작하는 것이 가장 빠르다.

## 시작점

- AI-Hardware: [[wiki/AI-Hardware/NPU-Architecture]] → [[wiki/AI-Hardware/Memory-Hierarchy-in-AI-Accelerators]]
- GenAI compression: [[wiki/GenAI/LLM-Quantization-and-Compression]] → [[wiki/GenAI/OCEAN-Compression-Deep-Dive]]
- Research workflow: [[wiki/Research/AI-Assisted-Research-Workflow]] → [[wiki/Research/Research-Tooling-Reviews]]
- Research theory: [[wiki/Research/Paper-Reviews]] → [[wiki/Research/Patent-MCM]]

## 개요

- `raw/`: 원본 소스 보관 레이어. 수정하지 않는다.
- `wiki/`: 합성 wiki 페이지 레이어
- `index.md`: wiki 우선 카탈로그 + raw source inventory
- `log.md`: append-only 작업 기록
- `graphify-out/`: knowledge graph 산출물

## 운영 규칙

- 새 자료는 먼저 `raw/`에 추가한다.
- 재사용 가치가 있으면 `wiki/`에 합성 페이지를 만든다.
- 카탈로그 변경은 `index.md`에 반영한다.
- 작업 기록은 `log.md`에 append-only로 남긴다.

## 허브 문서

- [[index]]: 전체 카탈로그와 추천 탐색 경로
- [[wiki/AI-Hardware/NPU-Architecture]]: AI-Hardware 상위 개요
- [[wiki/GenAI/LLM-Quantization-and-Compression]]: GenAI 압축 상위 개요
- [[wiki/Research/AI-Assisted-Research-Workflow]]: 연구 자동화와 agent workflow 축
- [[wiki/Misc/Vault-Lint-Checklist]]: vault 정합성 점검 기준
- [[wiki/Misc/Graphify-Post-Update-Review-Template]]: graph 업데이트 후 비교 기록 템플릿

## Graphify

- 구조나 주제 맥락을 파악할 때는 먼저 `graphify-out/GRAPH_REPORT.md`를 읽는다.
- 코드 파일 수정 후에는 graphify code rebuild를 실행한다.
- Markdown/PDF/image 변경 후에는 semantic update 필요 상태를 표시한다.
- graphify 관련 스크립트/스킬은 저장소 루트의 `.graphify_python` 인터프리터 경로를 기준으로 실행한다.

현재 graphify는 문서 변경분에 대해 `graphify-out/needs_update` 플래그를 사용해 stale 상태를 표시한다.

### Update Flow

1. 코드만 바뀌었으면 AGENTS.md에 적힌 `graphify.watch._rebuild_code` 명령으로 즉시 갱신한다.
2. Markdown/PDF/image가 바뀌었으면 `graphify.watch._notify_only`로 `graphify-out/needs_update`를 남긴다.
3. `needs_update`가 있으면 현재 `GRAPH_REPORT.md`는 stale일 수 있으므로, Claude Code에서 `/graphify --update`를 실행한 뒤 다시 읽는다.

업데이트 후에는 `GRAPH_REPORT.md`에서 `Knowledge Gaps`, `God Nodes`, `Surprising Connections`를 먼저 비교한다.
