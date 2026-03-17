# Project Research Summary

**Project:** Finance AI Skill for Claude Code (pyfi.com curriculum-aligned)
**Domain:** Claude Code slash command with Python finance code generation and execution
**Researched:** 2026-03-17
**Confidence:** HIGH (Claude Code architecture); MEDIUM (features, Python library versions)

## Executive Summary

This project is a Claude Code native slash command (`/finance`) that accepts natural language finance requests, generates Python analysis scripts, executes them via the Bash tool against the user's local Python environment, and returns plain-English interpretations of results. It is not a web app, not a Jupyter notebook, and not a plugin with its own runtime — the user's machine is the execution environment. Research across all four domains converges on a clear implementation model: a thin command file handles invocation mechanics, a separate SKILL.md holds domain expertise and intent routing, generated Python scripts follow a write-to-disk-then-execute pattern, and all output flows through a consistent `.finance-output/` directory. This separation of concerns is architecturally mandated by the Claude Code plugin system and should not be collapsed.

The recommended stack is unambiguous: yfinance 0.2.40+ for market data, pandas 2.2+ for data manipulation, scikit-learn 1.4+ with Pipeline for ML workflows, and matplotlib/seaborn for headless chart output. The scope aligns directly with the pyfi.com curriculum — ML 01 EDA, ML 03 regression, ML 05-06 classification — meaning the feature set is constrained and well-defined rather than open-ended. The competitive gap this skill fills is unique: no existing tool combines natural language interface, ML modeling on user-provided CSV data, code transparency, and native Claude Code integration at no additional cost.

The dominant risks are not implementation complexity but correctness and legal. Using unadjusted price data (`Close` instead of `Adj Close`) silently poisons all return calculations. Look-ahead bias from fitting sklearn transformers before the train/test split invalidates ML results entirely. And outputs that read as investment recommendations rather than educational analysis create regulatory exposure. All three of these must be addressed in Phase 1 infrastructure — they cannot be retrofitted. Build the data-fetch adapter, output formatting conventions, and disclaimer templates before writing a single analysis workflow.

---

## Key Findings

### Recommended Stack

The Claude Code skill architecture uses a two-file model per capability: a `.claude/commands/name.md` command file (user-facing entry point) and a `.claude/skills/name/SKILL.md` skill file (domain knowledge). Python execution is handled entirely through `Bash(python3:*)` — Claude generates a Python script, writes it to `.finance-output/last_run.py` with the Write tool, and executes it via Bash. There is no sandboxed runtime; the user's local Python environment is the runtime, which means dependency management is the user's responsibility with setup instructions provided by the skill.

**Core technologies:**
- Python 3.10+: Runtime language — required by scikit-learn 1.4+ and yfinance 0.2.x
- pandas 2.2+: DataFrames, time series, CSV I/O — course-standard; yfinance 0.2.x returns pandas 2.x-compatible data
- NumPy 1.26+: Array math — foundation beneath pandas and scikit-learn; 1.26 gives broadest compat
- scikit-learn 1.4+: Regression, classification, Pipeline API — covers 100% of course ML scope (ML 03, ML 05-06)
- yfinance 0.2.40+: Market data — free, no API key, course-confirmed; pin to 0.2.x to avoid silent breakage
- matplotlib 3.8+ / seaborn 0.13+: Charts — headless `Agg` backend only; always `savefig()`, never `show()`
- statsmodels 0.14+: OLS tables with p-values — supplement to sklearn for statistical inference output
- tabulate 0.9+: Markdown/ASCII table formatting — improves terminal readability of DataFrame summaries

**What NOT to use:** polars (breaks yfinance), plotly v1 (produces HTML not PNG), yfinance < 0.2.0, pandas < 2.0, any deep learning framework (overkill for course scope), PyTorch/TensorFlow.

### Expected Features

The feature research draws a sharp line between what finance professionals assume any tool has (table stakes), what makes this skill distinctive (differentiators), and what must be deliberately excluded from v1 (anti-features). The entire v1 feature set maps to the pyfi.com curriculum notebooks — this is both a constraint and an asset.

**Must have (table stakes):**
- Stock price data fetch by ticker via yfinance — first thing any user tries
- Price charts (line, return distribution histogram) — visual output is non-negotiable
- Returns calculation (daily, cumulative) and annualized volatility — every analyst needs these
- Sharpe ratio and basic risk metrics vs. benchmark — single most-asked portfolio question
- CSV/Excel file ingestion with format tolerance — required root dependency for both ML workflows
- Data exploration summary (EDA) — data quality gate before any ML; matches ML 01 curriculum
- Outlier detection and flagging — directly taught in ML 01
- Graceful error handling with finance-specific messages — cryptic errors kill adoption with non-programmers
- Plain-English interpretation on every output — the core differentiator vs. raw Python

**Should have (competitive differentiators):**
- Liquidity predictor (regression ML, ML 03) — real risk desk problem; first ML workflow
- Investor/client classifier (classification ML, ML 05-06) — replaces manual profiling spreadsheets
- Multi-ticker comparison with normalized price index — analysts never look at one stock in isolation
- Correlation matrix heatmap — standard portfolio construction tool
- Cross-validation score interpretation — explains model validity to non-ML users
- Persona-aware phrasing (Analyst vs. PM/Trader) — Phase 2 split commands

**Defer (v2+):**
- Real-time/streaming prices — yfinance does not support it; requires entirely different infrastructure
- Bloomberg/Alpha Vantage/Refinitiv data sources — API keys, paid tiers, authentication complexity
- Portfolio optimization (mean-variance/Black-Litterman) — mathematically brittle; high correctness risk
- Backtesting framework — separate product; look-ahead bias risk is high if built incorrectly
- DCF / fundamental valuation — requires different data source; formula assumptions vary by sector
- Web dashboard (Streamlit/Dash) — separate product surface entirely
- PDF/Word report export — formatting complexity without core value change

### Architecture Approach

The architecture follows a strict separation: the slash command is a thin router (invocation mechanics, tool allowances, dynamic context injection), and the SKILL.md is the brain (intent classification, code generation guidance, output conventions). Python scripts are always written to `.finance-output/last_run.py` before execution — never run inline in Bash. Environment state (available CSV files, installed packages, Python path) is injected into the prompt via `!`command`` syntax before Claude generates any code, ensuring generated code matches the actual environment. All chart output uses the matplotlib `Agg` backend and writes PNG files to `.finance-output/charts/`.

**Major components:**
1. Slash command file (`.claude/commands/finance.md`) — accepts `$ARGUMENTS`, defines `allowed-tools`, injects environment context, delegates to skill
2. Core SKILL.md (`.claude/skills/finance/SKILL.md`) — intent classification (stock analysis / liquidity model / investor classifier), code generation guidance, output format rules
3. Reference files (`references/task-types.md`, `references/libraries.md`, `references/output-format.md`) — detailed lookup tables loaded via `@path` to keep SKILL.md under ~300 lines
4. Code templates (`templates/stock-analysis.py.md`, `templates/liquidity-model.py.md`, `templates/investor-classifier.py.md`) — reduce hallucination in library API calls
5. Output layer (`.finance-output/last_run.py`, `.finance-output/charts/*.png`) — consistent locations for inspection and debugging
6. Persona overlays (`finance-analyst/SKILL.md`, `finance-pm/SKILL.md`) — thin framing layers added in Phase 2+, built only after the generalist command is proven

### Critical Pitfalls

Eight critical pitfalls were identified, all with HIGH recovery cost if discovered late. The top five that must be addressed in Phase 1:

1. **Adjusted vs. unadjusted price confusion** — Always use `Adj Close`, never `Close`, for return calculations. Check for column presence after every fetch and raise an explicit error if missing. Bake this into the data-fetch adapter before any analysis code is written on top.

2. **Look-ahead bias in ML feature engineering** — Always split data first (`train_test_split`), then fit all transformers exclusively on training data. Never call `.fit_transform()` on the full dataset. Use `TimeSeriesSplit` for time-ordered financial data. This must be a design constraint from the first ML workflow line, not a retrofit.

3. **yfinance rate limiting and silent data gaps** — After every `yf.download()`, check `df.empty` and validate row count against expected trading days. Surface friendly errors to users, never raw exceptions. Cache successful fetches to avoid re-fetching during iterative analysis.

4. **ML output treated as investment advice** — Every output that includes a prediction or classification MUST include a hardcoded disclaimer that it is for educational purposes only and not investment advice. This is a legal requirement, not a nicety. Build it into the output template in Phase 1.

5. **Date and timezone misalignment in multi-ticker merges** — Normalize all timestamps to UTC immediately after fetch. Use date-only index for daily data. Test with at least one non-US ticker. Misalignment silently produces wrong correlation calculations.

Additionally: yfinance API instability requires a thin adapter layer (single point of change when Yahoo breaks their API), and non-stationarity of financial time series requires using returns (not raw prices) as ML features.

---

## Implications for Roadmap

Based on the combined research, the architecture's own dependency graph dictates a natural phase structure. All four research files independently converge on the same ordering.

### Phase 1: Infrastructure and Data Foundation

**Rationale:** Everything else depends on this. Data correctness (adjusted prices, timezone normalization, empty DataFrame detection), output conventions (chart directory, last_run.py location, disclaimer templates), and the yfinance adapter layer must exist before any analysis workflow is built. Building analysis on top of shaky data infrastructure means every workflow inherits the bugs. The architecture research explicitly calls this out as a prerequisite. The pitfalls research identifies the highest-severity issues as Phase 1 concerns.

**Delivers:** A working `/finance` command skeleton with correct data fetching, output directory, requirements.txt, dependency checker, and output formatting conventions including the mandatory disclaimer template.

**Addresses:** Stock price data fetch, graceful error handling, chart output conventions, environment detection, CSV ingestion plumbing.

**Avoids:** Adjusted/unadjusted price confusion (baked in at foundation), timezone misalignment (normalized at fetch time), yfinance API instability (adapter layer created), investment advice language (disclaimer in output template), silent data gaps (validation wrapper built at foundation).

**Research flag:** Standard patterns — skip `/gsd:research-phase`. Claude Code command format is HIGH confidence from local official docs. yfinance adapter pattern is well-documented.

### Phase 2: Market Analysis Workflows

**Rationale:** With the data foundation solid, implement the stock analysis workflows that use live yfinance data. These are lower ML complexity and prove the write-then-execute pattern before adding the higher-complexity ML workflows. This phase also validates that the intent classification in SKILL.md correctly routes plain-language requests.

**Delivers:** Working stock price charts, returns and volatility calculations, Sharpe/Sortino/Beta vs. benchmark, multi-ticker comparison with normalized price index, correlation matrix heatmap. Plain-English interpretation on all outputs.

**Addresses:** All table-stakes features except CSV ingestion (Phase 1) and ML (Phase 3). Covers the "first thing any user tries" workflows.

**Implements:** Stock analysis code template, price chart output conventions, SKILL.md intent routing for market analysis task type.

**Avoids:** `plt.show()` (use `Agg` backend and `savefig()` — baked in from Phase 1 conventions), raw tracebacks (error handler from Phase 1), survivorship bias disclaimer (output formatter from Phase 1).

**Research flag:** Standard patterns — well-documented yfinance and matplotlib workflows. No additional research needed.

### Phase 3: ML Workflows (Liquidity Predictor + Investor Classifier)

**Rationale:** Both ML workflows share the same data preprocessing pipeline and must be built after CSV ingestion and EDA (Phase 1) are proven. They are the highest-complexity features and the primary competitive differentiators. Building them last means the code generation layer is proven and the output formatting conventions are stable, reducing rework. The look-ahead bias pitfall is most dangerous here and must be a design constraint from the first line.

**Delivers:** Liquidity predictor (linear/polynomial regression pipeline matching ML 03), investor classifier (stratified classification with cross-validation matching ML 05-06), plain-English coefficient interpretation, cross-validation reporting with baseline comparison, categorical variable handling with explanation.

**Addresses:** Both high-value differentiator features. Completes the full pyfi.com curriculum scope.

**Implements:** Liquidity model code template, investor classifier code template, sklearn Pipeline with train/test split ordering enforced in template, EDA report output format.

**Avoids:** Look-ahead bias (split first, fit after — enforced in template), non-stationarity (use returns as features, not raw prices — documented in SKILL.md), ML output as investment advice (disclaimer from Phase 1 template), `GridSearchCV` hang (cap grid size in skill defaults).

**Research flag:** Needs `/gsd:research-phase` during planning. The sklearn Pipeline with `ColumnTransformer` and `set_output(transform='pandas')` has version-specific behavior in sklearn 1.4+. The TimeSeriesSplit vs. random split decision depends on whether the liquidity data is cross-sectional or time-indexed. Validate these decisions before implementation.

### Phase 4: Persona Variants

**Rationale:** Persona overlays are thin framing layers that add value only when the underlying task implementations are solid. Building them before Phases 1-3 are complete creates rework — any bug in code generation must be fixed in multiple places. The architecture research explicitly warns against this (Anti-Pattern 4). Phase 4 adds `/finance-analyst` and `/finance-pm` commands with persona-specific SKILL.md overlays.

**Delivers:** Analyst-framed outputs (sector context, regression coefficient tables, equity analysis framing), PM/trader-framed outputs (drawdown emphasis, Sortino over Sharpe, portfolio risk framing), separate command files enabling clear user intent signaling.

**Addresses:** Persona-aware phrasing (v1.x differentiator), Sortino ratio and max drawdown (PM persona), analyst-style statistical output (statsmodels OLS tables).

**Implements:** `finance-analyst/SKILL.md`, `finance-pm/SKILL.md` as overlays referencing base skill conventions, `/finance-analyst` and `/finance-pm` command files.

**Avoids:** Monolithic skill growth (persona logic stays in overlay files, not base SKILL.md), premature complexity (only added after generalist command is validated).

**Research flag:** Standard patterns — persona overlay is a thin file addition with no new library dependencies. Skip `/gsd:research-phase`.

### Phase Ordering Rationale

- **Infrastructure first** because all analysis workflows share the data-fetch adapter, output directory conventions, and error handling patterns. Correctness bugs at this layer propagate everywhere.
- **Market analysis before ML** because market analysis validates the write-then-execute pattern and intent routing before adding the complexity of sklearn pipelines and CSV ingestion.
- **Both ML workflows in one phase** because they share the same CSV ingestion, EDA, and preprocessing infrastructure. Building them together avoids duplicating that foundation.
- **Personas last** because they are framing overlays with no new backend logic. They multiply the value of working task implementations rather than adding new capabilities.

### Research Flags

Needs `/gsd:research-phase` during planning:
- **Phase 3 (ML Workflows):** sklearn Pipeline with `ColumnTransformer` and `set_output(transform='pandas')` has version-specific behavior. TimeSeriesSplit vs. random split decision is data-dependent. Cross-validation strategy for small finance datasets needs validation.

Standard patterns (skip `/gsd:research-phase`):
- **Phase 1:** Claude Code command format is HIGH confidence from local official docs. yfinance adapter wrapping pattern is well-established.
- **Phase 2:** yfinance + matplotlib stock analysis workflows are extremely well-documented in training data and community resources.
- **Phase 4:** Persona overlays are thin SKILL.md additions with no new technical complexity.

---

## Confidence Assessment

| Area | Confidence | Notes |
|------|------------|-------|
| Stack | HIGH | Claude Code architecture verified from installed local official plugin docs. Python library versions from August 2025 training data — verify with `pip install --upgrade` on first deploy. |
| Features | MEDIUM | Web search unavailable during research; competitor feature analysis (OpenBB, Bloomberg Copilot) based on training data (cutoff August 2025). pyfi.com curriculum scope taken from PROJECT.md — accurate if that doc is current. |
| Architecture | HIGH | All key architectural claims sourced from installed plugin reference files on this machine, not training data. Command format, Bash execution, skill invocation, `$ARGUMENTS`, `!`cmd`` syntax all verified from authoritative local sources. |
| Pitfalls | HIGH (finance/ML) / MEDIUM (UX) | Finance and ML pitfalls (adjusted prices, look-ahead bias, non-stationarity, survivorship bias) are extremely well-documented in literature and training data. Claude Code skill UX pitfall patterns have limited production examples — MEDIUM confidence. |

**Overall confidence:** HIGH for build decisions; MEDIUM for feature prioritization (validate with real users after v1 launch).

### Gaps to Address

- **Bash tool timeout for long Python scripts:** Research flagged uncertainty about whether the Bash tool has a timeout that would affect ML model training on large datasets (potentially 30-60 seconds). Validate with a synthetic long-running script before building the ML workflows. If there is a timeout, add progress-printing to keep the connection alive.
- **yfinance column name stability in 0.2.40+:** The `Adj Close` vs. `adj_close` naming changed between 0.1.x and 0.2.x. The adapter layer must validate column names at runtime and raise a clear error rather than assuming the schema. Verify the exact column name from a live 0.2.40 install during Phase 1 implementation.
- **pandas-datareader maintenance status:** Listed as a fallback data source if yfinance becomes unreliable. Last known stable version was 0.10.x. Verify current maintenance status before documenting as a supported alternative.
- **Cross-sectional vs. time-series structure of liquidity data:** The pitfalls research recommends `TimeSeriesSplit` for time-ordered data. The liquidity predictor workflow may use cross-sectional fund data (not time series), in which case random split is correct. This depends on the actual dataset structure and must be determined before Phase 3 implementation.

---

## Sources

### Primary (HIGH confidence)

- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/frontmatter-reference.md` — Command frontmatter field definitions
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/examples/simple-commands.md` — Command patterns with Bash execution, `!`cmd`` syntax
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/claude-code-setup/skills/claude-automation-recommender/references/skills-reference.md` — Skill file format and invocation control
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/example-plugin/skills/example-skill/SKILL.md` — Skill vs. command distinction, SKILL.md schema
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/advanced-workflows.md` — Multi-step workflow patterns, state management
- `/Users/aarjay/.claude/plugins/marketplaces/claude-plugins-official/plugins/plugin-dev/skills/command-development/references/plugin-features-reference.md` — `CLAUDE_PLUGIN_ROOT`, namespaced commands

### Secondary (MEDIUM confidence)

- Training knowledge (August 2025 cutoff) — Python library versions, yfinance 0.2.x architecture, pandas 2.x changes, scikit-learn Pipeline API
- pyfi.com curriculum scope — described in `.planning/PROJECT.md`; assumed accurate
- OpenBB Terminal v2+ feature set — open source docs at openbb.co (training data)
- Bloomberg Copilot feature announcements (Bloomberg.com, 2024-2025) (training data)
- yfinance GitHub issue tracker and changelog (training data; verify against live repo)
- sklearn Pipeline documentation — https://scikit-learn.org/stable/modules/compose.html

### Tertiary (LOW confidence)

- pandas-datareader current maintenance status — unverified; treat as potentially unmaintained
- Bash tool timeout behavior in Claude Code — flagged as unknown; needs empirical validation
- Claude Code UX pitfall patterns — limited production examples in training data as of August 2025

---

*Research completed: 2026-03-17*
*Ready for roadmap: yes*
