---
phase: 11-ml-classifier-walkthrough
plan: "01"
subsystem: walkthroughs
tags: [investor_classifier, classify_investor, ingest_csv, private-equity, due-diligence, ml-classifier]

# Dependency graph
requires:
  - phase: 10-data-profiling-walkthroughs
    provides: walkthrough-accounting.md pattern (same MCP tools, same frontmatter format)
provides:
  - walkthrough-private-equity.md slash command (PE/VC due diligence scenario, 4 phases, 12 steps)
  - SKILL.md PE intent routing (intent table row, Role Walkthroughs row, routing paragraph)
  - 6th and final role-based walkthrough completing the v1.2 Role Walkthroughs milestone
affects:
  - 12-walkthrough-testing (tests all 6 walkthroughs together)

# Tech tracking
tech-stack:
  added: []
  patterns:
    - "PE/VC framing of investor classifier: classify_investor confidence scores as IC conviction level"
    - "Portfolio monitoring via confidence drift detection: low confidence = thesis drift signal"
    - "Pure analysis steps between tool calls for IC memo, deal screening matrix, portfolio dashboard"

key-files:
  created:
    - .claude/commands/walkthrough-private-equity.md
  modified:
    - .claude/skills/finance/SKILL.md

key-decisions:
  - "PE/VC reframing: investor_classifier trained model used as IC scoring baseline; accuracy = historical IC consistency rate"
  - "Confidence score dual-use: serves as prospect conviction level (new deals) and thesis drift signal (existing portfolio companies)"
  - "4-phase structure mirrors accounting walkthrough but substitutes PE terminology throughout (LBO, growth equity, venture, IC memo, LP reporting)"
  - "Three distinct prospect profiles chosen to span the fund's strategy spectrum: growth equity (age=42, income=145K, rt=0.55), buyout (age=58, income=190K, rt=0.25), venture (age=26, income=72K, rt=0.88)"

patterns-established:
  - "Dual-use classifier: scoring model output reused for both prospect screening and portfolio monitoring"
  - "IC memo synthesis as final pure analysis step synthesizing all quantitative findings"
  - "Feature importance as due diligence priority framework — top features define diligence interview agenda"

requirements-completed: [PE-01, PE-02, PE-03]

# Metrics
duration: 4min
completed: 2026-03-18
---

# Phase 11 Plan 01: Private Equity Walkthrough Summary

**PE/VC due diligence walkthrough using investor_classifier for prospect scoring, side-by-side comparison matrix, and portfolio monitoring via confidence drift detection**

## Performance

- **Duration:** 4 min
- **Started:** 2026-03-18T21:33:37Z
- **Completed:** 2026-03-18T21:37:14Z
- **Tasks:** 2
- **Files modified:** 2

## Accomplishments

- Created 404-line `/walkthrough-private-equity` slash command with 4 phases and 12 steps, fully framed in PE/VC language
- classify_investor called 3 times for distinct prospect archetypes (growth equity, buyout, venture) with detailed IC interpretation for each classification result
- Portfolio monitoring dashboard reinterprets confidence scores as thesis drift signals for existing holdings
- SKILL.md updated with PE intent routing in all 3 locations (intent table, Role Walkthroughs table, routing paragraph)

## Task Commits

Each task was committed atomically:

1. **Task 1: Create walkthrough-private-equity.md slash command** - `6bdacc7` (feat)
2. **Task 2: Update SKILL.md intent routing for private equity walkthrough** - `bc13e8e` (feat)

## Files Created/Modified

- `.claude/commands/walkthrough-private-equity.md` - 4-phase PE/VC due diligence walkthrough slash command (404 lines)
- `.claude/skills/finance/SKILL.md` - Added PE intent row, Role Walkthroughs row, and routing paragraph

## Decisions Made

- PE/VC reframing: investor_classifier accuracy interpreted as "historical IC consistency rate" — high accuracy = fund committee has been disciplined and consistent in its allocation decisions
- Confidence score dual-use: the same classify_investor call serves as both a prospect screening conviction score (high confidence = IC recommendation) and a portfolio monitoring drift signal (low confidence on existing holding = thesis has shifted since acquisition)
- Three prospect profiles span the full strategy spectrum — growth equity (moderate risk, established revenue), buyout (low risk, high revenue, debt-oriented), venture (high risk, early-stage, equity-oriented) — chosen to produce maximally distinct classifications for IC comparison
- Feature importance reframed as due diligence priority framework: the deal team front-loads whichever features drive the model into their management interview agenda

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required.

## Next Phase Readiness

- All 6 role-based walkthroughs are now complete (equity-research, hedge-fund, investment-banking, accounting, fpa, private-equity)
- Phase 12 (walkthrough testing) can now test all 6 walkthroughs end-to-end
- No blockers

---
*Phase: 11-ml-classifier-walkthrough*
*Completed: 2026-03-18*
