---
phase: 3
slug: ml-workflow-tools
status: draft
nyquist_compliant: false
wave_0_complete: false
created: 2026-03-17
---

# Phase 3 — Validation Strategy

> Per-phase validation contract for feedback sampling during execution.

---

## Test Infrastructure

| Property | Value |
|----------|-------|
| **Framework** | pytest 9.0.2 |
| **Config file** | `pyproject.toml` (already configured) |
| **Quick run command** | `.venv/bin/python3 -m pytest tests/test_ml_tools.py -x -q` |
| **Full suite command** | `.venv/bin/python3 -m pytest tests/ -q` |
| **Estimated runtime** | ~15 seconds (GridSearchCV on synthetic 100-row data) |

---

## Sampling Rate

- **After every task commit:** Run `.venv/bin/python3 -m pytest tests/test_ml_tools.py -x -q`
- **After every plan wave:** Run `.venv/bin/python3 -m pytest tests/ -q`
- **Before `/gsd:verify-work`:** Full suite must be green
- **Max feedback latency:** 15 seconds

---

## Per-Task Verification Map

| Task ID | Plan | Wave | Requirement | Test Type | Automated Command | File Exists | Status |
|---------|------|------|-------------|-----------|-------------------|-------------|--------|
| 3-01-01 | 03-01 | 1 | LQDX-01,02,03 | unit | `.venv/bin/python3 -m pytest tests/test_ml_tools.py -x -q` | Wave 0 | ⬜ pending |
| 3-01-02 | 03-01 | 1 | LQDX-01,02,03 | unit | `.venv/bin/python3 -m pytest tests/test_ml_tools.py::test_csv_structure_detection tests/test_ml_tools.py::test_data_cleaning tests/test_ml_tools.py::test_eda_output -x` | Wave 0 | ⬜ pending |
| 3-02-01 | 03-02 | 2 | LQDX-04,05,06 | unit | `.venv/bin/python3 -m pytest tests/test_ml_tools.py::test_split_before_fit tests/test_ml_tools.py::test_regression_evaluation tests/test_ml_tools.py::test_predict_liquidity -x` | Wave 0 | ⬜ pending |
| 3-03-01 | 03-03 | 2 | INVX-01,02,03,04,05,06 | unit | `.venv/bin/python3 -m pytest tests/test_ml_tools.py::test_investor_csv_detection tests/test_ml_tools.py::test_feature_engineering tests/test_ml_tools.py::test_stratified_split tests/test_ml_tools.py::test_gridsearch_runs tests/test_ml_tools.py::test_classifier_evaluation tests/test_ml_tools.py::test_classify_investor -x` | Wave 0 | ⬜ pending |
| 3-04-01 | 03-04 | 3 | LQDX-04..06, INVX-01..06 | unit | `.venv/bin/python3 -m pytest tests/test_ml_tools.py -q` | Wave 0 | ⬜ pending |
| 3-05-01 | 03-05 | 4 | all | functional | manual + `.venv/bin/python3 -m pytest tests/ -q` | existing | ⬜ pending |

*Status: ⬜ pending · ✅ green · ❌ red · ⚠️ flaky*

---

## Wave 0 Requirements

- [ ] `tests/test_ml_tools.py` — 14 test stubs covering all LQDX and INVX requirements
- [ ] `tests/data/liquidity_sample.csv` — synthetic 100-row CSV for unit tests
- [ ] `tests/data/liquidity_client_sample.csv` — 3-row CSV for predict_liquidity tests
- [ ] `tests/data/investor_sample.csv` — synthetic 120-row investor CSV
- [ ] `tests/data/investor_sample_2.csv` — second investor CSV for multi-file tests

*`finance_output/models/` created at runtime by the tool — not a Wave 0 requirement.*

---

## Manual-Only Verifications

| Behavior | Requirement | Why Manual | Test Instructions |
|----------|-------------|------------|-------------------|
| Residual plot is visually meaningful (residuals centered near zero) | LQDX-05 | Visual inspection | Open `finance_output/charts/residual_*.png` |
| Confusion matrix heatmap is readable | INVX-05 | Visual inspection | Open `finance_output/charts/confusion_*.png` |
| Feature importance chart names real columns | INVX-05 | Schema-dependent | Verify bar chart labels match CSV column names |
| Plain-English interpretations make domain sense | All LQDX/INVX | Semantic correctness | Read tool output and confirm it makes finance sense |

---

## Validation Sign-Off

- [ ] All tasks have `<automated>` verify or Wave 0 dependencies
- [ ] Sampling continuity: no 3 consecutive tasks without automated verify
- [ ] Wave 0 covers all MISSING references
- [ ] No watch-mode flags
- [ ] Feedback latency < 15s
- [ ] `nyquist_compliant: true` set in frontmatter

**Approval:** pending
