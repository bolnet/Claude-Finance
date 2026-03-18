---
phase: 2
slug: market-analysis-tools
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-17
---

# Phase 2 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | pytest 9.0.2 |
| **Config file** | `pyproject.toml` (already configured) |
| **Quick run command** | `.venv/bin/python3 -m pytest tests/test_market_tools.py -x -q` |
| **Full suite command** | `.venv/bin/python3 -m pytest tests/ -v` |
| **Estimated runtime** | ~5 seconds |

---

## Sampling Rate

- **After every task commit:** Run `.venv/bin/python3 -m pytest tests/test_market_tools.py -x -q`
- **After every plan wave:** Run `.venv/bin/python3 -m pytest tests/ -v`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 10 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 2-01-01 | 01 | 0 | MKTX-01 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py -x -q` | Wave 0 | ⬜ pending |
| 2-01-02 | 01 | 1 | MKTX-01 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_analyze_stock_returns_output -x` | Wave 0 | ⬜ pending |
| 2-01-03 | 01 | 1 | MKTX-01 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_analyze_stock_saves_chart -x` | Wave 0 | ⬜ pending |
| 2-02-01 | 02 | 1 | MKTX-02 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_get_returns_values -x` | Wave 0 | ⬜ pending |
| 2-02-02 | 02 | 1 | MKTX-02 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_get_returns_chart -x` | Wave 0 | ⬜ pending |
| 2-02-03 | 02 | 1 | MKTX-03 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_annualized_volatility_formula -x` | Wave 0 | ⬜ pending |
| 2-02-04 | 02 | 1 | MKTX-03 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_volatility_chart -x` | Wave 0 | ⬜ pending |
| 2-02-05 | 02 | 1 | MKTX-04 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_sharpe_sign -x` | Wave 0 | ⬜ pending |
| 2-02-06 | 02 | 1 | MKTX-04 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_max_drawdown_nonpositive -x` | Wave 0 | ⬜ pending |
| 2-02-07 | 02 | 1 | MKTX-04 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_beta_calculation -x` | Wave 0 | ⬜ pending |
| 2-03-01 | 03 | 1 | MKTX-05 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_normalized_prices_start_at_100 -x` | Wave 0 | ⬜ pending |
| 2-03-02 | 03 | 1 | MKTX-05 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_compare_tickers_chart -x` | Wave 0 | ⬜ pending |
| 2-03-03 | 03 | 1 | MKTX-06 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_correlation_uses_returns -x` | Wave 0 | ⬜ pending |
| 2-03-04 | 03 | 1 | MKTX-06 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_correlation_map_chart -x` | Wave 0 | ⬜ pending |
| 2-07-01 | all | 1 | MKTX-07 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_output_plain_english_first -x` | Wave 0 | ⬜ pending |
| 2-07-02 | all | 1 | MKTX-07 | unit | `.venv/bin/python3 -m pytest tests/test_market_tools.py::test_output_ends_with_disclaimer -x` | Wave 0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `tests/test_market_tools.py` — 15 test stubs covering MKTX-01 through MKTX-07
- [ ] `src/finance_mcp/tools/__init__.py` — empty init to make tools a package

*Existing pytest infrastructure (conftest.py, pyproject.toml) carries forward from Phase 1.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Chart PNG is visually correct (axes labelled, readable) | MKTX-01 | Visual inspection required | Open `finance_output/charts/` and inspect saved PNGs |
| Correlation heatmap color scale is meaningful | MKTX-06 | Visual inspection required | Verify heatmap uses diverging colormap (-1 to +1) |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 10s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
