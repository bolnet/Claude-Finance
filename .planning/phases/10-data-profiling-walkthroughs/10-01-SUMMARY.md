---
phase: 10-data-profiling-walkthroughs
plan: "01"
subsystem: walkthrough-commands
tags: [fpa, walkthrough, erp, budget-forecasting, liquidity-predictor, skill-routing]
dependency_graph:
  requires: []
  provides: [walkthrough-fpa-command, fpa-intent-routing]
  affects: [.claude/skills/finance/SKILL.md]
tech_stack:
  added: []
  patterns: [multi-phase-walkthrough, three-scenario-budget-framework, pure-analysis-steps]
key_files:
  created:
    - .claude/commands/walkthrough-fpa.md
  modified:
    - .claude/skills/finance/SKILL.md
decisions:
  - "FP&A walkthrough uses 4-phase structure matching equity-research/hedge-fund template: pre-flight, data profiling, model build, budget synthesis"
  - "Step 4 and 6 are pure analysis steps with no tool calls — compile ERP data quality report card and feature-to-budget-line-item table respectively"
  - "Three-scenario budget framework (Steps 7/9/10) is the signature FP&A deliverable — best case credit 750/0.3, base case 650/0.45, worst case 520/0.7"
  - "walkthrough-fpa added after walkthrough-accounting row in SKILL.md — consistent ordering reflects plan execution sequence"
metrics:
  duration_seconds: 197
  completed_date: "2026-03-18"
  tasks_completed: 2
  tasks_total: 2
  files_created: 1
  files_modified: 1
---

# Phase 10 Plan 01: FP&A Walkthrough Summary

**One-liner:** Four-phase FP&A walkthrough covering ERP data profiling with ingest_csv, regression model via liquidity_predictor, and three-scenario budget synthesis using predict_liquidity with FP&A-specific language throughout.

## What Was Built

### Task 1: `.claude/commands/walkthrough-fpa.md`

Created the `/walkthrough-fpa` slash command with 11 steps across 4 phases:

- **Phase 1 (1 step):** Pre-flight check via `validate_environment` — framed as "an FP&A forecasting workflow depends on pandas, scikit-learn, and matplotlib"
- **Phase 2 (3 steps):** ERP data profiling — full dataset profile with `ingest_csv` (no target), target column profile with `ingest_csv` (target_column="liquidity_risk"), and a pure-analysis ERP Data Quality Report Card table for VP of Finance review
- **Phase 3 (4 steps):** Forecasting model build — `liquidity_predictor` for regression model, pure-analysis Feature Importance table mapping ML features to budget line items (AR risk, balance sheet leverage, revenue capacity), Scenario 1 best-case `predict_liquidity` (credit 750, debt 0.3, Northeast), Scenario 2 worst-case `predict_liquidity` (credit 520, debt 0.7, South)
- **Phase 4 (3 steps):** Budget synthesis — Scenario 3 base case `predict_liquidity` (credit 650, debt 0.45, Midwest), pure-analysis Three-Scenario Budget Summary table with variance spread calculation, pure-analysis Quarterly Budget Recommendation memo with "What an FP&A Analyst Would Do Next" section

The command uses exclusively FP&A language: budget variance, ERP export, VP of Finance, reserve requirements, forecasting prep, resource allocation, quarterly close.

Traditional tool mapping table maps: ingest_csv to SAP/Oracle ERP export review, liquidity_predictor to Hyperion planning model, predict_liquidity to manual what-if scenario tables.

### Task 2: `.claude/skills/finance/SKILL.md`

Added three FP&A additions to SKILL.md:
1. Intent Classification table row: `walkthrough-fpa` intent with trigger phrases "fpa walkthrough", "budget variance analysis", "erp data profiling", "forecasting prep walkthrough", "fp&a scenario"
2. Role Walkthroughs table row: `/walkthrough-fpa` for FP&A analyst with "ERP data profiling, target column identification, and three-scenario budget forecasting"
3. Routing paragraph: directs users who ask about FP&A/budget variance queries to `/walkthrough-fpa`

All existing walkthrough routing preserved: hedge-fund, investment-banking, equity-research, accounting.

## Decisions Made

1. **4-phase structure** matches the equity-research/hedge-fund template exactly — ensures consistent user experience across all role walkthroughs.
2. **Pure analysis steps at Steps 4, 6, 10, 11** — these are the FP&A-native deliverables (data quality report card, feature-to-budget mapping, three-scenario table, quarterly recommendation memo) that don't require tool calls but represent the highest-value outputs for an FP&A analyst.
3. **Three-scenario credit profiles** chosen to span the full portfolio distribution from sample_portfolio.csv (scores range 514-813, debt ratios 0.11-0.78) — best case 750/0.3 is ~75th percentile, base case 650/0.45 is roughly median, worst case 520/0.7 is ~10th percentile.
4. **FP&A-specific feature importance framing** — credit_score maps to AR risk, debt_ratio to balance sheet leverage, income to revenue forecast input — bridges ML output to the budget line items an FP&A analyst actually manages.

## Deviations from Plan

None — plan executed exactly as written.

Note: SKILL.md already contained a `walkthrough-accounting` row from a concurrent plan (10-02). The FP&A additions were inserted after the accounting row to maintain chronological ordering. The plan's line number references (around line 33-34, around line 48-50, around line 54) were advisory — the actual insertion used content-based matching which correctly handled the already-modified file.

## Verification Results

All 6 plan verification checks passed:
1. walkthrough-fpa.md exists with valid frontmatter, 4 phases, 11 steps (>= 10 required)
2. Walkthrough references `demo/sample_portfolio.csv`
3. All 4 MCP tools present: validate_environment, ingest_csv, liquidity_predictor, predict_liquidity
4. FP&A framing keywords present: budget, forecast, erp, variance
5. SKILL.md has fpa intent routing row and Role Walkthroughs entry
6. All existing walkthrough routing preserved: hedge-fund, investment-banking, equity-research, accounting

## Commits

| Task | Commit | Description |
|------|--------|-------------|
| Task 1 | 4332210 | feat(10-01): create /walkthrough-fpa slash command |
| Task 2 | d48b7a5 | feat(10-01): add FP&A walkthrough intent routing to SKILL.md |

## Self-Check: PASSED

Files exist:
- `.claude/commands/walkthrough-fpa.md` — FOUND
- `.claude/skills/finance/SKILL.md` — FOUND (modified)

Commits exist:
- 4332210 — FOUND
- d48b7a5 — FOUND
