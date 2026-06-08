# Workspace Reorganization Example

This file captures the concrete output patterns from a successful brownfield
reorganization of a Chinese wildlife research workspace. Use as a template for
the diagnostic report and wiki pages.

## Diagnostic Report Template

When analyzing a messy workspace, produce a report covering:

1. **File Overview** — table of directories, file counts, and nature
2. **Core Problems** — numbered list with data backing each problem
   - Package proliferation (same data analyzed N times)
   - Duplicate filenames across packages (count)
   - Missing registry (which sub-systems are absent)
   - Raw/generated mixing
   - No knowledge index
3. **Organization Proposal** — directory tree with annotations
4. **Execution Path** — numbered tasks with files, verification steps
5. **Risks** — sync conflicts, author time, path encoding

## Wiki Page Templates

### wiki/data.md

Should contain:
- Source file table with file_id, filename, record count, description
- Key variables and definitions
- Known data issues (with specific counts)
- Sensitive data statement

### wiki/analysis.md

Should contain:
- Research question (1-2 sentences)
- Core results table (#, conclusion, effect size, strength)
- Method notes (unit of analysis, statistical methods)
- Analysis boundaries (what can/cannot be claimed)
- Output file paths
- Figure descriptions

### wiki/manuscript.md

Should contain:
- Current manuscript metadata table
- Passed checks table
- Promotion checklist (what's needed for next level)
- Author decision items (sourced from revision_tasks)

## Decision Log Format

Use markdown tables, not bare D-prefix lines:

```markdown
# Decision Log

| ID | Date | Description | Status |
|----|------|-------------|--------|
| D001 | 2026-06-08 | 当前有效分析包: wildlife_manuscript_rerun_20260605 | resolved |
| D002 | 2026-06-08 | 其余4个旧分析包全部归档 | resolved |
```

## File ID Allocation Pattern

When bulk-registering files in a package, organize by work_id groups:

```
F20260608001-003  W_raw_data         (3 dataset .xlsx)
F20260608004      W_raw_workbook     (1 provenance .xlsx)
F20260608005      W_reference_pdf    (1 background PDF)
F20260608006      W_manuscript_pkg   (1 README)
F20260608007-011  W_manuscript_draft (5 manuscript files)
F20260608012-016  W_evidence         (5 core evidence docs)
F20260608017-018  W_literature       (2 literature docs)
F20260608019-022  W_analysis_outputs (4 output files)
F20260608023-027  W_figures          (5 figures)
F20260608028-033  W_stage_docs       (6 stage documents)
F20260608034-037  W_checks           (4 quality checks)
F20260608038-045  W_supporting       (8 supporting docs)
```

Assign contiguous file_id ranges per work group for readability.
