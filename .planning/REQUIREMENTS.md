# Requirements: Finance AI Skill — v1.2 Role Walkthroughs

**Defined:** 2026-03-18
**Core Value:** Finance professionals get professional-grade analysis by describing what they want — the skill writes, runs, and interprets the code for them.

## v1.2 Requirements

Requirements for 5 remaining role-based walkthrough slash commands. Each follows the equity research pattern: multi-phase, auto-running scenario using existing MCP tools with role-specific framing.

### Hedge Fund / Trading Desk

- [x] **HF-01**: User can run `/walkthrough-hedge-fund` for a multi-phase volatility regime and pair trading scenario
- [x] **HF-02**: Walkthrough covers volatility regime detection across market conditions
- [x] **HF-03**: Walkthrough covers cross-sector diversification analysis
- [x] **HF-04**: Walkthrough covers correlation-based pair identification

### Investment Banking

- [ ] **IB-01**: User can run `/walkthrough-investment-banking` for a comparable company analysis scenario
- [ ] **IB-02**: Walkthrough covers 5-ticker comps with relative valuation framing
- [ ] **IB-03**: Walkthrough covers relative performance analysis for deal pitch materials

### FP&A Analyst

- [ ] **FPA-01**: User can run `/walkthrough-fpa` for a data profiling and forecasting prep scenario
- [ ] **FPA-02**: Walkthrough covers CSV data profiling pipeline with target column identification
- [ ] **FPA-03**: Walkthrough covers ERP export cleanup and ML forecasting prep using liquidity predictor

### Private Equity / VC

- [ ] **PE-01**: User can run `/walkthrough-private-equity` for a due diligence and portfolio monitoring scenario
- [ ] **PE-02**: Walkthrough covers due diligence scoring using investor classifier
- [ ] **PE-03**: Walkthrough covers multi-prospect comparison and portfolio company monitoring

### Accounting / Controller

- [ ] **ACCT-01**: User can run `/walkthrough-accounting` for a transaction profiling and anomaly detection scenario
- [ ] **ACCT-02**: Walkthrough covers transaction data profiling via CSV ingestion
- [ ] **ACCT-03**: Walkthrough covers anomaly detection prep and ERP consolidation patterns

### Testing

- [ ] **TEST-01**: Each walkthrough has a dedicated test file validating structure, phases, and MCP tool coverage

## Future Requirements

None — all 5 remaining walkthroughs scoped for this milestone.

## Out of Scope

| Feature | Reason |
|---------|--------|
| New MCP tools | Walkthroughs reuse existing 11 tools |
| Equity research walkthrough changes | Already shipped in v1.1 |
| Real-time data feeds | Batch analysis only (existing constraint) |
| Role-specific ML models | Reuse existing liquidity + investor models |

## Traceability

| Requirement | Phase | Status |
|-------------|-------|--------|
| HF-01 | Phase 9 | Complete |
| HF-02 | Phase 9 | Complete |
| HF-03 | Phase 9 | Complete |
| HF-04 | Phase 9 | Complete |
| IB-01 | Phase 9 | Pending |
| IB-02 | Phase 9 | Pending |
| IB-03 | Phase 9 | Pending |
| FPA-01 | Phase 10 | Pending |
| FPA-02 | Phase 10 | Pending |
| FPA-03 | Phase 10 | Pending |
| PE-01 | Phase 11 | Pending |
| PE-02 | Phase 11 | Pending |
| PE-03 | Phase 11 | Pending |
| ACCT-01 | Phase 10 | Pending |
| ACCT-02 | Phase 10 | Pending |
| ACCT-03 | Phase 10 | Pending |
| TEST-01 | Phase 12 | Pending |

**Coverage:**
- v1.2 requirements: 17 total
- Mapped to phases: 17
- Unmapped: 0 ✓

---
*Requirements defined: 2026-03-18*
*Last updated: 2026-03-18 — traceability mapped after roadmap creation*
