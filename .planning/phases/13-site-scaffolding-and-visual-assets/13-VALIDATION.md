---
phase: 13
slug: site-scaffolding-and-visual-assets
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-18
---

# Phase 13 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | Browser DevTools + filesystem checks (no pytest for HTML/CSS) |
| **Config file** | none — static HTML, no test framework needed |
| **Quick run command** | `ls docs/assets/images/*.png \| wc -l && du -sh docs/assets/images/*.png` |
| **Full suite command** | `open docs/index.html && echo "Check DevTools for 404s"` |
| **Estimated runtime** | ~5 seconds (filesystem), manual browser check ~30s |

---

## Sampling Rate

- **After every task commit:** Run `ls docs/assets/images/*.png | wc -l && du -sh docs/assets/images/*.png`
- **After every plan wave:** Open `docs/index.html` locally and check DevTools console for 404s
- **Before `/gsd:verify-work`:** Full browser check must show zero 404s + mobile viewport test
- **Max feedback latency:** 5 seconds (filesystem checks)

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 13-01-01 | 01 | 1 | INFRA-01 | filesystem | `test -f docs/.nojekyll && echo OK` | ❌ W0 | ⬜ pending |
| 13-01-02 | 01 | 1 | INFRA-01 | filesystem | `test -f docs/index.html && echo OK` | ❌ W0 | ⬜ pending |
| 13-01-03 | 01 | 1 | INFRA-02 | manual | Open index.html, verify nav/footer present | ❌ W0 | ⬜ pending |
| 13-01-04 | 01 | 1 | INFRA-03 | manual | Resize browser to 375px, check no horizontal scroll | ❌ W0 | ⬜ pending |
| 13-02-01 | 02 | 1 | VIS-01 | filesystem | `ls docs/assets/images/*.png \| wc -l` (expect 6-8) | ❌ W0 | ⬜ pending |
| 13-02-02 | 02 | 1 | VIS-02 | filesystem | `du -b docs/assets/images/*.png` (each <150KB) | ❌ W0 | ⬜ pending |
| 13-02-03 | 02 | 1 | VIS-03 | filesystem | `test -f docs/favicon.ico && echo OK` | ❌ W0 | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `docs/` directory structure created
- [ ] `docs/.nojekyll` file exists

*Existing Python test infrastructure is unaffected — Phase 13 delivers HTML/CSS/images only.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Placeholder page loads with zero 404s | INFRA-01 | Requires live browser DevTools | Open docs/index.html, open DevTools Console, verify no red errors |
| All paths are relative (no leading `/`) | INFRA-01 | Requires HTML source inspection | grep for `href="/"` and `src="/"` in all HTML files — expect zero matches |
| Mobile responsive at 375px | INFRA-03 | Requires visual viewport check | Chrome DevTools → Toggle device toolbar → iPhone SE (375px) → no horizontal scroll |
| Shared template across pages | INFRA-02 | Requires visual comparison | Compare head/nav/footer HTML blocks across index.html and placeholder pages |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 30s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
