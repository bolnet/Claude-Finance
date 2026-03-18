---
phase: 02-market-analysis-tools
plan: "03"
subsystem: market-tools
tags: [tdd, mcp-tool, comparison, correlation, heatmap, phase-2, mktx-07]
dependency_graph:
  requires:
    - 02-01 (test scaffold, analyze_stock, price_chart tool)
    - 02-02 (returns, volatility, risk_metrics tools — pre-existing in repo)
  provides:
    - src/finance_mcp/tools/comparison.py (compare_tickers MCP tool)
    - src/finance_mcp/tools/correlation.py (correlation_map MCP tool)
    - mcp.add_tool() for all 6 Phase 2 tools in server.py
    - All 15 test_market_tools.py tests passing (MKTX-01 through MKTX-07)
  affects:
    - Phase 3 ML tools (depend on Phase 2 data infrastructure being complete)
tech_stack:
  added:
    - seaborn (imported inside correlation_map function body — Agg backend already set)
  patterns:
    - Normalize-to-100 price comparison (prices_df / prices_df.iloc[0] * 100)
    - Return-based correlation (pct_change().dropna().corr()) not price-level correlation
    - seaborn imported inside function body to respect Agg backend ordering set by output.py
    - mcp.add_tool() registration pattern continued from Phase 2 decisions
key_files:
  created:
    - src/finance_mcp/tools/comparison.py
    - src/finance_mcp/tools/correlation.py
  modified:
    - src/finance_mcp/server.py
    - tests/test_market_tools.py
decisions:
  - "Import seaborn inside correlation_map function body — output.py sets Agg backend at module scope; seaborn must be imported after pyplot to respect this ordering"
  - "Correlation on pct_change().dropna() not raw prices — price levels are non-stationary; return-based correlation avoids spurious high correlations from shared trends"
  - "compare_tickers uses pd.DataFrame(prices_dict).dropna() for common date range — ensures all tickers have data on the same dates before normalization"
metrics:
  duration: 5 min
  completed: "2026-03-18"
  tasks_completed: 2
  files_changed: 4
---

# Phase 2 Plan 3: compare_tickers and correlation_map Tools Summary

**One-liner:** Normalized multi-ticker comparison chart and seaborn return-correlation heatmap completing all 6 Phase 2 MCP tools with 15/15 tests green

## What Was Built

Final plan of Phase 2. Implements the last two analysis tools and drives all remaining 6 failing test stubs to green, completing MKTX-05, MKTX-06, and MKTX-07.

### 1. compare_tickers tool (`src/finance_mcp/tools/comparison.py`)

MCP tool that accepts 2-5 comma-separated ticker symbols, fetches adjusted price history via `fetch_multi_ticker()`, restricts to common trading dates, normalizes all price series to 100 at the start date, and plots a multi-line performance chart. Final values are ranked to identify best and worst performer with plain-English interpretation. Output follows `format_output()` contract: plain-English summary first, data section with normalized returns, chart path, and mandatory DISCLAIMER last.

### 2. correlation_map tool (`src/finance_mcp/tools/correlation.py`)

MCP tool that computes pairwise return correlation (not price-level correlation) for 2-10 ticker symbols. Uses seaborn heatmap with coolwarm colormap, annotation, and square cells as specified in the research reference pattern. Seaborn is imported inside the function body to ensure `output.py` has already set the Agg matplotlib backend before seaborn loads. Output identifies the most and least correlated pairs and explicitly states that returns (not price levels) were used, satisfying the `test_correlation_uses_returns` assertion.

### 3. server.py updated with all 6 Phase 2 tools

All 6 tools registered: `analyze_stock`, `get_returns`, `get_volatility`, `get_risk_metrics`, `compare_tickers`, `correlation_map`. Server imports without error.

### 4. All 15 test_market_tools.py tests green

The 6 stub tests (previously `pytest.fail`) replaced with full implementations covering:
- `test_normalized_prices_start_at_100` — verifies base-100 normalization
- `test_compare_tickers_chart` — verifies PNG saved to finance_output/charts/
- `test_correlation_uses_returns` — verifies output references "return" or "daily"
- `test_correlation_map_chart` — verifies PNG saved to finance_output/charts/
- `test_output_plain_english_first` — all 6 tools output[0].isalpha()
- `test_output_ends_with_disclaimer` — all 6 tools output.strip().endswith(DISCLAIMER)

## Verification Results

- `.venv/bin/python3 -m pytest tests/test_market_tools.py -v` → 15/15 PASSED
- `.venv/bin/python3 -m pytest tests/ -q` → 20 passed, 13 xpassed, 0 failures, 0 errors
- `.venv/bin/python3 -c "from finance_mcp.server import mcp"` → OK (no ImportError)
- `.venv/bin/python3 -c "from finance_mcp.tools.correlation import correlation_map; print('OK')"` → OK

## Decisions Made

1. **seaborn imported inside function body** — `output.py` sets `matplotlib.use("Agg")` at module scope before any pyplot import. Importing seaborn at module level in correlation.py would risk the Agg setting being bypassed. Importing inside the function body guarantees output.py has already run its module-level code.

2. **Return-based correlation via pct_change().dropna().corr()** — Price levels are non-stationary time series. Computing correlation on price levels inflates correlation artificially when two stocks share an upward trend. Return-based correlation measures co-movement of day-over-day changes, which is the financially meaningful metric.

3. **compare_tickers uses pd.DataFrame(prices_dict).dropna()** — Restricts the comparison to dates where all tickers have data, avoiding NaN distortions in normalization and chart rendering.

## Deviations from Plan

### Context deviation — plan 02-02 partially executed before this plan started

**Found during:** Initial inspection (pre-execution)
**Issue:** The STATE.md said `stopped_at: Completed 02-01-PLAN.md`, but `returns.py`, `volatility.py`, and `risk_metrics.py` already existed in the tools directory. The test stubs for those tools (tests 3-9) had already been replaced with real implementations. Only the 6 comparison/correlation/MKTX-07 stubs remained as `pytest.fail`.
**Action:** Treated the pre-existing tool files as given infrastructure. Proceeded with plan 02-03 scope only (comparison.py, correlation.py, last 6 test stubs, server.py registrations for all 6 tools).
**Impact:** None — all deliverables from this plan were correctly scoped and executed.

## Self-Check: PASSED

- src/finance_mcp/tools/comparison.py: FOUND
- src/finance_mcp/tools/correlation.py: FOUND
- Task 1 commit b689f05: verified
- Task 2 commit 0960d74: verified
- 15/15 market tool tests passing: verified
