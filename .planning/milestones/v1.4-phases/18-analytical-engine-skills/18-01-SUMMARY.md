---
phase: 18-analytical-engine-skills
plan: 01
subsystem: skills
tags: [private-equity, machine-learning, MCP, skill, prospect-scoring, liquidity-risk, pipeline-profiling, public-comps, market-risk]

# Dependency graph
requires:
  - phase: 17-portfolio-and-value-creation-skills
    provides: Established SKILL.md pattern (200-400 lines, intent classification table, MCP tool sections, output format specs)
provides:
  - prospect-scoring SKILL.md — ML classifier train + score workflow (investor_classifier + classify_investor)
  - liquidity-risk SKILL.md — Regression model train + predict workflow (liquidity_predictor + predict_liquidity)
  - pipeline-profiling SKILL.md — CRM CSV EDA workflow (ingest_csv with PE-specific interpretation)
  - public-comp-analysis SKILL.md — Comp charts + correlation heatmaps (compare_tickers + correlation_map)
  - market-risk-scan SKILL.md — Three-tool risk analysis chain (get_volatility + get_risk_metrics + analyze_stock)
affects:
  - Any future commands that load these 5 skills (18-02-PLAN.md or similar)
  - Users of finance-mcp-plugin who trigger /project:score-prospect, /project:liquidity-risk, etc.

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "ML-chain skill pattern: train model via first tool, predict/classify via second tool (established for investor_classifier→classify_investor and liquidity_predictor→predict_liquidity)"
    - "Three-tool analytical chain: get_volatility → get_risk_metrics → analyze_stock for market risk scans"
    - "Pipeline health scorecard: weighted composite of completeness + distribution + freshness + integrity + volume"

key-files:
  created:
    - finance-mcp-plugin/skills/private-equity/prospect-scoring/SKILL.md
    - finance-mcp-plugin/skills/private-equity/liquidity-risk/SKILL.md
    - finance-mcp-plugin/skills/private-equity/pipeline-profiling/SKILL.md
    - finance-mcp-plugin/skills/private-equity/public-comp-analysis/SKILL.md
    - finance-mcp-plugin/skills/private-equity/market-risk-scan/SKILL.md
  modified: []

key-decisions:
  - "ML-chain skills established two-phase pattern: always train first (investor_classifier / liquidity_predictor), then predict (classify_investor / predict_liquidity) — model persists in memory between calls"
  - "Confidence threshold framework for prospect-scoring: >80% pursue, 50-80% nurture, 30-50% monitor, <30% pass"
  - "Risk tier mapping for liquidity-risk: >0.70 Low, 0.40-0.70 Moderate, 0.20-0.40 High, <0.20 Critical"
  - "pipeline-profiling pipeline health scorecard uses weighted dimensions: completeness 30%, distributions 25%, freshness 20%, integrity 15%, volume 10%"
  - "public-comp-analysis recommends 3-8 tickers per comparison, always include SPY as context anchor"
  - "market-risk-scan Sharpe thresholds: >1.0 good, 0.5-1.0 acceptable, <0.5 poor, <0.0 negative"

patterns-established:
  - "ML-chain SKILL.md pattern: Phase 1 = prepare data, Phase 2 = train via MCP tool 1, Phase 3 = predict via MCP tool 2, Phase 4 = batch workflow, Phase 5 = diagnostics/analysis, Phase 6 = retrain/stress-test"
  - "Risk tier framework with 4 tiers (Low/Moderate/High/Critical or Very High) and recommended actions per tier"
  - "PE context framing section in each skill translating public market metrics to PE decision-making terms"

requirements-completed: [SKILL-11, SKILL-12, SKILL-13, SKILL-14, SKILL-15]

# Metrics
duration: 7min
completed: 2026-03-19
---

# Phase 18 Plan 01: Analytical Engine Skills Summary

**5 MCP-heavy PE skills built: ML train-then-score chains (prospect-scoring, liquidity-risk), CSV EDA (pipeline-profiling), comp charts + correlation heatmaps (public-comp-analysis), and 3-tool risk chain (market-risk-scan)**

## Performance

- **Duration:** ~7 min
- **Started:** 2026-03-19T23:04:35Z
- **Completed:** 2026-03-19T23:11:48Z
- **Tasks:** 2 of 2
- **Files modified:** 5 created

## Accomplishments

- Created 5 SKILL.md files (total 1,627 lines) covering all analytical engine capabilities for PE professionals
- Established the ML-chain skill pattern: investor_classifier trains a classifier then classify_investor scores prospects, and liquidity_predictor trains a regression model then predict_liquidity predicts risk
- Built pipeline-profiling skill with PE-specific ingest_csv interpretation including a pipeline health scorecard (5 weighted dimensions)
- Built public-comp-analysis skill with sector-to-ticker mapping and two-tool valuation context workflow (compare_tickers + correlation_map)
- Built market-risk-scan skill with three-tool chain (get_volatility → get_risk_metrics → analyze_stock) and PE-specific risk tier thresholds

## Task Commits

Each task was committed atomically:

1. **Task 1: Create prospect-scoring and liquidity-risk skills (ML-chain skills)** - `94cb900` (feat)
2. **Task 2: Create pipeline-profiling, public-comp-analysis, and market-risk-scan skills** - `e3caaad` (feat)

## Files Created/Modified

- `finance-mcp-plugin/skills/private-equity/prospect-scoring/SKILL.md` — 319 lines: train ML classifier via investor_classifier, score individual prospects via classify_investor, batch scoring, model diagnostics, retrain workflow
- `finance-mcp-plugin/skills/private-equity/liquidity-risk/SKILL.md` — 341 lines: train regression model via liquidity_predictor, predict risk via predict_liquidity, portfolio scan, stress testing scenarios
- `finance-mcp-plugin/skills/private-equity/pipeline-profiling/SKILL.md` — 312 lines: full EDA on CRM CSV via ingest_csv, completeness audit, distribution analysis, data quality flags, pipeline health scorecard
- `finance-mcp-plugin/skills/private-equity/public-comp-analysis/SKILL.md` — 285 lines: comp charts via compare_tickers, correlation heatmap via correlation_map, valuation context framing, sector-to-ticker mapping
- `finance-mcp-plugin/skills/private-equity/market-risk-scan/SKILL.md` — 370 lines: volatility via get_volatility, Sharpe/drawdown/beta via get_risk_metrics, comprehensive analysis via analyze_stock

## Decisions Made

- ML-chain skills use two-phase pattern: always train first (investor_classifier / liquidity_predictor), then predict (classify_investor / predict_liquidity). Model persists in memory between calls so users do not re-train on every score.
- Confidence threshold framework for prospect-scoring: >80% pursue, 50-80% nurture, 30-50% monitor, <30% pass. These thresholds are presented as starting points the deal team can adjust.
- Risk tier mapping for liquidity-risk: >0.70 Low Risk, 0.40-0.70 Moderate Risk, 0.20-0.40 High Risk, <0.20 Critical. Maps predicted continuous score to actionable recommendations.
- market-risk-scan Sharpe thresholds follow standard PE guidance: >1.0 good, 0.5-1.0 acceptable, <0.5 poor for risk-adjusted return context.

## Deviations from Plan

None — plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None — no external service configuration required. Skills are markdown files loaded by the plugin.

## Next Phase Readiness

- All 5 SKILL-11 through SKILL-15 requirements delivered and verified
- Skills follow the exact same pattern as Phase 16 and 17 skills — ready for any command layer (18-02) that loads them
- PE professionals can now invoke: score-prospect, liquidity-risk, profile-pipeline, public-comps, market-risk workflows

---
*Phase: 18-analytical-engine-skills*
*Completed: 2026-03-19*
