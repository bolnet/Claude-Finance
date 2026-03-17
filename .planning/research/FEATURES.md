# Feature Research

**Domain:** Finance AI skill for Claude Code — natural language to Python finance workflows
**Researched:** 2026-03-17
**Confidence:** MEDIUM (web research unavailable; based on training knowledge of finance workflows, yfinance, OpenBB, Bloomberg Copilot, and pyfi.com curriculum scope; training cutoff August 2025)

---

## What Finance Professionals Actually Do Today

Before mapping features, it helps to understand the real daily workflows this skill replaces.

### Financial Analyst (buy-side / sell-side)
- Pull price history from Bloomberg/Refinitiv → paste into Excel → chart
- Build DCF models in Excel (manual formula chains, fragile)
- Run correlation analysis across a watchlist
- Compute returns (daily, weekly, monthly), rolling volatility, drawdowns
- Screen stocks by ratio (P/E, P/B, EV/EBITDA) using VLOOKUP / INDEX-MATCH
- Export findings to PowerPoint for client decks

### Portfolio Manager / Trader
- Track portfolio performance vs benchmark (Excel or internal system)
- Compute risk metrics: beta, Sharpe ratio, Sortino ratio, max drawdown
- Monitor sector/asset class exposure
- Run "what if" scenario analysis (manually adjusting weights in Excel)
- Monitor liquidity of holdings (bid/ask spread, volume relative to position)

### Risk Analyst
- Model credit or liquidity risk with regression
- Classify clients by risk profile
- Build and validate predictive models (historically done in R or SAS)
- Backtest risk thresholds

### Common Excel Patterns Being Replaced
- VLOOKUP / INDEX-MATCH for data joins → Pandas merge/join
- Pivot tables for aggregation → Pandas groupby
- Manual charting → matplotlib/seaborn
- Goal Seek / Solver → scipy optimize or scikit-learn
- VBA macros for automation → Python scripts
- Manual data cleaning (duplicates, blanks, outliers) → Pandas + sklearn preprocessing

---

## Feature Landscape

### Table Stakes (Users Expect These)

Features finance pros assume any Python/AI finance tool has. Missing these means the skill feels like a toy.

| Feature | Why Expected | Complexity | Notes |
|---------|--------------|------------|-------|
| Stock price data fetch by ticker | First thing any finance user tries — "give me AAPL data" | LOW | yfinance; must handle bad tickers gracefully |
| Price charts (line, candlestick) | Visual output is non-negotiable; analysts think in charts | LOW-MEDIUM | matplotlib/mplfinance; seaborn for return distributions |
| Returns calculation (daily, cumulative) | Core of any performance analysis | LOW | Pandas pct_change; log returns optional |
| Volatility (rolling std dev, annualized) | Risk metric every analyst knows | LOW | Rolling window; annualization factor must be correct |
| Multi-ticker comparison | Analysts never look at one stock in isolation | MEDIUM | Normalized price index (rebased to 100) |
| Correlation matrix across tickers | Standard portfolio construction tool | MEDIUM | Pandas corr(); heatmap output |
| CSV/Excel file ingestion | Users have their own data — client data, fund holdings | MEDIUM | Must handle messy headers, mixed types, encoding issues |
| Plain-English output interpretation | The whole point — translate numbers into meaning | MEDIUM | LLM generates narrative summary after computation |
| Error messages that explain what went wrong | Finance pros are not programmers; cryptic errors kill adoption | LOW | Catch bad tickers, empty data, wrong column names explicitly |
| Reproducible outputs | Results must be consistent; finance has audit requirements | LOW | Deterministic random seeds, dated outputs |

### Differentiators (Competitive Advantage)

Features that go beyond what a finance pro could get by Googling "Python yfinance tutorial." These are where the skill earns its place.

| Feature | Value Proposition | Complexity | Notes |
|---------|-------------------|------------|-------|
| Liquidity predictor (regression ML) | Predicts liquidity metrics for holdings — directly addresses a real risk desk need | HIGH | Matches ML 03 curriculum; linear/polynomial regression pipeline; interprets coefficients in plain English |
| Investor/client classifier (classification ML) | Segments clients by risk profile automatically — replaces manual profiling spreadsheets | HIGH | Matches ML 05-06; stratified sampling, cross-validation, hyperparameter tuning; outputs segment labels + feature importances |
| Sharpe / Sortino / Beta — one command | Requires just "analyze AAPL risk vs SPY" — no formula knowledge needed | MEDIUM | Benchmark comparison built in; annualization handled automatically |
| Outlier detection in user data | Finance data is dirty — bad trades, data vendor errors; automatic flagging saves hours | MEDIUM | IQR + Z-score detection; visual flagging on chart |
| Categorical variable handling for ML | Turns fund style / sector / rating strings into model-ready features automatically | MEDIUM | Matches ML 05 (dummy variables); explained to user |
| Cross-validation reporting | Finance analysts without ML background don't know if their model is overfit; skill explains it | MEDIUM | K-fold CV; reports mean and std of score; interprets result |
| Persona-aware phrasing | Analyst gets "revenue quality" framing; PM gets "drawdown risk" framing for same data | MEDIUM | System prompt persona selection; v1 generalist, v2 splits |
| Natural language metric requests | "Show me the Sharpe ratio for my portfolio over the last year" — no syntax to learn | LOW-MEDIUM | Prompt parsing; the skill's core interface |
| Interpret model coefficients in plain English | "Volume has the strongest positive effect on liquidity" — not just a table of numbers | MEDIUM | Template-based narration from model output |
| Data exploration summary (EDA) | Automatic: shape, dtypes, missing values, distributions — matches ML 01 curriculum | MEDIUM | Single command produces full EDA report |

### Anti-Features (Things to Deliberately NOT Build in v1)

These seem compelling but would sink v1 scope, create maintenance burden, or mislead users.

| Feature | Why Requested | Why Problematic | Alternative |
|---------|---------------|-----------------|-------------|
| Real-time streaming / live prices | "I want to see prices update live" | yfinance does not support streaming; implementing websockets adds a full infrastructure layer; delays v1 by months | Document clearly: skill fetches end-of-day or latest snapshot; real-time is v2+ with different data stack |
| Bloomberg / Refinitiv / Alpha Vantage API integration | Professional data is more accurate | API keys, rate limits, authentication, paid tiers — each integration is its own project; breaks course scope | yfinance for v1; design data layer so swapping in providers is clean later |
| Trading execution / order submission | "Can it place a trade?" | Brokerage API integrations, regulatory implications, error modes with real money; entirely out of course scope | Document as explicitly out of scope; never hint at it |
| Backtesting framework (vectorbt, Backtrader) | Traders want to backtest strategies | A proper backtester is a separate product; slippage, transaction cost modeling, look-ahead bias are subtle and dangerous | Defer to Phase 2; for v1, performance analysis is historical analysis only, not strategy backtesting |
| DCF / fundamental valuation models | Analysts build these in Excel | Requires company financials data (not yfinance price data); formula assumptions vary by sector; high correctness risk | Out of scope for v1; requires separate data source and domain-expert formula validation |
| Web UI / dashboard (Streamlit, Dash) | "I want a nice interface" | Entirely different product surface; Claude Code skill runs in terminal; web UI is a separate deployment | Skill outputs clean charts as files and tables as text; dashboard is a v3+ consideration |
| Portfolio optimization (mean-variance, Black-Litterman) | PM wants optimal weights | Optimization is mathematically brittle (covariance estimation, constraints); requires user to understand assumptions or gets dangerous outputs | Correlation analysis and risk metrics are safe; optimization deferred to Phase 2 with explicit caveats |
| Narrative report auto-generation to PDF/Word | "Send me a report" | Formatting libraries (reportlab, python-docx) add complexity; output format varies by firm | Skill outputs markdown summaries; user formats them; v2 could add export |
| Natural language querying of financial databases (Text-to-SQL) | Seems like AI feature | Not in scope — the skill runs analysis on fetched/uploaded data, not a queryable database; adds DB infrastructure | The natural language interface is for analysis requests, not database queries |

---

## Feature Dependencies

```
[Stock price data fetch]
    └──required by──> [Returns calculation]
                          └──required by──> [Volatility metrics]
                          └──required by──> [Sharpe / Sortino / Beta]
                          └──required by──> [Multi-ticker comparison]
                          └──required by──> [Correlation matrix]

[CSV/Excel file ingestion]
    └──required by──> [Data exploration summary (EDA)]
                          └──required by──> [Outlier detection]
                          └──required by──> [Categorical variable handling]
                                                └──required by──> [Investor classifier (ML)]
    └──required by──> [Liquidity predictor (ML)]

[Plain-English output interpretation]
    └──enhances──> [All analysis features]

[Liquidity predictor (ML)]
    └──requires──> [CSV ingestion + EDA + outlier handling]

[Investor classifier (ML)]
    └──requires──> [CSV ingestion + EDA + categorical handling + cross-validation]

[Persona-aware phrasing]
    └──enhances──> [Plain-English interpretation]
    └──conflicts──> [Single monolithic skill prompt] (persona requires prompt switching)
```

### Dependency Notes

- **Stock data fetch is the root dependency** for all market analysis features; it must be solid before any other market analysis feature is added.
- **CSV ingestion is the root dependency** for both ML workflows (liquidity predictor, investor classifier); messy real-world data makes this non-trivial.
- **EDA precedes ML**: The ML 01 data cleaning workflow must run before any model is fitted. This is both a curriculum dependency and a correctness dependency.
- **Plain-English interpretation is a cross-cutting concern**: it enhances every feature but is not a blocker for any individual feature. Add interpretation on top of working numeric outputs.
- **Persona switching conflicts with a single monolithic prompt**: the skill architecture must support swapping the system persona context cleanly.

---

## MVP Definition

### Launch With (v1)

The minimum that makes the skill genuinely useful and demonstrates the core value proposition.

- [ ] Natural language `/finance` slash command entry point — the interface everything else hangs on
- [ ] Stock price data fetch via yfinance (single ticker and multi-ticker) — first thing any user tries
- [ ] Price chart output (line chart, return distribution histogram) — visual is non-negotiable
- [ ] Returns calculation (daily, cumulative) and annualized volatility — table stakes metrics
- [ ] Sharpe ratio and basic risk metrics vs benchmark — single most-asked portfolio question
- [ ] CSV/Excel file ingestion with format tolerance — required for both ML workflows
- [ ] Data exploration summary (EDA) matching ML 01 curriculum — data quality gate before ML
- [ ] Outlier detection and flagging — data quality, directly taught in ML 01
- [ ] Plain-English interpretation narrative on every output — the actual differentiator vs raw Python
- [ ] Liquidity predictor (regression ML, ML 03) — first ML workflow; real-world risk desk problem
- [ ] Investor classifier (classification ML, ML 05-06) — second ML workflow; real-world client profiling
- [ ] Graceful error handling with finance-specific messages (bad ticker, empty data, wrong columns)

### Add After Validation (v1.x)

Add once the core workflows are validated with real users.

- [ ] Correlation matrix heatmap across portfolio tickers — trigger: users ask "how correlated is my portfolio?"
- [ ] Sortino ratio and max drawdown — trigger: PM persona users ask for downside-specific risk
- [ ] Categorical variable encoding explanation — trigger: users confused why dummy variables appear
- [ ] Cross-validation score interpretation — trigger: users ask "is my model good?"
- [ ] Persona-specific system prompts (Analyst vs PM/Trader) — trigger: enough users to validate distinct needs

### Future Consideration (v2+)

Defer until product-market fit is confirmed.

- [ ] Algo trading backtesting — not in current course notebooks; significant complexity and correctness risk
- [ ] Bloomberg / Alpha Vantage data sources — authentication + paid tier complexity
- [ ] Portfolio optimization (mean-variance) — mathematically brittle; requires user education
- [ ] Fundamental data (P/E, earnings, balance sheet) — requires different data source (yfinance fundamentals are incomplete)
- [ ] PDF/Word report export — formatting complexity without core value change
- [ ] Web dashboard (Streamlit/Dash) — separate product surface

---

## Feature Prioritization Matrix

| Feature | User Value | Implementation Cost | Priority |
|---------|------------|---------------------|----------|
| Stock price data fetch (yfinance) | HIGH | LOW | P1 |
| Price charts (line, return distribution) | HIGH | LOW | P1 |
| Returns + annualized volatility | HIGH | LOW | P1 |
| Plain-English output interpretation | HIGH | LOW | P1 |
| CSV/Excel file ingestion | HIGH | MEDIUM | P1 |
| Data exploration summary (EDA) | HIGH | MEDIUM | P1 |
| Outlier detection | MEDIUM | LOW | P1 |
| Sharpe / Sortino / Beta vs benchmark | HIGH | MEDIUM | P1 |
| Liquidity predictor (regression ML) | HIGH | HIGH | P1 |
| Investor classifier (classification ML) | HIGH | HIGH | P1 |
| Graceful error handling + finance messages | HIGH | LOW | P1 |
| Multi-ticker comparison (normalized) | HIGH | MEDIUM | P2 |
| Correlation matrix heatmap | MEDIUM | LOW | P2 |
| Categorical variable handling + explanation | MEDIUM | MEDIUM | P2 |
| Cross-validation score interpretation | MEDIUM | LOW | P2 |
| Persona-aware phrasing (Analyst vs PM) | MEDIUM | MEDIUM | P2 |
| Max drawdown | MEDIUM | LOW | P2 |
| Portfolio optimization | HIGH | HIGH | P3 |
| Backtesting framework | HIGH | HIGH | P3 |
| Fundamental valuation (DCF) | MEDIUM | HIGH | P3 |
| Bloomberg/Refinitiv integration | MEDIUM | HIGH | P3 |
| Export to PDF/Word | LOW | MEDIUM | P3 |

**Priority key:**
- P1: Must have for launch (v1)
- P2: Should have, add when v1 core is stable
- P3: Nice to have, defer to v2+

---

## Competitor Feature Analysis

Confidence: MEDIUM (based on training knowledge of OpenBB, Bloomberg Copilot, FinChat as of August 2025)

| Feature | OpenBB Terminal | Bloomberg Copilot | FinChat.io | This Skill |
|---------|----------------|-------------------|------------|-----------|
| Natural language interface | Yes (CLI + AI) | Yes (AI assistant) | Yes (chat) | Yes (slash command) |
| Stock price data | Many providers | Bloomberg data | Many providers | yfinance only (v1) |
| Portfolio analysis | Yes | Yes (full) | Partial | Core (Sharpe, vol, beta) |
| ML modeling | No | No | No | Yes (regression + classification) |
| CSV/user data upload | Limited | No | No | Yes (core feature) |
| Plain-English interpretation | Partial | Yes | Yes | Yes (every output) |
| Code transparency | Open source, Python | No | No | Yes (generated code visible) |
| Runs in Claude Code | No | No | No | Yes (native) |
| Cost | Free/paid tiers | Bloomberg Terminal (~$24k/yr) | Paid subscription | Free (Claude subscription) |
| Data freshness | Real-time capable | Real-time | Delayed | EOD / historical (v1) |

**Key gap this skill fills:** No existing tool combines natural language interface + ML modeling on user-provided CSV data + code transparency + runs in Claude Code + course-curriculum-aligned pedagogy. OpenBB is closest but requires Python/CLI knowledge to use effectively. Bloomberg Copilot requires a Bloomberg Terminal subscription.

---

## Finance Workflow Specifics: Excel to Skill Mapping

This maps specific Excel workflows to skill features, which is the adoption bridge.

| Excel Workflow | Excel Method | Skill Command Pattern | Complexity |
|---------------|--------------|----------------------|------------|
| Pull stock history | Bloomberg add-in / manual download | "get 2 years of AAPL prices" | LOW |
| Chart price over time | Insert > Chart | "chart AAPL vs SPY last year" | LOW |
| Calculate daily returns | Manual formula =(B2-B1)/B1 | "calculate returns for AAPL" | LOW |
| Rolling 30-day volatility | STDEV on rolling range | "show 30-day rolling volatility for AAPL" | LOW |
| Correlation between stocks | CORREL function | "correlation matrix for AAPL MSFT GOOGL" | LOW |
| Sharpe ratio calculation | Manual (return - rf) / std | "Sharpe ratio for portfolio vs SPY" | MEDIUM |
| Clean dirty CSV data | Manual delete, filter | "clean this investor data, flag outliers" | MEDIUM |
| Pivot table aggregation | Insert > Pivot Table | "summarize investor data by risk category" | MEDIUM |
| Regression model | LINEST or Data Analysis Toolpak | "predict liquidity from my data" | HIGH |
| Client segmentation | Manual grouping or k-means plugin | "classify investors by profile" | HIGH |
| VLOOKUP / data join | VLOOKUP / INDEX-MATCH | Part of EDA / merge on ingestion | MEDIUM |

---

## Sources

- Project context: `.planning/PROJECT.md` (pyfi.com curriculum scope, persona definition)
- Training knowledge: OpenBB Terminal v2+ feature set (open source, docs available at openbb.co)
- Training knowledge: Bloomberg Copilot feature announcements (Bloomberg.com, 2024-2025)
- Training knowledge: FinChat.io feature set (finchat.io, 2024-2025)
- Training knowledge: yfinance library capabilities (Yahoo Finance API wrapper, PyPI)
- Training knowledge: Common finance analyst workflows (CFA curriculum, sell-side analyst role descriptions)
- Training knowledge: pyfi.com course curriculum as described in PROJECT.md
- Confidence note: Web search unavailable during this research session; all competitor and workflow claims based on training data (cutoff August 2025); verify competitor features before using in marketing claims

---
*Feature research for: Finance AI Skill for Claude Code (pyfi.com curriculum-based)*
*Researched: 2026-03-17*
