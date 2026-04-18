# Graph Report - obsidian_notes  (2026-04-18)

## Corpus Check
- 190 files · ~292,955 words
- Verdict: corpus is large enough that graph structure adds value.

## Summary
- 719 nodes · 1286 edges · 63 communities detected
- Extraction: 89% EXTRACTED · 11% INFERRED · 0% AMBIGUOUS · INFERRED: 145 edges (avg confidence: 0.85)
- Token cost: 0 input · 0 output

## God Nodes (most connected - your core abstractions)
1. `LLM Quantization & Compression` - 34 edges
2. `NPU Architecture` - 23 edges
3. `Patent & MCM Research` - 21 edges
4. `RISC-V NPU SoC Simulator` - 20 edges
5. `Flash-Resident LLM and HBF for Edge Inference` - 20 edges
6. `Paper Reviews` - 19 edges
7. `NPU/LPU Simulator and Implementation Tools` - 19 edges
8. `NPU Simulator & Compiler` - 19 edges
9. `From Minimal NumPy LLaMA to Tile-based NPU Modeling` - 19 edges
10. `Wiki Index` - 17 edges

## Surprising Connections (you probably didn't know these)
- `Multiplierless Multiple Constant Multiplication` --semantically_similar_to--> `Xilinx FINN`  [INFERRED] [semantically similar]
  raw/Research/Patent-MCM/Multiplierless Design Paper Analysis/Multiplierless Design Paper Analysis (Part 1 of 3) 33a6cc566b0b81028b7bdfa2171ef078.md → wiki/AI-Hardware/NPU-Architecture.md
- `SpinQuant` --semantically_similar_to--> `SpinQuant as Partial Manifold Alignment`  [INFERRED] [semantically similar]
  wiki/topics/llm-quantization-compression.md → raw/Research/operator-coordinate-compression/docs/research_direction_review.md
- `Google TPU Sparse Core` --semantically_similar_to--> `HyperAccel LPU card`  [INFERRED] [semantically similar]
  wiki/AI-Hardware/NPU-Architecture.md → raw/AI-Hardware/Simulator/HyperAccel LPU Explanation/HyperAccel LPU Explanation (Part 2 of 3) 33a6cc566b0b8184b3e7c80f64b99235.md
- `Why LPDDR fits decode-heavy inference` --conceptually_related_to--> `Flash-Resident LLM and HBF for Edge Inference`  [INFERRED]
  raw/AI-Hardware/Simulator/HyperAccel LPU Explanation/HyperAccel LPU Explanation (Part 2 of 3) 33a6cc566b0b8184b3e7c80f64b99235.md → wiki/AI-Hardware/Flash-Resident-LLM-and-HBF-for-Edge-Inference.md
- `Tesla AI5/AI6 roadmap` --conceptually_related_to--> `Tesla Mixed-Precision Bridge`  [INFERRED]
  raw/AI-Hardware/Architecture/Tesla AI Chip Roadmap 33a6cc566b0b8129a0f3eef1ddc4dfa3.md → wiki/topics/llm-quantization-compression.md

## Hyperedges (group relationships)
- **Research workflow stack** — wiki_research_ai_assisted_research_workflow, wiki_research_research_tooling_reviews, wiki_research_paper_reviews, wiki_research_patent_mcm [EXTRACTED 0.95]
- **Graphify review stack** — readme_obsidian_notes, index_wiki_catalog, wiki_misc_vault_lint_checklist, wiki_misc_graphify_post_update_review_template [EXTRACTED 0.95]
- **Outlier mitigation stack** — wiki_genai_outlier_mitigation_methods_comparison, concept_smoothquant, concept_quarot, concept_spinquant, concept_ocean_coordinate_aware_compression [EXTRACTED 0.95]
- **AI Hardware Memory and Compute Stack** — npu_architecture_overview, memory_hierarchy_in_ai_accelerators, npu_simulator_and_implementation_tools, memory_bandwidth_bottleneck [INFERRED 0.90]
- **Research Automation Cluster** — ai_assisted_research_workflow, research_tooling_reviews, paper_reviews, patent_mcm [INFERRED 0.88]
- **GenAI Efficiency Cluster** — llm_quantization_and_compression_methods, outlier_mitigation_methods_comparison, hw_friendly_genai_model_design [INFERRED 0.89]
- **Deterministic Inference Cluster** — static_scheduling_determinism, prefill_decode_duality, npu_simulator_compiler_topic, riscv_npu_soc_sim_topic [INFERRED 0.90]
- **Operator Coordinate Compression Theory Stack** — operator_coordinate_compression_overview, operator_view, coordinate_relative_outliers, rotation_vs_alignment, manifold_alignment, turboquant_architecture [EXTRACTED 0.95]
- **Operator Coordinate Compression Writing and Validation Stack** — paper_draft, related_work, invention_summary, claim_candidates, toy_basis_vs_mlp_spec, tinyllama_validation_spec [EXTRACTED 0.95]
- **Quantization Rotation Stack** — smoothquant, gptq, quarot, spinquant, kurtail, turboquant_method, qjl, polarquant [INFERRED 0.86]
- **NPU-IR Core Model** — npu_program, hw_profile, tensor, memref, npu_graph, tag_sync, compute_tile [EXTRACTED 1.00]
- **Offline Compiler Pass Graph** — ir_builder_onnx_to_npu_ir, tile_graph, spm_allocation, schedule_dag, cmdq_json [EXTRACTED 1.00]
- **Simulator Core Runtime Stack** — control_fsm_issue_policy, dma_latency_bandwidth_model, te_tile_latency_model, ve_tile_latency_model, global_cycle_loop, trace_json [EXTRACTED 1.00]
- **Trace Visualization Suite** — gantt_timeline, bandwidth_heatmap, engine_utilization_dashboard, quantization_impact_plot [EXTRACTED 1.00]
- **Quantization Pipeline** — qconfig, npu_ir, tensor_metadata, bitwidth_memory_mapping, kv_cache_quantization [INFERRED 0.84]
- **LLM Execution Pipeline** — prefill_decode_semantics, kv_cache, cmdq, trace_event_schema [INFERRED 0.80]
- **System Architecture Stack** — system_architecture_overview_doc, system_architecture_doc, dataflow_overview_doc, memory_noc_overview_doc, module_responsibilities_doc [EXTRACTED 0.95]
- **Timing and Memory Stack** — bus_and_noc_model_spec, dma_timing_spec_doc, te_timing_spec_doc, ve_timing_spec_doc, spm_model_spec_doc [EXTRACTED 0.95]
- **Tile Semantics Stack** — tile_semantics_spec_doc, tile_contract_spec_doc, stb_adoption_rfc_doc, kv_cache_semantics_spec_doc [EXTRACTED 0.95]
- **Process Workflow Stack** — sdd_devflow_overview_doc, doc_study_milestone_plan_doc, vibe_coding_sdd_guide_doc, contribution_and_review_guide_doc, naming_convention_doc, doc_improvement_tasks_doc, tile_based_integration_mapping_doc [EXTRACTED 0.95]
- **Project Roadmap Architecture Stack** — project_roadmap_workload_loader, project_roadmap_compiler_pipeline, project_roadmap_cycle_based_simulator, project_roadmap_trace_visualization, project_roadmap_dse_scenario_engine [EXTRACTED 1.00]
- **SDD Workflow Sequence** — spec_workflow_spec_update, spec_workflow_design_update, spec_workflow_test_doc_update, spec_workflow_implementation, spec_workflow_review_merge [EXTRACTED 1.00]
- **Roadmap Stage Chain** — project_roadmap_stage_a_foundation, project_roadmap_stage_b_timing_engine_modeling, project_roadmap_stage_c_full_operator_support, project_roadmap_stage_d_llm_specialization, project_roadmap_stage_e_dse_platform [EXTRACTED 1.00]
- **MCM Research Family** — multiple_constant_multiplication, multiplierless_dwt, mcm_based_neural_network_hardware_pipeline, quicksrnet_mcm, domain_adaptive_mcm_isp [INFERRED 0.82]
- **Wiki Compiler Topic Expansion 2026-04-14** — operator_coordinate_compression_topic, trace_visualization_topic, npu_doc_process_topic [EXTRACTED 1.00]
- **Wiki Compiler Concept Expansion 2026-04-14** — tile_semantics_contract, mixed_precision_policy, kv_cache_dram_residency, trace_first_design [EXTRACTED 1.00]
- **Flash-Resident Edge Inference Stack** — wiki_ai_hardware_flash_resident_llm_and_hbf_for_edge_inference, wiki_ai_hardware_memory_hierarchy_in_ai_accelerators, kv_cache_dram_residency, prefill_decode_duality, llm_quantization_compression [INFERRED 0.92]
- **Helios Edge Platform Strategy Stack** — project_helios_edge_physical_ai_custom_soc_platform_strategy, edge_physical_ai_platform_provider, fabric_preset_strategy, hierarchical_multi_noc_architecture, helios_differentiation_layer [EXTRACTED 0.98]
- **RISCV_NPU_SoC_SIM Split Topics 2026-04-16** — riscv_npu_soc_sim_topic, npu_architecture_spec_topic, npu_timing_memory_model_topic, simulation_validation_topic, trace_visualization_topic, npu_doc_process_topic [EXTRACTED 1.00]
- **GenAI Deep-Dive Canonical Links 2026-04-17** — wiki_genai_hw_friendly_model_design, wiki_genai_llm_quantization_and_compression, wiki_genai_ocean_compression_deep_dive, wiki_genai_outlier_mitigation_methods_comparison, hw_friendly_model_design, llm_quantization_compression, mixed_precision_policy, kv_cache_dram_residency, prefill_decode_duality, memory_bandwidth_bottleneck [EXTRACTED 1.00]
- **Research Deep-Dive Canonical Links 2026-04-17** — wiki_research_ai_assisted_research_workflow, wiki_research_paper_reviews, wiki_research_patent_mcm, wiki_research_research_tooling_reviews, paper_reviews, patent_mcm_doc, static_scheduling_determinism [EXTRACTED 1.00]
- **Execution Spec Split Topics 2026-04-17** — riscv_npu_soc_sim_topic, npu_architecture_spec_topic, ir_cmdq_contract_topic, tile_semantics_quantization_topic, npu_timing_memory_model_topic, simulation_validation_topic, trace_visualization_topic, npu_doc_process_topic [EXTRACTED 1.00]
- **Operator-Coordinate Compression Canonical Links 2026-04-17** — operator_coordinate_compression_topic, llm_quantization_compression, mixed_precision_policy, memory_bandwidth_bottleneck, hw_friendly_model_design [EXTRACTED 1.00]
- **Overview Layer Absorbed Into Execution Spec 2026-04-17** — system_architecture_doc, dataflow_overview_doc, module_responsibilities_doc, npu_architecture_spec_topic [EXTRACTED 1.00]
- **Timing Specs Absorbed Into Timing Topic 2026-04-17** — te_timing_spec_doc, ve_timing_spec_doc, dma_timing_spec_doc, spm_model_spec_doc, bus_and_noc_model_spec, npu_timing_memory_model_topic [EXTRACTED 1.00]
- **Trace Specs Absorbed Into Trace Topic 2026-04-17** — trace_format_spec_doc, gantt_timeline_spec_doc, bandwidth_heatmap_spec_doc, visualization_requirements_doc, trace_visualization_topic [EXTRACTED 1.00]
- **Validation Specs Absorbed Into Validation Topic 2026-04-17** — unit_test_spec_doc, integration_test_spec_doc, test_plan_doc, performance_validation_protocol_doc, golden_trace_examples_doc, simulation_validation_topic [EXTRACTED 1.00]
- **Example Ownership Split 2026-04-17** — tutorial_minimal_llama_to_tile_npu_doc, trace_visualization_topic, pytorchsim_npu_ir_examples_doc, simulation_validation_topic [EXTRACTED 1.00]
- **Wiki Topic Watchlist 2026-04-17** — soc_spec_english, trace_visualization_topic, simulation_validation_topic, project_helios_edge_physical_ai_custom_soc_platform_strategy [EXTRACTED 1.00]
- **Deep-Dive Canonical Boundaries 2026-04-17** — wiki_ai_hardware_npu_architecture, npu_architecture, wiki_ai_hardware_simulator_and_implementation_tools, npu_simulator_compiler_topic [EXTRACTED 1.00]
- **Memory Deep-Dive Canonical Boundaries 2026-04-17** — wiki_ai_hardware_memory_hierarchy_in_ai_accelerators, npu_architecture, wiki_ai_hardware_flash_resident_llm_and_hbf_for_edge_inference, hw_friendly_model_design, kv_cache_dram_residency [EXTRACTED 1.00]
- **Helios Strategy Extraction Gate 2026-04-18** — project_helios_edge_physical_ai_custom_soc_platform_strategy, npu_architecture, npu_simulator_compiler_topic, fabric_preset_strategy, hierarchical_multi_noc_architecture, helios_differentiation_layer [EXTRACTED 1.00]
- **Operator-Coordinate Split Gate 2026-04-18** — operator_coordinate_compression_topic, manifold_alignment, turboquant_architecture, invention_summary, claim_candidates, tinyllama_validation_spec [EXTRACTED 1.00]
- **Research Review Workflow Boundary 2026-04-18** — paper_reviews, wiki_research_paper_reviews, wiki_research_ai_assisted_research_workflow, wiki_research_research_tooling_reviews [EXTRACTED 1.00]
- **Simulator Survey Canonical Boundary 2026-04-18** — npu_simulator_compiler_topic, wiki_ai_hardware_simulator_and_implementation_tools, supernode, hyperaccel_lpu, google_coral_npu, meta_mtia [EXTRACTED 1.00]
- **Architecture Canonical Boundary 2026-04-18** — npu_architecture, wiki_ai_hardware_npu_architecture, wiki_ai_hardware_memory_hierarchy_in_ai_accelerators, project_helios_edge_physical_ai_custom_soc_platform_strategy, memory_bandwidth_bottleneck, static_scheduling_determinism [EXTRACTED 1.00]
- **Compression Canonical Boundary 2026-04-18** — llm_quantization_compression, wiki_genai_llm_quantization_and_compression, wiki_genai_ocean_compression_deep_dive, wiki_genai_outlier_mitigation_methods_comparison, operator_coordinate_compression_topic, mixed_precision_policy, memory_bandwidth_bottleneck [EXTRACTED 1.00]

## Communities

### Community 0 - "LLM Quantization & Compression / Paper Draft / TurboQuant Architecture Note"
Cohesion: 0.07
Nodes (70): Blockwise Axis Alignment, Bits per Channel, Claim Candidates, Concentration, Coordinate-Relative Outliers, Coordinate-system view of quantization, Effective Rank, Entropy Coding for Functional Decomposition (+62 more)

### Community 1 - "RISC-V NPU SoC Simulator / NPU/LPU Simulator and Implementation Tools / NPU Simulator & Compiler"
Cohesion: 0.07
Nodes (68): AgentHub Collaboration Graph, AI-Assisted Research Workflow, AMBA Bus Recommendation, AMD Versal ACAP, Architecture Scope Alignment, Claude Code + Harness Pipeline, Google Coral NPU, Decode-centric pipeline (+60 more)

### Community 2 - "From Minimal NumPy LLaMA to Tile-based NPU Modeling / CMDQ Format Specification / CMDQ Overview"
Cohesion: 0.07
Nodes (64): Bitwidth-to-Memory Mapping, Bitwidth & Memory Mapping Specification, Capacity / Traffic Tradeoff, Command Queue (CMDQ), CMDQ Format Specification, CMDQ Overview, Why Mixed Precision LLM Support, Why Static Scheduling (+56 more)

### Community 3 - "Paper Reviews / Paper Reviews / AI-Assisted Research Workflow"
Cohesion: 0.05
Nodes (55): AgentHub, Autoresearch, AI-assisted research workflow, Chaos Theory and Prediction, Ensemble prediction for chaotic systems, Search-verify-refine loop, Graphify post-update review, Content-oriented index catalog (+47 more)

### Community 4 - "Cycle Loop Design / Integration Test Specification / Offline Compiler Design"
Cohesion: 0.07
Nodes (54): CMDQ Generator Design, CMDQ JSON, Control FSM Design, Control FSM Issue Policy, Cycle Loop Design, Cycle Loop Determinism, DMA Engine Design, DMA Latency and Bandwidth Model (+46 more)

### Community 5 - "NPU Architecture / Flash-Resident LLM and HBF for Edge Inference / NPU Architecture Overview"
Cohesion: 0.05
Nodes (50): Architecture Diary Format, Assumptions and Constraints, Cache Hybrid, AMBA bus recommendation, Google Coral Edge TPU, HyperAccel LPU, Memory hierarchy in AI accelerators, Meta MTIA vs Google TPU (+42 more)

### Community 6 - "Bus & NoC Timing Specification / KV Cache Semantics Specification / SPM Model Specification"
Cohesion: 0.07
Nodes (50): Allocate / Free Lifecycle, DMA > TE > VE Arbitration Priority, Architecture Semantics Spec Index, Back-Pressure Events, Burst Latency Model, Bus & NoC Timing Specification, Bus / NoC, CMDQ Generator (+42 more)

### Community 7 - "Patent & MCM Research / Patent Research — MCM and AI Infrastructure / Multiple Constant Multiplication"
Cohesion: 0.09
Nodes (38): A-graph, A-operation, AgentHub, AI-Hardware Architecture, AI-Hardware Simulator, Coefficient Programmable MCM Fabric, AgentHub collaboration model change, AI-assisted research workflow (+30 more)

### Community 8 - "HW-Friendly Model Design / HW-Friendly GenAI Model Design / HW-Friendly GenAI Model Design"
Cohesion: 0.07
Nodes (34): Action-free data, Gemma 3n MatFormer/PLE, Latent Action World Model, MSA and RAG, Normalization-free Transformers, Conditional Parameter Loading, Dynamic Control Flow Hurts NPU Deployment, Dynamic Transformer (+26 more)

### Community 9 - "System Architecture Specification / Documentation Improvement Tasks / SDD Devflow Overview"
Cohesion: 0.12
Nodes (27): Contribution & Review Guide, Documentation Improvement Tasks, Documentation Study Milestone Plan, docs/README_SPEC.md, Documentation Improvement Backlog, Documentation Review Summary, Host CPU Control Plane, Milestone Plan (+19 more)

### Community 10 - "Trace Format Specification / Golden Trace Examples / Visualization Requirements"
Cohesion: 0.11
Nodes (24): Bandwidth Heatmap, Bandwidth Heatmap Specification, Bandwidth Samples, CMD_EVENT, ENGINE_EVENT, Engine Utilization Dashboard, Gantt Timeline, Gantt Timeline Specification (+16 more)

### Community 11 - "Stage A - Foundation / Phase 2 Detail Buildup / Cycle-Based Simulator"
Cohesion: 0.1
Nodes (23): docs/design/control_fsm_design.md, docs/design/cycle_loop_design.md, docs/design/npu_simulator_core_design.md, docs/design/te_engine_design.md, docs/design/visualizer_design.md, docs/spec/trace/bandwidth_heatmap_spec.md, docs/spec/trace/gantt_timeline_spec.md, docs/spec/trace/visualization_requirements.md (+15 more)

### Community 12 - "Compiler Pipeline / Phase 1 Pipeline Alignment / docs/spec/isa/cmdq_format_spec.md"
Cohesion: 0.11
Nodes (22): docs/design/cmdq_generator_design.md, docs/design/ir_builder_design.md, docs/design/spm_allocator_design.md, docs/design/static_scheduler_design.md, docs/design/tiling_planner_design.md, docs/process/naming_convention.md, docs/spec/ir/npu_ir_spec.md, docs/spec/isa/cmdq_format_spec.md (+14 more)

### Community 13 - "LLM Quantization and Compression Methods / Outlier Mitigation Methods Comparison / Mixed Precision as a System Policy"
Cohesion: 0.19
Nodes (15): DC-LLM, LLM quantization/compression stack, Microscaling vs mixed precision, OCEAN coordinate-aware compression, OliVe outlier-victim pair, Outlier mitigation methods comparison, QuaRot, SmoothQuant (+7 more)

### Community 14 - "AI memory hierarchy / Tesla Mixed-Precision Bridge / HyperAccel LPU card"
Cohesion: 0.19
Nodes (14): Attention Sink, AWS Inferentia, AWS Trainium, Google TPU, Horner's Method, Why LPDDR fits decode-heavy inference, HyperAccel LPU card, AI memory hierarchy (+6 more)

### Community 15 - "Prior Art Mapping / Operator-Coordinate Compression Framework / Low-Rank Limitations"
Cohesion: 0.15
Nodes (13): Low-Rank Limitations, Low-Rank Structural Approximation, Manifold Gap Rationale, Novelty Claims, Operator-Coordinate Compression Framework, Outlier-Aware Limitations, Outlier-Aware Mixed Precision Methods, Prior Art Category Taxonomy (+5 more)

### Community 16 - "Versioning & Changelog Guide / Spec-Code Version Mapping / Breaking Change Recording"
Cohesion: 0.23
Nodes (12): Breaking Change Recording, Changelog, Changelog Automation, MAJOR.MINOR.PATCH, README.md, Semantic Versioning, Simulator Version, Spec-Code Version Mapping (+4 more)

### Community 17 - "Entropy Coding / OCEAN / Outlier-victim pair quantization"
Cohesion: 0.28
Nodes (9): Base / residual / exception streams, Dead-zone, Entropy Coding, Entropy Coding Requires Distribution Shaping, OCEAN, OliVe, Outlier-aware quantization, Outlier-victim pair quantization (+1 more)

### Community 18 - "DC-LLM / Adaptive basis selection / Dynamic linear combination"
Cohesion: 0.32
Nodes (8): Adaptive basis selection, Bayesian optimization DSE, DC-LLM, Dynamic linear combination, Explained Energy Ratio, LFSR-based basis generation, Systolic array, Weight generator

### Community 19 - "Multiplierless Multiple Constant Multiplication / AdderNet / Multiplier-less Artificial Neurons"
Cohesion: 0.5
Nodes (5): AdderNet, Multiplier-less Artificial Neurons, Multiplierless lifting DWT design, Multiplierless Multiple Constant Multiplication, Pattern-search Lefevre algorithm

### Community 20 - "MCM-first CNN-ISP block / Domain-invariant MCM-ISP via adversarial training / QuickSRNet target"
Cohesion: 0.5
Nodes (5): MCM-first CNN-ISP block, Domain-invariant MCM-ISP via adversarial training, QuickSRNet target, Why the CNN-ISP block is needed, Why domain adaptation strengthens MCM-ISP

### Community 21 - "Stronger Normalization-Free Transformers / Residual scaling / Variance control"
Cohesion: 0.5
Nodes (5): DyT, LayerNorm, Residual scaling, Stronger Normalization-Free Transformers, Variance control

### Community 22 - "docs/spec/quantization/quantization_model_overview.md / docs/spec/quantization/bitwidth_memory_mapping.md / docs/spec/ir/quantization_ir_extension.md"
Cohesion: 0.4
Nodes (5): docs/spec/ir/quantization_ir_extension.md, docs/spec/quantization/bitwidth_memory_mapping.md, docs/spec/quantization/kv_cache_quantization_spec.md, docs/spec/quantization/mixed_precision_policy.md, docs/spec/quantization/quantization_model_overview.md

### Community 23 - "docs/spec/trace/trace_format_spec.md / docs/test/golden_trace_examples.md / TraceCollector"
Cohesion: 0.67
Nodes (3): docs/spec/trace/trace_format_spec.md, docs/test/golden_trace_examples.md, TraceCollector

### Community 24 - "4-week practical usage guide / SoC negotiation technical English"
Cohesion: 1.0
Nodes (2): 4-week practical usage guide, SoC negotiation technical English

### Community 25 - "Phase 1 - MatMul-only End-to-End / Vertical-First"
Cohesion: 1.0
Nodes (2): Phase 1 - MatMul-only End-to-End, Vertical-First

### Community 26 - "Document -> Design -> Code Flow / Document First"
Cohesion: 1.0
Nodes (2): Document -> Design -> Code Flow, Document First

### Community 27 - "Test-First for Core Logic / Test Accompanied"
Cohesion: 1.0
Nodes (2): Test-First for Core Logic, Test Accompanied

### Community 28 - "docs/process/contribution_and_review_guide.md / Reviewability"
Cohesion: 1.0
Nodes (2): docs/process/contribution_and_review_guide.md, Reviewability

### Community 29 - "Phase 0 Overview Enhancement / Skeleton-Level Docs Issue"
Cohesion: 1.0
Nodes (2): Phase 0 Overview Enhancement, Skeleton-Level Docs Issue

### Community 30 - "doc_status.py"
Cohesion: 1.0
Nodes (0): 

### Community 31 - "Q/K/V Projection"
Cohesion: 1.0
Nodes (1): Q/K/V Projection

### Community 32 - "Softmax Tile"
Cohesion: 1.0
Nodes (1): Softmax Tile

### Community 33 - "LLM Workload Patterns"
Cohesion: 1.0
Nodes (1): LLM Workload Patterns

### Community 34 - "Cycle-Based Simulation"
Cohesion: 1.0
Nodes (1): Cycle-Based Simulation

### Community 35 - "Vibe Coding Workflow"
Cohesion: 1.0
Nodes (1): Vibe Coding Workflow

### Community 36 - "docs/spec/ir/tensor_metadata_spec.md"
Cohesion: 1.0
Nodes (1): docs/spec/ir/tensor_metadata_spec.md

### Community 37 - "docs/spec/timing/te_timing_spec.md"
Cohesion: 1.0
Nodes (1): docs/spec/timing/te_timing_spec.md

### Community 38 - "docs/spec/timing/ve_timing_spec.md"
Cohesion: 1.0
Nodes (1): docs/spec/timing/ve_timing_spec.md

### Community 39 - "docs/spec/timing/spm_model_spec.md"
Cohesion: 1.0
Nodes (1): docs/spec/timing/spm_model_spec.md

### Community 40 - "docs/design/dma_engine_design.md"
Cohesion: 1.0
Nodes (1): docs/design/dma_engine_design.md

### Community 41 - "docs/design/offline_compiler_design.md"
Cohesion: 1.0
Nodes (1): docs/design/offline_compiler_design.md

### Community 42 - "docs/test/unit_test_spec.md"
Cohesion: 1.0
Nodes (1): docs/test/unit_test_spec.md

### Community 43 - "Stage B - NPU Timing & Engine Modeling"
Cohesion: 1.0
Nodes (1): Stage B - NPU Timing & Engine Modeling

### Community 44 - "Stage C - Full Operator Support"
Cohesion: 1.0
Nodes (1): Stage C - Full Operator Support

### Community 45 - "Stage D - LLM Specialization"
Cohesion: 1.0
Nodes (1): Stage D - LLM Specialization

### Community 46 - "Stage E - DSE Platform"
Cohesion: 1.0
Nodes (1): Stage E - DSE Platform

### Community 47 - "Modular & Replaceable"
Cohesion: 1.0
Nodes (1): Modular & Replaceable

### Community 48 - "Trace-First Design"
Cohesion: 1.0
Nodes (1): Trace-First Design

### Community 49 - "End-to-End AI Hardware Simulation Platform"
Cohesion: 1.0
Nodes (1): End-to-End AI Hardware Simulation Platform

### Community 50 - "Skeleton-First SDD Implementation Roadmap"
Cohesion: 1.0
Nodes (1): Skeleton-First SDD Implementation Roadmap

### Community 51 - "Phase 0 - Project Code Skeleton"
Cohesion: 1.0
Nodes (1): Phase 0 - Project Code Skeleton

### Community 52 - "Phase 2 - Timing/Memory/Engine Realization"
Cohesion: 1.0
Nodes (1): Phase 2 - Timing/Memory/Engine Realization

### Community 53 - "Single Source of Truth"
Cohesion: 1.0
Nodes (1): Single Source of Truth

### Community 54 - "Spec Update"
Cohesion: 1.0
Nodes (1): Spec Update

### Community 55 - "Design Update"
Cohesion: 1.0
Nodes (1): Design Update

### Community 56 - "Test Doc Update"
Cohesion: 1.0
Nodes (1): Test Doc Update

### Community 57 - "Implementation"
Cohesion: 1.0
Nodes (1): Implementation

### Community 58 - "Review & Merge"
Cohesion: 1.0
Nodes (1): Review & Merge

### Community 59 - "PR/Issue Templates"
Cohesion: 1.0
Nodes (1): PR/Issue Templates

### Community 60 - "CI Integration"
Cohesion: 1.0
Nodes (1): CI Integration

### Community 61 - "Example Flows"
Cohesion: 1.0
Nodes (1): Example Flows

### Community 62 - "Global Integrated Diagram"
Cohesion: 1.0
Nodes (1): Global Integrated Diagram

## Knowledge Gaps
- **297 isolated node(s):** `AgentHub collaboration model change`, `Reasoning in 13M parameters`, `AI-assisted research workflow`, `HyperAccel LPU`, `Google Coral Edge TPU` (+292 more)
  These have ≤1 connection - possible missing edges or undocumented components.
- **Thin community `4-week practical usage guide / SoC negotiation technical English`** (2 nodes): `4-week practical usage guide`, `SoC negotiation technical English`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Phase 1 - MatMul-only End-to-End / Vertical-First`** (2 nodes): `Phase 1 - MatMul-only End-to-End`, `Vertical-First`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Document -> Design -> Code Flow / Document First`** (2 nodes): `Document -> Design -> Code Flow`, `Document First`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Test-First for Core Logic / Test Accompanied`** (2 nodes): `Test-First for Core Logic`, `Test Accompanied`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `docs/process/contribution_and_review_guide.md / Reviewability`** (2 nodes): `docs/process/contribution_and_review_guide.md`, `Reviewability`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Phase 0 Overview Enhancement / Skeleton-Level Docs Issue`** (2 nodes): `Phase 0 Overview Enhancement`, `Skeleton-Level Docs Issue`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `doc_status.py`** (1 nodes): `doc_status.py`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Q/K/V Projection`** (1 nodes): `Q/K/V Projection`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Softmax Tile`** (1 nodes): `Softmax Tile`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `LLM Workload Patterns`** (1 nodes): `LLM Workload Patterns`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Cycle-Based Simulation`** (1 nodes): `Cycle-Based Simulation`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Vibe Coding Workflow`** (1 nodes): `Vibe Coding Workflow`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `docs/spec/ir/tensor_metadata_spec.md`** (1 nodes): `docs/spec/ir/tensor_metadata_spec.md`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `docs/spec/timing/te_timing_spec.md`** (1 nodes): `docs/spec/timing/te_timing_spec.md`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `docs/spec/timing/ve_timing_spec.md`** (1 nodes): `docs/spec/timing/ve_timing_spec.md`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `docs/spec/timing/spm_model_spec.md`** (1 nodes): `docs/spec/timing/spm_model_spec.md`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `docs/design/dma_engine_design.md`** (1 nodes): `docs/design/dma_engine_design.md`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `docs/design/offline_compiler_design.md`** (1 nodes): `docs/design/offline_compiler_design.md`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `docs/test/unit_test_spec.md`** (1 nodes): `docs/test/unit_test_spec.md`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Stage B - NPU Timing & Engine Modeling`** (1 nodes): `Stage B - NPU Timing & Engine Modeling`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Stage C - Full Operator Support`** (1 nodes): `Stage C - Full Operator Support`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Stage D - LLM Specialization`** (1 nodes): `Stage D - LLM Specialization`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Stage E - DSE Platform`** (1 nodes): `Stage E - DSE Platform`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Modular & Replaceable`** (1 nodes): `Modular & Replaceable`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Trace-First Design`** (1 nodes): `Trace-First Design`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `End-to-End AI Hardware Simulation Platform`** (1 nodes): `End-to-End AI Hardware Simulation Platform`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Skeleton-First SDD Implementation Roadmap`** (1 nodes): `Skeleton-First SDD Implementation Roadmap`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Phase 0 - Project Code Skeleton`** (1 nodes): `Phase 0 - Project Code Skeleton`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Phase 2 - Timing/Memory/Engine Realization`** (1 nodes): `Phase 2 - Timing/Memory/Engine Realization`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Single Source of Truth`** (1 nodes): `Single Source of Truth`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Spec Update`** (1 nodes): `Spec Update`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Design Update`** (1 nodes): `Design Update`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Test Doc Update`** (1 nodes): `Test Doc Update`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Implementation`** (1 nodes): `Implementation`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Review & Merge`** (1 nodes): `Review & Merge`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `PR/Issue Templates`** (1 nodes): `PR/Issue Templates`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `CI Integration`** (1 nodes): `CI Integration`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Example Flows`** (1 nodes): `Example Flows`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.
- **Thin community `Global Integrated Diagram`** (1 nodes): `Global Integrated Diagram`
  Too small to be a meaningful cluster - may be noise or needs more connections extracted.