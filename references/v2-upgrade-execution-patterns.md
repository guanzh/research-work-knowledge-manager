# v2.0 Upgrade Execution Patterns

Proven Python patterns from a successful brownfield v1.0→v2.0 upgrade of a wildlife research
workspace. These are copy-modify templates, not abstract guidance.

## Pattern 1: Upgrade file_registry → source_registry

When you have an existing `file_registry.jsonl` from v1.0, add the six v2.0 fields
(`source_type`, `provenance`, `sensitivity`, `linked_claims`, `linked_gaps`, `linked_tasks`,
`impact_on_goal`) without losing any v1.0 fields.

```python
import os, json

base = "<workspace_root>"
fr_path = os.path.join(base, "registry", "file_registry.jsonl")
records = []
with open(fr_path, 'r', encoding='utf-8') as f:
    for line in f:
        if line.strip():
            records.append(json.loads(line))

# Map work_id → source_type (key decision table)
work_source_map = {
    "W_raw_data": "dataset",
    "W_raw_workbook": "dataset",
    "W_reference_pdf": "literature_pdf",
    "W_manuscript_draft": "manuscript_draft",
    "W_evidence": "experiment_log",
    "W_literature": "literature_pdf",
    "W_analysis_outputs": "dataset",
    "W_figures": "figure",
    "W_stage_docs": "experiment_log",
    "W_checks": "experiment_log",
    "W_supporting": "experiment_log",
}

work_impact_map = {
    "W_raw_data": "high",
    "W_manuscript_draft": "high",
    "W_evidence": "high",
    "W_analysis_outputs": "high",
    "W_literature": "medium",
    "W_figures": "medium",
    "W_checks": "medium",
    "W_stage_docs": "medium",
    "W_supporting": "low",
}

for rec in records:
    wid = rec.get("work_id", "")
    rec["source_type"] = work_source_map.get(wid, "experiment_log")
    rec["provenance"] = {
        "origin": "field_collection" if wid in ("W_raw_data", "W_raw_workbook") else "generated",
        "collected_by": "author_team",
        "collected_at": "unknown",
        "processing": "cleaned" if wid in ("W_raw_data", "W_analysis_outputs") else "raw"
    }
    rec["sensitivity"] = "internal"
    rec["sensitivity_note"] = ""
    rec["linked_claims"] = []
    rec["linked_gaps"] = []
    rec["linked_tasks"] = []
    rec["impact_on_goal"] = work_impact_map.get(wid, "low")

# Write to source_registry (keep file_registry for backward compat)
sr_path = os.path.join(base, "registry", "source_registry.jsonl")
with open(sr_path, 'w', encoding='utf-8') as f:
    for rec in records:
        f.write(json.dumps(rec, ensure_ascii=False) + '\n')
```

## Pattern 2: Batch-create claim_matrix from claim_ledger

When the project has a `claim_ledger.md` with a table of claims, extract claims
programmatically into `claim_matrix.jsonl`. Build each claim record with the
full v2.0 schema including `supporting_sources` (list of `{file_id, source_type, note}`).

Tip: the `supporting_sources` should reference the specific `file_id` of the
data_contract (`F...` from source_registry), not a filename.

```python
claims = [
    {
        "claim_id": "C001",
        "statement": "...",
        "status": "strongly_supported",  # strongly_supported | weakly_supported | contested | speculative | dropped
        "manuscript_section": "Abstract / Results",
        "supporting_sources": [
            {"file_id": "F20260608013", "source_type": "experiment_log", "note": "data contract: rate ratio 27.35, 95% CI 13.61-54.96"}
        ],
        "challenging_sources": [],
        "evidence_strength": "high",  # high | medium | low
        "weakness": "Observational patch comparison, not causal",
        "next_task": None,
        "updated": "2026-06-08"
    },
    # ... repeat for each claim
]

with open(os.path.join(base, "registry", "claim_matrix.jsonl"), 'w', encoding='utf-8') as f:
    for c in claims:
        f.write(json.dumps(c, ensure_ascii=False) + '\n')
```

## Pattern 3: Batch-create gap_register from revision_tasks

Gaps are nouns ("what is missing"), not verbs ("what to do"). Extract from
revision_tasks sections that list missing data, methods, literature, or evidence.

```python
gaps = [
    {
        "gap_id": "G001",
        "description": "Conflict processing rule not author-confirmed (854 raw → 570 processed)",
        "type": "data_gap",  # data_gap | method_gap | literature_gap | evidence_gap | analysis_gap
        "affects_claims": ["C001", "C002", "C007"],
        "severity": "blocking",  # blocking | high | medium | low
        "potential_sources": ["author_knowledge"],
        "status": "open",
        "created_at": "2026-06-08"
    },
    # ...
]
```

## Pattern 4: Batch-create task_backlog from revision_tasks

Tasks are verbs ("do X"), extracted from revision_tasks action items.
Each task notes what triggered it (`triggered_by` file_id) and what it affects
(claim_ids, gap_ids).

```python
tasks = [
    {
        "task_id": "T001",
        "description": "Confirm processed conflict table is the official version",
        "triggered_by": "F20260608003",
        "affects": ["claim_C001", "claim_C002", "claim_C007", "gap_G001"],
        "priority": "blocking",  # blocking | high | medium | low
        "status": "pending",
        "assigned_to": "author",
        "created_at": "2026-06-08"
    },
    # ...
]
```

## Pattern 5: Backfill linked_* fields in source_registry

After claim_matrix, gap_register, and task_backlog exist, backfill the
`linked_claims`, `linked_gaps`, `linked_tasks` arrays in source_registry
so the cross-reference graph is complete.

Build a `file_id → (claims, gaps, tasks)` mapping table, then patch:

```python
linking = {
    "F20260608013": (["C001","C002","C003","C004","C005","C006","C007"], [], []),
    "F20260608003": (["C001","C002","C007"], ["G001","G007"], ["T001","T008"]),
    "F20260608034": ([], ["G005","G006","G007","G009","G010","G011","G012"], 
                      ["T001","T002","T003","T004","T005","T006","T007","T008","T009",
                       "T010","T011","T012","T013","T014","T015","T016","T017","T018","T019"]),
    # ... one entry per file_id in source_registry
}

for rec in records:
    fid = rec.get("file_id")
    if fid in linking:
        lc, lg, lt = linking[fid]
        rec["linked_claims"] = lc
        rec["linked_gaps"] = lg
        rec["linked_tasks"] = lt

with open(sr_path, 'w', encoding='utf-8') as f:
    for rec in records:
        f.write(json.dumps(rec, ensure_ascii=False) + '\n')
```

## Pattern 6: Update wiki pages with claim_id/gap_id/task_id references

After the five tables exist, update `wiki/analysis.md` and `wiki/manuscript.md`
to use structured IDs instead of vague references:

- `wiki/analysis.md`: core results table gets a `claim_id` column
  linking to `claim_matrix.jsonl`
- `wiki/analysis.md`: evidence gaps section becomes a table with
  `gap_id` → `gap_register.jsonl`
- `wiki/manuscript.md`: promotion checklist becomes `gap_id` + `task_id` pairs

This closes the loop: wiki → registry tables → source files.
