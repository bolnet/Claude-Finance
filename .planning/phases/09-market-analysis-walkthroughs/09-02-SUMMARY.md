---
phase: 09-market-analysis-walkthroughs
plan: 02
subsystem: walkthroughs
tags: [investment-banking, comparable-company-analysis, deal-pitch, comps, valuation, MSFT, GOOGL, AMZN, CRM, ORCL]

# Dependency graph
requires:
  - phase: 08-interactive-demo
    provides: walkthrough-equity-research.md template pattern and SKILL.md base
provides:
  - walkthrough-investment-banking slash command (5 phases, 19 steps)
  - IB intent routing in SKILL.md
  - Deal pitch comparable company analysis scenario covering IB-01 through IB-03
affects:
  - 09-01-market-analysis-walkthroughs (adds IB row to shared SKILL.md Role Walkthroughs table)
  - 12-testing (walkthrough-investment-banking will be tested in system test phase)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "Phase/step structure for IB deal pitch scenarios mirrors equity-research template"
    - "Exhibit A/B/C/D naming convention for pitch book quantitative sections"
    - "Bloomberg/CapIQ tool mapping table for IB context"

key-files:
  created:
    - .claude/commands/walkthrough-investment-banking.md
  modified:
    - .claude/skills/finance/SKILL.md

key-decisions:
  - "Comps universe: MSFT, GOOGL, AMZN, CRM, ORCL (cloud/tech M&A deal pitch targets per IB-02)"
  - "IB framing uses Exhibit A/B/C/D naming to match pitch book conventions"
  - "Steps 9-10 dynamically pick top/bottom LTM performers from Step 7 output rather than hardcoding"
  - "Step 16 and Step 19 are no-tool synthesis steps — pure analysis, matching equity-research pattern"
  - "Tool mapping table uses Bloomberg/CapIQ/FactSet equivalents specific to IB workflows"

patterns-established:
  - "IB walkthrough pattern: per-comp profiles (Phase 2) -> relative performance exhibits (Phase 3) -> risk table (Phase 4) -> correlation/transaction rationale (Phase 5)"
  - "Deal pitch commentary framing for every step output"

requirements-completed: [IB-01, IB-02, IB-03]

# Metrics
duration: 3min
completed: 2026-03-18
---

# Phase 09 Plan 02: Investment Banking Walkthrough Summary

**Investment banking comparable company analysis walkthrough with 5-ticker comps (MSFT/GOOGL/AMZN/CRM/ORCL), Exhibit A–D pitch book exhibits, and deal pitch framing across 5 phases and 19 steps**

## Performance

- **Duration:** 3 min
- **Started:** 2026-03-18T20:44:46Z
- **Completed:** 2026-03-18T20:47:49Z
- **Tasks:** 2
- **Files modified:** 2

## Accomplishments
- Created `/walkthrough-investment-banking` slash command — 5-phase, 19-step auto-running scenario covering IB-01 through IB-03
- Built Phase 2 with per-comp LTM profiles for all 5 comps, Phase 3 with relative performance exhibits (LTM + 90-day), Phase 4 with risk profile Exhibit C (Sharpe/drawdown/beta), and Phase 5 with correlation Exhibit D and transaction rationale framing
- Updated SKILL.md intent classification and Role Walkthroughs table to route IB queries to the new command

## Task Commits

Each task was committed atomically:

1. **Task 1: Create walkthrough-investment-banking.md slash command** - `3c5cc4e` (feat)
2. **Task 2: Update SKILL.md intent routing for investment banking walkthrough** - `7944219` (feat)

**Plan metadata:** (docs commit follows)

## Files Created/Modified
- `.claude/commands/walkthrough-investment-banking.md` - 5-phase, 19-step IB deal pitch comparable company analysis walkthrough command
- `.claude/skills/finance/SKILL.md` - Added walkthrough-investment-banking intent row and IB entry to Role Walkthroughs table

## Decisions Made
- Comps universe (MSFT, GOOGL, AMZN, CRM, ORCL) chosen per plan specification covering cloud/tech M&A deal pitch names for IB-02
- Steps 9 and 10 instruct Claude to dynamically identify top/bottom LTM performers from Step 7 output rather than hardcoding tickers — makes the walkthrough accurate regardless of market conditions
- Exhibit A/B/C/D naming convention mirrors real pitch book section naming
- Step 16 and Step 19 are pure analysis steps (no tool calls), matching the equity-research template pattern for synthesis steps
- Tool mapping table uses Bloomberg/CapIQ/FactSet terminology specific to IB workflows (vs FactSet-focused equity research table)

## Deviations from Plan

None - plan executed exactly as written.

Note: SKILL.md already had Plan 01's Role Walkthroughs section and hedge-fund intent row (Plan 01 had run in parallel). The executor correctly detected existing content and appended the IB row rather than recreating the full section.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness
- Phase 9 complete: both walkthroughs (hedge fund Plan 01, investment banking Plan 02) are ready
- Phase 10 (FPA + Accounting walkthroughs) can begin
- `/walkthrough-investment-banking` is discoverable via `/finance` SKILL.md intent routing and directly via slash command

---
*Phase: 09-market-analysis-walkthroughs*
*Completed: 2026-03-18*
