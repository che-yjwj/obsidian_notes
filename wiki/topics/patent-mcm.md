# Patent & MCM Research

*last_compiled: 2026-04-12 | sources: 7*

---

## Summary [coverage: high -- 7 sources]

이 토픽은 **MCM(Multiple Constant Multiplication) 기반 multiplierless 하드웨어 설계**를 중심으로, AI 인프라 패러다임 전환(Taalas), 소규모 파라미터 추론(TinyLoRA/13M), 에이전트 기반 협업 플랫폼(AgentHub), AI-assisted 연구 워크플로우, 그리고 multiplierless DWT/CNN 논문 분석을 포괄한다.

핵심 연구 방향:
- **MCM(Multiple Constant Multiplication)**: 고정 상수 곱셈을 shift+add/sub DAG로 대체하여 곱셈기를 완전히 제거하는 VLSI 최적화 기법. Lefevre 알고리즘, Voronenko-Püschel A-graph 이론이 기반.
- **Multiplierless DWT**: JPEG2000 표준의 9/7 리프팅 기반 웨이블릿 변환에 MCM을 적용해 ASIC 면적 51%, 전력 43%, 지연 30% 감소 달성 (0.18 μm CMOS, 200 MHz).
- **MCM × LLM 하드와이어링**: Taalas의 "모델-실리콘 직접 매핑" 전략과 MCM을 결합하면 LLM 가중치(inference 시 상수)를 shift/add 네트워크로 구현해 DSP/곱셈기를 제거할 수 있다는 아이디어.
- **소규모 reasoning**: "Learning to Reason in 13 Parameters"(Meta FAIR, 2026)는 TinyLoRA(13 trainable parameters, 26 bytes)와 RL을 결합해 Qwen2.5-8B 기반 GSM8K 91% 달성 — reasoning capability는 parameter scale이 아닌 training signal에 의존한다는 발견.
- **응용 도메인**: ISP/SR, 멀티모달 이미지·오디오 인코더, edge AI 퍼셉션 프론트엔드.

---

## Core Concepts [coverage: high -- 7 sources]

### MCM (Multiple Constant Multiplication)

MCM은 상수 집합 `{c₁, c₂, ..., cₙ}`에 대해 각 `cᵢ·x`를 곱셈기 없이 이진 shift, add/sub 만으로 구현하는 문제다. Voronenko & Püschel (CMU)의 공식화에 따르면:

- **A-operation**: `A-op(u, v) = |2^l1 · u + (-1)^s · 2^l2 · v| >> r`
- **A-graph**: A-operation의 DAG 표현으로 공통 부분식(CSE) 공유를 가능하게 함
- **NP-완전 문제**이나 heuristic search로 실용적 최적해 탐색 가능
- 상수 100개 이상의 필터뱅크에도 확장 가능

#### 세 가지 관련 접근의 계층적 관계

| 접근 | 본질 | 컴파일러 친화성 | 정확도 영향 |
|---|---|---|---|
| **MCM (Voronenko)** | 구현 그래프 최적화 (shift/add DAG + CSE) | 최상 | 없음 |
| **Multiplierless Neuron (Sarwar)** | 가중치를 Po2로 제한 (독립 shift-add) | 보통 | 약간 |
| **AdderNet** | Conv 연산을 `−Σ|xᵢ − wᵢ|`로 재정의 | 어려움 | 구조적 변경 |

NPU 컴파일러 관점에서 MCM이 정석: 모델 수식 100% 유지, VE(shift/add) 활용 극대화, TE(MAC)와 hybrid 가능.

### PEFT × MCM 결합 구조

PEFT의 공통 구조 `y = Wx + g_θ(x)` (W: frozen base, g_θ: 작은 보정항)는 MCM과 이상적으로 결합된다:
- **Base 경로 (W)**: 상수 고정 → MCM 전제 조건 충족. Shift/Add DAG로 변환, CSE 공유.
- **PEFT 경로 (LoRA/Adapter)**: 소규모 MAC 유지, 빠른 모델 교체 대응.
- NPU 아키텍처 분해: MCM DAG → VE 실행, LoRA/Adapter → 작은 MAC TE 실행.

### TinyLoRA / 13-Parameter Reasoning

"Learning to Reason in 13 Parameters" (Meta FAIR + Cornell + CMU, arXiv 2602.04118, 2026):
- **TinyLoRA**: `W' = W + α·U` (α: scalar trainable, U: fixed random matrix) → trainable parameters = 1개
- 13개의 파라미터(26 bytes, bf16)로 Qwen2.5-8B GSM8K 91%, AIME/AMC/MATH500에서도 경쟁력 유지
- RL이 핵심: SFT는 100~1000× 더 많은 파라미터 필요 vs RL은 13파라미터로 reasoning 학습 가능
- 해석: Base model에 이미 latent reasoning circuits 존재 → TinyLoRA는 "behavior steering" 역할
- HW 함의: `base LLM(frozen SRAM) + 32 byte adapter` 구조 → reasoning upgrade = firmware patch

### AgentHub (Andrej Karpathy)

GitHub(human-centric)에서 AgentHub(agent-centric)으로의 패러다임 전환:
- **DAG 기반 commit 구조**: merge 없음, branch 없음, review 없음 → 수만 개 에이전트의 병렬 hypothesis testing
- **Autoresearch**: agent swarm이 literature scan → experiment generation → code modification → result analysis → paper draft를 자율적으로 수행
- **인간 역할 변화**: coder → architect (culture/instructions 설계자)
- NPU 연구 적용: tiling strategy, quantization, memory scheduling, compiler pass를 agent swarm이 자동 탐색

### Taalas: Model as Hardware

Taalas의 "The Path to Ubiquitous AI" 전략:
- **Hardcore Models**: 특정 AI 모델(Llama 3.1 8B) 전체를 ASIC에 hardwire → 가중치가 SRAM/DRAM이 아닌 트랜지스터+배선으로 존재
- **성능**: ~17,000 tokens/sec, GPU 대비 ≈10× inference 속도, 전력/비용 ≈10×/20× 절감 (HC1 보드 기준)
- **TSMC 파운드리** 협력: model → silicon 변환 → 제조 (2개월 사이클 목표)
- **3~6비트 양자화** 활용 가능성 (정확도/성능 트레이드오프 존재)
- **한계**: 모델 변경 시 재제작 필요, 유연성 제한

---

## Architecture [coverage: high -- 7 sources]

### Multiplierless DWT ASIC 구조

한국멀티미디어학회지 (2010, DBpia NODE01605693) 논문의 핵심 구조:

```
9/7 리프팅 DWT 계수
    ↓
Lefevre Pattern Search 알고리즘
    ↓
상수 계수 → CSD / shift+add 분해
    ↓
공통 부분식 공유 (CSE)
    ↓
Verilog HDL 구현 (0.18 μm CMOS, 200 MHz)
```

**ASIC 합성 결과**:
| 항목 | 기존 (곱셈기) | 제안 (Multiplierless) | 개선율 |
|---|---|---|---|
| 면적 | 기준 | 49% | 51% 감소 |
| 전력 | 기준 | 57% | 43% 감소 |
| 최대 지연 | 기준 | 70% | 30% 감소 |

### MCM-Based Neural Network Hardware Pipeline

```
ONNX 모델
    ↓
Constant Folding (가중치 고정 확인)
    ↓
MCM Graph Extraction
    ↓
DAG 최적화 (CSE, adder depth 최소화)
    ↓
RTL Auto-generation (Verilog emitter)
    ↓
VE(Shift/Add Engine) 실행
```

### Taalas vs GPU vs MCM ASIC 비교

| 구조 | Weight 처리 | Multiply | 메모리 |
|---|---|---|---|
| GPU | DRAM 로드 | 범용 MAC | HBM |
| 일반 NPU | SRAM 로드 | MAC array | SRAM |
| Taalas | 실리콘 고정 | 거의 없음 | On-chip |
| MCM ASIC | 실리콘 고정 | Shift/Add | On-chip |

### QuickSRNet-MCM 아키텍처 설계안

MCM-first 구조로 QuickSRNet을 재설계하는 End-to-End 파이프라인:

```
LR → MCM-ISP Front → Early MCM Blocks (주력) → Late MAC Blocks (보정) → Upscale Head (PixelShuffle)
```

- **MCM-ISP Front**:
  - (A) CCM/White-balance: 3×3 per-pixel, 상수 9개 → MCM 최적화 용이
  - (B) Denoise/Sharpen: Depthwise 3×3 × L layers, kernel codebook
  - (C) Piecewise nonlinearity: LUT (gamma, tone mapping) → Vector 엔진 처리

- **Depthwise Kernel Codebook**: 채널별 커널 `K_c ≈ B_{π(c)}, π(c) ∈ {1,..,M}` — 커널 종류를 M개로 제한해 MCM 그래프 수를 M개로 축소

- **MAC vs MCM Crossover 조건**:
  ```
  C^MCM < C^MAC
  C = C_compute + C_move + C_control
  ```
  - MCM 황금 구간: 초반 depthwise/3×3, ISP 성격, 코드북 공유 가능, 스트리밍 처리 가능 구간
  - 1×1 pointwise conv는 대부분 MAC 쪽이 유리

- **학습 레시피 (4단계)**:
  1. Stage 0: Baseline QuickSRNet 재현 (DIV2K, L1 Charbonnier loss)
  2. Stage 1: MCM-ISP Front를 float로 end-to-end 학습
  3. Stage 2: Depthwise 커널 코드북 적용 (QAT/STE)
  4. Stage 3: 보정 경로 추가 (LoRA 저랭크 보정 또는 1×1 conv)

### Domain-Adaptive MCM-ISP 구조 (논문 강화 버전)

"Learning End-to-End Deep Learning Based Image Signal Processing Pipeline" (AW-Net, 2024) 흡수:

```
RAW/LR
 → MCM-ISP Front + Early MCM Blocks
   → Domain Classifier + GRL (Gradient Reversal Layer)
 → Late MAC Blocks
 → SR Output
```

**핵심 주장**: MCM 제약(코드북/Po2)이 하드웨어 최적화일 뿐만 아니라, 카메라별 노이즈/색 응답 과적합을 억제하는 **domain generalization regularizer**로 작용한다.

### MCM 응용 분야 맵

| 분류 | 응용 영역 | MCM 적합도 | 논문/특허 가치 |
|---|---|---|---|
| A | ISP / Image Restoration | ★★★★★ | 매우 높음 |
| B | Super-Resolution / Enhancement | ★★★★★ | 매우 높음 |
| C | 멀티모달 이미지 인코더 (Conv stem, patch embed) | ★★★★ | 높음 |
| D | 멀티모달 오디오 인코더 (STFT, Mel filterbank) | ★★★★ | 높음 |

MCM은 멀티모달 시스템에서 **perception front-end**를 담당: 이미지 인코더의 Conv stem/early feature extractor, 오디오 인코더의 STFT/Mel filterbank가 교과서적 MCM 타깃.

---

## Key Findings [coverage: high -- 7 sources]

### Taalas 분석 핵심 takeaway

1. **MCM = Taalas를 이론적으로 완성하는 수단**: Taalas의 hardwired model은 inference 시 가중치가 상수 → MCM으로 곱셈기 완전 제거 가능.
2. **Coefficient Programmable MCM Fabric**: 모델 변경 문제 해결 아이디어 — shift network는 고정, sign/shift mask만 변경하는 LUT 기반 coefficient bank (FPGA와 ASIC의 중간 영역).
3. **LLM decode 관점**: KV cache만 외부 메모리, weight는 칩에 고정 → memory bandwidth = 0 (for weights). Prefill/Decode 모두 MCM이 유리.
4. **연구 제안 논문 제목**: "Hardwired Transformer via MCM-Based Multiplierless Architecture"

### Multiplierless DWT 분석 핵심 takeaway

1. **Lefevre 알고리즘**: pattern search로 상수 계수를 CSD(Canonical Signed Digit)로 분해 → 전통 곱셈기 대비 면적/전력/지연 전방위 개선.
2. **신경망 확장 흐름**: 최근(2023–2025) 논문들이 MCM 아이디어를 CNN/ViT에 적용:
   - **HaShiFlex (2025)**: 하드와이어드 레이어 방향까지 확장
   - **DenseShift (ICCV 2023)**: Po2 QAT 계열
   - **MCM-SR (TVLSI 2025)**: CNN에 MCM을 직접 적용
   - **ShiftAddViT (OpenReview 2026)**: Vision Transformer 적용
   - **Hardware-Aware Training for Multiplierless CNN (ARITH 2025)**
3. **NPU 컴파일러 MCM Pass 아이디어**: Conv/Linear에서 weight 상수 구간을 VE(ADD/SHIFT)로 매핑, TE(MAC)와 hybrid 혼합.

### Reasoning in 13M Parameters 핵심 takeaway

1. **Reasoning = training signal (not parameter scale)**: 13 parameters(26 bytes)로 GSM8K 91% — SFT 대비 RL이 결정적으로 우월.
2. **Hardware 함의**: fine-tuning이 register 수준 파라미터만 필요 → activation steering 구조 → `base LLM(frozen) + tiny adapter`가 NPU deployment 모델을 바꿀 수 있음.
3. **향후 연구 질문**: reasoning circuit의 정확한 위치(which layer? which attention head?), 1-parameter 학습 가능성, neuromodulation 스타일 훈련.

### AgentHub 핵심 takeaway

1. **DAG = 거대한 실험 그래프**: GitHub(human bandwidth ≈ 10 commits/day) → AgentHub(agent swarm ≈ 10^5 commits/day).
2. **Autoresearch = AI 연구실 자동화**: 수천 명 AI 박사 과정 학생이 24시간 연구하는 것과 동일한 효과.
3. **미해결 과제**: 결과 evaluation (어느 결과가 좋은가), hallucinated code 문제, 10^6 commits의 그래프 관리.
4. **NPU 연구 적용**: ISA design, tiling strategy, memory hierarchy를 agent swarm이 자동 탐색 → AI hardware design automation.

### AI-Assisted Research Workflow 핵심 takeaway

1. **Super-Acceleration 주장 검토**: A100→Rubin R100 5년간 에너지 효율 26.7×는 GPU 단일칩(1.6~2.2×/세대)이 아닌 시스템 스케일링(3~5×) + precision 축소(FP16→FP8→FP4) + sparse acceleration의 합산.
2. **Claude Code + Harness**: 프롬프트 4개, 20분으로 대시보드 + 논문 생성 — AI-assisted research의 시작.
3. **AI-driven architecture research**: Architecture idea → LLM simulation code → performance modeling → paper auto draft.

### 특허 전략 핵심 takeaway

내부 조직 개발자 관점에서 최적 순서:
- **실험 → 선출원 특허 → 논문** 순서가 가장 안전
- 논문이 먼저 나오면 prior art 문제 발생, 경쟁사 우회 특허 리스크
- **특허로 분리하기 좋은 내용**: MCM uEngine 마이크로아키텍처, 코드북 kernel 업데이트 규칙, 온디바이스 few-shot domain adaptation 흐름, ISP–NPU–메모리 인터페이스 구조

---

## Connections [coverage: medium -- 4 sources]

- **[[../../raw/AI-Hardware/Architecture]]** — Taalas hardwired ASIC, MCM ASIC, Coefficient Programmable MCM Fabric은 AI-Hardware Architecture의 핵심 사례
- **[[../../raw/AI-Hardware/Simulator]]** — MCM uEngine 시뮬레이션, NPU ISA에서 VE(shift/add) vs TE(MAC) 실행 분리
- **[[../../raw/GenAI/Compression]]** — TinyLoRA, LoRA, PEFT는 MCM base path와 결합 가능 (frozen W → MCM DAG, ΔW → small MAC)
- **[[../../raw/GenAI/HW-Friendly]]** — HaShiFlex, DenseShift, ShiftAddViT 등 Po2/shift-add 가중치 제약 학습 기법; QuickSRNet-MCM의 codebook QAT
- **wiki/topics/hw-friendly-model-design** — MCM-first 설계는 HW-friendly 모델 설계의 하드웨어 최적화 관점과 직접 연결
- **Groq LPU, HyperAccel LPU** — LPU 아키텍처는 MCM 기반 hardwired model과 유사한 데이터플로우 머신 개념

---

## Open Questions [coverage: medium -- 3 sources]

### MCM × LLM 하드웨어 설계

1. **LLaMA Linear layer를 실제 MCM DAG로 변환**: 토큰당 에너지 모델 수식화가 필요. MAC vs MCM crossover가 실제 LLM layer에서 어디에서 발생하는가?
2. **Coefficient Programmable MCM Fabric**: sign/shift mask만 변경하는 LUT 기반 coefficient bank의 실현 가능성과 면적/전력 트레이드오프.
3. **MCM IR 표현**: ONNX → Constant Folding → MCM Graph Extraction의 컴파일러 패스를 어떻게 공식화할 것인가?

### QuickSRNet-MCM 연구 방향

4. **Crossover 수치화**: 코드북 크기 M ∈ {4, 8, 16, 32, 64} sweep에서 PSNR/SSIM vs RelativeCost 그래프 — 실제 실험 미완료.
5. **Domain-adversarial + MCM**: MCM 제약이 실제로 domain generalization regularizer로 작용하는가? 실험으로 검증 필요.
6. **멀티모달 인코더로의 확장**: 논문 2편 또는 확장 섹션으로 MCM-based Vision/Audio Encoder 타당성 증명 필요.

### 소규모 Reasoning 연구

7. **Reasoning circuit 위치**: 어느 layer, 어느 attention head에 reasoning circuits가 존재하는가?
8. **1-parameter 학습 한계**: TinyLoRA의 최소 파라미터 하한선은 어디인가?
9. **On-device RL**: Edge AI에서 13-parameter 수준의 on-device RL이 실제로 동작하는가?

### AgentHub / 연구 자동화

10. **Agent swarm NPU 탐색**: ISA design, tiling strategy를 agent swarm이 자동 탐색하는 시스템의 evaluation 기준을 어떻게 설계할 것인가?
11. **AI-assisted architecture research 파이프라인**: Claude Code + Simulator를 연결한 architecture search AI의 구체적 구현 방안.

---

## Sources [coverage: medium -- 7 sources]

1. [[../../raw/Research/Patent-MCM/Taalas AI Infrastructure Innovation 33a6cc566b0b81378bc9f4ae1054ac16]] — Taalas "model as hardware" 전략 분석, MCM과의 연결, Hardwired Transformer 연구 제안
2. [[../../raw/Research/Patent-MCM/AgentHub and Collaboration Model Change 33a6cc566b0b815ab9faec6381cdfa23]] — AgentHub DAG 협업 구조, Autoresearch, agent swarm NPU 탐색 적용
3. [[../../raw/Research/Patent-MCM/Reasoning in 13M Parameters 33a6cc566b0b816480b1d91f9b8363c9]] — TinyLoRA/13-parameter reasoning (Meta FAIR 2026), RL vs SFT, NPU deployment 모델 변화
4. [[../../raw/Research/Patent-MCM/AI-assisted research workflow (Claude Code + Harne 33a6cc566b0b81ea9827ddba8abd5822]] — Super-Acceleration 가설 분석, Claude Code + Harness 20분 논문 생성, AI-driven architecture research
5. [[../../raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 1 of 3) 33a6cc566b0b81028b7bdfa2171ef078]] — 리프팅 DWT Multiplierless 논문 (Lefevre 알고리즘), Voronenko-Püschel MCM 형식화, AdderNet/Sarwar 비교, PEFT × MCM 결합
6. [[../../raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 2 of 3) 33a6cc566b0b81e1a9a9f08c25a9f5d0]] — QuickSRNet-MCM 설계안, CNN-ISP 계층 설계, MAC vs MCM crossover 수치화, 코드북 depthwise, QuickSRNet vs SESR 선택
7. [[../../raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 3 of 3) 33a6cc566b0b81c9a32eca2323221258]] — Domain adaptive ISP (AW-Net) 흡수 전략, 논문/특허 전략 (실험→선출원→논문), MCM 응용 분야 보고서 (ISP/SR/멀티모달 이미지·오디오 인코더)
