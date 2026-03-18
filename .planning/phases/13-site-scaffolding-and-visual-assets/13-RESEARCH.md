# Phase 13: Site Scaffolding and Visual Assets — Research

**Researched:** 2026-03-18
**Domain:** GitHub Pages static site scaffolding — docs/ folder, .nojekyll, relative paths, HTML template, image curation and optimization
**Confidence:** HIGH

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| INFRA-01 | Site deploys from `docs/` folder on main branch with `.nojekyll` file | GitHub Docs (official): docs/ branch source pattern + .nojekyll behavior fully documented; confirmed zero build-step needed |
| INFRA-02 | Shared HTML template (head, nav bar, footer) used by all pages | Standard HTML include pattern; 4-page flat structure makes copy-pasted template viable without a build tool |
| INFRA-03 | All pages are mobile responsive with proper viewport meta | `<meta name="viewport" content="width=device-width, initial-scale=1">` in shared head template; CSS media queries cover the rest |
| VIS-01 | 6-8 best charts curated from `finance_output/charts/` | 60+ source PNGs confirmed present; 6 primary candidates identified with sizes measured; curation criteria documented |
| VIS-02 | All site images web-optimized (800px wide, <150KB each) | Source file sizes measured: 5 of 6 primary candidates already under 150KB; compare-tech-stocks.png (185KB) marginally over and needs resize; macOS `sips` or Python PIL/Pillow for resize |
| VIS-03 | Favicon for browser tab | Standard `<link rel="icon">` in shared head; 32x32 or 16x16 PNG or ICO; can be generated from a chart color or finance-themed emoji via favicon generator |
</phase_requirements>

---

## Summary

Phase 13 is the infrastructure foundation for the entire v1.3 GitHub Pages site. It has no predecessor content pages — every task in this phase is a prerequisite for Phases 14 and 15. The phase delivers three concrete artifacts: (1) a working GitHub Pages deployment of a styled placeholder `index.html` with zero 404s in DevTools, (2) the complete `docs/` folder structure with shared HTML template, viewport meta, and relative-path conventions locked in, and (3) 6-8 curated chart PNGs in `docs/assets/images/` with stable descriptive filenames.

The project-level research (SUMMARY.md, ARCHITECTURE.md, STACK.md, PITFALLS.md) has already resolved all architectural questions for this phase. The stack decision is locked: plain HTML in `docs/`, no Jekyll, no build step. The `.nojekyll` file is the first committed artifact. All asset paths must be relative — the GitHub Pages project site URL prefix (`/machine_learning_skill/`) makes root-absolute paths a fatal mistake. The existing `finance_output/charts/` directory contains 60+ source PNGs; file sizes for the 6 primary candidate charts have been verified — five are already under 150KB and one (compare-tech-stocks, 185KB) needs a web-optimization resize pass.

The critical path for this phase is: (1) enable GitHub Pages in repository Settings before writing any HTML, (2) commit `.nojekyll` as the first file in `docs/`, (3) establish the docs/ folder structure and HTML template with relative paths, (4) verify deployment on the live URL before any content is written, (5) curate and optimize chart images. None of these steps can be reordered without creating rework.

**Primary recommendation:** Create the `docs/` folder structure and HTML template first, commit `.nojekyll` immediately, configure GitHub Pages settings before writing the first line of HTML, and verify the live deployment URL loads styled with zero DevTools 404s before proceeding to image curation.

---

## Standard Stack

### Core

| Technology | Version | Purpose | Why Standard |
|------------|---------|---------|--------------|
| Plain HTML5 | — | All 4 page files | No build step; immediate GitHub Pages compatibility; zero version conflicts with Python project |
| Plain CSS3 | — | Shared stylesheet at `docs/assets/css/style.css` | No preprocessor needed at 4-page scale; media queries handle mobile |
| GitHub Pages (docs/ source) | — | Static hosting | Native GitHub feature; branch source mode; auto-deploys on push to main; live in <60 seconds |
| `.nojekyll` (empty file) | — | Disables Jekyll processing | Prevents Jekyll from mis-processing Python project structure; required for plain HTML approach |

### Supporting

| Tool | Purpose | When to Use |
|------|---------|-------------|
| macOS `sips` command | Resize PNGs to 800px wide without installing anything | For the compare-tech-stocks.png (185KB) that marginally exceeds 150KB; available on macOS without any install |
| Python Pillow (PIL) | Alternative resize/optimize if sips produces unsatisfactory output | Available in the existing project Python environment; `pip install Pillow` |
| favicon.io or realfavicongenerator.net | Generate favicon.ico / PNG from text or image | One-time; produces the 16x16 and 32x32 PNGs needed |
| `gh` CLI | Verify Pages deployment URL and status post-push | `gh api repos/{owner}/{repo}/pages` shows deployment state |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| Plain HTML | Jekyll 4.4 + Minimal Mistakes | Jekyll adds Ruby toolchain, Gemfile, GitHub Actions workflow — unjustified for 4 pages |
| Plain HTML | Eleventy (11ty) | Node toolchain adds complexity; no benefit at 4-page scale |
| Relative paths | `<base href="/machine_learning_skill/">` | base tag works but complicates anchor links; relative paths are simpler and more portable |
| Manual image copy | Symlink `finance_output/charts/` | GitHub Pages does not follow symlinks; physical copy required |

**Installation:** No npm or gem install required. This phase has zero new package dependencies.

---

## Architecture Patterns

### Recommended Project Structure

```
machine_learning_skill/        # repo root — Python project unchanged
├── src/finance_mcp/           # existing — untouched
├── finance_output/charts/     # existing — source PNGs live here
├── docs/                      # NEW — GitHub Pages root
│   ├── .nojekyll              # FIRST FILE — disables Jekyll
│   ├── index.html             # placeholder landing page (Phase 13)
│   ├── features.html          # stub (Phase 14)
│   ├── walkthroughs.html      # stub (Phase 14)
│   ├── getting-started.html   # stub (Phase 15)
│   └── assets/
│       ├── css/
│       │   └── style.css      # shared stylesheet
│       ├── js/
│       │   └── main.js        # mobile nav toggle (10-15 lines)
│       └── images/            # 6-8 curated PNGs, stable names
│           ├── compare-tech-stocks.png
│           ├── compare-semiconductor-stocks.png
│           ├── correlation-heatmap.png
│           ├── volatility-analysis.png
│           ├── confusion-matrix.png
│           ├── feature-importance.png
│           ├── eda-credit-risk.png
│           └── residual-plot.png
├── pyproject.toml             # existing — untouched
└── requirements.txt           # existing — untouched
```

### Pattern 1: docs/ Folder Source (No Build Step)

**What:** GitHub Pages reads `docs/` on every push to main. No Actions workflow needed. Every push deploys automatically.

**When to use:** Always for this project. The plain HTML approach is the locked decision.

**Configuration (one-time, do first):**
```
GitHub repo → Settings → Pages
  Source: Deploy from a branch
  Branch: main
  Folder: /docs
```

Verify this is saved BEFORE committing any HTML. The environment must exist before the first push.

### Pattern 2: Shared HTML Template (Copy-Paste at 4 Pages)

**What:** All 4 pages share an identical `<head>`, `<nav>`, and `<footer>` block. At 4 pages, copy-pasting the shared sections is acceptable and avoids any build-tool dependency.

**When to use:** For Phase 13, the placeholder `index.html` establishes the canonical template. Phases 14-15 copy this template exactly.

**Head template:**
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Finance AI Skill</title>
  <meta name="description" content="[Page-specific 150-character description]">
  <meta property="og:title" content="Finance AI Skill">
  <meta property="og:description" content="[Description]">
  <meta property="og:image" content="[ABSOLUTE URL — Phase 15 task]">
  <meta property="og:url" content="[Canonical page URL]">
  <link rel="icon" href="assets/images/favicon.png" type="image/png">
  <link rel="stylesheet" href="assets/css/style.css">
</head>
```

**Nav template (all pages at docs/ root level):**
```html
<nav>
  <a href="index.html">Home</a>
  <a href="features.html">Features</a>
  <a href="walkthroughs.html">Walkthroughs</a>
  <a href="getting-started.html">Get Started</a>
</nav>
```

Note: Because all pages are at the same `docs/` directory level, nav hrefs are simple filenames with no `../` needed.

### Pattern 3: Relative Paths — The Non-Negotiable Rule

**What:** All `href` and `src` attributes use relative paths without a leading `/`.

**When to use:** Always. No exceptions. Root-absolute paths cause 404s on GitHub Pages project sites.

**Correct vs incorrect:**
```html
<!-- CORRECT — from docs/index.html -->
<link rel="stylesheet" href="assets/css/style.css">
<img src="assets/images/compare-tech-stocks.png" alt="Tech stock comparison">
<a href="features.html">Features</a>

<!-- WRONG — root-absolute paths cause 404s after deployment -->
<link rel="stylesheet" href="/assets/css/style.css">
<img src="/assets/images/compare-tech-stocks.png">
<a href="/features.html">Features</a>
```

### Pattern 4: Image Curation with Stable Filenames

**What:** Copy 6-8 selected PNGs from `finance_output/charts/` to `docs/assets/images/` with stable descriptive names (no date stamps, no ticker-date combinations).

**Source-to-destination mapping (verified file sizes):**

| Site Purpose | Source File | Source Size | Destination Name | Action |
|---|---|---|---|---|
| Hero / multi-stock comparison | `compare_AAPL_GOOGL_MSFT_NVDA_2025-03-18.png` | 185KB | `compare-tech-stocks.png` | Resize to 800px wide (exceeds 150KB target) |
| Semiconductor sector comparison | `compare_AMD_AVGO_INTC_NVDA_QCOM_2025-03-18.png` | 229KB | `compare-semiconductor-stocks.png` | Resize to 800px wide (exceeds 150KB target) |
| Correlation heatmap | `correlation_AAPL_GOOGL_MSFT_2025-03-18.png` | 37KB | `correlation-heatmap.png` | Copy as-is |
| Volatility analysis | `aapl_volatility_2025-03-18.png` | 68KB | `volatility-analysis.png` | Copy as-is |
| ML confusion matrix | `confusion_matrix.png` | 35KB | `confusion-matrix.png` | Copy as-is |
| ML feature importance | `feature_importance.png` | 50KB | `feature-importance.png` | Copy as-is |
| EDA / credit risk | `eda_credit_score.png` | 19KB | `eda-credit-risk.png` | Copy as-is |
| ML residual plot | `residual_plot.png` | 57KB | `residual-plot.png` | Copy as-is |

Total: 8 charts. 6 are already under 150KB. 2 require a resize pass.

**Resize command (macOS sips — no install needed):**
```bash
# Resize to 800px wide, preserving aspect ratio
sips -Z 800 compare_AAPL_GOOGL_MSFT_NVDA_2025-03-18.png --out compare-tech-stocks.png
sips -Z 800 compare_AMD_AVGO_INTC_NVDA_QCOM_2025-03-18.png --out compare-semiconductor-stocks.png
```

**Alternative (Python Pillow):**
```python
# Source: Pillow docs
from PIL import Image

def web_optimize(src_path, dst_path, max_width=800):
    img = Image.open(src_path)
    if img.width > max_width:
        ratio = max_width / img.width
        new_size = (max_width, int(img.height * ratio))
        img = img.resize(new_size, Image.LANCZOS)
    img.save(dst_path, "PNG", optimize=True)
```

### Pattern 5: Mobile Navigation Toggle

**What:** A minimal JavaScript hamburger menu that closes when any nav link is clicked (including anchor links). 10-15 lines.

**When to use:** Required for mobile UX. Pure-CSS `:checked` toggles do not close on anchor click, which leaves the menu covering page content.

```html
<!-- In HTML nav -->
<button class="nav-toggle" aria-label="Toggle navigation">&#9776;</button>
<ul class="nav-links" id="nav-links">
  <li><a href="index.html">Home</a></li>
  <li><a href="features.html">Features</a></li>
  <li><a href="walkthroughs.html">Walkthroughs</a></li>
  <li><a href="getting-started.html">Get Started</a></li>
</ul>
```

```javascript
// docs/assets/js/main.js — ~12 lines
const toggle = document.querySelector('.nav-toggle');
const navLinks = document.querySelector('.nav-links');

toggle.addEventListener('click', () => {
  navLinks.classList.toggle('open');
});

// Close menu on any nav link click (including anchor links)
navLinks.querySelectorAll('a').forEach(link => {
  link.addEventListener('click', () => {
    navLinks.classList.remove('open');
  });
});
```

### Anti-Patterns to Avoid

- **Root-absolute paths:** Any `href` or `src` starting with `/` produces 404s on GitHub Pages project sites. No exceptions.
- **Symlinking `finance_output/charts/`:** GitHub Pages does not resolve symlinks. Images must be physically present in `docs/assets/images/`.
- **Committing all 60+ charts:** Inflates repo size with no benefit. Showcase needs 6-8 named examples.
- **Skipping `.nojekyll`:** Jekyll processes the Python project structure and can fail builds silently. `.nojekyll` is the first committed file, not an afterthought.
- **GitHub Actions workflow before enabling Pages in Settings:** The `github-pages` deployment environment must exist in Settings before a workflow can deploy to it.
- **Pushing any workflow YAML for a plain HTML site:** Branch source mode (`/docs` folder) requires no workflow. Adding a workflow introduces a CI failure risk with no benefit.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Image resize | Custom Python script from scratch | macOS `sips` command or Pillow `Image.resize` | Both are one-liners; Pillow is already available in the Python environment |
| Favicon | Draw a custom icon | favicon.io text-to-favicon generator | Produces correct 16x16/32x32 formats instantly |
| Mobile nav close-on-click | CSS-only :checked hack | 12-line vanilla JS (documented above) | CSS-only solution cannot detect anchor link clicks |
| Deployment pipeline | GitHub Actions workflow | GitHub Pages branch source mode | Branch source mode deploys automatically on push; no YAML needed for plain HTML |

**Key insight:** This phase involves zero custom logic. Every problem in Phase 13 has a one-step standard solution. Time should be spent on configuration, curation decisions, and verification — not on building infrastructure.

---

## Common Pitfalls

### Pitfall 1: Root-Absolute Asset Paths

**What goes wrong:** CSS, images, and inter-page links all 404 after deployment. The site looks correct on localhost but loads completely unstyled after `git push`.

**Why it happens:** Localhost resolves `/assets/css/style.css` to `http://localhost:port/assets/css/style.css`. GitHub Pages resolves it to `https://aarjay.github.io/assets/css/style.css` — missing the `/machine_learning_skill/` prefix.

**How to avoid:** Never use a leading `/` in any href or src. Establish and document the relative-path rule in the first commit so it carries forward to Phase 14 authoring.

**Warning signs:** Any DevTools Network tab 404 after live deployment; styles present locally but absent on the live URL.

### Pitfall 2: Configuring GitHub Pages After Writing HTML

**What goes wrong:** Pages aren't enabled; first push produces no deployment; troubleshooting wastes time finding the Settings step that should have been first.

**How to avoid:** Enable GitHub Pages in Settings → Pages (branch: main, folder: /docs) BEFORE writing any HTML. This is the first task in Phase 13.

### Pitfall 3: Jekyll Processing Without .nojekyll

**What goes wrong:** GitHub Pages runs Jekyll on the repository by default. Jekyll may silently drop files, fail on Python source, or behave unpredictably with the existing project structure.

**How to avoid:** `.nojekyll` is the first file committed to `docs/`. Not the second or third — the first.

### Pitfall 4: Unoptimized Images Exceeding 150KB

**What goes wrong:** The two comparison charts (`compare-tech-stocks.png` at 185KB and `compare-semiconductor-stocks.png` at 229KB) exceed the 150KB requirement. Lighthouse Performance will flag them. Mobile load time is degraded.

**How to avoid:** Run `sips -Z 800` on both files before copying them to `docs/assets/images/`. Verify output file size with `ls -la` before committing. The other 6 charts are already under 150KB.

### Pitfall 5: Testing Only on Desktop at Root Level

**What goes wrong:** Nav links work from `index.html` but 404 from `features.html`; mobile hamburger menu stays open after link tap; images overflow container at 375px.

**How to avoid:** After each significant commit, open the live GitHub Pages URL (not localhost), inspect with DevTools Network tab, switch to mobile device emulation (375px), and test nav links from every page.

### Pitfall 6: Forgetting `loading="lazy"` on Below-Fold Images

**What goes wrong:** Even optimized PNGs slow initial page load if all images are downloaded immediately. On the placeholder index.html this is minor, but the pattern needs to be established before Phase 14 adds more images.

**How to avoid:** Add `loading="lazy"` to every `<img>` tag that is not the hero/above-the-fold image. Establish this in the Phase 13 template so Phase 14 inherits it.

---

## Code Examples

### Verified HTML Template Pattern

```html
<!-- Source: ARCHITECTURE.md (verified against GitHub Pages official docs) -->
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Finance AI Skill</title>
  <meta name="description" content="Professional-grade financial analysis in plain English. No Python required.">
  <link rel="icon" href="assets/images/favicon.png" type="image/png">
  <link rel="stylesheet" href="assets/css/style.css">
</head>
<body>
  <header>
    <nav>
      <a href="index.html" class="nav-brand">Finance AI Skill</a>
      <button class="nav-toggle" aria-label="Toggle navigation">&#9776;</button>
      <ul class="nav-links" id="nav-links">
        <li><a href="index.html">Home</a></li>
        <li><a href="features.html">Features</a></li>
        <li><a href="walkthroughs.html">Walkthroughs</a></li>
        <li><a href="getting-started.html">Get Started</a></li>
      </ul>
    </nav>
  </header>

  <main>
    <!-- Placeholder content for Phase 13 -->
    <section class="hero">
      <h1>Finance AI Skill</h1>
      <p>Professional-grade financial analysis in plain English. No Python required.</p>
      <a href="getting-started.html" class="cta-primary">Get Started</a>
    </section>
  </main>

  <footer>
    <p>For educational/informational purposes only. Not financial advice.</p>
  </footer>

  <script src="assets/js/main.js"></script>
</body>
</html>
```

### CSS Mobile Responsive Baseline

```css
/* docs/assets/css/style.css — minimum viable mobile responsive */
*, *::before, *::after {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  font-size: 16px;
  line-height: 1.6;
  color: #1a1a2e;
}

/* Nav */
nav {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 1.5rem;
  background: #0f3460;
  flex-wrap: wrap;
}

.nav-toggle {
  display: none;
  background: none;
  border: none;
  color: white;
  font-size: 1.5rem;
  cursor: pointer;
}

.nav-links {
  list-style: none;
  display: flex;
  gap: 1.5rem;
  margin: 0;
  padding: 0;
}

/* Mobile breakpoint — 375px finance professional target */
@media (max-width: 640px) {
  .nav-toggle { display: block; }
  .nav-links {
    display: none;
    flex-direction: column;
    width: 100%;
    padding: 1rem 0;
  }
  .nav-links.open { display: flex; }
}

/* Images never overflow their containers */
img {
  max-width: 100%;
  height: auto;
}

/* Lazy-load below-fold images: add loading="lazy" in HTML */
```

### Image Resize Command

```bash
# Source: macOS sips man page — available without any install
# Resize to 800px wide maximum, preserve aspect ratio, output to new file
cd /path/to/finance_output/charts/
sips -Z 800 compare_AAPL_GOOGL_MSFT_NVDA_2025-03-18.png \
     --out /path/to/docs/assets/images/compare-tech-stocks.png
sips -Z 800 compare_AMD_AVGO_INTC_NVDA_QCOM_2025-03-18.png \
     --out /path/to/docs/assets/images/compare-semiconductor-stocks.png

# Verify output sizes
ls -la /path/to/docs/assets/images/*.png
```

### Verify Deployment — Zero 404s Check

```bash
# After pushing to main, confirm Pages is live
gh api repos/{OWNER}/{REPO}/pages --jq '.html_url'

# Manual check: open DevTools → Network tab → reload the live URL
# Filter by status 404 — must show zero results
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|---|---|---|---|
| `gh-pages` branch | `docs/` folder on main branch | GitHub Pages added docs/ source option ~2016; now the recommended default for project sites | Eliminates separate branch; one PR covers source and site |
| Jekyll required for GitHub Pages | `.nojekyll` disables Jekyll for plain HTML | GitHub Pages native feature | No Ruby toolchain needed for static HTML sites |
| GitHub Actions required for all Pages deploys | Branch source mode auto-deploys without Actions | Always available; Actions mode added for SSG flexibility | Branch source mode is simpler for no-build-step sites |

**Deprecated/outdated:**
- `gh-pages` npm package: Useful for pushing built output from a CI step. Not needed when using `docs/` folder branch source. Do not add to this project.
- Jekyll `_config.yml` with `baseurl:`: Only needed if using Jekyll. The `.nojekyll` approach avoids this entirely.

---

## Open Questions

1. **Exact GitHub Pages URL (owner/repo slug)**
   - What we know: URL will be `https://[github-username].github.io/machine_learning_skill/`
   - What's unclear: The exact GitHub username — needed for absolute OG image URLs in Phase 15
   - Recommendation: Placeholder `og:image` URL in Phase 13 template is fine; Phase 15 task fills it in. Do not block Phase 13 on this.

2. **Favicon design decision**
   - What we know: Phase 13 requires a favicon; no design constraint documented
   - What's unclear: Whether a text-based favicon (e.g., "FA" initials in a dark blue box) or a chart icon is preferred
   - Recommendation: Use favicon.io with text "FA" and the site's dark blue (#0f3460) background. Takes 2 minutes. If preferred design differs, swap the file — no other changes needed.

3. **Image optimization verification on Phase 14+ pages**
   - What we know: The two comparison charts need resize in Phase 13; other 6 charts are already under 150KB
   - What's unclear: Whether Phase 14 will add additional chart images beyond the 8 curated here
   - Recommendation: Phase 13 establishes the image optimization workflow (sips resize + verify size). Phase 14 applies the same workflow to any additional images it introduces.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | pytest 7.0+ |
| Config file | `pyproject.toml` (confirmed — `[tool.pytest.ini_options]` present) |
| Quick run command | `pytest tests/ -q --tb=short -m "not network"` |
| Full suite command | `pytest tests/ -q --tb=short` |

### Phase Requirements → Test Map

Phase 13 delivers infrastructure (HTML files, folder structure, image files, GitHub Pages configuration) and visual assets — not Python code or MCP logic. The existing pytest test suite covers the Python/MCP layer and is unaffected by Phase 13. Phase 13 deliverables are verified through browser-based checks and file system assertions, not pytest.

| ID | Behavior | Test Type | Automated Command | File Exists? |
|----|----------|-----------|-------------------|-------------|
| INFRA-01 | `docs/.nojekyll` present; GitHub Pages deploys from `docs/` | Manual/filesystem | `ls docs/.nojekyll` + browser DevTools | ❌ Wave 0 — file created in Phase 13 |
| INFRA-01 | `docs/index.html` loads on live GitHub Pages URL with zero 404s | Manual (browser) | Open live URL; DevTools Network tab; filter 404 | ❌ Wave 0 |
| INFRA-02 | All HTML pages share identical head/nav/footer structure | Manual inspection | `diff <(head -20 docs/index.html) <(head -20 docs/features.html)` | ❌ Wave 0 |
| INFRA-03 | `<meta name="viewport">` present in all pages | Filesystem/grep | `grep -l "viewport" docs/*.html` | ❌ Wave 0 |
| INFRA-03 | No horizontal scrolling at 375px viewport | Manual (DevTools) | DevTools device emulation at 375px | ❌ Wave 0 |
| VIS-01 | 6-8 PNG files present in `docs/assets/images/` with stable names | Filesystem | `ls docs/assets/images/*.png \| wc -l` | ❌ Wave 0 |
| VIS-02 | All image files under 150KB | Filesystem | `find docs/assets/images -name "*.png" -size +150k` (must return empty) | ❌ Wave 0 |
| VIS-03 | `favicon.png` present and referenced in HTML head | Filesystem | `ls docs/assets/images/favicon.png && grep -l "favicon" docs/index.html` | ❌ Wave 0 |

### Sampling Rate

- **Per task commit:** `pytest tests/ -q --tb=short -m "not network"` (confirms no Python regressions; Phase 13 does not modify Python code, so this should always pass)
- **Per wave merge:** `pytest tests/ -q --tb=short` + manual browser check on live URL
- **Phase gate:** Existing pytest suite green + live URL loads styled with zero DevTools 404s + mobile 375px no horizontal scroll

### Wave 0 Gaps

Phase 13 creates new files in `docs/` — none are Python test files. The existing test suite does not need modification for Phase 13. The validation is entirely browser-based (DevTools) and filesystem-based (ls, grep, find).

- [ ] `docs/.nojekyll` — created in Phase 13 Wave 1
- [ ] `docs/index.html` — created in Phase 13 with correct relative paths and viewport meta
- [ ] `docs/assets/css/style.css` — created in Phase 13 with mobile breakpoints
- [ ] `docs/assets/js/main.js` — created in Phase 13 with nav toggle
- [ ] `docs/assets/images/*.png` — 8 curated images created in Phase 13
- [ ] `docs/assets/images/favicon.png` — created in Phase 13

None of these gaps require new pytest test files — they are verified through manual DevTools inspection and file system commands documented in the phase gate above.

---

## Sources

### Primary (HIGH confidence)
- GitHub Docs: Configuring a publishing source for GitHub Pages — https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site
- GitHub Docs: Creating a GitHub Pages site — https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site
- Project ARCHITECTURE.md (2026-03-18) — docs/ folder pattern, .nojekyll behavior, relative path requirement, curated image table with specific source files
- Project STACK.md (2026-03-18) — plain HTML decision locked; Jekyll alternatives evaluated and deferred
- Project PITFALLS.md (2026-03-18) — 9 pitfalls documented with root causes and recovery strategies
- Project SUMMARY.md (2026-03-18) — phase rationale, critical path, and quality gates
- macOS sips man page — `sips -Z` resize flag, no install required on macOS

### Secondary (MEDIUM confidence)
- Maxim Orlov — "Deploying to Github Pages? Don't Forget to Fix Your Links": https://maximorlov.com/deploying-to-github-pages-dont-forget-to-fix-your-links/ — real-world root-absolute path failure report, verified against GitHub Docs
- Project finance_output/charts/ file size measurements — performed 2026-03-18 via `ls -la`; 8 candidate files measured; 2 over 150KB identified

### Tertiary (LOW confidence)
- None. All findings in this research are sourced from official docs or project-level research already at HIGH confidence.

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — locked decision from project-level research; plain HTML, .nojekyll, docs/ folder are official GitHub Pages patterns
- Architecture: HIGH — sourced from official GitHub Docs; confirmed in project ARCHITECTURE.md
- Image curation: HIGH — candidate files measured directly; resize tooling (sips) is macOS built-in
- Pitfalls: HIGH — sourced from official GitHub Docs + project PITFALLS.md with real-world verification
- Validation architecture: HIGH — existing pytest infrastructure confirmed; Phase 13 deliverables are infrastructure/assets verified by browser + filesystem, not Python unit tests

**Research date:** 2026-03-18
**Valid until:** 2026-09-18 (GitHub Pages publishing mechanics are stable; plain HTML patterns do not expire)
