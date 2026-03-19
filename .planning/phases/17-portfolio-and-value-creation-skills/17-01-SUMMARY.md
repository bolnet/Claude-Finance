---
phase: 17-portfolio-and-value-creation-skills
plan: "01"
subsystem: finance-mcp-plugin/skills/private-equity
tags: [private-equity, portfolio-monitoring, returns-analysis, unit-economics, value-creation, ai-readiness, skills]
dependency_graph:
  requires: [16-02-SUMMARY.md]
  provides: [SKILL-06, SKILL-07, SKILL-08, SKILL-09, SKILL-10]
  affects: [finance-mcp-plugin/skills/private-equity/]
tech_stack:
  added: []
  patterns:
    - SKILL.md pattern (frontmatter + intent table + phased workflow + MCP tool integration + output templates + error handling)
    - PE workflow frameworks: EBITDA bridge, 100-day plan, cohort waterfall, LTV/CAC, AI readiness scoring
key_files:
  created:
    - finance-mcp-plugin/skills/private-equity/portfolio-monitoring/SKILL.md
    - finance-mcp-plugin/skills/private-equity/returns-analysis/SKILL.md
    - finance-mcp-plugin/skills/private-equity/unit-economics/SKILL.md
    - finance-mcp-plugin/skills/private-equity/value-creation-plan/SKILL.md
    - finance-mcp-plugin/skills/private-equity/ai-readiness/SKILL.md
  modified: []
decisions:
  - "All 5 skills follow the deal-sourcing/ic-memo density target (200–400 lines) established in Phase 16"
  - "No MCP tool integration in value-creation-plan or ai-readiness — both are pure framework skills"
  - "AI readiness gate uses aggregate weighted score: GO ≥ 3.5, WAIT 2.0–3.4, NOT READY < 2.0"
metrics:
  duration_seconds: 464
  completed_date: "2026-03-19"
  tasks_completed: 2
  tasks_total: 2
  files_created: 5
  files_modified: 0
---

# Phase 17 Plan 01: Portfolio and Value Creation Skills Summary

**One-liner:** 5 PE portfolio-stage SKILL.md files (1,588 total lines) covering KPI monitoring with classify_investor drift detection, IRR/MOIC sensitivity with public comp benchmarking, ARR cohort analysis with ingest_csv profiling, EBITDA bridge with 100-day plan, and AI readiness go/wait gate with EBITDA-ranked quick wins.

---

## Tasks Completed

| Task | Name | Commit | Files |
|------|------|--------|-------|
| 1 | Create portfolio-monitoring and returns-analysis skills | 0567ac1 | portfolio-monitoring/SKILL.md, returns-analysis/SKILL.md |
| 2 | Create unit-economics, value-creation-plan, and ai-readiness skills | 9aa97f7 | unit-economics/SKILL.md, value-creation-plan/SKILL.md, ai-readiness/SKILL.md |

---

## Skills Created

### SKILL-06: portfolio-monitoring (342 lines)
- **Intent coverage:** kpi-dashboard, drift-detection, benchmark-compare, alert-review, board-prep
- **MCP tools:** `classify_investor` (drift detection vs. initial investment thesis score), `get_risk_metrics` (Sharpe/drawdown/beta for public market benchmarks)
- **Key frameworks:** Traffic-light KPI thresholds, classification drift report template, benchmark comparison table, underperformer action framework, board deck narrative template
- **Output formats:** KPI dashboard with traffic-light status, drift detection report, benchmark table, board reporting package

### SKILL-07: returns-analysis (294 lines)
- **Intent coverage:** irr-moic, sensitivity, public-comps, exit-modeling, fund-attribution
- **MCP tools:** `get_returns` (public comp return data for PME comparison), `get_risk_metrics` (Sharpe/drawdown/beta for risk-adjusted benchmarking)
- **Key frameworks:** IRR/MOIC inputs template with gross/net computation, 5x5 sensitivity matrices (entry vs. exit, hold period vs. exit), probability-weighted exit scenario comparison
- **Output formats:** IRR/MOIC summary, sensitivity tables, public comp returns with PME ratio, exit scenario comparison, fund attribution table

### SKILL-08: unit-economics (295 lines)
- **Intent coverage:** arr-cohort, ltv-cac, net-retention, revenue-quality, cohort-profile
- **MCP tools:** `ingest_csv` (cohort CSV profiling — data quality, column completeness, distributions before analysis)
- **Key frameworks:** ARR cohort waterfall (expansion/contraction/churn by vintage), LTV model with discount rate, blended CAC and payback period, NRR decomposition with expansion breakdown, 5-dimension revenue quality scorecard
- **Output formats:** Cohort data quality summary, waterfall table, LTV/CAC assessment, NRR decomposition, revenue quality scorecard with traffic-light

### SKILL-09: value-creation-plan (311 lines)
- **Intent coverage:** ebitda-bridge, hundred-day, kpi-targets, lever-analysis, initiative-track
- **MCP tools:** None (pure framework skill)
- **Key frameworks:** EBITDA bridge (revenue growth, gross margin, OpEx, one-time costs), 100-day plan (3 phases with activities/deliverables/decision points), KPI target dashboard by function through Year 3, value lever analysis with priority scoring
- **Output formats:** EBITDA bridge waterfall, 100-day plan table, KPI target dashboard, lever analysis table, initiative tracker with revised EBITDA forecast

### SKILL-10: ai-readiness (345 lines)
- **Intent coverage:** portfolio-scan, company-assess, quick-wins, roadmap-build, risk-assess
- **MCP tools:** None (pure framework skill)
- **Key frameworks:** 5-dimension scoring (Data Infrastructure, Technical Capability, Process Maturity, Organization Readiness, Use Case Clarity), weighted aggregate gate (GO/WAIT/NOT READY), quick wins ranked by (EBITDA × confidence / effort), phased 3-phase implementation roadmap
- **Output formats:** Portfolio readiness heatmap, company assessment scorecard with gate decision, quick wins ranked table, implementation roadmap with EBITDA targets, risk assessment matrix

---

## Verification Results

All 5 success criteria met:

- [x] All 5 SKILL.md files exist at `finance-mcp-plugin/skills/private-equity/{name}/SKILL.md`
- [x] Each skill is 200–400 lines with YAML frontmatter (name, description, version)
- [x] portfolio-monitoring references both `classify_investor` and `get_risk_metrics`
- [x] returns-analysis references both `get_returns` and `get_risk_metrics`
- [x] unit-economics references `ingest_csv`
- [x] value-creation-plan contains EBITDA bridge framework and 100-day plan
- [x] ai-readiness contains go/wait gate scoring and EBITDA-ranked quick wins

---

## Deviations from Plan

None — plan executed exactly as written.

---

## Self-Check: PASSED

Files verified:
- FOUND: finance-mcp-plugin/skills/private-equity/portfolio-monitoring/SKILL.md (342 lines)
- FOUND: finance-mcp-plugin/skills/private-equity/returns-analysis/SKILL.md (294 lines)
- FOUND: finance-mcp-plugin/skills/private-equity/unit-economics/SKILL.md (295 lines)
- FOUND: finance-mcp-plugin/skills/private-equity/value-creation-plan/SKILL.md (311 lines)
- FOUND: finance-mcp-plugin/skills/private-equity/ai-readiness/SKILL.md (345 lines)

Commits verified:
- FOUND: 0567ac1 — feat(17-01): add portfolio-monitoring and returns-analysis PE skills
- FOUND: 9aa97f7 — feat(17-01): add unit-economics, value-creation-plan, and ai-readiness PE skills
