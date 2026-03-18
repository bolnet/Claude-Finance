---
phase: 08-persona-demos
plan: 01
subsystem: demo
tags: [persona, demo, slash-command, equity-analyst, portfolio-manager, get_risk_metrics]

# Dependency graph
requires:
  - phase: 07-ml-workflow-demos
    provides: demo.md Steps 1-11 covering all 11 MCP tools
  - phase: 04-web-publishing-personas
    provides: finance-analyst and finance-pm persona commands with framing patterns
provides:
  - demo.md Steps 12-14 showing persona framing live with get_risk_metrics data
  - Structural tests confirming persona demo steps exist in demo.md
affects: [08-persona-demos]

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Persona framing via instructional prose — demo.md simulates persona roles by telling Claude how to frame tool results, without invoking slash commands"
    - "Reuse-data pattern — Step 13 explicitly does not re-call the tool; reuses Step 12 output for contrast demonstration"

key-files:
  created:
    - tests/test_demo_personas.py
  modified:
    - .claude/commands/demo.md

key-decisions:
  - "demo.md cannot invoke /finance-analyst or /finance-pm slash commands directly — persona framing is simulated inline by instructing Claude with role-specific explanation rules"
  - "Step 13 explicitly skips get_risk_metrics call and reuses Step 12 data — this is intentional to demonstrate same-data-different-framing contrast"
  - "Demo summary updated to say '11 tools AND 2 personas (13 steps)' and added Personas table alongside tool tables"

patterns-established:
  - "Persona contrast pattern: run tool once (analyst framing), re-explain same data (PM framing), then print table showing difference — three steps total"

requirements-completed: [PERS-01, PERS-02, PERS-03]

# Metrics
duration: 5min
completed: 2026-03-18
---

# Phase 8 Plan 01: Persona Demos Summary

**Replaced text-only Persona Showcase with three executable demo steps (12-14) showing get_risk_metrics output framed through equity analyst and portfolio manager lenses with a side-by-side contrast table**

## Performance

- **Duration:** ~5 min
- **Started:** 2026-03-18T16:08:49Z
- **Completed:** 2026-03-18T16:13:30Z
- **Tasks:** 2
- **Files modified:** 2

## Accomplishments

- Step 12 calls get_risk_metrics for AAPL and explains output with equity analyst framing (Sharpe first, single-stock lens)
- Step 13 reuses Step 12 data and re-explains with portfolio manager framing (drawdown/beta first, holdings lens)
- Step 14 prints a 4-row contrast table showing exactly how the two personas differ in priority, framing, and next actions
- Old text-only Persona Showcase section removed
- Demo summary updated to reflect 11 tools + 2 personas with Personas table
- 5 structural tests in tests/test_demo_personas.py confirm all persona content present

## Task Commits

Each task was committed atomically:

1. **Task 1: Replace Persona Showcase with executable Steps 12-14** - `97921aa` (feat)
2. **Task 2: Structural tests for persona demo steps** - `ea09b64` (test)

**Plan metadata:** (docs commit below)

## Files Created/Modified

- `.claude/commands/demo.md` - Replaced Persona Showcase (lines 200-208) with Steps 12-14 and updated Demo Complete summary
- `tests/test_demo_personas.py` - 5 structural tests confirming Steps 12-14 exist with correct keywords and contrast table

## Decisions Made

- demo.md cannot invoke `/finance-analyst` or `/finance-pm` slash commands directly — persona framing is simulated inline by instructing Claude with role-specific explanation rules
- Step 13 explicitly does not re-call `get_risk_metrics` — reuses Step 12 output to demonstrate same-data-different-framing
- Demo summary updated to cover "11 tools AND 2 personas" with a Personas table alongside the tool summary tables

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- Phase 8 Plan 01 complete — persona demo steps live in demo.md
- Ready for Phase 8 Plan 02 (if defined) or milestone completion
- PERS-01, PERS-02, PERS-03 requirements fulfilled

## Self-Check

- [x] `.claude/commands/demo.md` — updated with Steps 12-14
- [x] `tests/test_demo_personas.py` — 5 tests, all pass
- [x] `97921aa` commit exists (Task 1)
- [x] `ea09b64` commit exists (Task 2)
- [x] `grep -c "## Step 12\|## Step 13\|## Step 14" .claude/commands/demo.md` returns 3
- [x] `grep -c "Persona Showcase" .claude/commands/demo.md` returns 0

## Self-Check: PASSED

---
*Phase: 08-persona-demos*
*Completed: 2026-03-18*
