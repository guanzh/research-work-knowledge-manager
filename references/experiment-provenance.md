# Experiment Provenance: Claim → Experiment Alignment

Automatically link experiment results to the claim matrix during ingest.

## Purpose

When a new experiment_log is ingested, the system should not just register the file — it should trace every quantitative result back to the claims it supports or challenges. This prevents the common failure mode where experimental results exist in the evidence_matrix but are never linked to specific claims, making them invisible during manuscript writing.

## When to Run

During `extract` and `link` steps of the 8-step pipeline for any source with `source_type = experiment_log` or `source_type = code` (when the code produces model outputs).

## Provenance Schema

Each experiment result written to evidence_matrix MUST include:

```json
{
  "evidence_id": "E...",
  "source_file_id": "F...",
  "evidence_statement": "Occupancy probability declined from 0.73 (95% CI 0.61–0.82) in 2020 to 0.51 (0.39–0.63) in 2024",
  "related_claim": "C001",           // auto-matched from claim_matrix
  "evidence_direction": "supports",
  "reliability": "verified",
  "provenance": {
    "method": "occupancy_model",
    "parameters": {"covariates": ["year", "forest_cover"], "detection_covariates": ["survey_hour"]},
    "input_data": ["F20260608001"],
    "runtime_env": "R 4.3.2 / unmarked 1.3.0",
    "run_timestamp": "2026-06-08T14:30:00",
    "model_selection": "AICc = 1245.3"
  }
}
```

## Auto-Matching Rules

When an experiment_log is ingested, the system attempts to match each extracted result to the claim_matrix:

### Match by keyword overlap (fuzzy)

For each extracted quantitative result, compute overlap with each claim in claim_matrix:
- Tokenize both the result statement and the claim statement
- Compute Jaccard similarity on keywords (species names, method terms, variable names)
- If similarity ≥ 0.3, propose as candidate match
- Write to evidence_matrix with `related_claim` = best-match claim_id

### Match by data lineage (deterministic)

If the experiment_log's provenance includes `input_data` file_ids:
- Look up those file_ids in source_registry
- Check their `linked_claims` field
- If any linked_claim exists, directly map results to those claims

### Match by work_id

If the experiment belongs to a known work_id (e.g., `W_gibbon_acoustic_2025`):
- Pull all claims from claim_matrix where `work_id` matches
- Attempt keyword matching only within this subset

## Unmatched Results

Results that cannot be auto-matched to any claim are still written to evidence_matrix with `related_claim = null`. These appear in Evolution Reports under "Unlinked Evidence" — they may indicate:

- A new finding that warrants a new claim
- An incidental result that is not manuscript-relevant
- A mismatch caused by claim wording that is too vague

## Handling Anomalies

When an experiment_log contains unexpected or negative results:
- Write them as evidence with `direction = "challenges"` and `reliability = "needs_review"`
- Link to the closest matching claim
- Flag in Evolution Report as "Potential Claim Challenge"

## Integration with Statistical Delivery Gate

When used with `wildlife-manuscript-builder`:
- The statistical delivery gate (`references/statistical-delivery-gate.md`) can consume evidence_matrix entries marked with `reliability = "verified"` and `provenance.method`
- The claim-ledger (`references/claim-ledger.md`) can cross-reference evidence entries to verify claim-evidence alignment

## Anti-Patterns

- Do not skip provenance fields (method, parameters, runtime_env). Without them, a reviewer or future-you cannot reproduce the result.
- Do not auto-create claims for every extracted result. Only claim_matrix entries with human-reviewed statements should exist.
- Do not treat auto-matched results as confirmed. Every auto-match should be flagged for human review in the Evolution Report.
