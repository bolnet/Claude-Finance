---
phase: 10-data-profiling-walkthroughs
plan: "02"
subsystem: walkthroughs
tags: [accounting, walkthrough, slash-command, anomaly-detection, csv-ingestion, audit, controller]
dependency_graph:
  requires:
    - demo/sample_portfolio.csv
    - .claude/commands/walkthrough-equity-research.md (structural template)
    - src/finance_mcp MCP tools (ingest_csv, investor_classifier, classify_investor, validate_environment)
  provides:
    - .claude/commands/walkthrough-accounting.md
    - /walkthrough-accounting slash command
    - accounting intent routing in SKILL.md
  affects:
    - .claude/skills/finance/SKILL.md (intent routing, Role Walkthroughs table)
tech_stack:
  added: []
  patterns:
    - 4-phase walkthrough structure (established in equity-research, extended to accounting domain)
    - Pure-analysis steps (no tool calls) for audit report, accuracy interpretation, anomaly summary, and close recommendation
    - Controller framing: investor_classifier reframed as segment misclassification detector
key_files:
  created:
    - .claude/commands/walkthrough-accounting.md
  modified:
    - .claude/skills/finance/SKILL.md
decisions:
  - "investor_classifier reframed as anomaly detection / misclassification detection tool (not investment tool) — consistent with accounting domain without changing tool behavior"
  - "classify_investor reframed as individual GL record spot-check validation (not investor profiling) — matches substantive testing in audit workflows"
  - "4 phases chosen: pre-flight, data profiling, segment classification, consolidation synthesis — maps to quarterly close workflow sequence"
  - "Step 4 (ERP Audit Report) is a pure analysis step that compiles Phase 2 findings into audit committee format — no tool call needed, avoids redundant MCP invocations"
  - "Traditional tool mapping covers SAP ABAP, Oracle GL, Excel pivot tables — chosen as most common ERP/audit tools for controller audience"
metrics:
  duration_minutes: 3
  completed_date: "2026-03-18"
  tasks_completed: 2
  tasks_total: 2
  files_created: 1
  files_modified: 1
---

# Phase 10 Plan 02: Accounting Walkthrough Summary

**One-liner:** Controller-framed /walkthrough-accounting command using ingest_csv for ERP data profiling and investor_classifier as a segment misclassification detector for quarterly close audit workflows.

---

## What Was Built

### Task 1: walkthrough-accounting.md slash command

Created `.claude/commands/walkthrough-accounting.md` — a 4-phase, 11-step autonomous walkthrough for controllers and accounting teams. The scenario frames the existing MCP tools in controller language without any new Python code.

**Phase structure:**
- **Phase 1 (1 step):** Pre-flight check via `validate_environment`
- **Phase 2 (3 steps):** Transaction data profiling — full ERP export via `ingest_csv` (no target), segment distribution via `ingest_csv` (target="segment"), pure-analysis ERP Data Audit Report table
- **Phase 3 (4 steps):** Segment classification using `investor_classifier`, pure-analysis accuracy-as-audit-metric interpretation, two `classify_investor` record validations (expected conservative, ambiguous profile)
- **Phase 4 (3 steps):** Third `classify_investor` cross-check, pure-analysis Anomaly Detection Summary table, pure-analysis Quarterly Close Recommendation with "What a Controller Would Do Next"

**Key framing decisions:**
- `investor_classifier` framed as anomaly detection model that establishes expected segment patterns; mismatches = GL misclassifications
- `classify_investor` framed as individual record spot-check during substantive testing (not investor profiling)
- Accounting vocabulary throughout: audit trail, general ledger, trial balance, consolidation entity, ERP, audit committee, quarterly close, CECL/IFRS 9 reference for credit_score column

### Task 2: SKILL.md intent routing

Added three additions to `.claude/skills/finance/SKILL.md`:
1. Intent table row: `walkthrough-accounting` with trigger phrases covering controller, quarterly close, audit, anomaly detection
2. Role Walkthroughs table row: `/walkthrough-accounting | Controller / accounting | ...`
3. Routing paragraph: directs accounting/controller/audit queries to `/walkthrough-accounting`

All existing walkthrough routing preserved. FPA rows (from Plan 10-01) also present and correctly ordered.

---

## Verification Results

Both automated verification scripts passed:
- 11 steps confirmed (requirement: >= 10)
- All 4 MCP tools present in allowed-tools frontmatter
- `sample_portfolio.csv` reference confirmed
- Phases 1-4 all present
- Accounting/audit/controller language confirmed
- "not financial advice" disclaimer confirmed
- SKILL.md: all 4 existing walkthroughs preserved; accounting row added

---

## Deviations from Plan

None — plan executed exactly as written.

---

## Self-Check

### Files exist:
- `.claude/commands/walkthrough-accounting.md` — FOUND
- `.claude/skills/finance/SKILL.md` (modified) — FOUND

### Commits exist:
- `212873f` feat(10-02): create walkthrough-accounting slash command
- `70b636f` feat(10-02): update SKILL.md intent routing for accounting walkthrough

## Self-Check: PASSED
