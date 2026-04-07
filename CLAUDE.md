# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 이 vault의 목적

AI 하드웨어(NPU/가속기 아키텍처, 시뮬레이터)와 GenAI(양자화·압축, HW-friendly 기법) 분야의 개인 연구 지식베이스. LLM이 wiki를 작성·유지하고 사용자는 소싱과 방향 설정을 담당한다.

---

## 디렉토리 구조

```
obsidian_notes/
├── raw/                    ← 원본 소스 (ChatGPT 대화 export, 클립된 아티클 등). 수정 금지.
│   ├── AI-Hardware/
│   │   ├── Architecture/
│   │   └── Simulator/
│   ├── GenAI/
│   │   ├── Compression/
│   │   └── HW-Friendly/
│   ├── Research/
│   │   ├── Paper-Review/
│   │   └── Patent-MCM/
│   └── Misc/
├── wiki/                   ← LLM이 생성·유지하는 합성 페이지 (아직 비어 있음, 필요 시 생성)
├── index.md                ← 전체 페이지 카탈로그 (모든 ingest 후 업데이트)
├── log.md                  ← append-only 활동 로그
└── CLAUDE.md               ← 이 파일 (schema)
```

**raw/** 파일은 절대 수정하지 않는다. wiki/ 와 index.md, log.md 만 LLM이 쓴다.

---

## 노트 포맷 (raw/)

현재 raw/ 안의 노트는 ChatGPT 대화 export 형식이다:

```
# 제목

ChatTime: YYYY년 MM월 DD일 HH:MM (GMT+9)
Created time: ...
ProjectName: ...
URL: https://chatgpt.com/c/...

- **You: ...**
    (사용자 질문 또는 논문 제목/내용)
- **ChatGPT: ...**
    (ChatGPT 분석·정리 내용)
```

파일명 규칙: `<한국어 또는 영어 제목> <32자 hex id>.md`
긴 노트는 `(Part N of M)` 형식으로 분할되어 있다.

---

## Ingest 워크플로우

새 소스를 추가할 때:

1. 파일을 `raw/<카테고리>/` 에 저장
2. 내용을 읽고 사용자와 핵심 takeaway 논의
3. 필요 시 `wiki/` 에 합성 페이지 작성 (엔티티 페이지, 비교 분석, 개념 요약 등)
4. `index.md`에 새 항목 추가 (카테고리 테이블에 링크 + 한 줄 요약)
5. `log.md` 맨 아래에 항목 추가:
   ```
   ## [YYYY-MM-DD] ingest | <제목>
   - <무엇을 추가했는지, 어느 wiki 페이지에 영향을 미쳤는지>
   ```

---

## Query 워크플로우

질문을 받으면:
1. `index.md`를 먼저 읽어 관련 페이지를 파악
2. 관련 raw/ 또는 wiki/ 페이지를 읽고 답변 합성
3. 답변이 재사용 가치 있는 분석이라면 `wiki/`에 새 페이지로 저장하고 index.md에 추가

---

## Lint 워크플로우

`lint`를 요청받으면:
- index.md에 없는 raw/ 파일 탐지 (orphan source)
- wiki/ 페이지 간 모순·outdated 내용 확인
- 여러 페이지에서 언급되지만 독립 페이지가 없는 개념 파악
- log.md에 lint 결과 기록

---

## 카테고리 가이드

| 카테고리 | 포함 내용 |
|---|---|
| AI-Hardware/Architecture | 칩 아키텍처, 버스/인터페이스, 메모리 계층 |
| AI-Hardware/Simulator | NPU/LPU 시뮬레이터, C/RTL 구현 분석 |
| GenAI/Compression | 양자화(PTQ/QAT), pruning, distillation |
| GenAI/HW-Friendly | HW-aware 모델 설계, KV-cache, normalization-free |
| Research/Paper-Review | 논문 요약 및 비판적 분석 |
| Research/Patent-MCM | MCM·chiplet 관련 특허 및 논문 |
| Misc | 기술 영어, 기타 참고 자료 |
