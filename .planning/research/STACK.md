# Stack Research

**Domain:** Claude Code native skill — Python finance analysis
**Researched:** 2026-03-17
**Confidence:** HIGH (Claude Code architecture from local authoritative docs; Python library versions from training data August 2025, flagged where uncertain)

---

## Claude Code Skill Architecture

**Confidence:** HIGH — verified from local official plugin documentation at `~/.claude/plugins/marketplaces/claude-plugins-official/`

This project is a **slash command** (`/finance`), not a skill. The distinction matters:

| Component | File location | Who invokes | Use for |
|-----------|--------------|-------------|---------|
| **Command** (`.md` file) | `.claude/commands/finance.md` | User types `/finance` | User-initiated analysis tasks |
| **Skill** (`SKILL.md` file) | `.claude/skills/finance/SKILL.md` | Claude invokes automatically | Background knowledge/context |
| **Agent** (`.md` file) | `.claude/agents/finance-analyst.md` | Claude spawns via Task tool | Long multi-step sub-workflows |

### Recommended Structure for This Project

```
.claude/
└── commands/
    ├── finance.md                    # /finance — generalist entry point
    ├── finance-analyst.md            # /finance-analyst — analyst persona
    └── finance-trader.md             # /finance-trader — PM/trader persona
```

Optionally add a skill for passive context injection:

```
.claude/
└── skills/
    └── finance-conventions/
        └── SKILL.md                  # Auto-applied finance output conventions
```

### Command File Format

```markdown
---
description: Run financial analysis from plain English
argument-hint: [request]
allowed-tools: Bash(python3:*), Read, Write
model: sonnet
---

Analyze the following finance request: $ARGUMENTS

[Instructions to Claude for code generation, execution, and interpretation...]
```

### Key Architecture Facts (HIGH confidence)

- **File format:** Markdown with optional YAML frontmatter
- **Invocation:** User types `/finance [natural language request]`
- **Arguments:** `$1`, `$2`, ..., or `$ARGUMENTS` (full string) — accessed in prompt body
- **Bash execution:** `!`command`` syntax injects live command output into the prompt before Claude sees it; `allowed-tools: Bash(python3:*)` permits script execution
- **Python execution mechanism:** Claude generates a Python script, runs it via the Bash tool (`python3 script.py`), reads stdout/stderr, interprets results — all within a single command invocation
- **File references:** `@filename` syntax includes file contents inline — useful for loading user CSV files
- **Plugin root:** `${CLAUDE_PLUGIN_ROOT}` resolves to plugin directory for portable paths when distributed as a plugin
- **Model selection:** `model: sonnet` is correct for this use case — complex code generation with interpretation; avoid `haiku` (too weak for finance ML), reserve `opus` for architectural decisions only
- **State management:** Multi-step workflows can write/read `.claude/*.local.md` files to carry state across command invocations

### What "Runs Python" Means

There is no sandboxed Python runtime in Claude Code. The skill generates Python code as a string, writes it to a temp file, and executes it via `Bash(python3:*)`. The user's local Python environment is the runtime. This means:

1. The user must have the required libraries installed (`pip install yfinance pandas scikit-learn ...`)
2. The skill should check for dependencies on first run and print install instructions if missing
3. Charts saved as PNG files (not interactive) are the reliable output format for the terminal environment
4. Stdout from the script becomes the data Claude interprets and summarizes

---

## Recommended Stack

### Core Technologies

| Technology | Version | Purpose | Why Recommended |
|------------|---------|---------|-----------------|
| Python | 3.10+ | Runtime language | Course teaches Python 3; 3.10+ required for `match` statements and current scikit-learn; 3.12 is current stable (August 2025) |
| pandas | 2.x (2.2+) | DataFrames, time series, CSV I/O | Standard for tabular finance data; course uses it throughout all ML notebooks; 2.x required — yfinance 0.2+ returns pandas 2.x-compatible data |
| NumPy | 1.26+ / 2.x | Array math, vectorized operations | Foundation beneath pandas and scikit-learn; course PF 01-02 teaches NumPy directly; use 1.26.x for broadest compatibility or 2.0+ if pandas 2.2+ is confirmed |
| scikit-learn | 1.4+ | Regression, classification, pipelines, cross-validation | Covers 100% of course ML scope (ML 03, ML 05-06); Pipeline API, GridSearchCV, StratifiedKFold all directly used |
| yfinance | 0.2.x (0.2.40+) | Download stock price/fundamentals data from Yahoo Finance | Free, no API key required, course-confirmed data source; 0.2.x rewrote the download internals for reliability; earlier versions break frequently |
| matplotlib | 3.8+ | Charts: line plots, histograms, scatter plots | Course standard; required for all ML 01 visualizations; saves PNG files which work reliably in terminal context |
| seaborn | 0.13+ | Statistical charts: heatmaps, pair plots, box plots | Course ML 01 uses seaborn directly; built on matplotlib; 0.13 added object-oriented API |

### Supporting Libraries

| Library | Version | Purpose | When to Use |
|---------|---------|---------|-------------|
| statsmodels | 0.14+ | OLS regression with p-values, R², confidence intervals | When liquidity predictor needs statistical inference beyond sklearn's metrics; produces finance-readable regression tables |
| scipy | 1.12+ | Statistical tests (t-test, normality), optimization | When analyst persona needs hypothesis testing on returns data |
| openpyxl | 3.1+ | Read `.xlsx` Excel files | When users provide Excel files instead of CSV; pandas `read_excel()` backend |
| tabulate | 0.9+ | Format DataFrames as clean markdown/ASCII tables | When outputting summary tables to Claude's terminal response — more readable than default DataFrame repr |
| PyPortfolioOpt | 1.5+ | Mean-variance optimization, efficient frontier | Phase 2 (PM/Trader persona) — portfolio construction workflows; NOT needed for v1 |
| arch | 6.x | GARCH volatility modeling | Phase 2 (PM/Trader persona) — deferred; overkill for v1 course scope |

### Development Tools

| Tool | Purpose | Notes |
|------|---------|-------|
| pip | Dependency management | Use `pip install` in skill setup instructions; no virtual env needed at runtime since user environment is the runtime |
| pytest | Unit testing the skill command logic | Test Python helper scripts that the command generates/runs |
| black | Code formatting | Ensure generated Python scripts are readable when users inspect them |

---

## Installation

The skill itself requires no installation. The user's machine needs the Python libraries.

Provide this as a setup block in the command's first-run check:

```bash
# Core (required for all /finance commands)
pip install yfinance pandas numpy scikit-learn matplotlib seaborn

# Supporting (for richer output)
pip install statsmodels scipy openpyxl tabulate

# Verify
python3 -c "import yfinance, pandas, numpy, sklearn, matplotlib, seaborn; print('Finance stack ready')"
```

---

## Alternatives Considered

| Recommended | Alternative | When to Use Alternative |
|-------------|-------------|-------------------------|
| yfinance 0.2.x | pandas-datareader | When Yahoo Finance data is unreliable and FRED/Quandl data sources are preferred; pandas-datareader has broader source support but requires API keys for most premium data |
| yfinance 0.2.x | Alpha Vantage API | When real-time intraday data is needed (v1 out of scope) |
| matplotlib/seaborn | plotly | When interactive HTML charts are needed; plotly produces `.html` files that are harder to integrate into terminal output; defer to Phase 2 if web UI is ever added |
| matplotlib/seaborn | altair | When vega-lite JSON specs are needed for embedding; unnecessary complexity for this use case |
| scikit-learn | statsmodels | Use statsmodels as supplement for statistical inference tables, not as replacement; sklearn has superior pipeline/cross-validation tooling |
| scikit-learn | PyTorch / TensorFlow | Deep learning overkill for course scope (linear regression, classification); heavy installation burden for users |
| sklearn Pipeline | Manual preprocessing | Pipeline prevents data leakage across train/test split; course ML 03 teaches Pipeline explicitly — match the course |
| pandas 2.x | polars | Polars is faster but breaks yfinance compatibility and changes the API surface the course teaches; stay with pandas |
| Python script via Bash | Jupyter kernel execution | Jupyter requires a running server; Bash + python3 is stateless, simple, and reliable in Claude Code context |

---

## What NOT to Use

| Avoid | Why | Use Instead |
|-------|-----|-------------|
| yfinance < 0.2.0 | Yahoo Finance changed their API; pre-0.2 versions return malformed or empty data unpredictably | yfinance 0.2.40+ |
| pandas < 2.0 | Copy-on-write semantics changed in 2.0; `.loc` assignment patterns from older tutorials silently fail; yfinance 0.2.x expects pandas 2.x types | pandas 2.2+ |
| numpy < 1.26 | Incompatible with pandas 2.x and scikit-learn 1.4+; type promotion changed | numpy 1.26+ |
| QuantLib | Extremely heavy C++ dependency for derivatives pricing; far beyond course scope; install is non-trivial | scipy for basic math |
| Zipline / backtrader | Backtesting engines requiring event loop architecture; out of scope for v1; course does not cover backtesting | Deferred to Phase 2 |
| pyfolio | Depends on zipline/empyrical ecosystem; unmaintained (last release 2021); not course scope | statsmodels + manual returns analysis |
| BloombergGPT / FinGPT | Specialized LLMs for finance NLP tasks (earnings analysis, sentiment); Claude already handles NLP; these add infrastructure complexity with no benefit here | Claude's built-in reasoning |
| OpenBB Terminal | Full terminal UI for finance data; conflicts with the slash command interaction model; wraps many of the same underlying libraries | Direct library calls |
| plotly (v1) | Produces HTML files, not PNG; terminal display requires extra steps; harder to include in Claude's response | matplotlib savefig() → PNG |
| `*` wildcard in allowed-tools | Grants all bash commands; security over-exposure | `Bash(python3:*)` — scoped to only Python execution |

---

## Stack Patterns by Persona Variant

**Analyst persona (`/finance-analyst`):**
- Emphasize: pandas data cleaning, matplotlib/seaborn charts, regression output tables via statsmodels
- Key workflows: outlier detection, categorical encoding, correlation heatmaps, liquidity regression
- No portfolio optimization needed

**Portfolio Manager / Trader persona (`/finance-trader`):**
- Emphasize: returns calculations, Sharpe ratio, volatility (rolling std), yfinance multi-ticker downloads
- Phase 2 additions: PyPortfolioOpt for efficient frontier, arch for GARCH volatility
- Charts: multi-line price charts, drawdown charts

**Generalist persona (`/finance`) — v1 default:**
- Covers both above at lighter depth
- Detects task type from natural language input and routes to appropriate workflow
- Uses the full stack minus Phase 2 libraries

---

## Version Compatibility

| Package | Compatible With | Notes |
|---------|-----------------|-------|
| yfinance 0.2.40+ | pandas 2.x, numpy 1.26+ | 0.2.x is a ground-up rewrite; do not mix with yfinance 0.1.x patterns from older tutorials |
| pandas 2.2 | numpy 1.26 or numpy 2.0 | pandas 2.2 supports both numpy generations; prefer numpy 1.26 for widest library compat |
| scikit-learn 1.4 | numpy 1.17–2.0, scipy 1.5+ | 1.4 added metadata routing; Pipeline API stable since 1.0 |
| seaborn 0.13 | matplotlib 3.3+, pandas 1.1+ | 0.13 rewrote the API to objects interface; older seaborn tutorials use deprecated `distplot()` — use `histplot()` instead |
| matplotlib 3.8 | numpy 1.21+, Python 3.8+ | No compatibility issues with rest of stack |
| statsmodels 0.14 | numpy 1.22+, pandas 1.4+, scipy 1.7+ | Compatible with full recommended stack |

---

## Claude Code Skill Architecture — Confidence Notes

**HIGH confidence** (from local official docs, read directly):
- Command format: `.claude/commands/name.md` with YAML frontmatter
- Bash execution via `!`cmd`` syntax and `allowed-tools: Bash(python3:*)`
- Argument access via `$1`, `$2`, `$ARGUMENTS`
- File inclusion via `@filename`
- Skill auto-invocation via `SKILL.md` description triggers
- `${CLAUDE_PLUGIN_ROOT}` variable for plugin-distributed paths
- State persistence via `.claude/*.local.md` files

**MEDIUM confidence** (from training data, not verified against live Anthropic docs due to tool restrictions):
- Exact Python version floor (3.10+ assumed from scikit-learn 1.4 requirements)
- Library versions are as of August 2025 knowledge cutoff — verify with `pip install --upgrade` on first deploy

**LOW confidence** (training data only, flag for validation before ship):
- Whether Claude Code's Bash tool has a timeout on long-running Python scripts (financial ML on large datasets could take 30-60s)
- Whether matplotlib `plt.show()` blocks execution; use `plt.savefig()` + `plt.close()` pattern unconditionally
- pandas-datareader current maintenance status (last known stable: 0.10.x for FRED data)

---

## Sources

- `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md` — Command frontmatter format (HIGH confidence)
- `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/examples/simple-commands.md` — Command patterns including Bash execution (HIGH confidence)
- `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/plugin-features-reference.md` — Plugin paths, CLAUDE_PLUGIN_ROOT, component integration (HIGH confidence)
- `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/advanced-workflows.md` — State management, multi-step patterns (HIGH confidence)
- `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/example-plugin/skills/example-skill/SKILL.md` — Skill vs command distinction (HIGH confidence)
- `~/.claude/plugins/marketplaces/claude-plugins-official/plugins/claude-code-setup/skills/claude-automation-recommender/references/skills-reference.md` — Skill frontmatter fields including `user-invocable`, `disable-model-invocation`, `context: fork` (HIGH confidence)
- Training data (August 2025 cutoff) — Python library versions, yfinance 0.2.x architecture, pandas 2.x changes, scikit-learn Pipeline API (MEDIUM confidence)

---

*Stack research for: Finance AI Skill for Claude Code*
*Researched: 2026-03-17*
