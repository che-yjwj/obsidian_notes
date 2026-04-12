# SoC Specification Negotiation English

*last_compiled: 2026-04-12 | sources: 2*

---

## Summary [coverage: medium -- 2 sources]

This topic covers English language patterns and vocabulary for SoC engineers, specifically System Architects, who need to negotiate SoC specifications in customer-facing settings. The content is structured as a practical training program targeting engineers transitioning into or working in System Architect roles who must conduct technical discussions in English about NPU/SoC architecture, PPA trade-offs, and specification alignment.

The core thesis is that the goal is **not** fluent English but rather the ability to express architectural decisions, assumptions, and trade-offs clearly and structurally — enough to lead a spec negotiation meeting with confidence.

---

## Core Concepts [coverage: medium -- 2 sources]

### The Spec Negotiation Language Framework

All SoC spec negotiation conversations follow a five-step decision flow:

```
[1] Requirement Clarification
  → [2] Assumptions & Constraints
    → [3] Impact Analysis (PPA)
      → [4] Soft No & Options
        → [5] Decision / Action Items
```

Every phrase in the cheat sheet maps directly to one of these five stages.

### Key Vocabulary Domains

**Technical English**
- SoC block terminology: NPU, memory subsystem, interconnect, accelerator, CPU subsystem
- Performance model language: prefill vs decode, latency, throughput, memory bandwidth, bottleneck
- Specification document terms: `shall` / `should` / assumption / constraint / TBD / out of scope

**Negotiation English**
- Requirement clarification and rephrasing
- Risk and trade-off explanation
- Scope control (range management)
- Ownership boundary definition (SoC vs IP vs SW/Compiler/Runtime)

**Structured Communication**
- Bullet-based explanation
- Conditional framing (if–then)
- Option presentation (Option A / B / C)
- Assumption-first structuring

### Architecture English Diary Format

A daily writing practice designed to train both English expression and architect-level thinking simultaneously. Fixed five-line format:

```markdown
## 1. What I explained or practiced today
## 2. Key assumption I used
## 3. One sentence I want to reuse
## 4. What was difficult to express
## 5. One thing to improve tomorrow
```

---

## Architecture [coverage: low -- 0 sources]

Not applicable as a primary technical topic. However, the source material does cover SoC architectural concepts as the *subject matter* of the English practice:

- SoC consists of CPU subsystem, accelerator subsystem (NPU), and memory/interconnect subsystem
- NPU handles matrix and vector operations for AI inference; provides higher energy efficiency vs CPU
- Memory bandwidth and latency often dominate overall performance in AI workloads, not raw compute
- Prefill and decode phases have different performance characteristics and require different optimization strategies
- System bottleneck shifts between compute and memory depending on the workload and architecture configuration

These concepts are the raw material for practicing the negotiation language, not topics analyzed in depth.

---

## Key Findings [coverage: medium -- 2 sources]

### The Cheat Sheet: 14 Fixed Sentences

The minimum viable set for conducting a spec negotiation meeting. Organized by negotiation stage:

**1. Requirement Clarification**
- *"So what you're asking is essentially…"* — reframes customer requirement in one sentence, transfers initiative
- *"Let me rephrase your requirement to make sure I understand correctly."* — prevents misunderstanding
- *"From a system perspective, this translates to…"* — converts business language to system constraint

**2. Assumptions & Constraints**
- *"Our baseline assumption is that…"* — anchors numbers to explicit premises
- *"This analysis assumes…"*
- *"Under the current constraints…"*
- *"This holds true only if…"* — marks conditional validity

**3. Impact / Trade-off Explanation**
- *"The main bottleneck comes from…"* — identifies the limiting factor
- *"This directly impacts…"* — chains the consequence
- *"There is a trade-off between…"* — frames PPA trade-off
- *"From a latency / power / area perspective…"* — shifts analytical viewpoint

**4. Soft No & Options**
- *"Technically possible, but…"* — conditional yes that signals high cost
- *"This would require a different architecture."* — indicates scope change
- *"This is not feasible within the current scope."* — scope boundary
- *"One possible option is…"* — Option A
- *"An alternative approach would be…"* — Option B
- *"Another direction we can explore is…"* — exploratory option

**5. Ownership & Scope**
- *"This is handled at the SoC level."* — assigns SoC responsibility
- *"This falls under the compiler/runtime responsibility."* — assigns SW responsibility
- *"This is out of scope for this phase."* — freezes scope
- *"We suggest freezing this parameter."* — locks a spec parameter

**6. Wrap-up & Action Items**
- *"Let's align on the following points."*
- *"The action item from our side is…"*
- *"We will follow up with…"*

### Meeting Scenarios

**Scenario 1: Unrealistic Performance Request**

Customer: *"Can you double the performance without increasing power?"*

Response structure:
1. Clarify: *"So what you're asking is essentially doubling throughput under the same power budget."*
2. Assumptions: *"Our current performance assumes a fixed memory bandwidth."*
3. Impact: *"Doubling throughput shifts the bottleneck to memory, not compute."*
4. Soft No + Options: *"Technically possible, but only if we relax either power or precision. One possible option is to use lower precision for the decode phase."*
5. Wrap-up: *"Let's align on which constraint is more flexible."*

**Scenario 2: Ambiguous Latency Requirement**

Customer: *"We want low latency for LLM inference."*

Response structure:
1. Clarify: *"Low latency for which phase — prefill or decode?"*
2. Assumption: *"These two phases have very different characteristics."*
3. Impact: *"Optimizing both simultaneously usually increases area and power."*
4. Option: *"We can prioritize decode latency, or balance both."*

**Scenario 3: SoC vs Compiler Ownership Dispute**

Customer: *"Can the SoC guarantee this scheduling behavior?"*

Response:
- *"The hardware provides the necessary primitives, but the exact scheduling behavior is handled by the compiler."*
- *"From the SoC side, we guarantee bandwidth and latency bounds."*

### Experience-Based Language (Trust-Building Sentences)

Sentences that convey credibility by demonstrating real design experience, not just English fluency:

- *"My main role was to define the system-level SoC architecture, including NPU integration, memory hierarchy, and interconnect assumptions."*
- *"Rather than relying on RTL, I used an abstract performance model to explore architectural trade-offs in the early phase."*
- *"Our performance numbers are based on clearly defined assumptions regarding memory latency and data reuse."*
- *"Before committing to this target, I think we need to align on the memory access pattern assumptions."*
- *"From a system perspective, this requirement shifts the bottleneck from compute to memory."*
- *"I focused on clarifying the boundary between SoC-level responsibilities and IP-level responsibilities early in the project."*
- *"I built a cycle-level performance model to evaluate prefill vs decode latency under different KV-cache tiling strategies."*

### 4-Week Intensive Training Plan

Structured for engineers with 1 month before entering a System Architect role. Daily commitment: 60–90 minutes.

| Week | Theme | Daily Tasks |
|------|-------|-------------|
| Week 1 | Architecture Explanation | Read 5 fixed sentences aloud; write 5-sentence SoC description; 2-min spoken explanation (recorded); Architecture Diary |
| Week 2 | Performance & Trade-off | Practice 5 trade-off sentences; write 3 assumption sentences; answer 1 customer question; Diary |
| Week 3 | Spec Negotiation & Scope Control | Practice 3 Soft No sentences; write 2 option sentences; practice 2 ownership sentences; Diary |
| Week 4 | Real Meeting Simulation | Practice 2 opening sentences; practice 2 wrap-up sentences; 5-min full mock meeting; Diary |

Day-by-day focus topics within each week:
- Week 1: Overall SoC Architecture → NPU Block & Execution Model → Memory Hierarchy → Interconnect & Data Movement → SoC vs IP Responsibility → Full SoC Explanation → Review
- Week 2: Latency vs Throughput → Compute vs Memory Bottleneck → Power vs Performance → Area Impact → Prefill vs Decode → Quantization Impact → Review Q&A
- Week 3: Requirement Rephrasing → Unrealistic Target → Scope Control → SoC vs Compiler Boundary → Risk Explanation → Parameter Freeze → Role-play
- Week 4: Meeting Opening → Clarification Q&A → Conflict Handling → Decision Summary → Action Items → Full Mock Meeting → Final Review

**Daily success criterion:** Spoke + Wrote + Organized (not grammar, not fluency — structure)

### Obsidian + ChatGPT Workflow

Recommended integration pattern for self-study:

```
[ChatGPT] → generate / refine / expand
     ↓
[Obsidian] → accumulate / connect / reuse
     ↓
[Meeting / Document / Speaking]
```

Recommended Obsidian vault structure for this training:
```
00_Daily/           — Architecture English Diary entries
01_CheatSheets/     — SoC_Spec_Negotiation_English.md, Meeting_Opening_Closing.md, Tradeoff_Language.md
02_Scenarios/       — Performance_Request.md, Unrealistic_Spec.md, Scope_and_Ownership.md
03_Fixed_Sentences/ — Architecture_Explanation.md, Assumptions_and_Constraints.md, Soft_No_and_Options.md, Wrapup_and_Action_Items.md
04_Projects/        — Current project context in English
99_Index/           — Start_Here.md (vault navigation hub)
```

Effective ChatGPT prompts for refining sentences:
- *"Rewrite this sentence for use in an SoC spec negotiation meeting. Keep it non-confrontational, architect tone."*
- *"Refine this into a fixed sentence I can reuse across meetings."*

---

## Connections [coverage: low -- 1 source]

- [[../../raw/Misc/SoC Specification Negotiation English/SoC Specification Negotiation English (Part 1 of 2) 33a6cc566b0b8129952df13d553ab241]] — source material, Part 1
- [[../../raw/Misc/SoC Specification Negotiation English/SoC Specification Negotiation English (Part 2 of 2) 33a6cc566b0b811a9636ed54f24fd862]] — source material, Part 2
- Related vault topics: SoC/NPU architecture (AI-Hardware/Architecture), performance modeling (AI-Hardware/Simulator), quantization trade-offs (GenAI/Compression)
- The KV-cache tiling and prefill/decode distinction mentioned in example sentences maps to GenAI topics in this vault

---

## Open Questions [coverage: low -- 0 sources]

- No source coverage of email writing or async written communication patterns (e.g., spec review comments, meeting minutes)
- Scenarios do not cover multi-stakeholder meetings (customer + internal design team)
- No coverage of handling ambiguity in spec documents (e.g., responding to "shall" vs "should" language in PRDs)
- No examples for presenting quantitative data (benchmarks, power numbers) in spoken English
- The 4-week plan assumes daily availability; no guidance on compressed (2-week) or extended (3-month) schedules
- No coverage of cross-cultural communication nuances specific to semiconductor customer interactions (e.g., US vs Korean vs Japanese customer styles)

---

## Sources [coverage: medium -- 2 sources]

- [[../../raw/Misc/SoC Specification Negotiation English/SoC Specification Negotiation English (Part 1 of 2) 33a6cc566b0b8129952df13d553ab241]]
- [[../../raw/Misc/SoC Specification Negotiation English/SoC Specification Negotiation English (Part 2 of 2) 33a6cc566b0b811a9636ed54f24fd862]]
