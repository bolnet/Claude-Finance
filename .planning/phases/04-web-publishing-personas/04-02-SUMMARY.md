---
phase: 04-web-publishing-personas
plan: 02
subsystem: commands
tags: [claude-commands, persona, equity-analyst, portfolio-manager, mcp-tools]

# Dependency graph
requires:
  - phase: 04-web-publishing-personas
    provides: base finance.md command as template for persona variants
  - phase: 02-market-analysis-tools
    provides: MCP tools (analyze_stock, compare_tickers, get_returns, get_volatility, get_risk_metrics, correlation_map)
provides:
  - finance-analyst.md: sell-side equity analyst Claude Code command (PERS-01)
  - finance-pm.md: portfolio manager Claude Code command (PERS-02)
  - test_persona_commands.py: unit test suite for both persona command files
affects: [04-web-publishing-personas]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Persona command variant pattern: specialized role framing + intent routing over shared MCP tools, no new Python code"
    - "TDD for markdown command files: read_text() + regex frontmatter split + string assertions"

key-files:
  created:
    - .claude/commands/finance-analyst.md
    - .claude/commands/finance-pm.md
    - tests/test_persona_commands.py
  modified: []

key-decisions:
  - "Persona commands reuse all existing MCP tools — zero new Python code; differentiation is entirely in role framing and intent routing"
  - "!-injection context block included in persona files — these are .claude/commands/ files for Claude Code only, not claude.ai web commands"
  - "finance-analyst emphasizes peer comparison and Sharpe/drawdown vs S&P 500; finance-pm prioritizes drawdown/beta before Sharpe and treats tickers as portfolio holdings"
  - "Tests use raw text parsing (re.split on --- delimiters) — no yaml library needed; avoids adding test dependency"

patterns-established:
  - "Persona variant pattern: copy frontmatter from base command, expand allowed-tools list, replace Instructions with role-specific framing and intent routing table"
  - "Mandatory disclaimer: both persona commands end every response with the same disclaimer text — enforced in Output Rules section"

requirements-completed: [PERS-01, PERS-02]

# Metrics
duration: 4min
completed: 2026-03-18
---

# Phase 04 Plan 02: Persona Command Files Summary

**Two sell-side equity analyst and portfolio manager Claude Code command variants created with role-specific intent routing over the full existing MCP tool set — zero new Python code.**

## Performance

- **Duration:** 4 min
- **Started:** 2026-03-18T04:24:29Z
- **Completed:** 2026-03-18T04:28:35Z
- **Tasks:** 2
- **Files modified:** 3

## Accomplishments

- finance-analyst.md: equity analyst persona with peer-comparison-first emphasis, Sharpe/drawdown vs S&P 500 framing, seven intent routing rows
- finance-pm.md: portfolio manager persona with portfolio-level aggregation emphasis, drawdown/beta as primary risk metrics, eight intent routing rows
- test_persona_commands.py: nine unit tests (TDD) verify file existence, frontmatter fields, role framing text, disclaimer, and MCP tool presence for both files

## Task Commits

Each task was committed atomically:

1. **Task 1: Write test stubs for persona commands (Wave 0)** - `569a793` (test)
2. **Task 2: Create finance-analyst and finance-pm persona command files** - `8a7bd82` (feat)

**Plan metadata:** *(this commit)*

_Note: TDD tasks have two commits: test (RED) → feat (GREEN)_

## Files Created/Modified

- `.claude/commands/finance-analyst.md` — Equity analyst persona: sell-side framing, peer comparison emphasis, full MCP tool set
- `.claude/commands/finance-pm.md` — Portfolio manager persona: portfolio-level framing, drawdown/beta-first risk metrics, full MCP tool set
- `tests/test_persona_commands.py` — Nine unit tests for both command files; parse frontmatter with regex split, assert role framing and disclaimer text

## Decisions Made

- Persona commands reuse all existing MCP tools with zero new Python code. The differentiation is entirely in role framing and the intent routing table, keeping maintenance minimal.
- The `!`-injection context block is included in both persona files. The anti-pattern note in the plan research applied to claude.ai web commands; these are `.claude/commands/` files that only run in Claude Code, so `!` injection is valid.
- finance-analyst emphasizes Sharpe ratio and drawdown vs S&P 500 benchmark in a single-stock context; finance-pm leads with drawdown/beta before Sharpe and frames all tickers as portfolio holdings.
- Tests use `re.split(r'^---\s*$', text, flags=re.MULTILINE)` for frontmatter parsing — no yaml library needed, keeping the test file dependency-free.

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- PERS-01 and PERS-02 persona commands are ready for use via `/finance-analyst` and `/finance-pm` in Claude Code
- Full test suite: 47 passed, 13 xpassed (no failures)
- Phase 04 Plan 02 complete; remaining plans in phase 04 can proceed

---
*Phase: 04-web-publishing-personas*
*Completed: 2026-03-18*
