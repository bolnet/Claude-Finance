---
phase: 14
slug: content-pages
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-18
---

# Phase 14 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Browser DevTools + HTML content checks (no pytest for HTML) |
| **Config file** | none — static HTML content |
| **Quick run command** | `grep -c 'class="hero"' docs/index.html && grep -c 'class="tool-card"' docs/features.html && grep -c 'class="role-card"' docs/walkthroughs.html` |
| **Full suite command** | `open docs/index.html && echo "Check content, links, images"` |
| **Estimated runtime** | ~5 seconds (grep), manual browser check ~60s |

---

## Sampling Rate

- **After every task commit:** Run quick content structure checks (grep for key CSS classes)
- **After every plan wave:** Open pages locally and verify content renders correctly
- **Before `/gsd:verify-work`:** Full browser check on all 3 pages + mobile viewport
- **Max feedback latency:** 5 seconds (grep checks)

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 14-01-01 | 01 | 1 | LAND-01 | content | `grep -c 'class="hero"' docs/index.html` | ✅ | ⬜ pending |
| 14-01-02 | 01 | 1 | LAND-02 | content | `grep -c '<img' docs/index.html` (expect ≥1 chart) | ✅ | ⬜ pending |
| 14-01-03 | 01 | 1 | LAND-03 | content | `grep -c 'walkthroughs.html#' docs/index.html` (expect ≥6 links) | ✅ | ⬜ pending |
| 14-01-04 | 01 | 1 | LAND-04 | content | `grep -c 'class="stats' docs/index.html` | ✅ | ⬜ pending |
| 14-02-01 | 02 | 1 | FEAT-01 | content | `grep -c 'class="tool-card"' docs/features.html` (expect ≥11) | ✅ | ⬜ pending |
| 14-02-02 | 02 | 1 | FEAT-02 | content | `grep -c '<img' docs/features.html` (expect ≥2 category images) | ✅ | ⬜ pending |
| 14-03-01 | 03 | 1 | WALK-01 | content | `grep -c 'class="role-card"' docs/walkthroughs.html` (expect 6) | ✅ | ⬜ pending |
| 14-03-02 | 03 | 1 | WALK-02 | content | `grep -c '<img' docs/walkthroughs.html` (expect ≥6 chart images) | ✅ | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

*Existing infrastructure covers all phase requirements — Phase 13 delivered HTML template, CSS, and curated images.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Hero copy is outcome-led (no jargon) | LAND-01 | Requires human judgment on copy quality | Read hero text — should say what finance pros GET, not how it works |
| Role entry points link to correct walkthrough cards | LAND-03 | Requires cross-page navigation check | Click each role link on landing page → verify it scrolls to correct card on walkthroughs.html |
| Tool descriptions use plain language | FEAT-01 | Requires human judgment on jargon | Read each tool description — no mentions of MCP, FastMCP, stdio, transport |
| Walkthrough cards match role scenarios | WALK-01 | Requires content accuracy check | Compare each card's scenario text against source walkthrough command files |
| Mobile layout renders correctly | LAND-01 | Requires visual viewport check | Check all 3 pages at 375px — cards stack, images resize, no overflow |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 30s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
