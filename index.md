# Wiki Index

이 vault의 wiki 우선 카탈로그. 합성 wiki 페이지를 먼저 정리하고, 하단에는 raw source inventory를 둔다.

---

## Recommended Reading Paths

처음 들어오는 사람이 시작점을 빠르게 잡을 수 있도록 대표 탐색 경로를 둔다.

### AI-Hardware 입문

1. [[wiki/AI-Hardware/NPU-Architecture]]
2. [[wiki/AI-Hardware/Memory-Hierarchy-in-AI-Accelerators]]
3. [[wiki/AI-Hardware/Simulator-and-Implementation-Tools]]
4. [[wiki/Misc/SoC-Specification-Negotiation-English]]

아키텍처 개요 → 메모리 병목 → 구현 사례 → 실제 사양 협의 표현 순서로 읽는다.

### GenAI 압축/하드웨어 친화 설계

1. [[wiki/GenAI/LLM-Quantization-and-Compression]]
2. [[wiki/GenAI/Outlier-Mitigation-Methods-Comparison]]
3. [[wiki/GenAI/OCEAN-Compression-Deep-Dive]]
4. [[wiki/GenAI/HW-Friendly-Model-Design]]

양자화 상위 개요 → outlier 완화 비교 → OCEAN 심화 → HW-friendly 설계 축으로 내려간다.

### Research Workflow와 자동화

1. [[wiki/Research/AI-Assisted-Research-Workflow]]
2. [[wiki/Research/Research-Tooling-Reviews]]
3. [[wiki/Research/Paper-Reviews]]
4. [[wiki/Research/Patent-MCM]]

연구 자동화 구조 → 생산성 도구 → reasoning/theory 리뷰 → 하드웨어 특수화 아이디어 순서로 연결한다.

---

## Synthesized Wiki Pages

| 페이지 | 타입 | 한 줄 요약 |
|---|---|---|
| [[wiki/AI-Hardware/Memory-Hierarchy-in-AI-Accelerators]] | topic | H100, B100, AHPM, MI300를 중심으로 AI 가속기 메모리 계층과 설계 시사점 정리 |
| [[wiki/AI-Hardware/NPU-Architecture]] | topic | Tesla, AMD, Xilinx, TPU Sparse Core, AMBA 권장안을 묶은 NPU 아키텍처 개요 |
| [[wiki/AI-Hardware/Simulator-and-Implementation-Tools]] | topic | HyperAccel LPU, Coral, MTIA, C 구현, 슈퍼노드 등 시뮬레이터 및 구현 관점 정리 |
| [[wiki/GenAI/HW-Friendly-Model-Design]] | topic | KV-cache, normalization-free, latent world model, RAG 관점에서 HW-friendly 설계 정리 |
| [[wiki/GenAI/LLM-Quantization-and-Compression]] | topic | 양자화, 저비트 압축, outlier 처리, microscaling, OCEAN을 묶은 상위 개요 페이지 |
| [[wiki/GenAI/OCEAN-Compression-Deep-Dive]] | deep-dive | OCEAN의 축 변환, 함수적 표현, residual, rate-distortion, 검증 실험 설계를 상세 정리 |
| [[wiki/GenAI/Outlier-Mitigation-Methods-Comparison]] | comparison | SmoothQuant, QuaRot, SpinQuant를 중심으로 outlier 완화 계열을 비교하고 OCEAN과의 차이를 정리 |
| [[wiki/Misc/SoC-Specification-Negotiation-English]] | reference | SoC 사양 협의에 쓰는 실무 영어 표현 요약 |
| [[wiki/Misc/Graphify-Post-Update-Review-Template]] | reference | `/graphify --update` 후 `GRAPH_REPORT.md` 비교 기록용 템플릿 |
| [[wiki/Misc/Vault-Lint-Checklist]] | reference | orphan source, 끊어진 링크, source coverage, log 정합성을 점검하는 lint 체크리스트 |
| [[wiki/Research/AI-Assisted-Research-Workflow]] | topic | AgentHub, Claude Code 기반 연구 자동화, tiny reasoning을 하나의 workflow 축으로 정리 |
| [[wiki/Research/Paper-Reviews]] | topic | STEM, Dr Zero, Chaos Theory를 reasoning/theory 축으로 묶는 리뷰 허브 |
| [[wiki/Research/Patent-MCM]] | topic | Multiplierless 설계와 Taalas식 model-centric infrastructure의 접점 정리 |
| [[wiki/Research/Research-Tooling-Reviews]] | topic | PaperDebugger와 PaperBanana를 연구 생산성 도구 축으로 정리 |

## Raw Source Inventory

원본 소스는 `raw/` 아래에 보존한다. 파트 분할 문서는 대표 주제 단위로 묶어서 기록한다.

### AI-Hardware / Architecture

| 소스 | 형태 | 한 줄 요약 |
|---|---|---|
| [[raw/AI-Hardware/Architecture/Memory Hierarchy in AI 33a6cc566b0b81f4b111e6d0e5d21553.md]] | single | H100, B100, AHPM, MI300 기반 메모리 계층 비교 |
| [[raw/AI-Hardware/Architecture/Xilinx FINN Overview 33a6cc566b0b81068908d40fc473eaef.md]] | single | Xilinx FINN 프레임워크와 FPGA 추론 파이프라인 개요 |
| [[raw/AI-Hardware/Architecture/Tesla AI Chip Roadmap 33a6cc566b0b8129a0f3eef1ddc4dfa3.md]] | single | Tesla FSD 칩 세대별 로드맵 정리 |
| [[raw/AI-Hardware/Architecture/Tesla Edge AI Innovation 33a6cc566b0b81c8aa60dc6f39c11a1c.md]] | single | 테슬라의 엣지 AI 추론 혁신 내용 |
| [[raw/AI-Hardware/Architecture/AMD Versal ACAP Overview 33a6cc566b0b81d6b8d8ca55e7b2ba1d.md]] | single | AMD Versal ACAP 구조와 역할 |
| [[raw/AI-Hardware/Architecture/Synopsys Platform Architect Introduction 33a6cc566b0b81c78ed7c21c89862ab6.md]] | single | Platform Architect 소개와 시스템 레벨 탐색 용도 |
| [[raw/AI-Hardware/Architecture/NPU Performance-based AMBA Recommendation 33a6cc566b0b81d1a6f8fd7a1f0b2ffe.md]] | single | NPU 성능 티어별 버스 인터페이스 권장안 |
| [[raw/AI-Hardware/Architecture/TPU Sparse Core Explanation 33a6cc566b0b81f7a2a5f1e85f847ccb.md]] | single | TPU Sparse Core 구조 및 sparse workload 처리 |

### AI-Hardware / Simulator

| 소스 | 형태 | 한 줄 요약 |
|---|---|---|
| [[raw/AI-Hardware/Simulator/MTIA TPU Paper Analysis 33a6cc566b0b816ebdbfedfedbb11fb6.md]] | single | Meta MTIA와 Google TPU 비교 논문 분석 |
| [[raw/AI-Hardware/Simulator/Transformer C Implementation Analysis 33a6cc566b0b818585ddf76217de5112.md]] | single | Transformer 순전파의 C 구현 분석 |
| [[raw/AI-Hardware/Simulator/NPU Supernode Concept 33a6cc566b0b81aca099f1e6a9a8061e.md]] | single | NPU 슈퍼노드 개념과 분산 처리 관점 |
| [[raw/AI-Hardware/Simulator/coral npu]] | 2 parts | Coral NPU와 Edge TPU 구조 분석 |
| [[raw/AI-Hardware/Simulator/HyperAccel LPU Explanation]] | 3 parts | HyperAccel LPU 구조와 LLM 추론용 아키텍처 설명 |

### GenAI / Compression

| 소스 | 형태 | 한 줄 요약 |
|---|---|---|
| [[raw/GenAI/Compression/DC-LLM Paper Summary 33a6cc566b0b810abe58cd3c651ad3ad.md]] | single | Dynamic Linear Combination 기반 HW-friendly 압축 |
| [[raw/GenAI/Compression/OliVe Paper Summary 33a6cc566b0b81419e7bfc22f35bdf25.md]] | single | Outlier-victim pair를 이용한 양자화 아이디어 |
| [[raw/GenAI/Compression/LLM Quantization Architecture 33a6cc566b0b8181a2d9e4bd12d3d4c8.md]] | single | LLM 양자화 전반 구조와 설계 선택지 |
| [[raw/GenAI/Compression/Microscaling vs Mixed-Precision 33a6cc566b0b81798986d497a0b25f67.md]] | single | Microscaling과 mixed-precision의 비교 |
| [[raw/GenAI/Compression/TurboQuant Concept Summary]] | 2 parts | TurboQuant 개념 정리 |
| [[raw/GenAI/Compression/TurboQuant PyTorch Implementation 33a6cc566b0b8163a307f12187edcc91.md]] | single | TurboQuant PyTorch 구현 분석 |
| [[raw/GenAI/Compression/OCEAN-based LLM Compression]] | 9 parts | OCEAN의 좌표계, residual, entropy coding, 실험 설계까지 이어지는 심층 대화 |

### GenAI / HW-Friendly

| 소스 | 형태 | 한 줄 요약 |
|---|---|---|
| [[raw/GenAI/HW-Friendly/KV-cache Optimization Explanation 33a6cc566b0b811890a0df4bc615b114.md]] | single | KV-cache 최적화와 HW 구현 포인트 |
| [[raw/GenAI/HW-Friendly/Latent Action World Model 33a6cc566b0b816ba286eaff4c35c587.md]] | single | Latent Action World Model 구조 분석 |
| [[raw/GenAI/HW-Friendly/Stronger Normalization-Free Transformers 33a6cc566b0b81648fc4ec9576dc68a0.md]] | single | Normalization-free Transformer 설계 방향 |
| [[raw/GenAI/HW-Friendly/MSA and RAG Analysis 33a6cc566b0b81b6b40ec9ead1726cf4.md]] | single | Multi-head Self-Attention과 RAG 연계 분석 |

### Research / Paper-Review

| 소스 | 형태 | 한 줄 요약 |
|---|---|---|
| [[raw/Research/Paper-Review/STEM Structure and Scalability 33a6cc566b0b8134bba4f04b6ccf5786.md]] | single | STEM 임베딩 모듈 기반 확장성 정리 |
| [[raw/Research/Paper-Review/PaperDebugger Research Productivity Innovation 33a6cc566b0b81438d17de8e443da8de.md]] | single | PaperDebugger 기반 연구 생산성 향상 |
| [[raw/Research/Paper-Review/Dr Zero Concept Summary 33a6cc566b0b81dea270edb325090317.md]] | single | Dr Zero 개념 정리 |
| [[raw/Research/Paper-Review/PaperBanana AI Research 33a6cc566b0b81f5b652f4c9128db47b.md]] | single | PaperBanana 연구 도구 분석 |
| [[raw/Research/Paper-Review/Chaos Theory and Prediction]] | 2 parts | 카오스 이론과 LLM 예측 능력 분석 |

### Research / Patent-MCM

| 소스 | 형태 | 한 줄 요약 |
|---|---|---|
| [[raw/Research/Patent-MCM/Taalas AI Infrastructure Innovation 33a6cc566b0b81378bc9f4ae1054ac16.md]] | single | Taalas 기반 AI 인프라 혁신 사례 |
| [[raw/Research/Patent-MCM/AgentHub and Collaboration Model Change 33a6cc566b0b815ab9faec6381cdfa23.md]] | single | AgentHub 도입에 따른 협업 모델 변화 |
| [[raw/Research/Patent-MCM/Reasoning in 13M Parameters 33a6cc566b0b816480b1d91f9b8363c9.md]] | single | 13M 파라미터 소형 모델의 reasoning 분석 |
| [[raw/Research/Patent-MCM/AI-assisted research workflow (Claude Code + Harne 33a6cc566b0b81ea9827ddba8abd5822.md]] | single | Claude Code 기반 AI 보조 연구 워크플로우 |
| [[raw/Research/Patent-MCM/Multiplierless Design Paper Analysis]] | 3 parts | Multiplierless 하드웨어 설계 논문 분석 |

### Misc

| 소스 | 형태 | 한 줄 요약 |
|---|---|---|
| [[raw/Misc/SoC Specification Negotiation English]] | 2 parts | SoC 사양 협의에 필요한 영어 표현 정리 |
