# ab-test-landing-page-analysis

# A/B Test — Landing Page Experiment (Python)

Statistical analysis of an A/B test comparing two landing page versions, using hypothesis testing to decide which version to ship.

## Business Objective

An ecommerce company ran an A/B test on its landing page over 28 days. The analysis answers four questions and closes with a shipping recommendation:

- Do converted users spend more on version A or version B?
- Which version converts a higher share of visitors?
- Does conversion depend on the traffic channel the user arrived from?
- Does conversion depend on whether the user is new or returning?

## Dataset

`landing_experiment.csv` — 40,000 users exposed to the landing page between 2026-01-01 and 2026-01-28. No nulls, no duplicates, one row per user.

| Column | Type | Description |
|---|---|---|
| `user_id` | UUID | Unique user identifier |
| `date` | Date | Day the user was exposed to the page |
| `landing` | Categorical | A (control) / B (variant) |
| `region` | Categorical | Norte, Centro, Sur, Occidente, Oriente |
| `dispositivo` | Categorical | Mobile, Desktop |
| `traffic_source` | Categorical | Organic, Ads, Email, Referral |
| `user_type` | Categorical | Nuevo, Recurrente |
| `converted` | Binary | 1 if the user converted, 0 otherwise |
| `gasto` | Numeric | Amount spent — **0 when the user did not convert** |

Groups are balanced: 19,982 users saw version A and 20,018 saw version B.

> The dataset belongs to TripleTen and is not redistributed in this repository. The notebook stores its outputs, so all results are visible without running it.

## Methodology

Each business question is matched to a test by variable type and comparison type, not by the size of the difference:

| Question | Variable | Comparison | Test |
|---|---|---|---|
| Spend per converted user | Numeric | Two means | Welch's t-test (after Levene) |
| Conversion rate | Binary | Two proportions | Two-proportion z-test |
| Traffic source vs conversion | Categorical | Association | Chi-square of independence |
| User type vs conversion | Categorical | Association | Chi-square of independence |

Significance level: α = 0.05. Assumptions are verified before each test — Levene's test for variance homogeneity, and expected frequencies above 5 for both chi-square tests.

A note on `gasto`: the column is 0 for users who did not convert, so the spend comparison filters on `converted == 1` first. Comparing raw averages would drag both groups toward zero and mask the real difference.

## Key Findings

### Version B wins on both components of revenue

| Metric | Page A | Page B | Difference | p-value |
|---|---|---|---|---|
| Conversion rate | 12.57% | 15.96% | +3.38 p.p. (+26.9% relative) | 3.76e-22 |
| Spend per converted user | 61.09 | 68.75 | +7.66 (+12.5%) | 3.63e-21 |
| **Revenue per exposed user** | **7.68** | **10.97** | **+43%** | — |

Levene's test returned p = 6.88e-08, indicating heterogeneous variances, so Welch's t-test was used instead of the classic t-test.

The two effects compound: B converts more visitors *and* each converted visitor spends more. That is what produces the 43% gap in revenue per exposed user.

### Traffic source matters, but far less than the page

Chi-square: χ² = 8.66, p = 0.0341 — statistically significant.

| Channel | Conversion rate | Volume |
|---|---|---|
| Email | 14.99% | 6,123 |
| Ads | 14.74% | 11,935 |
| Referral | 13.88% | 3,955 |
| Organic | 13.79% | 17,987 |

The gap between the best and worst channel is 1.20 p.p. — roughly a third of the 3.38 p.p. gained by switching pages. With n = 40,000, small differences reach significance easily, so this result is statistically detectable but of limited practical relevance. Organic has the lowest rate and the highest volume, so it still contributes the largest absolute number of conversions.

### User type does not matter

Chi-square: χ² = 0.51, p = 0.4736 — not significant. New users convert at 14.36% and returning users at 14.09%, a 0.27 p.p. gap indistinguishable from noise.

This is operationally useful: the redesign performs consistently across both profiles, so the winning version can be rolled out uniformly with no need to maintain segmented variants.

## Business Recommendation

1. **Ship version B to all traffic.** It beats A on both revenue components — 26.9% more conversions and a 12.5% higher ticket — for roughly 43% more revenue per exposed user. The rollout can be uniform, since the effect holds for new and returning users alike.
2. **Keep the current channel budget allocation.** A 1.20 p.p. spread between channels does not justify reallocation, and cutting Organic for its lower rate would be counterproductive: it generates the most conversions in absolute terms. Any channel decision should be based on acquisition cost, which this experiment did not measure.
3. **Monitor before declaring the result final.** The analysis does not account for the cost of implementing B, nor for novelty effects. Conversion and spend should be tracked over the first weeks after rollout, and a follow-up should test the interaction between page version and user type, which the aggregate analysis cannot capture.

## Skills Demonstrated

- Hypothesis formulation (H₀ / H₁) tied to explicit business questions
- Test selection by variable type and comparison type
- Assumption checking: Levene's test, expected frequency thresholds
- Welch's t-test, two-proportion z-test, chi-square of independence
- Contingency tables with `pd.crosstab`, raw counts and row-normalized
- Distinguishing statistical significance from practical relevance
- Explicit reporting of limitations in every conclusion
- Visualization of categorical relationships in both volume and rate

## How to Run

```bash
pip install pandas matplotlib seaborn scipy statsmodels
jupyter notebook ab_test_landing_experiment.ipynb
```

The notebook looks for the dataset at `/datasets/landing_experiment.csv` and falls back to `data/landing_experiment.csv`. Since the dataset is not included, the saved outputs in the notebook are the reference.

## Tools

- Python 3
- pandas, matplotlib, seaborn
- scipy.stats, statsmodels

## Author

**Sebastian Ladino Novoa** — Data Analytics Portfolio
