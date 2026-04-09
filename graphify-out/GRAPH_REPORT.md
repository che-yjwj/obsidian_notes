# Graph Report - .  (2026-04-09)

## Corpus Check
- 66 files · ~186,644 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 172 nodes · 213 edges · 18 communities detected
- Extraction: 84% EXTRACTED · 16% INFERRED · 0% AMBIGUOUS · INFERRED: 35 edges (avg confidence: 0.84)
- Token cost: 0 input · 0 output

## God Nodes (most connected - your core abstractions)
1. `Paper Reviews` - 13 edges
2. `NPU Architecture Overview` - 10 edges
3. `AI-Assisted Research Workflow` - 10 edges
4. `LLM Quantization and Compression Methods` - 9 edges
5. `Outlier Mitigation Methods Comparison` - 9 edges
6. `Wiki Index` - 8 edges
7. `Patent Research — MCM and AI Infrastructure` - 8 edges
8. `obsidian_notes` - 7 edges
9. `Vault Lint Checklist` - 7 edges
10. `NPU/LPU Simulator and Implementation Tools` - 7 edges

## Surprising Connections (you probably didn't know these)
- `obsidian_notes` --references--> `NPU Architecture Overview`  [EXTRACTED]
  README.md → wiki/AI-Hardware/NPU-Architecture.md
- `obsidian_notes` --references--> `LLM Quantization and Compression Methods`  [EXTRACTED]
  README.md → wiki/GenAI/LLM-Quantization-and-Compression.md
- `obsidian_notes` --references--> `AI-Assisted Research Workflow`  [EXTRACTED]
  README.md → wiki/Research/AI-Assisted-Research-Workflow.md
- `Wiki Index` --references--> `AI-Assisted Research Workflow`  [EXTRACTED]
  index.md → wiki/Research/AI-Assisted-Research-Workflow.md
- `Wiki Index` --references--> `Research Tooling Reviews`  [EXTRACTED]
  index.md → wiki/Research/Research-Tooling-Reviews.md

## Hyperedges (group relationships)
- **Research workflow stack** — wiki_research_ai_assisted_research_workflow, wiki_research_research_tooling_reviews, wiki_research_paper_reviews, wiki_research_patent_mcm [EXTRACTED 0.95]
- **Graphify review stack** — readme_obsidian_notes, index_wiki_catalog, wiki_misc_vault_lint_checklist, wiki_misc_graphify_post_update_review_template [EXTRACTED 0.95]
- **Outlier mitigation stack** — wiki_genai_outlier_mitigation_methods_comparison, concept_smoothquant, concept_quarot, concept_spinquant, concept_ocean_coordinate_aware_compression [EXTRACTED 0.95]

## Communities

### Community 0 - "Community 0"
Cohesion: 0.15
Nodes (19): Ensemble prediction for chaotic systems, Search-verify-refine loop, PaperBanana multi-agent visualization, Executable paper / paper debugging, Research tooling / productivity loop, STEM embedding modules, Why chaos prediction moves to distributions, Why Dr. Zero removes training dependence (+11 more)

### Community 1 - "Community 1"
Cohesion: 0.12
Nodes (18): AMBA bus recommendation, Google Coral Edge TPU, HyperAccel LPU, Memory hierarchy in AI accelerators, Meta MTIA vs Google TPU, NPU architecture overview, NPU supernode, SoC specification negotiation phrases (+10 more)

### Community 2 - "Community 2"
Cohesion: 0.18
Nodes (15): AMD Versal ACAP, Coral to mobile AP NPU evolution, Coral-based multi-MatMul cluster, Why Coral does not scale to mobile AP as-is, Coral NPU full-stack platform, AdderNet, Multiplier-less Artificial Neurons, Multiplierless lifting DWT design (+7 more)

### Community 3 - "Community 3"
Cohesion: 0.23
Nodes (14): Graphify post-update review, Content-oriented index catalog, Persistent compounding wiki, Append-only activity log, Raw / wiki / schema layers, Vault root navigation hub, Vault lint / operational consistency, Wiki Index (+6 more)

### Community 4 - "Community 4"
Cohesion: 0.19
Nodes (14): DC-LLM, LLM quantization/compression stack, Microscaling vs mixed precision, OCEAN coordinate-aware compression, OliVe outlier-victim pair, Outlier mitigation methods comparison, QuaRot, SmoothQuant (+6 more)

### Community 5 - "Community 5"
Cohesion: 0.19
Nodes (14): Attention Sink, AWS Inferentia, AWS Trainium, Google TPU, Horner's Method, Why LPDDR fits decode-heavy inference, HyperAccel LPU card, AI memory hierarchy (+6 more)

### Community 6 - "Community 6"
Cohesion: 0.31
Nodes (9): Coordinate-system view of quantization, Functional + residual decomposition, SRHT / Hadamard transform, Inner-product preservation, Manifold-aligned coordinates, PolarQuant, QJL residual correction, Random rotation (+1 more)

### Community 7 - "Community 7"
Cohesion: 0.25
Nodes (9): Gemma 3n, 5-to-1 interleaved attention, KV Cache Sharing, MatFormer, Learnable memory hierarchy, MSA, RAG, Router Key (+1 more)

### Community 8 - "Community 8"
Cohesion: 0.32
Nodes (8): AgentHub collaboration model change, AI-assisted research workflow, Multiplierless / MCM shift-add design, Reasoning in 13M parameters, Model-as-computer ASIC, Why Taalas hardwires the model into silicon, Taalas AI 인프라 혁신, Patent Research — MCM and AI Infrastructure

### Community 9 - "Community 9"
Cohesion: 0.36
Nodes (8): AgentHub, Autoresearch, AI-assisted research workflow, AI-assisted research automation, RL steering of latent reasoning circuits, TinyLoRA reasoning adapter, Learning to Reason in 13M Parameters, AI-Assisted Research Workflow

### Community 10 - "Community 10"
Cohesion: 0.32
Nodes (8): Adaptive basis selection, Bayesian optimization DSE, DC-LLM, Dynamic linear combination, Explained Energy Ratio, LFSR-based basis generation, Systolic array, Weight generator

### Community 11 - "Community 11"
Cohesion: 0.32
Nodes (8): Base / residual / exception streams, Dead-zone, Entropy coding, OCEAN, OliVe, Outlier-aware quantization, Outlier-victim pair quantization, Uniform integer GEMM

### Community 12 - "Community 12"
Cohesion: 0.4
Nodes (6): Decode-centric pipeline, ESL token-level pipeline link, Groq LPU, HyperAccel LPU, Pipeline parallel, Static scheduling

### Community 13 - "Community 13"
Cohesion: 0.4
Nodes (5): Gemma 3n MatFormer/PLE, Latent Action World Model, MSA and RAG, Normalization-free Transformers, HW-Friendly GenAI Model Design

### Community 14 - "Community 14"
Cohesion: 0.5
Nodes (5): MCM-first CNN-ISP block, Domain-invariant MCM-ISP via adversarial training, QuickSRNet target, Why the CNN-ISP block is needed, Why domain adaptation strengthens MCM-ISP

### Community 15 - "Community 15"
Cohesion: 0.5
Nodes (5): DyT, LayerNorm, Residual scaling, Stronger Normalization-Free Transformers, Variance control

### Community 16 - "Community 16"
Cohesion: 0.5
Nodes (4): Action-free data, Latent action, Latent Action World Model, World model

### Community 17 - "Community 17"
Cohesion: 1.0
Nodes (2): 4-week practical usage guide, SoC negotiation technical English

## Knowledge Gaps
- **69 isolated node(s):** `AgentHub collaboration model change`, `Reasoning in 13M parameters`, `AI-assisted research workflow`, `HyperAccel LPU`, `Google Coral Edge TPU` (+64 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **Thin community `Community 17`** (2 nodes): `4-week practical usage guide`, `SoC negotiation technical English`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.

## Suggested Questions
_Questions this graph is uniquely positioned to answer:_

- **Why does `AI-Assisted Research Workflow` connect `Community 9` to `Community 0`, `Community 8`, `Community 3`?**
  _High betweenness centrality (0.105) - this node is a cross-community bridge._
- **Why does `obsidian_notes` connect `Community 3` to `Community 1`, `Community 4`, `Community 9`?**
  _High betweenness centrality (0.078) - this node is a cross-community bridge._
- **Why does `Wiki Index` connect `Community 3` to `Community 0`, `Community 9`, `Community 4`?**
  _High betweenness centrality (0.064) - this node is a cross-community bridge._
- **What connects `AgentHub collaboration model change`, `Reasoning in 13M parameters`, `AI-assisted research workflow` to the rest of the system?**
  _69 weakly-connected nodes found - possible documentation gaps or missing edges._
- **Should `Community 1` be split into smaller, more focused modules?**
  _Cohesion score 0.12 - nodes in this community are weakly interconnected._