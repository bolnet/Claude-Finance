---
phase: 04-web-publishing-personas
plan: 03
subsystem: infra
tags: [mcp, plugin, ngrok, claude.ai, marketplace, shell-script, pyproject]

# Dependency graph
requires:
  - phase: 04-web-publishing-personas
    provides: HTTP transport server (server_http.py) and persona commands (finance-analyst.md, finance-pm.md)

provides:
  - scripts/start_web.sh — one-command ngrok launcher with three-step claude.ai connector guide (WEB-02)
  - finance-mcp-plugin/ — full plugin directory structure for marketplace submission (WEB-03)
  - finance-mcp-plugin/.claude-plugin/plugin.json — validated marketplace manifest
  - finance-mcp-plugin/.mcp.json — pip-installed entry point (no .venv path)
  - finance-mcp-plugin/commands/ — finance.md, finance-analyst.md, finance-pm.md
  - finance-mcp-plugin/skills/finance/SKILL.md — full skill definition for plugin consumers
  - pyproject.toml [project.scripts] — finance-mcp-http console entry point
  - tests/test_plugin_manifest.py — 6-test suite validating plugin manifest structure

affects: [distribution, marketplace-submission, onboarding]

# Tech tracking
tech-stack:
  added: [ngrok (external prereq, not installed), pyproject.toml scripts entry point]
  patterns: [TDD RED/GREEN for file-existence assertions, plugin manifest validation via pathlib]

key-files:
  created:
    - scripts/start_web.sh
    - finance-mcp-plugin/.claude-plugin/plugin.json
    - finance-mcp-plugin/.mcp.json
    - finance-mcp-plugin/commands/finance.md
    - finance-mcp-plugin/commands/finance-analyst.md
    - finance-mcp-plugin/commands/finance-pm.md
    - finance-mcp-plugin/skills/finance/SKILL.md
    - tests/test_plugin_manifest.py
  modified:
    - pyproject.toml

key-decisions:
  - "Plugin .mcp.json uses 'python -m finance_mcp.server' (pip-installed), not a .venv relative path — distributable without project clone"
  - "start_web.sh reads ngrok URL from localhost:4040 API after 3s sleep — avoids parsing ngrok stdout which varies by version"
  - "Cloudflare Tunnel noted in start_web.sh as persistent-URL alternative to ngrok free tier"

patterns-established:
  - "Plugin package mirrors .claude/ structure — commands/ and skills/finance/ subdirectories identical to development layout"
  - "TDD RED via file-existence assertions — pytest collect + run before files exist guarantees RED before GREEN"

requirements-completed: [WEB-02, WEB-03]

# Metrics
duration: 8min
completed: 2026-03-18
---

# Phase 4 Plan 03: Plugin Package and Launch Script Summary

**ngrok one-command launcher (start_web.sh) + full claude.ai plugin directory (finance-mcp-plugin/) with validated manifest and 6-test suite**

## Performance

- **Duration:** 8 min
- **Started:** 2026-03-18T04:30:25Z
- **Completed:** 2026-03-18T04:38:00Z
- **Tasks:** 2 (TDD: RED commit + GREEN commit)
- **Files modified:** 9

## Accomplishments

- scripts/start_web.sh: checks ngrok + .venv prerequisites, starts HTTP server on configurable port, tunnels via ngrok, prints exact three-step URL for claude.ai Connectors UI
- finance-mcp-plugin/: complete marketplace-ready directory with plugin.json manifest, three command files, SKILL.md, and .mcp.json pointing to pip-installed module
- pyproject.toml [project.scripts] entry point added for finance-mcp-http
- 6 plugin manifest tests: all RED before implementation, all GREEN after (53 passed, 13 xpassed in full suite — zero regressions)

## Task Commits

Each task was committed atomically:

1. **Task 1: Write test stubs for plugin manifest (TDD RED)** - `3cef681` (test)
2. **Task 2: Create plugin package, launch script, and pyproject.toml entry point** - `b965f94` (feat)

**Plan metadata:** (docs commit — see final commit)

## Files Created/Modified

- `scripts/start_web.sh` - ngrok-based claude.ai web launcher with prerequisite checks and URL output
- `finance-mcp-plugin/.claude-plugin/plugin.json` - marketplace manifest with name, version, description, author.name, keywords
- `finance-mcp-plugin/.mcp.json` - MCP server config using python -m (pip install path)
- `finance-mcp-plugin/commands/finance.md` - general finance command (copied from .claude/commands)
- `finance-mcp-plugin/commands/finance-analyst.md` - equity analyst persona command
- `finance-mcp-plugin/commands/finance-pm.md` - portfolio manager persona command
- `finance-mcp-plugin/skills/finance/SKILL.md` - full skill definition for plugin consumers
- `tests/test_plugin_manifest.py` - 6 tests validating plugin directory structure and manifest fields
- `pyproject.toml` - added [project.scripts] finance-mcp-http entry point

## Decisions Made

- Plugin .mcp.json uses `python -m finance_mcp.server` (not a `.venv` path) — makes the plugin distributable to users who pip-install without cloning the full project repo
- start_web.sh reads the ngrok public URL from the localhost:4040 API (after a 3-second sleep) rather than parsing ngrok stdout — ngrok stdout format varies by version
- Cloudflare Tunnel documented in start_web.sh as the recommended alternative for a persistent URL (ngrok free tier URL changes on each restart)

## Deviations from Plan

None - plan executed exactly as written.

## Issues Encountered

None.

## User Setup Required

None - no external service configuration required beyond ngrok installation (documented in start_web.sh usage header).

## Next Phase Readiness

Phase 4 is complete. All four plans delivered:
- 04-01: HTTP transport (server_http.py, streamable-http on /mcp)
- 04-02: Persona commands (finance-analyst.md, finance-pm.md)
- 04-03: Plugin package + launch script (finance-mcp-plugin/, scripts/start_web.sh)

The skill is now distributable via:
1. Claude Code (stdio MCP via .mcp.json in project root)
2. claude.ai web (bash scripts/start_web.sh then paste URL into Connectors)
3. Plugin marketplace (finance-mcp-plugin/ directory, pending marketplace submission)

---
*Phase: 04-web-publishing-personas*
*Completed: 2026-03-18*
