# Phase 15: Getting Started and Polish — Research

**Researched:** 2026-03-18
**Domain:** Static HTML getting-started page authoring, Open Graph social card creation, navigation link audit, Lighthouse performance verification
**Confidence:** HIGH

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| START-01 | Claude Code installation path with copy-paste commands | Claude Code uses stdio transport; install commands confirmed from `finance-mcp-plugin/.mcp.json` and project structure; exact MCP config JSON documented below |
| START-02 | claude.ai installation path with step-by-step instructions | claude.ai requires HTTP transport + ngrok; ngrok free tier supports this; step-by-step documented below |
| INFRA-04 | Social card (OG meta image) displays when site is shared on LinkedIn/Twitter | OG meta tags already stubbed on all 4 pages with empty `og:image`; absolute URL required; 1200x630 PNG social card must be created; validated via opengraph.xyz |
</phase_requirements>

---

## Summary

Phase 15 has three independent workstreams: (1) replace the `getting-started.html` stub with full two-path installation content, (2) create the social card image and populate `og:image` across all four pages with the absolute URL, and (3) audit and fix every navigation link on every page plus verify Lighthouse Performance scores.

The getting-started page is the terminal conversion page — every CTA on the other three pages (`href="getting-started.html"`) lands here. It must present two clearly separated install paths without requiring technical fluency: the Claude Code path (stdio, simple JSON config) and the claude.ai path (HTTP transport, ngrok tunnel). The primary audience is finance professionals, not developers, so each step must be in plain English with zero assumed knowledge of MCP, Python environments, or JSON syntax.

The social card is a one-time visual asset task. All four pages already have `<meta property="og:image" content="">` in place — the empty string is the only thing blocking rich LinkedIn previews. The fix is to: create a 1200x630 PNG (best candidate: `compare-tech-stocks.png` cropped to ratio), commit it to `docs/assets/images/social-card.png`, and fill in the absolute URL `https://bolnet.github.io/Claude-Finance/assets/images/social-card.png` across all four pages. Validation is via opengraph.xyz before sign-off.

Navigation link audit and Lighthouse verification are polish tasks. The nav HTML is copied across all four pages from the shared template pattern — a single error propagates everywhere. The Lighthouse gate (score >= 80) guards against image weight regressions introduced during Phase 14 content authoring.

**Primary recommendation:** Do the three workstreams in order — (1) getting-started.html content, (2) social card + OG meta update across all 4 pages, (3) link audit + Lighthouse verification. This order catches any links into the new page during the audit pass.

---

## Standard Stack

### Core (already in place)

| Component | Detail | Purpose | Status |
|-----------|--------|---------|--------|
| `docs/getting-started.html` | Stub already exists (45 lines) | Replace with full two-path install content | In place — replace `<section class="page-stub">` |
| `docs/assets/css/style.css` | 453 lines, all classes defined | Shared styles — install step layout must use existing classes | In place |
| `docs/assets/js/main.js` | Mobile nav toggle (IIFE) | No changes needed | In place |
| Relative asset paths | Convention from Phase 13 | `assets/images/...` never `/assets/...` | Enforced |
| `og:image` meta tags | Empty string on all 4 pages | Fill with absolute URL after social card created | Present but empty |

### New CSS Required for Getting Started Page

Phase 14 established all card and grid patterns. The getting-started page needs install-step layout CSS that does not exist yet:

| Component | CSS class to add | Purpose |
|-----------|-----------------|---------|
| Install path tabs/sections | `.install-path` | Visual separation between Claude Code and claude.ai sections |
| Step number indicator | `.step` | Numbered step blocks (1, 2, 3...) |
| Code block for copy-paste | `.code-block` | Monospace, dark background, copy-friendly display |
| Path heading | `.path-heading` | Bold section h2 with colored left border, like `.feature-category h2` |
| Warning/note callout | `.callout-note` | Yellow-tinted box for ngrok rate-limit warning |

All new classes go in the single shared `style.css`. Do not create a `getting-started.css`.

### Social Card: Toolchain

| Tool | Purpose | Available |
|------|---------|-----------|
| `sips` (macOS built-in) | Resize PNG to 1200x630 | Yes — used in Phase 13 for image optimization |
| `convert` (ImageMagick) | Crop to exact 1200x630 with text overlay | Available if ImageMagick installed; not required if sips handles it |
| Source image | `docs/assets/images/compare-tech-stocks.png` | In place — 800px wide, <150KB |

Note: `compare-tech-stocks.png` at 800px is narrower than the required 1200px. The canonical recommendation is to create the social card at native 1200x630 by exporting a wider chart, or to center-pad/letterbox the existing image on a dark blue (`#0f3460`) background to reach 1200x630. The letterbox approach requires no new chart generation.

**Installation:** No new packages. `sips` handles resize; Python Pillow (already installed from Phase 13 favicon work) handles canvas-pad.

---

## Architecture Patterns

### Getting Started Page Structure

```
docs/
├── getting-started.html    # PHASE 15: replace page-stub with full content
├── index.html              # Phase 14 complete — update og:image URL
├── features.html           # Phase 14 complete — update og:image URL
├── walkthroughs.html       # Phase 14 complete — update og:image URL
└── assets/
    ├── css/style.css       # extend with install-step classes
    ├── js/main.js          # no changes
    └── images/
        └── social-card.png # NEW: 1200x630 OG social card image
```

### Pattern 1: Two-Path Install Section Structure

**What:** The getting-started page presents two clearly labeled, visually separated sections. Each section is a numbered step sequence. Section divider uses a contrasting background or bold heading border so a user scanning quickly can identify "I'm on Path A" vs "Path B" at a glance.

**When to use:** Whenever a tool has multiple install methods targeting different audiences.

**Page section order:**
1. **Page hero** — outcome-focused headline: "Start analyzing finance data in minutes"
2. **Path A: Claude Code** — audience label, 4 steps, copy-pasteable commands
3. **Path B: claude.ai** — audience label, 5 steps including ngrok setup
4. **First analysis** — one example prompt the user types after install, same for both paths
5. **Troubleshooting** — two pre-empted failure points (yfinance rate limit, ngrok auth)

**Example structure:**
```html
<section class="install-path" id="claude-code">
  <h2 class="path-heading">Path A: Claude Code (recommended)</h2>
  <p class="path-audience">Best if you use Claude from the terminal or IDE</p>
  <div class="step">
    <span class="step-num">1</span>
    <div class="step-content">
      <strong>Clone the repository</strong>
      <pre class="code-block"><code>git clone https://github.com/bolnet/Claude-Finance.git
cd Claude-Finance</code></pre>
    </div>
  </div>
  <!-- steps 2–4 -->
</section>

<section class="install-path" id="claude-ai">
  <h2 class="path-heading">Path B: claude.ai (browser)</h2>
  <p class="path-audience">Best if you use Claude in the browser at claude.ai</p>
  <!-- steps 1–5 -->
</section>
```

### Pattern 2: Install Step — Claude Code (stdio)

**What:** Claude Code connects to MCP servers via the `.mcp.json` file in the project root (project-scoped) or via `~/.claude.json` (global). The finance-mcp-plugin already ships a `.mcp.json` at `finance-mcp-plugin/.mcp.json`. The planner should document the exact steps.

**Confirmed from `finance-mcp-plugin/.mcp.json`:**
```json
{
  "mcpServers": {
    "finance": {
      "command": "python",
      "args": ["-m", "finance_mcp.server"]
    }
  }
}
```

**Full Claude Code install path (4 steps):**

1. Clone the repository:
   ```bash
   git clone https://github.com/bolnet/Claude-Finance.git
   cd Claude-Finance
   ```

2. Install Python dependencies:
   ```bash
   pip install -e ".[dev]"
   ```

3. Add the MCP server to Claude Code by copying `.mcp.json` to the project root or by running Claude Code from the `Claude-Finance/` directory (it reads `.mcp.json` automatically):
   ```bash
   # Claude Code auto-discovers .mcp.json in the working directory
   claude
   ```

4. Verify the skill is active by typing `/finance` in Claude Code — you should see a confirmation response.

**Note to planner:** Verify the exact pip install command against `pyproject.toml` or `setup.py` before finalizing copy. The confirmed working approach from Phase 1 is `pip install -e .` from the repo root. The `[dev]` extra may or may not be needed for end users.

### Pattern 3: Install Step — claude.ai (HTTP + ngrok)

**What:** claude.ai connects to MCP servers only via HTTP transport (not stdio). The finance MCP server must be exposed as an HTTP endpoint. ngrok creates a public HTTPS tunnel from the user's localhost to the internet, which claude.ai can reach.

**Full claude.ai install path (5 steps):**

1. Clone the repository and install dependencies (same as Claude Code steps 1–2 above).

2. Start the MCP HTTP server locally:
   ```bash
   python -m finance_mcp.server --transport streamable-http --port 8000
   ```
   (Verify the exact start command from `src/finance_mcp/server.py` — the `test_http_server.py` test file confirms HTTP transport is implemented.)

3. Install ngrok and create a tunnel:
   ```bash
   # Install ngrok (macOS)
   brew install ngrok
   # Authenticate (free tier requires signup at ngrok.com)
   ngrok config add-authtoken YOUR_TOKEN
   # Start tunnel
   ngrok http 8000
   ```
   ngrok outputs a public URL like `https://abc123.ngrok-free.app`.

4. Add the MCP server to claude.ai:
   - Go to claude.ai Settings → Integrations → Add integration
   - Enter the ngrok URL: `https://abc123.ngrok-free.app`
   - Save and refresh.

5. Verify by starting a new conversation and typing `/finance` — you should see the skill respond.

**Troubleshooting to pre-empt:**
- **ngrok URL changes on restart** — free tier ngrok generates a new URL each time. Users must update the integration URL in claude.ai Settings each session unless they use a paid ngrok plan with a static domain.
- **yfinance rate limiting** — Yahoo Finance blocks rapid API calls; if charts fail with network errors, wait 30 seconds and retry. Not a setup issue.
- **Python not found** — if `python -m finance_mcp.server` fails, try `python3 -m finance_mcp.server`. The commands are equivalent on macOS.

### Pattern 4: Open Graph Social Card

**What:** Every page already has `<meta property="og:image" content="">`. LinkedIn reads the `og:image` absolute URL and shows the image in the link preview card. An empty string produces a blank card.

**Required tags (all 4 pages):**
```html
<meta property="og:title" content="[Page-specific title]">
<meta property="og:description" content="[Page-specific 155-char description]">
<meta property="og:type" content="website">
<meta property="og:image" content="https://bolnet.github.io/Claude-Finance/assets/images/social-card.png">
<meta property="og:url" content="https://bolnet.github.io/Claude-Finance/[page].html">
```

**OG image requirements (LinkedIn spec):**
- Minimum: 1200x627 pixels (LinkedIn recommends 1200x628 or 1200x630)
- Aspect ratio: 1.91:1
- Format: PNG or JPG
- File size: under 5MB (target under 300KB for performance)
- The image must be at an absolute HTTPS URL — relative paths are silently ignored

**Social card image creation approach:**
The best source image is `docs/assets/images/compare-tech-stocks.png` (the hero image — already web-optimized, visually strong, shows real financial data). Since it is 800px wide, it needs to be placed on a 1200x630 canvas. Use Pillow (already installed) to:
- Create a 1200x630 image with `#0f3460` (dark blue brand color) background
- Center the 800px chart with top/bottom padding
- Optionally add a text label "Finance AI Skill" in white at the bottom

**Validation:** Visit `https://opengraph.xyz/url/https://bolnet.github.io/Claude-Finance/` after deploying. If the social card shows title, description, and chart image, INFRA-04 is satisfied.

### Anti-Patterns to Avoid

- **Relative path for og:image:** `og:image` must be an absolute `https://` URL. A relative path like `assets/images/social-card.png` is silently ignored by LinkedIn and produces a blank card.
- **Root-absolute paths:** Never use `/assets/...`. Keep `assets/...` (no leading slash) for all `src` and `href` attributes. This was established in Phase 13 and must not drift.
- **Developer-only language in install steps:** "Configure the MCP stdio transport" means nothing to a finance professional. Write "Tell Claude Code where to find the finance server" instead.
- **ngrok without the auth warning:** Free-tier ngrok generates a new URL on every restart. If the page does not warn about this, users will report a "broken" integration after their first ngrok restart.
- **New CSS file for getting-started:** All styles go in the shared `style.css`. Do not create `getting-started.css`.
- **Missing `og:url` tag:** LinkedIn uses `og:url` for deduplication. Include the canonical absolute URL for each page.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Social card image | Custom SVG/HTML-to-image renderer | Pillow canvas on `compare-tech-stocks.png` | Pillow already installed (Phase 13 favicon); one-time asset creation |
| Step numbering UI | JS accordion or stepper component | CSS counter + flexbox `.step` divs | No JS needed; CSS counters are supported in all browsers; static HTML is simpler |
| Install path "tab switcher" | JS click-to-reveal tabs | Two `<section class="install-path">` blocks stacked vertically | Finance professionals scroll; tabs hide content from search indexers; sections work without JS |
| Link checker | Custom crawl script | Browser DevTools Network tab (zero 404s) | Four pages; manual check is faster than setting up a tool; verify at deployed URL, not localhost |
| Lighthouse runner | Local Lighthouse CLI | Chrome DevTools → Lighthouse tab | Already available; no install needed; four pages takes under 10 minutes |

**Key insight:** This phase is documentation and asset creation. Every new technology introduced creates a new failure point. Minimize tooling surface.

---

## Common Pitfalls

### Pitfall 1: og:image With a Relative URL

**What goes wrong:** `<meta property="og:image" content="assets/images/social-card.png">` — LinkedIn fetches the URL literally; a relative path has no origin and is treated as invalid. The social card preview shows a blank grey box.

**Why it happens:** Relative paths work everywhere else in the HTML; developers assume the same convention applies to meta tags.

**How to avoid:** The `og:image` content MUST be an absolute HTTPS URL: `https://bolnet.github.io/Claude-Finance/assets/images/social-card.png`. The current stub already has `content=""` — the fix is to fill in the absolute URL, not a relative path.

**Warning signs:** opengraph.xyz shows title and description but no image (not a blank card — LinkedIn will show the title/desc even if the image fails).

---

### Pitfall 2: Social Card Image Under 200x200 or Wrong Aspect Ratio

**What goes wrong:** LinkedIn shows a small thumbnail rather than a large preview card when the OG image is smaller than 1200x627 or has an aspect ratio not close to 1.91:1.

**Why it happens:** Using an existing web-optimized 800px-wide chart directly without resizing to 1200x630.

**How to avoid:** Create the social card image at exactly 1200x630 by placing the chart on a padded canvas. The letterbox approach (chart centered on dark blue background) ensures correct dimensions regardless of the source image's original size.

**Warning signs:** opengraph.xyz shows a tiny thumbnail in the corner of the card instead of a full-width image.

---

### Pitfall 3: Install Instructions Written for Developers

**What goes wrong:** Step 3 reads "Configure the stdio transport in your MCP client settings JSON." A finance professional does not know what stdio transport means, what an MCP client is, or where settings JSON lives.

**Why it happens:** The writer knows the technical domain and collapses the abstraction layers.

**How to avoid:** For each step, test: "Could a senior equity analyst who has never opened a terminal follow this without asking a question?" Plain-English rewrites: "stdio transport" → "how Claude Code talks to the finance server"; "MCP client settings" → "Claude Code settings file"; "JSON" → "settings file (a plain text file that looks like a list)."

**Warning signs:** Any install step contains the words "MCP", "stdio", "transport", "streamable-HTTP", "FastMCP", "server process", or "environment variable" without an immediate plain-English translation in the same sentence.

---

### Pitfall 4: ngrok URL Regeneration Not Warned

**What goes wrong:** User sets up ngrok, adds the URL to claude.ai, everything works. Next day they restart ngrok; a new URL is generated; the integration in claude.ai now points to the old URL and returns errors. User believes the tool is broken.

**Why it happens:** Free-tier ngrok generates a random subdomain on each startup. This is well-documented in ngrok's own docs but not obvious to first-time users.

**How to avoid:** Include a clearly labeled callout on the claude.ai install path: "Note: the ngrok URL changes every time you restart ngrok. Update your integration URL in claude.ai Settings each time. Upgrade to ngrok's free static domain (one per account) to avoid this."

**Warning signs:** No mention of ngrok URL regeneration in the claude.ai install path.

---

### Pitfall 5: Broken Navigation Links Discovered Post-Phase

**What goes wrong:** Phase 14 copied nav HTML across three pages. A link that was correct in Phase 14 may have a typo in one copy. If the link audit is skipped, users click a nav link and hit a 404 — on a 4-page site.

**Why it happens:** Manual copy-paste of nav HTML across files; no automated link checker enforced.

**How to avoid:** Open each of the 4 pages in a browser (at the live `bolnet.github.io/Claude-Finance` URL, not localhost) and click every nav link from every page. Four pages × four nav links = 16 link checks. Record result. Zero 404s is the gate.

**Warning signs:** Any navigation link that opens to a blank browser tab or a 404 page.

---

### Pitfall 6: Lighthouse Performance Failing on Image-Heavy Pages

**What goes wrong:** Phase 14 added 6–8 chart PNGs across features.html and walkthroughs.html. If any image has `loading="eager"` that should be lazy, or if an image slipped through optimization at >150KB, Lighthouse Performance may fall below 80.

**Why it happens:** Phase 14 content authoring focused on structure and copy; image weight verification may not have been re-run after each page was completed.

**How to avoid:** Run Lighthouse on all 4 pages from the live URL before sign-off. If a page fails, use Chrome DevTools Network tab to identify the heaviest images and re-optimize with `sips`.

**Warning signs:** Lighthouse Performance below 80 on any of features.html, walkthroughs.html, or getting-started.html.

---

## Code Examples

### Install Step CSS (new additions to style.css)

```css
/* Source: designed to match existing style.css patterns */

/* ===== GETTING STARTED PAGE ===== */
.install-path {
  margin-bottom: 3rem;
  padding-bottom: 2rem;
  border-bottom: 1px solid #e0e0e0;
}

.install-path:last-of-type {
  border-bottom: none;
}

.path-heading {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f3460;
  padding-bottom: 0.5rem;
  border-bottom: 3px solid #e94560;
  margin-bottom: 0.5rem;
}

.path-audience {
  font-size: 0.95rem;
  color: #666;
  margin-bottom: 1.5rem;
}

.step {
  display: flex;
  gap: 1.25rem;
  align-items: flex-start;
  margin-bottom: 1.5rem;
}

.step-num {
  flex-shrink: 0;
  width: 2rem;
  height: 2rem;
  background: #0f3460;
  color: #fff;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: 700;
  font-size: 0.9rem;
}

.step-content {
  flex: 1;
}

.step-content strong {
  display: block;
  color: #1a1a2e;
  margin-bottom: 0.4rem;
  font-size: 1rem;
}

.step-content p {
  color: #555;
  font-size: 0.9rem;
  margin-bottom: 0.5rem;
}

.code-block {
  background: #1a1a2e;
  color: #e8e8e8;
  padding: 0.75rem 1rem;
  border-radius: 4px;
  font-family: "SFMono-Regular", Consolas, "Liberation Mono", Menlo, monospace;
  font-size: 0.85rem;
  overflow-x: auto;
  line-height: 1.5;
  margin: 0.5rem 0;
}

.callout-note {
  background: #fff8e1;
  border-left: 4px solid #f59e0b;
  padding: 0.75rem 1rem;
  border-radius: 0 4px 4px 0;
  font-size: 0.9rem;
  color: #555;
  margin: 1rem 0;
}

.callout-note strong {
  color: #1a1a2e;
}
```

### Social Card Creation Script (Python / Pillow)

```python
# Source: uses Pillow (already installed from Phase 13 favicon work)
# Run from repo root: python3 scripts/make_social_card.py

from PIL import Image, ImageDraw, ImageFont
import os

CANVAS_W, CANVAS_H = 1200, 630
BG_COLOR = (15, 52, 96)   # #0f3460 brand dark blue
LABEL_COLOR = (255, 255, 255)

chart_path = "docs/assets/images/compare-tech-stocks.png"
output_path = "docs/assets/images/social-card.png"

chart = Image.open(chart_path).convert("RGB")

# Scale chart to fill width, maintain aspect ratio
scale = CANVAS_W / chart.width
new_h = int(chart.height * scale)
chart_resized = chart.resize((CANVAS_W, new_h), Image.LANCZOS)

# Create canvas and paste chart centered vertically
canvas = Image.new("RGB", (CANVAS_W, CANVAS_H), BG_COLOR)
y_offset = (CANVAS_H - new_h) // 2
canvas.paste(chart_resized, (0, y_offset))

canvas.save(output_path, "PNG", optimize=True)
print(f"Social card saved to: {output_path}")
# Verify: file size should be under 300KB
print(f"File size: {os.path.getsize(output_path) / 1024:.0f} KB")
```

### OG Meta Tags — All 4 Pages (updated pattern)

```html
<!-- index.html — add og:url, fill og:image -->
<meta property="og:title" content="Finance AI Skill">
<meta property="og:description" content="Professional-grade financial analysis in plain English. No Python required. Powered by Claude AI.">
<meta property="og:type" content="website">
<meta property="og:image" content="https://bolnet.github.io/Claude-Finance/assets/images/social-card.png">
<meta property="og:url" content="https://bolnet.github.io/Claude-Finance/index.html">

<!-- features.html -->
<meta property="og:title" content="Features | Finance AI Skill">
<meta property="og:description" content="11 built-in finance analyses — price charts, risk metrics, peer comparisons, correlation heatmaps, and ML workflows.">
<meta property="og:type" content="website">
<meta property="og:image" content="https://bolnet.github.io/Claude-Finance/assets/images/social-card.png">
<meta property="og:url" content="https://bolnet.github.io/Claude-Finance/features.html">

<!-- walkthroughs.html -->
<meta property="og:title" content="Walkthroughs | Finance AI Skill">
<meta property="og:description" content="Step-by-step scenarios for equity research, hedge funds, investment banking, FP&A, private equity, and accounting.">
<meta property="og:type" content="website">
<meta property="og:image" content="https://bolnet.github.io/Claude-Finance/assets/images/social-card.png">
<meta property="og:url" content="https://bolnet.github.io/Claude-Finance/walkthroughs.html">

<!-- getting-started.html -->
<meta property="og:title" content="Get Started | Finance AI Skill">
<meta property="og:description" content="Install the Finance AI Skill in minutes — two paths for Claude Code and claude.ai users.">
<meta property="og:type" content="website">
<meta property="og:image" content="https://bolnet.github.io/Claude-Finance/assets/images/social-card.png">
<meta property="og:url" content="https://bolnet.github.io/Claude-Finance/getting-started.html">
```

### Navigation Link Audit Checklist (16 checks)

```
From index.html:
  [ ] Home        → index.html
  [ ] Features    → features.html
  [ ] Walkthroughs → walkthroughs.html
  [ ] Get Started  → getting-started.html

From features.html:
  [ ] Home        → index.html
  [ ] Features    → features.html
  [ ] Walkthroughs → walkthroughs.html
  [ ] Get Started  → getting-started.html

From walkthroughs.html:
  [ ] Home        → index.html
  [ ] Features    → features.html
  [ ] Walkthroughs → walkthroughs.html
  [ ] Get Started  → getting-started.html

From getting-started.html:
  [ ] Home        → index.html
  [ ] Features    → features.html
  [ ] Walkthroughs → walkthroughs.html
  [ ] Get Started  → getting-started.html

Also verify CTA links:
  [ ] index.html "Get Started Free"  → getting-started.html
  [ ] walkthroughs.html anchor links → walkthroughs.html#equity-research etc.
```

---

## State of the Art

| Old Approach | Current Approach | Impact |
|--------------|-----------------|--------|
| `og:image` relative paths | Absolute HTTPS URL required | Relative paths are silently ignored by LinkedIn — blank cards |
| ngrok HTTP paths (deprecated) | `ngrok http 8000` (current syntax) | Old `ngrok http 8000 --host-header=...` flags no longer needed for basic tunneling |
| LinkedIn 1200x627 OG image spec | 1200x630 (commonly cited) | Either works; 630 is a safe choice that satisfies both specs |
| Manual Lighthouse via CLI | Chrome DevTools Lighthouse tab | No install needed; DevTools version is current and matches production Chrome |

**Deprecated/outdated:**
- `og:image:width` and `og:image:height` meta tags: optional in the Open Graph spec; LinkedIn does not require them; omitting is fine
- `twitter:card` meta tags: Twitter/X still reads them but most sharing platforms now use `og:*` natively; omit unless Twitter sharing is a target

---

## Open Questions

1. **Exact pip install command for end users**
   - What we know: `finance-mcp-plugin/.mcp.json` specifies `python -m finance_mcp.server`; Phase 13 image scripts used Pillow which is already installed
   - What's unclear: Whether `pip install -e .` or `pip install finance-mcp` (PyPI package) is the correct end-user install command; whether `[dev]` extras are required for normal use
   - Recommendation: Planner should verify against `pyproject.toml` before finalizing copy; use `pip install -e .` if no PyPI package exists

2. **HTTP server start command for claude.ai path**
   - What we know: `tests/test_http_server.py` confirms HTTP transport is implemented; `finance-mcp-plugin/commands/finance.md` references streamable-HTTP
   - What's unclear: The exact CLI command to start the HTTP server (port, transport flag names)
   - Recommendation: Planner should verify `python -m finance_mcp.server --help` output and use the confirmed flag names; do not guess

3. **`og:image` alt text / `og:image:alt`**
   - What we know: `og:image:alt` is a valid Open Graph tag that provides accessible alt text for the social card
   - What's unclear: Whether LinkedIn reads it (it is in the spec but implementation varies)
   - Recommendation: Add `<meta property="og:image:alt" content="Finance AI Skill — stock comparison chart showing AAPL, GOOGL, MSFT, and NVDA performance">` to be spec-compliant; low cost, potential accessibility benefit

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | pytest (existing) |
| Config file | none detected — runs via `python3 -m pytest tests/` |
| Quick run command | `python3 -m pytest tests/ -x -q` |
| Full suite command | `python3 -m pytest tests/ -v` |

Phase 15 introduces no new Python code. The existing pytest suite verifies the Python MCP skill is unaffected. HTML content verification is manual.

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| START-01 | getting-started.html has a Claude Code install path with numbered steps and code blocks | Manual | Open page in browser; verify path A section visible with `<pre class="code-block">` elements | N/A — HTML content |
| START-01 | Claude Code MCP config JSON is copy-pasteable and syntactically valid | Manual / validation | `python3 -c "import json; json.loads(open('finance-mcp-plugin/.mcp.json').read()); print('valid')"` | ✅ file exists |
| START-02 | getting-started.html has a claude.ai install path with ngrok steps | Manual | Open page; verify path B section with ngrok install instructions visible | N/A — HTML content |
| START-02 | ngrok URL regeneration warning is present | Manual | Check for `.callout-note` element in getting-started.html claude.ai section | N/A — HTML content |
| INFRA-04 | og:image on all 4 pages is an absolute URL (not empty, not relative) | Automated grep | `grep -h 'og:image' docs/*.html` — all 4 results must contain `https://` | N/A — HTML content |
| INFRA-04 | Social card displays on LinkedIn (verified via opengraph.xyz) | Manual | Visit `opengraph.xyz/url/https://bolnet.github.io/Claude-Finance/` | N/A — requires live deployment |
| Phase success criterion | Every nav link on every page resolves without 404 | Manual | Browser test: 16 nav link clicks at live URL, all resolve | N/A — requires live deployment |
| Phase success criterion | Lighthouse Performance >= 80 on all 4 pages | Manual / Lighthouse | Chrome DevTools → Lighthouse → Performance on all 4 pages at live URL | N/A — requires live deployment |

### Sampling Rate

- **Per task commit:** `python3 -m pytest tests/ -x -q` (verify no Python regression)
- **Per wave merge:** Python suite + browser spot-check of getting-started.html at live URL
- **Phase gate:** All success criteria verified at `https://bolnet.github.io/Claude-Finance/`:
  1. `grep 'og:image' docs/*.html` — all 4 contain absolute `https://` URL
  2. opengraph.xyz shows rich card with title, description, and chart image
  3. 16 nav link clicks: zero 404s
  4. Lighthouse Performance >= 80 on all 4 pages

### Wave 0 Gaps

None — existing Python test infrastructure covers all phase requirements. Phase 15 introduces no new Python code. All verification is manual or HTML-grep based.

---

## Sources

### Primary (HIGH confidence)

- `/Users/aarjay/Documents/machine_learning_skill/docs/getting-started.html` — confirmed stub state: `page-stub` section, empty `og:image`, correct nav structure
- `/Users/aarjay/Documents/machine_learning_skill/docs/assets/css/style.css` — verified all 453 lines of existing CSS; confirmed what classes exist and what does not
- `/Users/aarjay/Documents/machine_learning_skill/docs/index.html`, `features.html`, `walkthroughs.html` — confirmed all three have `og:image content=""` stub
- `/Users/aarjay/Documents/machine_learning_skill/finance-mcp-plugin/.mcp.json` — confirmed Claude Code MCP config: `python -m finance_mcp.server`
- `/Users/aarjay/Documents/machine_learning_skill/.planning/research/SUMMARY.md` — INFRA-04 `og:image` absolute URL requirement, ngrok-based claude.ai path, opengraph.xyz validation method
- `/Users/aarjay/Documents/machine_learning_skill/.planning/phases/14-content-pages/14-RESEARCH.md` — confirmed Phase 14 design system, CSS class inventory, confirmed all phase 14 plans are complete
- `/Users/aarjay/Documents/machine_learning_skill/.planning/STATE.md` — confirmed project GitHub remote URL (`bolnet/Claude-Finance`)
- Open Graph Protocol specification — `og:image` must be absolute URL: https://ogp.me/#metadata
- LinkedIn developer docs — OG image size: 1200x627 minimum, 1.91:1 aspect ratio

### Secondary (MEDIUM confidence)

- ngrok documentation (2026) — `ngrok http 8000` basic tunnel syntax; free tier URL regeneration behavior — https://ngrok.com/docs/getting-started/
- opengraph.xyz — free OG tag validator; shows exactly what LinkedIn will display before sharing — https://www.opengraph.xyz/

---

## Metadata

**Confidence breakdown:**
- Getting Started content (START-01, START-02): HIGH for structure and steps; MEDIUM for exact HTTP server start command (needs verification against `python -m finance_mcp.server --help`)
- Social card / OG meta (INFRA-04): HIGH — OG spec is authoritative, absolute URL requirement is unambiguous, opengraph.xyz validation is well-established
- Navigation audit: HIGH — 4-page site, manual 16-click audit is definitive
- Lighthouse: HIGH — Chrome DevTools is available, gates are clear

**Research date:** 2026-03-18
**Valid until:** 2026-04-18 (stable domain: static HTML, OG spec, ngrok free-tier behavior)
