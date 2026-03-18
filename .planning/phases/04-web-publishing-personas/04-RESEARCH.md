# Phase 4: Web Publishing & Personas - Research

**Researched:** 2026-03-18
**Domain:** FastMCP HTTP transport, claude.ai MCP Integrations, Claude Code plugin packaging, command persona variants
**Confidence:** MEDIUM-HIGH (transport layer verified via official docs; marketplace submission process MEDIUM — verified via official Claude docs but review criteria not fully documented)

---

<phase_requirements>
## Phase Requirements

| ID | Description | Research Support |
|----|-------------|-----------------|
| WEB-01 | Finance professional with no Claude Code install can connect MCP server to claude.ai and use all tools via browser chat | FastMCP HTTP transport (streamable-http) + public URL; claude.ai Settings > Connectors flow |
| WEB-02 | MCP server packaged with connection guide for non-technical users (no terminal knowledge required to connect) | Server startup script + ngrok/Cloudflare tunnel; step-by-step guide written for non-developers |
| WEB-03 | Skill listed and documented for Claude plugin marketplace submission | plugin.json manifest, directory structure, submission at claude.ai/settings/plugins/submit |
| PERS-01 | `/finance-analyst` command — equity-focused variant (stock analysis, peer comparison, valuation ratios emphasis) | New `.claude/commands/finance-analyst.md` with frontmatter; focus routing on stock-analysis and multi-ticker intents |
| PERS-02 | `/finance-pm` command — portfolio manager variant (risk/return attribution, portfolio-level metrics) | New `.claude/commands/finance-pm.md` with frontmatter; focus routing on market-metrics and risk/return intents |
</phase_requirements>

---

## Summary

Phase 4 has three distinct work streams: (1) wiring the existing stdio MCP server to accept remote HTTP connections so claude.ai can reach it, (2) writing two persona command variants that reuse all existing MCP tools with different emphasis, and (3) packaging everything as a distributable Claude Code plugin with marketplace metadata.

The most technically complex work is WEB-01/WEB-02. The existing server uses FastMCP with stdio transport. For claude.ai integration, the server must expose an HTTP endpoint (streamable-http, the current standard) at a publicly reachable URL with TLS. The simplest path for non-technical users is an ngrok tunnel that starts alongside the server — no VPS required. Claude.ai on Pro/Max/Team/Enterprise connects via Settings > Connectors by entering the HTTPS URL.

PERS-01 and PERS-02 are low-risk: they are new `.claude/commands/` Markdown files with tailored frontmatter and intent routing instructions. They call the same underlying MCP tools; only the framing and emphasis differ. No Python code changes required.

WEB-03 requires packaging the project as a Claude Code plugin with `.claude-plugin/plugin.json`, `commands/`, and `.mcp.json` at the plugin root, then submitting via the in-app form.

**Primary recommendation:** Run the HTTP server with `mcp.run(transport="streamable-http", host="0.0.0.0", port=8000)` in a separate `server_http.py` entry point; expose it via ngrok for non-technical users; document the three-step connect flow for claude.ai.

---

## Standard Stack

### Core

| Library | Version | Purpose | Why Standard |
|---------|---------|---------|--------------|
| fastmcp | >=2.0 (3.1.1 current) | HTTP transport for remote MCP | Built-in streamable-http support; same library already in use |
| ngrok | free tier | Tunnel local HTTP port to public HTTPS URL | Zero infrastructure; non-technical users get a stable URL; free tier suitable for demo/personal use |

### Supporting

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| uvicorn | latest | ASGI server for production HTTP mode | Only needed if running FastMCP as ASGI app (not required for simple `mcp.run()`) |
| cloudflare tunnel | free | Alternative to ngrok; no session time limit on free tier | When ngrok's 2-hour free-tier limit is a problem |

### Alternatives Considered

| Instead of | Could Use | Tradeoff |
|------------|-----------|----------|
| ngrok | Cloudflare Tunnel (`cloudflared`) | Cloudflare has no session time limit on free tier but requires one-time account setup and CLI install; ngrok is simpler for first-time setup |
| ngrok | Railway / Render / Fly.io | Cloud hosting gives permanent URL but requires deployment knowledge — defeats WEB-02 non-technical goal |
| `mcp.run(transport="streamable-http")` | Full ASGI + uvicorn + nginx | ASGI stack needed only for multi-worker production; single-user skill does not need it |

**Installation (HTTP server dependencies):**
```bash
# FastMCP HTTP transport is built-in — no extra install needed
# For ngrok tunnel:
pip install pyngrok  # optional Python wrapper, OR install ngrok CLI directly
```

---

## Architecture Patterns

### Recommended Project Structure

```
finance-analyst-plugin/          # Plugin root (for WEB-03 packaging)
├── .claude-plugin/
│   └── plugin.json              # Plugin manifest
├── commands/
│   ├── finance.md               # Copy of .claude/commands/finance.md
│   ├── finance-analyst.md       # PERS-01
│   └── finance-pm.md            # PERS-02
├── skills/
│   └── finance/
│       └── SKILL.md             # Copy of .claude/skills/finance/SKILL.md
└── .mcp.json                    # MCP server config pointing to server.py

src/finance_mcp/
└── server_http.py               # New entry point: same tools, HTTP transport
```

### Pattern 1: FastMCP HTTP Transport Entry Point

**What:** A second server entry point that runs the same FastMCP app with `transport="streamable-http"` instead of stdio.
**When to use:** When serving claude.ai (browser-based) clients that cannot spawn a local process.
**Example:**
```python
# src/finance_mcp/server_http.py
# Source: https://gofastmcp.com/deployment/http
import sys
from finance_mcp.server import mcp   # reuses the same FastMCP instance with all tools

if __name__ == "__main__":
    port = int(sys.argv[1]) if len(sys.argv) > 1 else 8000
    print(f"[finance-mcp] Starting HTTP server on port {port}", file=sys.stderr)
    mcp.run(transport="streamable-http", host="0.0.0.0", port=port)
```

The server is then accessible at `http://localhost:8000/mcp` locally, and behind ngrok at `https://<id>.ngrok-free.app/mcp`.

### Pattern 2: ngrok One-Command Launch

**What:** A startup shell script that starts both the FastMCP HTTP server and an ngrok tunnel, printing the public URL for the user.
**When to use:** WEB-02 non-technical user setup — they run one script, get one URL to paste into claude.ai.
**Example:**
```bash
#!/bin/bash
# scripts/start_web.sh
# Source: ngrok documentation pattern
set -e
cd "$(dirname "$0")/.."
.venv/bin/python -m finance_mcp.server_http 8000 &
SERVER_PID=$!
echo "Finance MCP server started (PID $SERVER_PID)"
echo ""
echo "Starting ngrok tunnel..."
ngrok http 8000 --log stdout &
sleep 3
# ngrok API returns current tunnel URL
PUBLIC_URL=$(curl -s http://localhost:4040/api/tunnels | python3 -c "import sys,json; print(json.load(sys.stdin)['tunnels'][0]['public_url'])" 2>/dev/null || echo "Check http://localhost:4040 for your public URL")
echo ""
echo "============================================"
echo "  Connect claude.ai to: ${PUBLIC_URL}/mcp"
echo "============================================"
echo ""
echo "In claude.ai: Settings > Connectors > Add custom connector"
echo "Paste the URL above, then click Add."
wait $SERVER_PID
```

### Pattern 3: Persona Command Variants

**What:** Separate `.claude/commands/` Markdown files with identical structure to `finance.md` but different system role framing, intent routing emphasis, and output guidance.
**When to use:** PERS-01 and PERS-02 — analyst-focused and PM-focused variants.

**finance-analyst.md frontmatter and role framing:**
```markdown
---
description: Equity analyst view — stock analysis, peer comparison, and valuation metrics
argument-hint: "[e.g. 'compare AAPL vs MSFT vs GOOGL for 2024']"
allowed-tools: Bash(python3:*), Bash(pip:*), Write, Read, mcp__finance__ping, mcp__finance__validate_environment
model: sonnet
---

You are a **sell-side equity analyst**. Your default lens is single-stock and peer-group analysis:
price performance, relative returns, correlation between peers, and risk-adjusted metrics vs benchmarks.

For every analysis:
1. Lead with equity-analyst framing ("From an equity perspective…")
2. Prefer multi-ticker comparisons, peer normalization, and Sharpe/drawdown vs sector peers
3. Flag valuation context when discussing price trends (e.g., "AAPL's 18% gain compares to the S&P 500 +14%")
4. Disclaimer is mandatory on every response
```

**finance-pm.md frontmatter and role framing:**
```markdown
---
description: Portfolio manager view — risk/return attribution, portfolio-level metrics
argument-hint: "[e.g. 'risk metrics for a portfolio of AAPL MSFT JNJ for 2023']"
allowed-tools: Bash(python3:*), Bash(pip:*), Write, Read, mcp__finance__ping, mcp__finance__validate_environment
model: sonnet
---

You are a **portfolio manager**. Your default lens is portfolio-level attribution:
correlation between holdings, portfolio volatility vs benchmark, Sharpe ratio attribution,
and maximum drawdown across the book.

For every analysis:
1. Lead with portfolio-level framing ("From a portfolio perspective…")
2. Default to multi-ticker risk aggregation — treat tickers as portfolio holdings, not isolated securities
3. Emphasize correlation heatmaps and diversification implications
4. Highlight drawdown and beta vs S&P 500 as primary risk metrics
5. Disclaimer is mandatory on every response
```

### Pattern 4: Claude Code Plugin Packaging

**What:** Convert the existing `.claude/` standalone configuration into a distributable plugin with `.claude-plugin/plugin.json`.
**When to use:** WEB-03 marketplace submission.
**Example:**
```json
// finance-analyst-plugin/.claude-plugin/plugin.json
// Source: https://code.claude.com/docs/en/plugins-reference
{
  "name": "finance-mcp",
  "version": "1.0.0",
  "description": "Finance AI skill — stock analysis, market metrics, ML workflows for finance professionals",
  "author": {
    "name": "Finance MCP Project"
  },
  "homepage": "https://github.com/[owner]/machine_learning_skill",
  "repository": "https://github.com/[owner]/machine_learning_skill",
  "license": "MIT",
  "keywords": ["finance", "stocks", "yfinance", "ml", "market-analysis"]
}
```

### Anti-Patterns to Avoid

- **Modifying stdio server.py for HTTP**: Do not add `transport="streamable-http"` to the existing `server.py`. The existing stdio server is used by Claude Code via `.mcp.json`. Keep them separate entry points.
- **Putting `commands/` inside `.claude-plugin/`**: The manifest directory contains ONLY `plugin.json`. All other directories (commands, skills, hooks) must be at the plugin root.
- **Dynamic context injection (`!command`) in persona variants**: The `!` syntax only works in Claude Code commands, not in claude.ai chat. Persona command files work for Claude Code; for claude.ai, the MCP tools themselves provide context.
- **Authless HTTP server on a public URL**: FastMCP HTTP transport supports bearer token auth. For personal/demo use via ngrok, authless is acceptable. For any shared deployment, add token auth.

---

## Don't Hand-Roll

| Problem | Don't Build | Use Instead | Why |
|---------|-------------|-------------|-----|
| Public HTTPS URL for local server | Custom reverse proxy with SSL | ngrok or cloudflare tunnel | SSL cert management, DNS, and uptime are solved problems; tunnel tools handle all of it |
| HTTP transport for FastMCP | Custom Flask/FastAPI wrapper around MCP protocol | `mcp.run(transport="streamable-http")` | FastMCP has native HTTP transport; wrapping it introduces protocol translation bugs |
| Claude Code plugin manifest | Custom installation scripts | Standard `.claude-plugin/plugin.json` + marketplace submission form | Official plugin system handles installation, scoping, and updates |
| Persona routing logic | New Python code | Markdown command files with different framing | Persona behavior is entirely prompt-driven; no code needed |

**Key insight:** All four deliverables in Phase 4 are configuration and documentation work, not new Python code. The MCP tools are complete. Phase 4 wires existing tools to new surfaces.

---

## Common Pitfalls

### Pitfall 1: stdio vs HTTP Transport Confusion

**What goes wrong:** `mcp.run()` with no arguments defaults to stdio. If a developer starts the server expecting HTTP, it silently reads from stdin instead of listening on a port.
**Why it happens:** FastMCP's default transport is stdio for Claude Code compatibility.
**How to avoid:** Always pass `transport="streamable-http"` explicitly in `server_http.py`. Never rely on defaults when switching transports.
**Warning signs:** Server starts but no port binding message appears; `curl http://localhost:8000/mcp` returns connection refused.

### Pitfall 2: ngrok Free Tier Session Limits

**What goes wrong:** ngrok's free tier terminates tunnels after 2 hours and randomizes the URL on each restart. Users who saved the URL from the previous session get a connection error.
**Why it happens:** ngrok free tier does not support persistent URLs without a paid plan.
**How to avoid:** Document in the user guide that the URL changes on restart. Alternatively, use Cloudflare Tunnel (`cloudflared tunnel --url http://localhost:8000`) which has no session time limit on the free tier.
**Warning signs:** claude.ai Connectors shows the server as disconnected after a few hours.

### Pitfall 3: Plugin Commands Are Namespaced

**What goes wrong:** Users try `/finance` and `/finance-analyst` after installing the plugin, but the commands are actually `/finance-mcp:finance` and `/finance-mcp:finance-analyst`.
**Why it happens:** All plugin commands are namespaced by the plugin `name` field in `plugin.json`.
**How to avoid:** Document the namespaced command names prominently. The standalone `.claude/commands/` files continue to provide un-namespaced `/finance`, `/finance-analyst`, `/finance-pm` commands for project-level use.
**Warning signs:** `/finance` works as a standalone command but not after plugin install.

### Pitfall 4: Dynamic Context Injection Not Available in claude.ai

**What goes wrong:** The `!` injection syntax (e.g., `!python3 --version`) in command Markdown files only executes in Claude Code's local terminal context. In claude.ai chat, these lines are passed as literal text or silently dropped.
**Why it happens:** Dynamic context injection is a Claude Code feature, not part of the MCP protocol or claude.ai interface.
**How to avoid:** Persona command files for claude.ai must NOT rely on `!` injections. The MCP tools (`validate_environment`, `ping`) provide equivalent context via tool calls. Keep `!` injections only in the Claude Code–specific command files.
**Warning signs:** claude.ai shows raw `!python3 --version` text in the conversation instead of the version output.

### Pitfall 5: Plugin `.mcp.json` Paths Break After Install

**What goes wrong:** The plugin's `.mcp.json` hardcodes the Python path to `.venv/bin/python` relative to the original project directory. After marketplace install, the plugin is in `~/.claude/plugins/cache/` and the relative path is wrong.
**Why it happens:** Plugin files are copied to a cache directory; relative paths that assume the project root break.
**How to avoid:** Use `${CLAUDE_PLUGIN_ROOT}` in plugin `.mcp.json` for all paths. Note that the Finance MCP plugin's full Python environment cannot be bundled in a marketplace plugin — the `.mcp.json` in the plugin must point to a system-installed or user-installed server, not a project-local `.venv`.
**Warning signs:** Plugin MCP server fails to start after installation from marketplace.

---

## Code Examples

Verified patterns from official sources:

### FastMCP HTTP Server Startup

```python
# Source: https://gofastmcp.com/deployment/http
# src/finance_mcp/server_http.py
from finance_mcp.server import mcp  # reuse existing FastMCP instance

if __name__ == "__main__":
    mcp.run(transport="streamable-http", host="0.0.0.0", port=8000)
    # Accessible at http://localhost:8000/mcp
    # Behind ngrok: https://<id>.ngrok-free.app/mcp
```

### FastMCP CLI Alternative (no code change needed)

```bash
# Source: https://gofastmcp.com/deployment/running-server
# Run HTTP server directly from CLI — no server_http.py needed
fastmcp run src/finance_mcp/server.py --transport http --port 8000
```

### Claude.ai Connector URL Format

```
https://<ngrok-or-custom-domain>/mcp

# Example with ngrok:
https://a1b2c3d4e5f6.ngrok-free.app/mcp

# User flow (verified via support.claude.com):
# claude.ai > Settings > Connectors > Add custom connector
# Enter URL > (optional OAuth) > Add
# In conversation: click "+" > Connectors > toggle Finance MCP on
```

### Plugin Manifest Minimum Viable

```json
// Source: https://code.claude.com/docs/en/plugins-reference
// .claude-plugin/plugin.json
{
  "name": "finance-mcp",
  "version": "1.0.0",
  "description": "Finance AI skill — stock analysis, market metrics, and ML workflows",
  "author": { "name": "Finance MCP Project" },
  "keywords": ["finance", "stocks", "ml", "yfinance"]
}
```

### Plugin Directory Structure

```
finance-mcp-plugin/
├── .claude-plugin/
│   └── plugin.json
├── commands/
│   ├── finance.md
│   ├── finance-analyst.md
│   └── finance-pm.md
├── skills/
│   └── finance/
│       └── SKILL.md
└── .mcp.json
```

---

## State of the Art

| Old Approach | Current Approach | When Changed | Impact |
|--------------|------------------|--------------|--------|
| SSE (Server-Sent Events) HTTP transport | Streamable HTTP transport | MCP spec update 2025 | SSE may be deprecated; new servers should use streamable-http |
| Claude Code only (no browser support) | claude.ai "Integrations" feature for Pro/Max/Team/Enterprise | May 2025 | Browser users can now connect remote MCP servers |
| Manual plugin distribution (copy files) | Claude Code plugin system with marketplaces | Early 2026 (v1.0.33+) | One-command install; scoped deployment; submission form |
| Separate `mcp-remote` proxy for Claude Desktop → remote | Native streamable-http support in Claude Code | 2025 | No proxy needed; direct URL in config |

**Deprecated/outdated:**
- **SSE transport**: Still works but may be deprecated per Anthropic docs. New implementations should use streamable-http.
- **`claude_desktop_config.json` for remote servers**: Claude Desktop does not connect remote MCP servers via config file — only local stdio servers. Remote connections go through claude.ai Settings > Connectors.

---

## Open Questions

1. **Authentication requirement for claude.ai connectors**
   - What we know: Authless (no auth) servers are supported by claude.ai per official docs. OAuth is optional.
   - What's unclear: Whether Anthropic will require authentication for marketplace-listed servers.
   - Recommendation: Build authless for now (sufficient for WEB-01/WEB-02). Document that OAuth can be added if required for marketplace approval.

2. **Marketplace review timeline and criteria**
   - What we know: Submission is via `claude.ai/settings/plugins/submit` or `platform.claude.com/plugins/submit`. Anthropic performs basic automated review, with additional review for "Anthropic Verified" badge.
   - What's unclear: Review turnaround time; specific rejection criteria; whether a demo of working tools is required.
   - Recommendation: Submit WEB-03 with complete README, screenshots of each tool in action, and functional plugin.json. Plan for multi-week review cycle.

3. **ngrok free tier vs Cloudflare Tunnel for WEB-02 guide**
   - What we know: ngrok free tier randomizes URL on restart; Cloudflare Tunnel has no time limit but requires account.
   - What's unclear: Which provides a better user experience for the target (finance professional, non-technical).
   - Recommendation: Document both. Use ngrok for the primary quick-start guide (simpler CLI), note Cloudflare Tunnel as the persistent alternative.

4. **plugin .mcp.json vs project .mcp.json for WEB-03**
   - What we know: Plugin `.mcp.json` uses `${CLAUDE_PLUGIN_ROOT}` for paths; the Finance MCP server requires a Python environment with 7 specific packages.
   - What's unclear: Whether marketplace plugins can require the user to have a pre-installed Python package set, or if the plugin must be self-contained.
   - Recommendation: For WEB-03 (marketplace packaging), document that the server requires Python + packages as a prerequisite. The plugin `.mcp.json` points to a pip-installed `finance-mcp` package, not a `.venv` path. This means `pyproject.toml` needs a `[project.scripts]` entry point for `finance-mcp-server`.

---

## Validation Architecture

### Test Framework

| Property | Value |
|----------|-------|
| Framework | pytest 7+ |
| Config file | `pyproject.toml` (`[tool.pytest.ini_options]`) |
| Quick run command | `.venv/bin/python -m pytest tests/ -q --tb=short` |
| Full suite command | `.venv/bin/python -m pytest tests/ -v` |

### Phase Requirements → Test Map

| Req ID | Behavior | Test Type | Automated Command | File Exists? |
|--------|----------|-----------|-------------------|-------------|
| WEB-01 | HTTP server starts and responds at `/mcp` endpoint | smoke | `pytest tests/test_http_server.py -x` | ❌ Wave 0 |
| WEB-02 | Connection guide script produces a URL (ngrok mock) | manual-only | Manual verification — ngrok requires live internet | N/A |
| WEB-03 | Plugin manifest is valid JSON with required fields | unit | `pytest tests/test_plugin_manifest.py -x` | ❌ Wave 0 |
| PERS-01 | `finance-analyst.md` exists with correct frontmatter | unit | `pytest tests/test_persona_commands.py::test_analyst_command -x` | ❌ Wave 0 |
| PERS-02 | `finance-pm.md` exists with correct frontmatter | unit | `pytest tests/test_persona_commands.py::test_pm_command -x` | ❌ Wave 0 |

> WEB-02 is manual-only: verifying that ngrok produces a valid public HTTPS URL and that claude.ai can connect through it requires a live network and a claude.ai session — not automatable in unit tests.

### Sampling Rate

- **Per task commit:** `.venv/bin/python -m pytest tests/ -q --tb=short`
- **Per wave merge:** `.venv/bin/python -m pytest tests/ -v`
- **Phase gate:** Full suite green before `/gsd:verify-work`

### Wave 0 Gaps

- [ ] `tests/test_http_server.py` — covers WEB-01: tests that `server_http.py` imports cleanly and that `mcp.run` can be called with `transport="streamable-http"` without error (mock the run call)
- [ ] `tests/test_plugin_manifest.py` — covers WEB-03: reads `finance-mcp-plugin/.claude-plugin/plugin.json`, validates required fields (`name`, `version`, `description`), checks commands directory contains expected `.md` files
- [ ] `tests/test_persona_commands.py` — covers PERS-01 and PERS-02: reads `finance-analyst.md` and `finance-pm.md`, checks frontmatter fields (`description`, `allowed-tools`, `model`) and that each file contains persona-specific role framing text

---

## Sources

### Primary (HIGH confidence)

- `https://gofastmcp.com/deployment/http` — FastMCP HTTP/streamable-http transport configuration
- `https://gofastmcp.com/deployment/running-server` — FastMCP CLI transport modes
- `https://code.claude.com/docs/en/plugins` — Claude Code plugin structure and creation
- `https://code.claude.com/docs/en/plugins-reference` — Complete plugin.json manifest schema
- `https://code.claude.com/docs/en/discover-plugins` — Marketplace submission process and URLs
- `https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp` — claude.ai connector setup for end users
- `https://support.claude.com/en/articles/11503834-building-custom-connectors-via-remote-mcp-servers` — Remote MCP server technical requirements for claude.ai

### Secondary (MEDIUM confidence)

- `https://pypi.org/project/fastmcp/` — FastMCP 3.1.1 latest version confirmed (March 2026)
- `https://ngrok.com/docs/using-ngrok-with/using-mcp` — ngrok as MCP gateway pattern
- `https://developers.cloudflare.com/agents/guides/remote-mcp-server/` — Cloudflare Tunnel as alternative hosting

### Tertiary (LOW confidence)

- `https://www.getclockwise.com/blog/claude-remote-mcp-server-getting-started` — End-user claude.ai connector walkthrough (third-party blog, verified against official docs)
- `https://max-productive.ai/blog/claude-ai-connectors-guide-2025/` — Connector setup UX description

---

## Metadata

**Confidence breakdown:**
- Standard stack: HIGH — FastMCP HTTP transport verified via official docs; ngrok/cloudflare tunnel verified
- Architecture: HIGH — Plugin structure verified via official Claude Code plugin reference docs
- Pitfalls: MEDIUM — stdio/HTTP confusion and namespace pitfalls verified; marketplace review criteria LOW (not documented publicly)
- Persona variants: HIGH — command file structure matches existing `finance.md` pattern already in project

**Research date:** 2026-03-18
**Valid until:** 2026-04-18 (stable APIs; plugin system docs may evolve with new Claude Code versions)
