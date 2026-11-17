# A/B Test Analysis: Does a New Landing Page Increase Conversions?

## Problem
**Question:** Does the new landing page (B) give a higher conversion rate than the old page (A)?

An e-commerce site showed one of two landing pages to each visitor at random. The goal is to decide whether the new page should replace the old one.

## Data
- **Source:** [A/B testing dataset on Kaggle](https://www.kaggle.com/datasets/zhangluyuan/ab-testing) (`ab_data.csv`, 294,478 rows)
- **Columns:** `user_id`, `timestamp`, `group` (control/treatment), `landing_page` (old_page/new_page), `converted` (0/1)
- **Period:** 2–24 January 2017
- **Cleaning:**
  - Removed 3,893 rows where the group and the page did not match, for example a control user who saw the new page.
  - Removed 1 duplicate user, so each user is counted once.
  - This left 290,584 users.
- `ab_data.csv` is included in this repository (originally from Kaggle, linked above).

## Method
1. **Baseline:** compare the raw conversion rates of the two groups.
2. **Two-proportion z-test** (two-sided, α = 0.05).
   - H₀: rate_B = rate_A
   - H₁: rate_B ≠ rate_A
3. **95% confidence interval** for the difference rate_B − rate_A.
4. **Sample-size calculation:** users needed per group to detect a lift of 0.5 percentage points with 80% power.
5. **Checks on the method:**
   - **A/A test:** 1,000 random splits of control users, who all saw the same page, to check the false-positive rate.
   - **Peeking:** 500 A/A tests checked after every day, to show how stopping early inflates false positives.
6. **Stability check:** the same z-test is run separately on the first and second half of the test period.
7. **Sanity checks:** the formulas are tested on small known cases inside the notebook.

All formulas are written out in the notebook using SciPy.

**Note on evaluation:** This project uses a statistical test, not a prediction model, so there is no held-out test set, cross-validation or set of wrong predictions. Instead, the method is evaluated by: (1) A/A tests, to check the false-positive rate is about 5%; (2) a peeking simulation, to show how stopping early causes errors; (3) a stability check on each half of the test period; (4) a sample-size check, to confirm the test had enough data; and (5) sanity checks of the formulas on known cases.

## Results

| | Control (old page) | Treatment (new page) |
|---|---|---|
| Users | 145,274 | 145,310 |
| Conversions | 17,489 | 17,264 |
| Conversion rate | 12.04% | 11.88% |

| Measure | Value |
|---|---|
| Difference (B − A) | −0.16 percentage points |
| 95% confidence interval | [−0.39, +0.08] percentage points |
| z, p-value | z = −1.31, p = 0.19 |
| Users needed per group (0.5 pp lift, 80% power) | 67,676 |
| Users in the smaller group | 145,274 |
| A/A test false-positive rate | 5.3% (expected 5%) |
| False positives, checking once at the end | 5.4% |
| False positives, checking every day | 28.4% |
| Stability: first half / second half | −0.04 pp (p = 0.83) / −0.28 pp (p = 0.10) |

**Conclusion:** The new page did not increase conversions. The difference is not statistically significant, and the confidence interval rules out any lift of 0.5 percentage points or more. The test had more than twice the users it needed, so this is a reliable null result. **Recommendation: keep the old page.**

## Error analysis and interpretation
- **Statistical vs practical significance:** the CI includes zero, so there is no evidence of a difference. More importantly, its upper end (+0.08 pp) is far below the 0.5 pp minimum useful lift, so even the best case is not worth acting on.
- **False positives:** the A/A test gave 5.3% false positives, which confirms the test works as expected.
- **Peeking:** checking the result every day and stopping at the first p < 0.05 raised false positives from 5.4% to 28.4%. A real test should fix its length in advance.
- **Stability:** both halves of the test period showed a small, non-significant decrease (first half −0.04 pp, p = 0.83; second half −0.28 pp, p = 0.10), so the result is not caused by a few unusual days. The halves differ slightly, but their confidence intervals overlap heavily.
- **Sample size:** smaller lifts need far more users. This test could reliably detect lifts of about 0.35 pp or more.

## Limitations
- The test ran for about 3 weeks. Long-term effects, such as novelty wearing off, are not measured.
- The dataset has no context about the business or the page design, so the 0.5 pp "useful lift" is an assumption.
- Only one metric (conversion) is analysed. Revenue per user or other metrics might tell a different story.
- Users who visited more than once are counted on their first visit only.

## How to run
Requires Python 3 with pandas, numpy, scipy, matplotlib and Jupyter.

```bash
pip install -r requirements.txt
jupyter notebook ab_test_analysis.ipynb
```

Then use **Kernel → Restart & Run All**. The data file is already in the same folder.

## Files
```
├── ab_test_analysis.ipynb   # the full analysis
├── ab_data.csv              # the dataset (from Kaggle)
├── README.md
├── requirements.txt
└── .gitignore
```
