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
