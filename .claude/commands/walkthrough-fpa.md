---
description: "FP&A walkthrough -- ERP data profiling, target column identification, and budget forecasting prep using liquidity predictor"
allowed-tools: Bash(python3:*), Bash(pip:*), Write, Read, mcp__finance__ping, mcp__finance__validate_environment, mcp__finance__ingest_csv, mcp__finance__liquidity_predictor, mcp__finance__predict_liquidity
model: sonnet
---

## FP&A Walkthrough: ERP Data Profiling & Budget Forecasting

> *You are a Senior FP&A Analyst at a mid-market company preparing quarterly budget variance analysis. Finance has exported a client portfolio dataset from the ERP system. Before building the forecast model, you need to profile the data quality, identify which columns drive liquidity risk (your forecasting target), clean up the ERP export anomalies, and run a regression model to generate budget projections. The scenario uses the bundled `demo/sample_portfolio.csv`.*

This walkthrough takes approximately 10–15 minutes and runs automatically. Each phase produces data that a real FP&A team would present to the VP of Finance or use directly in the quarterly budget model.

---

## Walkthrough Instructions

Execute each phase below IN ORDER. After each step:
1. Run the tool or code as instructed
2. Show the result (output, metrics, or data summary)
3. Provide a PLAIN-ENGLISH EXPLANATION framed as FP&A commentary — write as if you are preparing notes for the VP of Finance or drafting a data quality memo
4. Print a separator line (`---`)
5. Proceed immediately to the next step

Do NOT ask the user for input between steps. If any step fails, explain the error in plain English and continue.

---

## Phase 1: Pre-Flight Check

### Step 1: Environment Validation

Call `validate_environment` with no arguments.

After running:
- Confirm all packages are ready
- Note: An FP&A forecasting workflow depends on pandas for data manipulation, scikit-learn for regression modeling, and matplotlib for variance visualization. All must be present before the ERP data analysis begins. A missing library at this stage is equivalent to opening a Hyperion session and finding the add-in is not installed — the work stops before it starts.

---

## Phase 2: ERP Data Profiling

> *Before building any forecast, the FP&A team profiles the ERP export to understand data completeness, column types, and distribution. This is the data audit step that prevents garbage-in-garbage-out forecasting. Every experienced FP&A analyst has been burned by a model that produced clean-looking numbers from dirty ERP data — this phase prevents that.*

### Step 2: Full Dataset Profile — ERP Export Audit

Call `ingest_csv` with:
- csv_path: "demo/sample_portfolio.csv"
- (no target_column argument)

After running:
- Show the column listing, row count, data types, and basic summary statistics
- Interpret the ERP export in FP&A terms:
  - **Numeric columns** (credit_score, debt_ratio, age, income, risk_tolerance, liquidity_risk): These are the quantitative inputs and targets that the forecast model will use. In ERP terms, these are the financial and client-profile fields that flow from CRM and loan origination systems.
  - **Categorical columns** (region, product_preference, segment): These are the dimensional attributes — the same categories used to slice budget variance reports by region or product line.
  - **Row count:** Note how many client records are in the ERP export. A dataset this size [confirm count] is [sufficient/borderline/insufficient] for regression modeling — budget models typically require at least 50 data points to produce reliable coefficients.
  - **Missing values:** Flag any columns with null values. In an ERP export, missing values usually indicate incomplete data entry, system migration gaps, or fields that were optional in the source system. These must be resolved before the forecast model is submitted.
- Note: "This initial profile is what the FP&A team presents to the data owner (usually IT or the ERP system administrator) to confirm the export is complete and correctly formatted before any modeling begins."

---

### Step 3: Target Column Profile — Liquidity Risk Distribution

Call `ingest_csv` with:
- csv_path: "demo/sample_portfolio.csv"
- target_column: "liquidity_risk"

After running:
- Show the target column distribution, min, max, mean, and standard deviation
- Interpret in FP&A terms:
  - "Now that we have identified liquidity_risk as the forecasting target, we re-profile with the target column specified. This isolates the dependent variable and shows its distribution — essential for understanding whether the target is balanced or skewed, which affects model selection and forecast confidence intervals."
  - "Liquidity risk scores range from [min] to [max], with a mean of approximately [mean]. This tells us the portfolio's overall risk posture for budget planning."
  - **Skew assessment:** If the distribution is right-skewed (long tail of high-risk clients), the budget model must account for tail scenarios. If it is roughly symmetric, a linear regression will fit well and the forecast intervals will be symmetric around the mean.
  - **Budget planning implication:** "A mean liquidity risk of [mean] with standard deviation of [std] means the FP&A team should budget for a base case of [mean], a stress case of approximately [mean + 1.5*std], and a best case of approximately [mean - 1.5*std] — this is the standard three-scenario framework for reserve requirement setting."

---

### Step 4: ERP Data Quality Assessment

Do NOT call any tools. This is a pure analysis step.

Compile the findings from Steps 2 and 3 into a formal data quality report card. Frame this as the memo the FP&A Analyst sends to the VP of Finance before proceeding to the forecast model.

---

#### ERP Export Data Quality Report — Client Portfolio Dataset

**Export Summary:**
- Source file: demo/sample_portfolio.csv
- Total records: [from Step 2]
- Total columns: [from Step 2]
- Export date: [today's date]

**Column Inventory:**

| Column | Type | Role in Forecast | Data Quality |
|--------|------|------------------|--------------|
| credit_score | Numeric | Predictor — Accounts Receivable risk proxy | [complete/partial/missing] |
| debt_ratio | Numeric | Predictor — leverage exposure (Balance Sheet risk) | [complete/partial/missing] |
| region | Categorical | Predictor — regional budget slice | [complete/partial/missing] |
| liquidity_risk | Numeric | **Forecasting Target** — dependent variable | [complete/partial/missing] |
| age | Numeric | Predictor — client lifecycle stage | [complete/partial/missing] |
| income | Numeric | Predictor — Revenue Forecast input | [complete/partial/missing] |
| risk_tolerance | Numeric | Predictor — portfolio risk appetite | [complete/partial/missing] |
| product_preference | Categorical | Predictor — product line split | [complete/partial/missing] |
| segment | Categorical | Predictor — client segment (conservative/moderate/aggressive) | [complete/partial/missing] |

**Target Variable Assessment:**
- Forecasting target: liquidity_risk
- Distribution: [symmetric/skewed] — [implication for model selection]
- Range: [min] to [max] — [interpretation]
- Modeling suitability: [regression is appropriate / transformation recommended]

**Red Flags Identified:**
- [List any missing values, outliers, or formatting issues found in Steps 2-3, or "None — ERP export is clean and complete"]

**Conclusion for VP of Finance:**
"The ERP export is [fit for purpose / requires remediation before modeling]. [If clean:] We can proceed directly to the regression model build. [If issues:] The following fields require attention before the forecast is submitted: [list issues]."

---

## Phase 3: Forecasting Model Build

> *With the ERP data profiled and the target variable identified, the FP&A team builds a regression model to quantify which financial factors most strongly drive liquidity risk. This is the equivalent of running a Hyperion planning model or a multi-factor Excel regression — except automated and reproducible.*

### Step 5: Train the Liquidity Risk Regression Model

Call `liquidity_predictor` with:
- csv_path: "demo/sample_portfolio.csv"
- (no target_column — it will auto-detect liquidity_risk)

After running:
- Show the model performance metrics (R-squared, RMSE, MAE) and feature importances
- Frame as FP&A commentary: "The FP&A team builds a regression model to predict liquidity risk from the profiled ERP data. This model quantifies which financial factors (credit score, debt ratio, income) most strongly drive the forecasting target — the same factors that explain budget variance quarter over quarter."
- Interpret R-squared: "An R-squared of [X] means the model explains [X*100]% of the variation in liquidity risk scores across the portfolio. [If R^2 > 0.7:] This is a strong fit — the model captures most of the signal in the ERP data and is suitable for budget forecasting. [If R^2 is lower:] The model is [moderate/weak] — additional feature engineering or data enrichment would improve forecast accuracy."
- Interpret RMSE: "The RMSE of [X] means the model's liquidity risk predictions are typically off by [X] score points. In budget terms, this translates to a forecast error band of ±[X] on the liquidity risk scale — which the VP of Finance should factor into reserve requirement calculations."

---

### Step 6: Feature Importance — Budget Drivers Analysis

Do NOT call any tools. This is a pure analysis step.

Interpret the feature importances from Step 5 in FP&A terms. Map each top predictor to a budget line item and explain its budget implications.

---

#### Feature Importance for Budget Drivers

Build a table mapping ML feature importance to FP&A budget line items:

| Feature | Importance Rank | Budget Line Item | Impact on Forecast |
|---------|----------------|------------------|-------------------|
| credit_score | [rank from Step 5] | Accounts Receivable Risk / Client Creditworthiness | Higher credit scores = lower expected defaults = lower loss provisions in budget |
| debt_ratio | [rank from Step 5] | Balance Sheet Leverage Exposure | Higher debt ratios = higher liquidity stress = larger reserve requirement in budget |
| income | [rank from Step 5] | Revenue Forecast Input / Client Revenue Capacity | Higher income = lower liquidity strain = supports revenue budget assumptions |
| region | [rank from Step 5] | Regional Budget Variance Driver | Regional differences = geography-specific reserve rates in the budget model |
| age | [rank from Step 5] | Client Lifecycle Stage | Age cohorts may have systematically different liquidity patterns |
| risk_tolerance | [rank from Step 5] | Portfolio Risk Appetite | Influences product mix and associated budget risk profile |

**Key Budget Insight:**
Identify the top 3 features by importance and frame: "The top drivers of liquidity risk in this portfolio are [Feature 1], [Feature 2], and [Feature 3]. For the quarterly budget, this means the FP&A team should prioritize monitoring these three metrics in the monthly ERP pull — they account for the majority of forecast variance and should be the focus of variance commentary in the CFO report."

---

### Step 7: Scenario 1 — Strong Client Profile (Best Case)

Call `predict_liquidity` with:
- credit_score: 750
- debt_ratio: 0.3
- region: "Northeast"

After running:
- Show the predicted liquidity risk score
- Frame as FP&A commentary: "Scenario 1 — Strong Client Profile. The FP&A team tests the model with a high-credit (750), low-leverage (0.3 debt ratio) client in the Northeast to establish the baseline for best-case budget assumptions. A predicted liquidity risk of [score] represents the lower bound of the portfolio's risk range — the favorable end of the budget spectrum."
- Budget interpretation: "In budget terms, a portfolio composed entirely of clients with this profile would require the minimum reserve requirement allocation and would support the most aggressive revenue forecast assumptions."

---

### Step 8: Scenario 2 — Stressed Client Profile (Worst Case)

Call `predict_liquidity` with:
- credit_score: 520
- debt_ratio: 0.7
- region: "South"

After running:
- Show the predicted liquidity risk score
- Frame as FP&A commentary: "Scenario 2 — Stressed Client Profile. Testing with a low-credit (520), high-leverage (0.7 debt ratio) client in the South establishes the worst-case budget assumptions. The spread between Scenario 1 and Scenario 2 defines the variance range the budget model must accommodate."
- Budget interpretation: "A predicted liquidity risk of [score] is [X] points higher than the best-case scenario from Step 7. This spread of [delta] represents the budget uncertainty range — the FP&A team must ensure reserves are sized to cover this variance before submitting the quarterly budget."

---

## Phase 4: Budget Variance Synthesis

> *The three-scenario budget framework is the core FP&A deliverable. Best case, base case, and worst case give the VP of Finance the range of outcomes and allow the organization to set reserve requirements, resource allocation targets, and go/no-go thresholds for capital deployment.*

### Step 9: Scenario 3 — Base Case (Most Likely)

Call `predict_liquidity` with:
- credit_score: 650
- debt_ratio: 0.45
- region: "Midwest"

After running:
- Show the predicted liquidity risk score
- Frame as FP&A commentary: "Scenario 3 — Base Case. The median-profile client (credit score 650, debt ratio 0.45, Midwest region) represents the most likely budget outcome — the central estimate the FP&A team submits as the official quarterly forecast. Comparing base case to best case and worst case gives the VP of Finance the three-scenario budget range."
- Budget interpretation: "A base case liquidity risk of [score] sits [above/below] the portfolio mean of approximately [mean from Step 3]. This [confirms/contradicts] the dataset-wide risk profile and is [consistent/inconsistent] with the regression model's population-level predictions."

---

### Step 10: Three-Scenario Budget Summary

Do NOT call any tools. This is a pure analysis step.

Compile the predictions from Steps 7, 8, and 9 into the standard FP&A three-scenario budget table.

---

#### Three-Scenario Budget Framework — Liquidity Risk Forecast

| Scenario | Client Profile | Predicted Liquidity Risk | vs. Base Case |
|----------|---------------|--------------------------|---------------|
| Best Case (Step 7) | Credit 750, Debt 0.30, Northeast | [score from Step 7] | -[delta] |
| Base Case (Step 9) | Credit 650, Debt 0.45, Midwest | [score from Step 9] | — (baseline) |
| Worst Case (Step 8) | Credit 520, Debt 0.70, South | [score from Step 8] | +[delta] |

**Variance Spread Analysis:**
- Total variance range: [worst case] - [best case] = [spread] liquidity risk points
- Upside from base: [best case delta] points ([percentage]% below base)
- Downside from base: [worst case delta] points ([percentage]% above base)
- Asymmetry: [Is the upside larger than the downside or vice versa? What does this mean for budget planning?]

**Reserve Requirement Implication:**
"This three-scenario framework is the standard FP&A deliverable for quarterly budget presentations. The VP of Finance uses the spread of [total spread] liquidity risk points to set reserve requirements and resource allocation targets. A [symmetric/asymmetric] spread suggests [balanced / conservative / optimistic] budget positioning is appropriate."

---

### Step 11: Quarterly Budget Recommendation

Do NOT call any tools. This is the final synthesis step.

Compile the complete FP&A findings into the quarterly budget recommendation memo.

---

#### Quarterly Budget Recommendation — Liquidity Risk Forecast

**Executive Summary**

This analysis profiled the ERP client portfolio export, built a liquidity risk regression model, and generated a three-scenario budget forecast. Key findings are summarized below for VP of Finance review.

---

**1. Data Quality Assessment (Phase 2)**

- ERP export: [row count] client records, [column count] fields
- Forecasting target: liquidity_risk — [suitable/requires adjustment] for regression modeling
- Data quality: [overall assessment — clean, minor issues, requires remediation]
- Action required before final budget submission: [None / [specific remediation steps]]

---

**2. Key Budget Drivers (Phase 3)**

The regression model (R-squared: [X], RMSE: [Y]) identifies the following primary drivers of liquidity risk, in order of impact:

1. **[Feature 1]** — [Budget line item and implication]
2. **[Feature 2]** — [Budget line item and implication]
3. **[Feature 3]** — [Budget line item and implication]

Budget monitoring recommendation: "The monthly ERP pull should track these three metrics explicitly. Variance in [Feature 1] or [Feature 2] is an early warning indicator for liquidity risk deviation from the forecast baseline."

---

**3. Three-Scenario Forecast Range (Phase 4)**

| Scenario | Liquidity Risk Score | Budget Interpretation |
|----------|---------------------|-----------------------|
| Best Case | [score] | Minimum reserve requirement; supports aggressive revenue assumptions |
| Base Case | [score] | Official quarterly forecast; standard reserve allocation |
| Worst Case | [score] | Maximum reserve requirement; triggers contingency budget activation |

**Recommended VP of Finance Actions:**
- Set quarterly reserve ratio based on the [base/worst] case scenario, with a [X]% buffer for model uncertainty
- Activate contingency budget review if actual liquidity risk exceeds [worst case score + model RMSE]
- Schedule monthly model re-run after each ERP export to track forecast drift

---

**4. What an FP&A Analyst Would Do Next**

This walkthrough produced the quantitative foundation for the quarterly budget submission. In a real FP&A workflow, the analyst would next:

1. **Export to Excel** — paste the three-scenario table into the quarterly budget workbook; link liquidity risk scores to the reserve requirement schedule
2. **Present variance analysis to CFO** — frame the spread between best and worst case as the budget uncertainty range; explain the top 3 drivers that create this variance
3. **Adjust reserve ratios** — use the model's RMSE as the minimum reserve buffer; add a management overlay for known risks not captured in the ERP data
4. **Schedule monthly re-runs** — re-run the full profiling and forecasting workflow after each monthly ERP extract to detect model drift and flag early warning signals before quarter-end
5. **Build the rolling forecast** — replace the static three-scenario table with a rolling 12-month forecast, updating each month as new ERP data arrives

---

### How This Maps to Traditional FP&A Tools

| This Walkthrough | Traditional Tool | Time Saved |
|-----------------|------------------|------------|
| `ingest_csv` (data profile, column types, stats) | SAP/Oracle ERP export review + manual Excel audit | Minutes vs hours of manual column inspection |
| `ingest_csv` with target_column (distribution analysis) | Excel histogram + pivot table on target field | Auto-computed distribution vs manual formula build |
| `liquidity_predictor` (regression model + feature importance) | Excel regression add-in or Hyperion planning model | Automated model training vs manual parameter setup |
| `predict_liquidity` (scenario scoring) | Manual what-if scenario tables in Excel | Instant score vs manual formula recalculation |
| Three-scenario table (Step 10) | Finance team builds manually from multiple data sources | Synthesized automatically from all prior steps |

**Total estimated time savings:** 4–6 hours of ERP export auditing, regression setup, and scenario table construction → ~10 minutes with the Finance AI Skill.

---

Walkthrough complete. Use `/finance` for general financial analysis queries.

> For educational/informational purposes only. Not financial advice. Past results do not guarantee future performance.
