---
title: Vault Lint Checklist
type: reference
sources:
  - CLAUDE.md
  - index.md
  - log.md
tags: [vault, lint, checklist, maintenance]
updated: 2026-04-07
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
- 새 wiki 페이지 추가 뒤 `log.md`가 빠지지 않았는지 확인
