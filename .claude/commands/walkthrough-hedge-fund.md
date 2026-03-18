---
description: Hedge fund walkthrough -- volatility regime detection, cross-sector diversification, and pair trading analysis
allowed-tools: Bash(python3:*), Bash(pip:*), Write, Read, mcp__finance__ping, mcp__finance__validate_environment, mcp__finance__analyze_stock, mcp__finance__get_returns, mcp__finance__get_volatility, mcp__finance__get_risk_metrics, mcp__finance__compare_tickers, mcp__finance__correlation_map
model: sonnet
---

## Hedge Fund Walkthrough: Volatility Regime & Pair Trading

> *You are a quantitative portfolio manager at a multi-strategy hedge fund. Your CIO wants a volatility regime assessment and pair trading opportunities across two sectors — semiconductors (NVDA, AMD, AVGO) and mega-cap tech (AAPL, MSFT, GOOGL). You need to identify volatility dislocations, cross-sector diversification potential, and correlation-based pair candidates for the systematic trading book.*

This walkthrough takes approximately 10–15 minutes and runs automatically. Each phase produces data that a real trading desk would use for book construction and morning risk meeting preparation.

---

## Walkthrough Instructions

Execute each phase below IN ORDER. After each step:
1. Run the tool or code as instructed
2. Show the result (output, metrics, or chart path)
3. Provide a PLAIN-ENGLISH EXPLANATION framed as trading desk commentary — write as if you are presenting findings at the morning risk meeting
4. Print a separator line (`---`)
5. Proceed immediately to the next step

Do NOT ask the user for input between steps. If any step fails, explain the error in plain English and continue.

---

## Phase 1: Pre-Flight Check

### Step 1: Environment Validation

Call `validate_environment` with no arguments.

After running:
- Confirm all packages are ready
- Note: A trading desk workflow depends on real-time data feeds and quantitative libraries. yfinance provides the market data, pandas handles time-series manipulation, matplotlib generates publication-quality charts, and scikit-learn supports any quantitative screening. All must be present before the analysis begins.

---

## Phase 2: Volatility Regime Detection

> *The first question the systematic book asks: "What volatility regime are we in?" Position sizing, option premiums, and stop-loss levels all depend on where each name sits relative to its historical vol range. This phase answers that question across both sleeves.*

### Step 2: Volatility Profile — NVDA

Call `get_volatility` with:
- ticker: "NVDA"
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show annualized volatility and note the rolling 21-day chart
- Frame as trading desk: "NVDA's annualized vol of X% places it in a [high/normal/low] regime relative to a typical semiconductor name. For book construction, this vol level implies [wider/tighter] position sizing — a 1% risk budget on a high-vol name requires a smaller notional than on a low-vol name. The rolling vol chart reveals whether we are in a [compressing/expanding] regime, which drives option premium and stop placement."

---

### Step 3: Volatility Profile — AMD

Call `get_volatility` with:
- ticker: "AMD"
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show annualized volatility
- Frame as trading desk: "AMD's annualized vol of X% is [higher/lower/in line] with NVDA's vol from Step 2. When two names in the same sector diverge on volatility, it creates a vol dislocation — one name's options are cheap relative to the other's, which is a potential trade signal. Are NVDA and AMD in sync, or has one decoupled from the sector regime?"

---

### Step 4: Volatility Profile — AAPL

Call `get_volatility` with:
- ticker: "AAPL"
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show annualized volatility
- Frame as trading desk: "AAPL's annualized vol of X% represents the mega-cap tech regime baseline. Compare to the semi names: [higher/lower/similar] vol in mega-cap tech suggests a [different/same] risk regime across the two sleeves. A low-vol mega-cap tech sleeve alongside a high-vol semi sleeve creates natural diversification in realized vol, which is exactly what a multi-strategy book is designed to exploit."

---

### Step 5: Volatility Regime Summary

Do NOT call any tools. This is a pure analysis step.

Build a regime table using the data from Steps 2–4:

#### Volatility Regime Assessment — Trading Desk Summary

| Ticker | Sector | Ann. Vol | Regime | Position Sizing Implication |
|--------|--------|----------|--------|----------------------------|
| NVDA | Semiconductors | [Step 2] | [High/Normal/Low] | [Wider/Normal/Tighter] |
| AMD | Semiconductors | [Step 3] | [High/Normal/Low] | [Wider/Normal/Tighter] |
| AAPL | Mega-Cap Tech | [Step 4] | [High/Normal/Low] | [Wider/Normal/Tighter] |

After building the table:
- **Identify the highest and lowest vol names** — these set the bounds of the vol regime spectrum
- **Assess vol divergence within semis (NVDA vs AMD):** "The vol spread between NVDA and AMD of [X]pp creates a [convergence/divergence] trade setup — when two correlated names diverge on realized vol, mean reversion is the systematic expectation..."
- **Assess cross-sector vol spread (semis vs tech):** "The vol differential between the semi sleeve and mega-cap tech sleeve suggests [adding/not adding] the tech sleeve [does/does not] provide meaningful volatility diversification..."
- **Note for book construction:** The regime classification drives all subsequent position sizing decisions in this analysis.

---

## Phase 3: Cross-Sector Diversification Analysis

> *Diversification alpha comes from combining return streams that do not move together. This phase tests whether the semiconductor and mega-cap tech sleeves actually provide diversification benefit, or whether they are closet substitutes that would move together in a risk-off event.*

### Step 6: Semiconductor Sleeve — Relative Performance

Call `compare_tickers` with:
- tickers: "NVDA,AMD,AVGO"
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show the normalized performance ranking
- Frame as trading desk: "Within the semi sleeve, normalizing all three names to a $100 starting value reveals [X] as the outperformer at $[Y] and [Z] as the laggard. The within-sleeve performance dispersion of [A]pp matters for sleeve construction: do we equal-weight, or do we tilt toward the best-performing name? A high-dispersion sleeve rewards active stock selection over passive equal-weighting."

---

### Step 7: Mega-Cap Tech Sleeve — Relative Performance

Call `compare_tickers` with:
- tickers: "AAPL,MSFT,GOOGL"
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show the normalized performance ranking
- Frame as trading desk: "Within the mega-cap tech sleeve, [X] leads at $[Y] and [Z] trails. Compare the within-sleeve dispersion to the semi sleeve from Step 6: [higher/lower/similar] dispersion suggests [more/less] active selection alpha available in tech vs semis. The sleeve with higher internal dispersion rewards stock picking; the low-dispersion sleeve behaves more like an index."

---

### Step 8: Full Cross-Sector Comparison

Call `compare_tickers` with:
- tickers: "NVDA,AMD,AVGO,AAPL,MSFT,GOOGL"
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show the full 6-name normalized performance ranking
- Frame as trading desk: "Cross-sector, the full 6-name comparison reveals whether semiconductors or mega-cap tech generated superior absolute returns over the trailing 12 months. The performance dispersion across all six names of [X]pp suggests [high/limited] diversification benefit from holding both sleeves — if both sleeves moved together, dispersion would be low; if they delivered distinct return streams, dispersion would be high."

---

### Step 9: Cross-Sector Diversification Summary

Do NOT call any tools. This is a pure analysis step.

Using data from Steps 6–8, build the diversification brief:

#### Cross-Sector Diversification Assessment

**Semi Sleeve (NVDA/AMD/AVGO):**
- Best performer: [X] (+[A]%)
- Worst performer: [Y] (+[B]%)
- Within-sleeve dispersion: [C]pp

**Tech Sleeve (AAPL/MSFT/GOOGL):**
- Best performer: [X] (+[A]%)
- Worst performer: [Y] (+[B]%)
- Within-sleeve dispersion: [C]pp

**Cross-Sector Assessment:**
- Semiconductor sleeve 12-month return: [average of Step 6 names]
- Mega-cap tech sleeve 12-month return: [average of Step 7 names]
- Cross-sector return differential: [X]pp

**Portfolio Construction Implication:**
Frame as trading desk: "Adding the mega-cap tech sleeve [does/does not] provide meaningful diversification to a semiconductor-only book. A [large/small] cross-sector return differential over 12 months suggests the sleeves [have/have not] delivered distinct return streams. The PM should [equal-weight both sleeves / overweight the outperforming sleeve / underweight the correlated sleeve] based on this analysis."

---

## Phase 4: Correlation-Based Pair Identification

> *Pair trading extracts alpha from temporary dislocations in historically correlated names. The systematic setup: identify two names with high correlation (they tend to move together), wait for a divergence (a dislocation from the mean), and trade the convergence. This phase identifies the best candidates across both sleeves.*

### Step 10: Intra-Semiconductor Correlations

Call `correlation_map` with:
- tickers: "NVDA,AMD,AVGO"
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show the 3x3 correlation matrix
- Frame as trading desk: "Within the semi sleeve, the [NVDA/AMD / NVDA/AVGO / AMD/AVGO] pair at r=[X] shows the highest correlation — these names move together closely enough for a mean-reversion pair trade. The lowest correlated pair at r=[Y] is a [better diversifier / weaker pair candidate]. For intra-sector pairs, we want high correlation (r > 0.7) to ensure the relationship is stable, but not perfect correlation (r < 0.95) to ensure there is room for dislocations."

---

### Step 11: Intra-Tech Correlations

Call `correlation_map` with:
- tickers: "AAPL,MSFT,GOOGL"
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show the 3x3 correlation matrix
- Frame as trading desk: "Within the mega-cap tech sleeve, the [AAPL/MSFT / AAPL/GOOGL / MSFT/GOOGL] pair at r=[X] shows the highest correlation. Compare to the semi sleeve: [higher/lower] intra-sector correlation in tech vs semis suggests [stronger/weaker] pair trade opportunities within tech. A sleeve with uniformly high correlations (all names at r > 0.8) behaves more like an index — pair trades there offer less alpha than in a differentiated sleeve."

---

### Step 12: Full 6x6 Cross-Sector Correlation Matrix

Call `correlation_map` with:
- tickers: "NVDA,AMD,AVGO,AAPL,MSFT,GOOGL"
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show the full 6x6 correlation matrix
- Identify:
  - **Top intra-sector pair:** The highest correlated pair within either sector
  - **Top cross-sector pair:** The highest correlated pair between a semi name and a tech name — this is the primary pair trade candidate
  - **Best diversification pair:** The lowest correlated cross-sector pair — names that genuinely provide uncorrelated return streams
- Frame as trading desk: "The [X]/[Y] cross-sector pair at r=[Z] is the top pair trade candidate — high enough correlation for mean-reversion setups but representing two different sectors, which provides fundamental justification for temporary dislocations. When [X] and [Y] diverge, the systematic book fades the divergence and bets on convergence."
- **Identify the top cross-sector pair for Steps 13–14:** Pick the cross-sector pair with the highest correlation (one semi name, one tech name). Name them explicitly — e.g., "Top cross-sector pair: NVDA / MSFT (r=0.78). Running risk metrics on both."

---

### Step 13: Risk Metrics — Top Cross-Sector Pair (Name 1)

Using the top cross-sector pair identified in Step 12, call `get_risk_metrics` with:
- ticker: [the semi name from the top pair]
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show Sharpe ratio, max drawdown, and beta
- Frame as trading desk: "For the pair trade, [semi name] shows a Sharpe of [A], max drawdown of [B]%, and beta of [C]. These metrics define the risk profile of the [long/short] leg candidate — a higher Sharpe name is preferred for the long leg, while a lower Sharpe / higher drawdown name is preferred for the short leg."

---

### Step 14: Risk Metrics — Top Cross-Sector Pair (Name 2)

Call `get_risk_metrics` with:
- ticker: [the tech name from the top pair identified in Step 12]
- start: [365 days before today, in YYYY-MM-DD format]

After running:
- Show Sharpe ratio, max drawdown, and beta
- Frame as trading desk: "Comparing the pair: [semi name] (Sharpe [A], DD [B]%, beta [C]) vs [tech name] (Sharpe [D], DD [E]%, beta [F]). The risk differential of [|A-D|] Sharpe points suggests the [long/short] structure should be [long semi/short tech] or [long tech/short semi] depending on current dislocation direction. The higher-Sharpe, lower-drawdown name is the preferred long leg — it has historically delivered better risk-adjusted returns and is more likely to recover after a divergence."

---

## Phase 5: Trading Desk Summary

### Step 15: Trading Desk Brief — Volatility Regime & Pair Trading Assessment

Do NOT call any tools. This is a pure synthesis step.

Compile all findings into the morning risk meeting brief:

---

#### Trading Desk Brief — Volatility Regime & Pair Trading Assessment

**Volatility Regime (Steps 2–5):**
- Current regime: [list all three names and their vol levels from the table in Step 5]
- Regime divergence: [which names show vol dislocation relative to sector peers]
- Vol spread (highest vs lowest): [X]pp — [narrow/wide] by historical standards
- Position sizing implication: [adjust notional sizing for high-vol names — smaller position for same 1% risk budget]

**Cross-Sector Diversification (Steps 6–9):**
- Semi sleeve performance: Best [X] (+[A]%), Worst [Y] (+[B]%)
- Tech sleeve performance: Best [X] (+[A]%), Worst [Y] (+[B]%)
- Cross-sector return differential: [X]pp
- Diversification benefit: [Yes/No — the sleeves provided distinct return streams / the sleeves moved together and offer limited diversification]

**Pair Trade Candidates (Steps 10–14):**
- Top intra-sector pair: [X]/[Y] (r=[Z]) — strong mean-reversion candidate within semis/tech
- Top cross-sector pair: [A]/[B] (r=[C]) — primary pair trade candidate
- Risk differential: [semi name] Sharpe [D] vs [tech name] Sharpe [E]
- Recommended structure: [Long X / Short Y] — rationale: [the higher-Sharpe name is the long leg; the lower-Sharpe name is the short leg; the fundamental sector difference justifies the expectation of temporary dislocations]

**CIO Recommendation:**
[2–3 sentences: What is the single most actionable trade? Frame it as: "The systematic book should [initiate/avoid/monitor] the [pair] pair trade given [vol regime observation] and [correlation reading]. The biggest risk is [X] — specifically, [what market event or regime change would cause the pair relationship to break down]. The [risk/reward] setup is [favorable/neutral/unfavorable] at current levels."]

---

#### What a PM Would Do Next

This walkthrough produced the data foundation for pair trade construction. In a real multi-strategy fund, the PM would next:

1. **Size the pair trade based on vol-adjusted notional** — use the annualized vol from Phase 2 to compute the notional that achieves a 1% risk budget per leg, then net to market-neutral
2. **Set entry and exit triggers using rolling correlation z-scores** — when the spread between the two names deviates beyond 2 standard deviations of its rolling 30-day mean, enter; exit when it reverts to mean
3. **Run the same analysis on 90-day and 30-day windows for regime confirmation** — a pair trade based only on 1-year data may be trading a regime that has already ended; shorter windows confirm the relationship is still live
4. **Present at the morning risk meeting** — the vol regime table (Step 5), diversification assessment (Step 9), and pair recommendation (Step 15) are the three slides the CIO will ask for

---

### How This Maps to Traditional Trading Desk Tools

| This Walkthrough | Traditional Tool | Time Saved |
|-----------------|------------------|------------|
| get_volatility (annualized + rolling) | Bloomberg `HVG` function or manual pandas formula | Auto-computed for any ticker/period in 1 call |
| compare_tickers (normalized performance) | Bloomberg `COMP` or FactSet charting | Multi-name normalized chart in 1 call vs manual setup |
| correlation_map (6x6 matrix) | Excel CORREL matrix (36 manual formulas) | Full matrix in 1 call vs manual construction |
| get_risk_metrics (Sharpe/DD/beta) | FactSet portfolio analytics + Excel formulas | 3 metrics per ticker in 1 call |
| Volatility regime table (Step 5) | Manual Bloomberg pull + Excel formatting | Auto-generated from live data |
| Trading Desk Brief (Step 15) | PM writes manually from multiple data sources | Synthesized from all prior steps automatically |

**Total estimated time savings:** 3–5 hours of Bloomberg/FactSet data pulling, correlation matrix construction, and risk metrics calculation → ~10 minutes with the Finance AI Skill.

---

Walkthrough complete. Use `/finance-analyst` to run ad-hoc equity research queries, or `/finance` for general analysis.

> For educational/informational purposes only. Not financial advice. Past results do not guarantee future performance.
