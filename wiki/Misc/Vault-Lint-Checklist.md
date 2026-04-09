---
title: Vault Lint Checklist
type: reference
sources:
  - CLAUDE.md
  - index.md
  - log.md
tags: [vault, lint, checklist, maintenance]
updated: 2026-04-09
---

# Vault Lint Checklist

이 vault의 `lint`는 문법 검사보다 운영 정합성 점검에 가깝다. 아래 순서대로 보면 된다.

## 1. Orphan Raw Sources

- `raw/` 아래 파일이 `index.md` raw inventory에 반영되어 있는지 확인한다
- 파트 분할 문서는 개별 파일 대신 대표 디렉토리 항목으로 묶여도 된다
- 새 raw가 추가됐는데 wiki나 log에 아무 흔적이 없으면 ingest 후보로 표시한다

## 2. Wiki Coverage

- wiki 페이지의 `sources:`가 실제 raw 경로를 가리키는지 확인한다
- 여러 파트로 구성된 원본은 필요한 파트가 누락되지 않았는지 본다
- 재사용 가치가 큰 질의 응답 결과가 raw만 있고 wiki에 아직 없으면 신규 합성 페이지 후보로 기록한다

## 3. Broken Links

- `index.md`와 wiki 내부 링크가 실제 페이지를 가리키는지 확인한다
- 아직 없는 페이지로 향하는 링크는 즉시 만들거나 TODO 문구로 낮춘다
- raw 링크는 대표 디렉토리 링크인지 실제 파일 링크인지 표기 규칙을 일관되게 유지한다

## 4. Index Consistency

- `index.md` 상단에는 wiki 합성 페이지가, 하단에는 raw source inventory가 있어야 한다
- 새 wiki 페이지를 만들면 synthesized table에 추가한다
- 새 raw source를 ingest하면 raw inventory에 추가한다

## 5. Log Consistency

- wiki 생성, 대규모 재구성, lint 결과는 `log.md`에 append-only로 남긴다
- 같은 날 여러 작업을 했더라도 작업 단위가 다르면 별도 항목으로 남긴다
- 과거 작업이 누락돼 있으면 backfill 항목으로 보완하되 기존 기록은 수정하지 않는다

## 6. Content Drift

- 여러 wiki 페이지가 같은 개념을 다르게 설명하는지 확인한다
- 상위 개요 페이지와 상세 페이지가 서로 모순되지 않는지 본다
- 오래된 TODO, `작성 권장`, `추후 정리` 문구가 남아 있으면 실제 작업 후보로 승격한다

## Current High-Value Checks

- `LLM Quantization and Compression`가 OCEAN 9개 파트를 모두 source로 포함하는지 확인
- `index.md`가 raw-only 목록으로 되돌아가지 않았는지 확인
- `README.md`가 `index`와 주요 허브 문서로 실제 진입 링크를 제공하는지 확인
- `Paper-Reviews` / `Research-Tooling-Reviews` / `AI-Assisted-Research-Workflow` 분리 구조가 `index.md`와 서로 일치하는지 확인
- 새 wiki 페이지 추가 뒤 `index.md`와 `log.md`가 함께 갱신됐는지 확인
- `graphify-out/needs_update` 플래그가 남아 있으면 semantic re-extraction이 필요한 상태로 간주하고 stale graph를 전제로 읽는다

## Graphify Update Rule

- 코드 변경만 있으면 `_rebuild_code`로 graph를 바로 갱신한다
- 문서나 이미지 변경이 섞이면 `_notify_only`로 플래그만 남기고 semantic update는 별도로 처리한다
- `needs_update`가 남은 상태에서는 graph 구조 해석 결과를 확정 사실처럼 쓰지 않는다
- 구조 정비 작업 직후에는 가능하면 Claude Code에서 `/graphify --update`를 실행해 새 `GRAPH_REPORT.md`를 기준으로 다시 본다

## After Update Review

- `Knowledge Gaps`에서 isolated node 수가 줄었는지 본다
- `SoC Specification Negotiation English`, `Memory Hierarchy in AI Accelerators`, `obsidian_notes`가 여전히 thin community인지 확인한다
- `Paper Reviews` god node 비중이 줄고 Research 축이 더 분리된 community로 보이는지 확인한다
- 새 허브 문서인 `AI-Assisted Research Workflow`, `Research Tooling Reviews`가 독립 축으로 잡히는지 본다
- `Surprising Connections`에 새로 추가된 cross-page bridge가 이번 구조 변경과 일치하는지 확인한다
