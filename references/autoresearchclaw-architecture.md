# AutoResearchClaw Architecture Reference

> Source: https://github.com/aiming-lab/AutoResearchClaw (13.3k stars, v0.5.0)
> Analyzed: 2026-06-08

## Overview

Fully autonomous & self-evolving research pipeline: "Chat an Idea. Get a Paper."
23 stages across 8 phases, with multi-agent debate, self-learning, and HITL co-pilot.

## Pipeline: 23 Stages, 8 Phases

```
Phase A: Research Scoping          Phase E: Experiment Execution
  1. TOPIC_INIT                      12. EXPERIMENT_RUN
  2. PROBLEM_DECOMPOSE               13. ITERATIVE_REFINE  ← self-healing

Phase B: Literature Discovery      Phase F: Analysis & Decision
  3. SEARCH_STRATEGY                 14. RESULT_ANALYSIS    ← multi-agent
  4. LITERATURE_COLLECT  ← real API  15. RESEARCH_DECISION  ← PIVOT/REFINE
  5. LITERATURE_SCREEN   [gate]
  6. KNOWLEDGE_EXTRACT               Phase G: Paper Writing
                                     16. PAPER_OUTLINE
Phase C: Knowledge Synthesis         17. PAPER_DRAFT
  7. SYNTHESIS                       18. PEER_REVIEW        ← evidence check
  8. HYPOTHESIS_GEN    ← debate      19. PAPER_REVISION

Phase D: Experiment Design         Phase H: Finalization
  9. EXPERIMENT_DESIGN   [gate]      20. QUALITY_GATE      [gate]
 10. CODE_GENERATION                 21. KNOWLEDGE_ARCHIVE
 11. RESOURCE_PLANNING               22. EXPORT_PUBLISH     ← LaTeX
                                     23. CITATION_VERIFY    ← relevance check
```

Gate stages (5, 9, 20) pause for human approval.

## Key Architectural Patterns

### 1. PIVOT/REFINE Decision Loop
Stage 15 autonomously decides: PROCEED → REFINE (tweak params, → Stage 13) → PIVOT (new direction, → Stage 8). Artifacts auto-versioned.

### 2. Multi-Agent Debate
Hypothesis generation, result analysis, and peer review each use structured multi-perspective debate — not single-agent linear reasoning.

### 3. Self-Learning (MetaClaw)
Lessons captured per run (failures, warnings) → converted to skills → injected into LLM prompts for all 23 stages on subsequent runs. Proven results: -24.8% stage retry rate, -40% refine cycles.

### 4. Sentinel Watchdog
Background quality monitor: NaN/Inf detection, paper-evidence consistency, citation relevance scoring, anti-fabrication guard. Runs continuously, not just at gates.

### 5. 4-Layer Citation Verification
arXiv ID check → CrossRef/DataCite DOI → Semantic Scholar title match → LLM relevance scoring. Hallucinated refs auto-removed.

### 6. HITL Co-Pilot (6 modes)
full-auto | gate-only | checkpoint | co-pilot | step-by-step | express
With SmartPause (confidence-driven auto-pause), Idea Workshop, Baseline Navigator, Paper Co-Writer.

### 7. Structured Knowledge Base
6 categories per run: decisions, experiments, findings, literature, questions, reviews.

### 8. Reproducibility
SHA256 checksums for all stage artifacts. Immutable manifests. Multi-level undo with versioned snapshots.

## Relevance to Goal-Driven Research System

These patterns directly inform the design of research pipeline skills:

| ARC Pattern | Maps to GDRS concept |
|-------------|---------------------|
| PIVOT/REFINE → | decision_log + task_backlog |
| Multi-Agent Debate → | evidence_matrix (supporting vs challenging sources) |
| Self-Learning → | Hermes curator + arc-* skills from run lessons |
| Sentinel Watchdog → | conflict_log automated checks |
| Structured KB → | 5-table architecture (source_registry + claim_matrix + evidence_matrix + gap_register + task_backlog) |
| 4-Layer Citation → | reference-verification as part of source_registry reliability scoring |
