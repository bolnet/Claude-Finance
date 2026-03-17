# Pitfalls Research

**Domain:** Finance AI Claude Code Skill (yfinance + scikit-learn + natural language interface)
**Researched:** 2026-03-17
**Confidence:** HIGH (finance data and ML pitfalls) / MEDIUM (Claude Code skill UX patterns — limited production examples in training data)

---

## Critical Pitfalls

### Pitfall 1: Adjusted vs. Unadjusted Price Confusion

**What goes wrong:**
yfinance returns split-adjusted and dividend-adjusted closing prices by default (`Close` column), but also provides `Adj Close`. Code that mixes these two columns — or presents raw `Close` as the price history without noting adjustments — produces misleading returns calculations. A 2-for-1 stock split makes it look like the stock dropped 50% overnight in unadjusted data. Users who don't know to ask will receive silently wrong numbers.

**Why it happens:**
Developers assume `Close` is already adjusted (it partially is, depending on yfinance version and the ticker). The semantics changed between yfinance 0.1.x and 0.2.x. Code written against older docs silently breaks. The skill generates code that may not pin this behavior explicitly.

**How to avoid:**
- Always use `Adj Close` explicitly for any return or performance calculation, never `Close`.
- Add a column presence check: if `Adj Close` is missing from the DataFrame, raise a clear error rather than silently falling back to `Close`.
- Include a comment in every generated code block stating which column is used and why.
- Document for the user: "Prices are split and dividend adjusted as of [date]."

**Warning signs:**
- Overnight price drops of 30–90% in historical charts that don't match news.
- Returns that look implausibly good or bad around earnings dates.
- Correlation calculations that look wrong because one series is adjusted and another isn't.

**Phase to address:**
Phase 1 (core data fetch module). Bake the correct column selection into the data-fetch helper function before any analysis workflow is built on top.

---

### Pitfall 2: Look-Ahead Bias in ML Feature Engineering

**What goes wrong:**
When building the liquidity predictor or investor classifier, features are accidentally constructed using information that would not have been available at prediction time. Classic examples: using the target variable's future value in a lag calculation, computing normalization statistics (mean, std) on the full dataset before train/test split, or using a `fillna(method='ffill')` that fills test data with values that leaked from future training rows.

**Why it happens:**
The sklearn `Pipeline` pattern is meant to prevent this, but only when `.fit()` is called exclusively on training data. If a developer fits a `StandardScaler` on `X` before splitting, the scaler has seen the test set. This is a subtle mistake that is trivially easy to make and produces optimistic, unreproducible results.

**How to avoid:**
- ALWAYS split first, then fit transformers on training data only. Order: `train_test_split` → `Pipeline.fit(X_train, y_train)` → `Pipeline.predict(X_test)`.
- Never call `.fit_transform()` on the full dataset. Call `.fit_transform(X_train)` then `.transform(X_test)`.
- For time series data, use `TimeSeriesSplit` from sklearn instead of random `train_test_split`, which preserves temporal order.
- Add a code comment in every generated pipeline stating the split order explicitly.

**Warning signs:**
- Model accuracy on test data is suspiciously close to training accuracy (gap < 1%).
- Model performs perfectly on historical data but fails on new data.
- Cross-validation scores are much better than walk-forward validation scores.

**Phase to address:**
Phase 2 (ML workflows — liquidity predictor and investor classifier). Must be a design constraint from the first line of the ML modules, not a retrofit.

---

### Pitfall 3: Non-Stationarity of Financial Time Series

**What goes wrong:**
Linear regression and classification models assume that the statistical properties of features (mean, variance, correlations) are stable over time. Stock prices and most raw financial series are non-stationary — they have trends, regime changes, and heteroscedastic volatility. A model trained on 2019–2022 data will have learned patterns from a low-rate, high-growth regime that do not hold in 2023's rate-shock environment. The skill generates models that appear to work but extrapolate dangerously.

**Why it happens:**
The course curriculum focuses on model mechanics, not time series theory. Analysts trained in cross-sectional statistics apply those tools directly to time series without transformation. The skill inherits this gap if not explicitly addressed.

**How to avoid:**
- Use returns (percent change) rather than raw prices as model inputs. Returns are closer to stationary.
- For features that represent levels (e.g., volume, market cap), log-transform before using.
- Add a stationarity check (Augmented Dickey-Fuller test from `statsmodels`) as a diagnostic step in any regression workflow. If a feature fails the test, log a warning in the output.
- Communicate to the user: "Note: this model was trained on data from [date range]. Performance in different market regimes may differ."

**Warning signs:**
- Model uses raw price as a feature (not returns, not log-returns).
- R² on training data is high (>0.9) but the feature is a lagged price level.
- Model was trained in one volatility regime (e.g., COVID period) and applied to another.

**Phase to address:**
Phase 2 (ML workflows). Include a stationarity diagnostic wrapper function that runs before any model fit and logs a warning if non-stationary features are detected.

---

### Pitfall 4: Survivorship Bias in Universe Selection

**What goes wrong:**
When the skill fetches data for "the S&P 500" or "tech stocks," it fetches data for companies that are currently in the index. Companies that went bankrupt, were acquired, or were delisted between the historical start date and today are excluded. Any analysis of that historical period is therefore biased toward winners, making returns look better and volatility look lower than they actually were.

**Why it happens:**
yfinance has no API for historical index composition. Fetching current S&P 500 constituents and pulling 10 years of history is the easy path and the one the course teaches. The bias is invisible in the output.

**How to avoid:**
- For v1, limit scope: the skill should not claim to analyze "the S&P 500 historically." It should analyze user-specified tickers.
- When a user requests a sector or index basket, add a disclaimer in the output: "Note: this analysis uses current index constituents. Companies that were removed from the index before today are excluded, which may overstate historical performance."
- Do not build a "screen the best performing stocks" workflow — that is pure survivorship bias by construction.

**Warning signs:**
- Any workflow that fetches a list of tickers from a current index and then runs historical analysis.
- Sharpe ratios or returns that look materially better than published index returns for the same period.

**Phase to address:**
Phase 1 (data fetch) and the output formatting layer. Add the disclaimer at the data-fetch level so it propagates to all analysis outputs.

---

### Pitfall 5: yfinance Rate Limiting and Silent Data Gaps

**What goes wrong:**
yfinance wraps Yahoo Finance's unofficial API, which has no rate limit documentation and has changed its throttling behavior multiple times. When rate limited, yfinance does not raise an exception by default — it returns an empty DataFrame or a partial DataFrame with NaN rows for the throttled period. Code that does not check for this produces analyses on incomplete data without any warning to the user.

**Why it happens:**
The error mode is silent. `df = yf.download("AAPL", start="2020-01-01")` returns a valid-looking DataFrame even when rows are missing. Developers test with one ticker and it works; production use with many tickers hits limits.

**How to avoid:**
- After every `yf.download()` or `ticker.history()` call, check: `if df.empty: raise DataFetchError(...)` and `if df.isnull().sum().sum() > threshold: warn(...)`.
- Add a row-count sanity check: for daily data over N trading days, expect approximately `N * 0.99` rows. If significantly fewer are returned, warn the user.
- For multi-ticker fetches, download serially with a small sleep between requests, or use `yf.download()` with `group_by='ticker'` and validate each sub-frame.
- Cache successful fetches to disk (using `joblib` or `pickle`) to avoid re-fetching during iterative analysis.

**Warning signs:**
- Empty or nearly-empty DataFrames returned for valid tickers.
- NaN clusters appearing at specific date ranges (rate limit periods).
- Analysis that runs on fewer rows than expected without warning.

**Phase to address:**
Phase 1 (data fetch module). Build the validation wrapper at the foundation layer.

---

### Pitfall 6: yfinance API Instability and Version-Breaking Changes

**What goes wrong:**
yfinance is not an official API. Yahoo Finance has changed its underlying endpoints multiple times, breaking yfinance releases with no notice. Between 0.1.x and 0.2.x, column names changed (`Adj Close` vs `adj_close`), the multi-ticker download format changed, and timezone handling changed. Code written for one version silently produces wrong results on another.

**Why it happens:**
The library is maintained by community contributors, not Yahoo. Yahoo has no obligation to maintain backward compatibility with yfinance. The skill's generated code will encode assumptions about the API that may break within months.

**How to avoid:**
- Pin yfinance version in requirements: `yfinance==0.2.x` (pin to a specific minor version tested at build time).
- Wrap all yfinance calls in a thin adapter layer (e.g., `data_fetcher.py`) so that when yfinance breaks, there is a single point of change.
- In generated code, add the version used as a comment: `# yfinance 0.2.x — column names and API shape validated against this version`.
- Monitor the yfinance GitHub releases page for breaking changes when updating.

**Warning signs:**
- `KeyError: 'Adj Close'` or `KeyError: 'adj_close'` errors after a package update.
- `AttributeError` on `Ticker` methods that previously worked.
- DataFrame shape or column structure unexpectedly different.

**Phase to address:**
Phase 1 (infrastructure setup). Create the adapter layer before writing any analysis code.

---

### Pitfall 7: Treating ML Output as Investment Advice

**What goes wrong:**
The skill produces outputs like "predicted liquidity score: HIGH" or "investor classification: aggressive growth." Without explicit framing, finance professionals may act on these outputs as if they are recommendations. In the US, providing specific investment recommendations without being a registered investment advisor (RIA) is regulated activity. An AI tool that produces outputs indistinguishable from investment advice creates legal and reputational exposure for the developers and users.

**Why it happens:**
ML output looks authoritative. Confidence scores feel like certainty. Finance professionals are trained to act on quantitative signals. The skill's UX — "describe what you want, get an answer" — reinforces the perception that the output is a decision.

**How to avoid:**
- Every output that includes a prediction, classification, or recommendation MUST include a boilerplate disclaimer: "This output is for educational and analytical purposes only. It is not investment advice and should not be the sole basis for any investment decision. Past model performance does not guarantee future results."
- Frame outputs as "the model suggests" or "based on historical patterns, the model identifies" — not "you should buy" or "this is classified as high risk."
- Include model accuracy metrics prominently alongside predictions so users understand the model's error rate.
- Do not build features that directly say "buy," "sell," or "hold."

**Warning signs:**
- Output text that uses imperative language ("buy X," "avoid Y").
- Outputs that omit model confidence or accuracy context.
- Users asking "should I act on this?" — indicates the framing is insufficiently clear.

**Phase to address:**
Phase 1 (output formatting layer). The disclaimer must be hardcoded into the skill template, not left to per-analysis generation.

---

### Pitfall 8: Date and Timezone Handling in Financial Data

**What goes wrong:**
yfinance returns timezone-aware timestamps for intraday data and timezone-naive timestamps for daily data, inconsistently across tickers and time periods. When date filtering, merging, or plotting, mixing aware and naive datetimes causes either a `TypeError` crash or silent misalignment where rows from different date series are merged on the wrong dates (e.g., one series in EST, another in UTC, producing a one-day offset in correlations).

**Why it happens:**
US market data comes as Eastern Time. International tickers come as their local market time. When merging AAPL (NYSE, ET) with HSBA.L (London, GMT), the timestamps represent different moments and a naive merge aligns wrong dates.

**How to avoid:**
- After every data fetch, normalize timestamps to UTC immediately: `df.index = df.index.tz_convert('UTC')` (for tz-aware) or `df.index = df.index.tz_localize('UTC')` (for tz-naive daily data).
- Use only date (not datetime) as the index for daily data to avoid timezone issues entirely: `df.index = df.index.date`.
- When merging multi-ticker DataFrames, use `pd.concat()` with `join='inner'` to align on common dates, then validate that the resulting row count is reasonable.
- Test with at least one non-US ticker (e.g., a London or Tokyo listing) in the test suite.

**Warning signs:**
- `TypeError: Cannot compare tz-naive and tz-aware datetime-like objects` errors.
- Correlation matrices where a pair of related stocks shows near-zero correlation (likely a one-day misalignment).
- Charts where two series that should track each other are offset by one period.

**Phase to address:**
Phase 1 (data fetch module). Apply timezone normalization as a mandatory post-processing step in the data fetcher, not in individual analysis workflows.

---

## Technical Debt Patterns

| Shortcut | Immediate Benefit | Long-term Cost | When Acceptable |
|----------|-------------------|----------------|-----------------|
| Calling `yf.download()` directly in analysis functions | Simpler code, faster to write | Breaks everywhere when yfinance API changes; no caching | Never — always go through an adapter |
| Hardcoding date ranges (e.g., `start="2020-01-01"`) | Deterministic results during development | Analyses become stale; no way to run "last 3 years" dynamically | Never in production skill code |
| Skipping `train_test_split` for quick model demos | Faster iteration | Models appear to work but are overfit; cannot assess generalization | Never for any output shown to users |
| Using `Close` instead of `Adj Close` | One less column to remember | Returns are wrong around splits/dividends | Never — the cost is correctness |
| Generating code without error handling | Cleaner generated code | Any data gap or rate limit crashes the entire workflow | Never in user-facing skill output |
| Omitting disclaimer from ML output | Less visual clutter | Legal and reputational liability for investment advice | Never — disclaimer is mandatory |
| Fitting sklearn Pipeline on full dataset before split | Simpler code | Look-ahead bias invalidates all model results | Never |
| No version pinning for yfinance | Easier dependency management | Silent breakage on next Yahoo Finance endpoint change | Only in local development, never in skill distribution |

---

## Integration Gotchas

| Integration | Common Mistake | Correct Approach |
|-------------|----------------|------------------|
| yfinance `download()` multi-ticker | Column MultiIndex becomes ambiguous when only 1 ticker is passed | Always check `if isinstance(df.columns, pd.MultiIndex)` and handle both cases |
| yfinance `Ticker.history()` | `period="max"` silently truncates at Yahoo's data limit (varies by ticker) | Use explicit `start=` date and validate row count against expected trading days |
| yfinance delisted tickers | Returns empty DataFrame with no error | Check `df.empty` immediately after fetch; surface "ticker not found or delisted" to user |
| pandas `read_csv()` for user CSVs | Date columns loaded as strings, not datetime | Always specify `parse_dates=['date_column']` or parse explicitly after load |
| sklearn `Pipeline` with `ColumnTransformer` | Feature names are lost; output is a numpy array | Use `set_output(transform='pandas')` (sklearn >= 1.2) to preserve column names |
| matplotlib in Claude Code terminal | `plt.show()` blocks execution and may not render | Use `plt.savefig('output.png')` and tell the user the file path |
| pandas `merge()` on date index | Timezone-aware vs naive mismatch causes all rows to fail to merge (empty result) | Normalize timezone before merge (see Pitfall 8) |

---

## Performance Traps

| Trap | Symptoms | Prevention | When It Breaks |
|------|----------|------------|----------------|
| Fetching full history for every analysis run | 10–30 second load times per request | Cache fetched data to disk with a TTL; re-fetch only if data is stale | Every run — immediately |
| Downloading 50+ tickers in a loop synchronously | Minutes-long execution, rate limit hits | Batch with `yf.download(tickers_list)` (vectorized); add progress output | >10 tickers |
| Running `GridSearchCV` with large hyperparameter grids | Skill hangs for minutes with no output | Limit grid size in skill defaults; use `RandomizedSearchCV` with `n_iter` cap | Any grid with >100 combinations |
| Generating charts without clearing `plt.figure()` | Memory accumulates; later charts contain artifacts from earlier ones | Call `plt.clf()` and `plt.close()` after every `savefig()` | After 3–5 charts in one session |
| Loading user CSV with `pd.read_csv()` and no dtype specification | Mixed-type columns inferred incorrectly; processing slows on large files | Specify `dtype` for known columns; use `chunksize` for files >10MB | Files >50k rows |

---

## Security Mistakes

| Mistake | Risk | Prevention |
|---------|------|------------|
| Executing user-provided CSV column names directly in generated code without sanitization | Code injection if a column is named `__import__('os').system('rm -rf /')` | Sanitize all user-supplied identifiers before interpolating into generated code strings; validate column names against `[A-Za-z0-9_\- ]+` pattern |
| Logging user-provided ticker symbols or file paths to disk without sanitization | Path traversal or log injection | Sanitize ticker input (uppercase, strip whitespace, validate against `[A-Z0-9\.\-]{1,10}`); never interpolate raw user input into file paths |
| Hardcoding API keys for premium data sources in skill templates | Key leakage if the skill file is shared | Use environment variables exclusively; add a check at skill startup that rejects hardcoded fallbacks |
| Generated code that writes to arbitrary file paths from user input | User could specify `~/Documents/sensitive_file.csv` as output path | Sandbox output paths to a designated working directory; reject paths containing `..` or starting with `/` |

---

## UX Pitfalls

| Pitfall | User Impact | Better Approach |
|---------|-------------|-----------------|
| Outputting a raw Python traceback when data fetch fails | Finance professional sees `ConnectionError: ('Connection aborted.', RemoteDisconnected(...))` — meaningless and alarming | Catch all exceptions in the skill wrapper; output "Could not retrieve data for AAPL. Yahoo Finance may be temporarily unavailable. Try again in a few minutes." |
| Displaying a dense DataFrame printout (100 rows, 20 columns) as the primary output | User is overwhelmed; cannot extract insight | Lead with a 3–5 sentence plain-English summary; put the full DataFrame in a collapsed section or save to CSV |
| Outputting only code with no interpretation | Non-technical user gets Python code they cannot read | Every analysis output must include a "What this means" section in plain English before any code or tables |
| Long-running tasks with no progress feedback | User sees nothing for 30+ seconds; assumes the skill crashed | Print progress markers (`Fetching data for AAPL...`, `Training model...`, `Done.`) at each major step |
| Asking the user to specify parameters a finance expert shouldn't need to know | "What look-back window and confidence interval should I use?" alienates non-technical users | Apply sensible defaults silently (252-day rolling window, 95% confidence interval); only expose these as optional overrides |
| Generating charts saved to a path the user cannot find | User never sees the chart | Always print the full absolute path to saved files: "Chart saved to: /path/to/chart.png" |
| Vague ML output with no accuracy context | User doesn't know if 72% accuracy is good or useless for their use case | Always show: model accuracy, benchmark (e.g., majority class baseline), and a one-line interpretation ("This model correctly classifies investors 72% of the time, compared to 53% for a random baseline") |
| Skill that only works for exact phrasings | Non-technical users rephrase naturally and get errors or wrong analysis | Design prompt parsing to be liberal: accept "show me Apple stock" and "get AAPL data" as equivalent intents |

---

## "Looks Done But Isn't" Checklist

- [ ] **Data fetch:** Appears to return data — verify `Adj Close` is being used, not `Close`, for any return calculation.
- [ ] **ML model:** Model trains and predicts — verify `train_test_split` happened BEFORE any preprocessing step.
- [ ] **Backtest / historical analysis:** Shows results — verify disclaimer about survivorship bias is present in output.
- [ ] **Multi-ticker analysis:** Correlation matrix renders — verify timezone normalization was applied before merge.
- [ ] **Classification model:** Accuracy metric shown — verify it is compared to majority-class baseline, not presented in isolation.
- [ ] **Output formatting:** Analysis text displayed — verify every output includes the investment advice disclaimer.
- [ ] **User CSV ingestion:** File loaded successfully — verify date columns were parsed as datetime, not strings.
- [ ] **Chart generation:** `plt.savefig()` called — verify `plt.clf()` and `plt.close()` are called immediately after.
- [ ] **Error handling:** Skill runs in the happy path — verify it handles empty DataFrames, rate limit errors, and delisted tickers gracefully.
- [ ] **Version pinning:** Works in development — verify `yfinance` version is pinned in `requirements.txt` or equivalent.

---

## Recovery Strategies

| Pitfall | Recovery Cost | Recovery Steps |
|---------|---------------|----------------|
| Adjusted vs. unadjusted price confusion discovered in production | MEDIUM | Identify all code paths using `Close`; swap to `Adj Close`; regenerate any saved analyses; notify users that historical results may have changed |
| Look-ahead bias discovered in trained model | HIGH | Model results are invalid; must re-split data, re-train from scratch, re-validate all reported metrics; any published accuracy numbers must be retracted |
| yfinance API breaking change | MEDIUM | All calls go through the adapter layer (if built correctly); update adapter only; re-test against new API shape |
| Legal complaint about investment advice language | HIGH | Immediate: add/strengthen disclaimers; review all output templates; consider legal review before re-publishing; potentially remove specific recommendation language entirely |
| Survivorship bias baked into analysis outputs | MEDIUM | Add disclaimer to all historical basket analysis; document the limitation prominently; cannot fully recover without historical index composition data |
| User CSV date parsing failure discovered late | LOW | Fix the `read_csv()` call in the CSV ingestion module; test with edge-case date formats |
| Timezone misalignment discovered after user reports wrong correlations | MEDIUM | Add normalization to the data fetcher; all previously generated multi-ticker analyses that span markets are suspect and should be regenerated |

---

## Pitfall-to-Phase Mapping

| Pitfall | Prevention Phase | Verification |
|---------|------------------|--------------|
| Adjusted vs. unadjusted prices | Phase 1 — data fetch foundation | Unit test: fetch AAPL around a known split date; verify returns do not show artificial spike |
| Look-ahead bias in ML | Phase 2 — ML workflow implementation | Code review: confirm `train_test_split` precedes all `fit()` calls in every pipeline |
| Non-stationarity of financial time series | Phase 2 — ML workflow implementation | Add ADF test diagnostic; review features list for raw price levels |
| Survivorship bias | Phase 1 — output formatting layer | Verify disclaimer appears in all basket/index analysis outputs |
| yfinance rate limiting / silent data gaps | Phase 1 — data fetch foundation | Integration test: mock empty DataFrame response; verify user-facing error message appears |
| yfinance API instability | Phase 1 — infrastructure setup | Create adapter module; pin version in requirements; confirm single point of change |
| ML output as investment advice | Phase 1 — output template design | Verify disclaimer appears in every output; search generated text for imperative language |
| Date/timezone handling | Phase 1 — data fetch foundation | Test with one US ticker and one non-US ticker merged; verify row alignment |
| Prompt ambiguity for non-technical users | Phase 1 — skill command design | User test with a non-Python finance professional; measure how often they get wrong or empty output |
| Chart file path opacity | Phase 1 — output formatting layer | Verify every `savefig()` call is followed by a print of the absolute path |
| No progress feedback on long tasks | Phase 1 — skill execution layer | Time a 50-ticker fetch; verify progress markers appear before timeout |
| Raw tracebacks exposed to users | Phase 1 — error handling layer | Force a network error in test; verify only the friendly message appears |

---

## Sources

- yfinance GitHub issue tracker and changelog (versions 0.1.x → 0.2.x breaking changes): https://github.com/ranaroussi/yfinance
- "Advances in Financial Machine Learning" (Lopez de Prado, 2018) — canonical reference for look-ahead bias, purging, and embargo methodology in finance ML
- SEC guidance on investment adviser definition (Advisers Act of 1940, Section 202(a)(11)) — relevant to "investment advice" language
- sklearn Pipeline documentation on fit/transform ordering: https://scikit-learn.org/stable/modules/compose.html
- pandas timezone documentation: https://pandas.pydata.org/docs/user_guide/timeseries.html#time-zone-handling
- "Python for Finance" (Yves Hilpisch, O'Reilly) — chapters on financial time series non-stationarity
- Survivorship bias in backtesting: documented extensively in academic literature (e.g., Brown, Goetzmann, Ibbotson 1992); practical implications described in Hilpisch and Prado
- Claude Code slash command skill design: based on Claude Code documentation and skill architecture patterns (training data, MEDIUM confidence — limited production examples publicly available as of August 2025)

---

*Pitfalls research for: Finance AI Claude Code Skill (yfinance + scikit-learn)*
*Researched: 2026-03-17*
