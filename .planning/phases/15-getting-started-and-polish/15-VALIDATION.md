---
phase: 15
slug: getting-started-and-polish
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-18
---

# Phase 15 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | pytest (existing) |
| **Config file** | none — runs via `python3 -m pytest tests/` |
| **Quick run command** | `python3 -m pytest tests/ -x -q` |
| **Full suite command** | `python3 -m pytest tests/ -v` |
| **Estimated runtime** | ~15 seconds |

---

## Sampling Rate

- **After every task commit:** Run `python3 -m pytest tests/ -x -q`
- **After every plan wave:** Run `python3 -m pytest tests/ -v` + browser spot-check
- **Before `/gsd:verify-work`:** Full suite must be green + all manual checks passed
- **Max feedback latency:** 15 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 15-01-01 | 01 | 1 | START-01 | manual | Open getting-started.html; verify Claude Code path with numbered steps | N/A | ⬜ pending |
| 15-01-02 | 01 | 1 | START-02 | manual | Open getting-started.html; verify claude.ai path with ngrok steps | N/A | ⬜ pending |
| 15-01-03 | 01 | 1 | START-01 | automated | `python3 -c "import json; json.loads(open('finance-mcp-plugin/.mcp.json').read()); print('valid')"` | ✅ | ⬜ pending |
| 15-02-01 | 02 | 1 | INFRA-04 | automated | `grep -h 'og:image' docs/*.html` — all 4 must contain `https://` | N/A | ⬜ pending |
| 15-02-02 | 02 | 1 | INFRA-04 | manual | opengraph.xyz shows rich card with title, description, chart image | N/A | ⬜ pending |
| 15-02-03 | 02 | 2 | Phase SC | manual | 16 nav link clicks at live URL — zero 404s | N/A | ⬜ pending |
| 15-02-04 | 02 | 2 | Phase SC | manual | Lighthouse Performance >= 80 on all 4 pages | N/A | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements. Phase 15 introduces no new Python code — all verification is manual or HTML-grep based.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Claude Code install path readable by non-developer | START-01 | Subjective readability check | Read each step; verify plain English, no unexplained jargon |
| claude.ai install path with ngrok warning | START-02 | Subjective readability + callout check | Verify `.callout-note` element warns about URL regeneration |
| LinkedIn rich preview card | INFRA-04 | Requires live deployment + external validator | Visit opengraph.xyz with deployed URL; verify image appears |
| Navigation links resolve | Phase SC | Requires live deployment | Click all 16 nav links at bolnet.github.io/Claude-Finance |
| Lighthouse Performance >= 80 | Phase SC | Requires live deployment + Chrome DevTools | Run Lighthouse on all 4 pages via DevTools |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 15s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
