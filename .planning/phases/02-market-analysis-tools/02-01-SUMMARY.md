---
phase: 02-market-analysis-tools
plan: "01"
subsystem: market-tools
tags: [tdd, mcp-tool, price-chart, test-scaffold, phase-2]
dependency_graph:
  requires: []
  provides:
    - tests/test_market_tools.py (15 stubs for MKTX-01 through MKTX-07)
    - src/finance_mcp/tools/ (importable tools package)
    - src/finance_mcp/tools/price_chart.py (analyze_stock MCP tool)
    - mcp.add_tool(analyze_stock) registration in server.py
  affects:
    - 02-02 (depends on test stubs for TDD cycles)
    - 02-03 (depends on test stubs for TDD cycles)
tech_stack:
  added:
    - src/finance_mcp/tools/ Python package
  patterns:
    - mcp.add_tool() for Phase 2 tool registration (not @mcp.tool decorator)
    - output.py imported first to set Agg backend before pyplot
key_files:
  created:
    - tests/test_market_tools.py
    - src/finance_mcp/tools/__init__.py
    - src/finance_mcp/tools/price_chart.py
  modified:
    - src/finance_mcp/server.py
decisions:
  - "Use mcp.add_tool() (not @mcp.tool decorator) for all Phase 2 tool imports — keeps registration explicit and avoids decorator binding issues on imported functions"
  - "Import finance_mcp.output before matplotlib.pyplot in price_chart.py — ensures Agg backend is set before any pyplot call"
metrics:
  duration: 2 min
  completed: "2026-03-18"
  tasks_completed: 2
  files_changed: 4
---

# Phase 2 Plan 1: Test Scaffold and analyze_stock Tool Summary

**One-liner:** TDD scaffold with 15 test stubs and analyze_stock price-chart tool registered via mcp.add_tool()

## What Was Built

Wave 0 gate for Phase 2. This plan establishes two foundations that downstream plans depend on:

1. **Test scaffold** (`tests/test_market_tools.py`) — 15 named test stubs that fail loudly (`pytest.fail`) with exact names matching VALIDATION.md requirements MKTX-01 through MKTX-07. Downstream plans (02-02, 02-03) implement tools by replacing these stubs.

2. **tools/ package** (`src/finance_mcp/tools/__init__.py`) — Empty package marker making `from finance_mcp.tools.x import y` importable. Establishes the module pattern for all Phase 2 tools.

3. **analyze_stock tool** (`src/finance_mcp/tools/price_chart.py`) — MCP tool that fetches adjusted price history via the adapter, builds a matplotlib price chart, saves PNG to `finance_output/charts/`, and returns a formatted plain-English output string including chart path and mandatory disclaimer. Raises `ToolError` on `DataFetchError`.

4. **server.py updated** — `analyze_stock` imported from `tools/price_chart` and registered with `mcp.add_tool(analyze_stock)`. Existing `ping` and `validate_environment` tools unchanged.

## Verification Results

- `.venv/bin/python3 -m pytest tests/test_market_tools.py` → 2 passed, 13 failed (stubs), 0 errors
- `.venv/bin/python3 -c "from finance_mcp.server import mcp"` → OK (no ImportError)
- `.venv/bin/python3 -m pytest tests/ ` → 7 passed, 13 xpassed, 13 failed (stubs only), 0 regressions

## Decisions Made

1. **mcp.add_tool() over @mcp.tool decorator for Phase 2** — Functions imported from submodules must be registered explicitly; the decorator approach only works reliably when defined in the same module scope as the `mcp` instance.

2. **output.py imported first in price_chart.py** — `finance_mcp.output` calls `matplotlib.use("Agg")` at module scope before any `pyplot` import. This ordering is mandated and enforced by test_output_format.py.

## Deviations from Plan

None — plan executed exactly as written.

## Self-Check: PASSED

- tests/test_market_tools.py: FOUND
- src/finance_mcp/tools/__init__.py: FOUND
- src/finance_mcp/tools/price_chart.py: FOUND
- Task 1 commit 8438766: verified
- Task 2 commit 8fa5d68: verified
