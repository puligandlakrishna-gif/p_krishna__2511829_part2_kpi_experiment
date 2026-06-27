# Hypothesis Test Notes

**File:** `analysis/hypothesis_test_notes.md`
**Experiment:** Onboarding & Activation Campaign — Control vs Treatment
**Data:** `data/campaign_experiment_data.xlsx` (cleaned: 1,400 unique users after removing 8 duplicate rows)
**Live formulas:** `analysis/experiment_analysis.xlsx`, sheet `hypothesis_testing`
**Screenshot evidence:** `screenshots/hypothesis_test_output.png`

---

## 1. Primary Hypothesis — Paid Conversion Rate (North Star Metric)

| Item | Detail |
|---|---|
| **Metric being tested** | Paid conversion rate (`converted_to_paid`) |
| **Reason for choosing this metric** | It is the North Star metric — the single clearest signal that a user found enough value to pay, and the metric leadership's launch decision hinges on. |
| **Null hypothesis (H0)** | There is no difference in paid conversion rate between Control and Treatment (p_treatment = p_control). |
| **Alternate hypothesis (H1)** | Paid conversion rate differs between Control and Treatment (p_treatment ≠ p_control). |
| **Tail** | Two-tailed. Although the business hopes for an improvement, a two-tailed test is used so that a meaningful *decline* in Treatment would also be caught and not dismissed by an assumption of improvement. |
| **Significance level** | α = 0.05 |
| **Test used** | Two-proportion Z-test (pooled variance), appropriate for comparing two independent binomial conversion rates at this sample size (n > 500 per group). |

### Test Inputs
- Control: 22 conversions / 690 users = 3.19%
- Treatment: 50 conversions / 710 users = 7.04%
- Pooled proportion: 5.14%
- Standard error: 0.0118

### Test Output
- **Z = 3.264**
- **P-value (two-tailed) = 0.0011**
- Relative lift: **+120.9%**

### Decision Rule
If p-value < 0.05 → reject H0 and conclude the conversion rates are significantly different.

### Interpretation
P = 0.0011 < 0.05, so **we reject H0**. The Treatment's conversion rate is significantly higher than Control's, and the effect size (more than double the baseline rate) is large enough to be business-relevant, not just statistically detectable.

### Business Interpretation
The new onboarding experience meaningfully increases the rate at which signed-up users become paying customers. Taken alone, this is strong evidence in favor of launch — but a conversion-only view is exactly what Task 8 (guardrails) warns against, so this result must be read together with the guardrail tests below before any launch decision is made.

---

## 2. Guardrail Hypotheses

A win on the North Star metric is only "real" if it does not come at the cost of degraded guardrails. Each guardrail below uses the same two-tailed, α = 0.05 framework (or a Welch's t-test for continuous metrics), because we want to detect harm in either direction, not just confirm an expected one.

### 2a. Refund Rate

| H0 | Refund rate is equal in Control and Treatment |
|---|---|
| H1 | Refund rate differs between Control and Treatment |
| Test | Two-proportion Z-test, two-tailed, α = 0.05 |
| Result | Control 0.00% vs Treatment 0.42%; Z = 1.71; **p = 0.0874** |
| Decision | Fail to reject H0 — not statistically significant at the 5% level, though it is directionally worse for Treatment and worth monitoring post-launch given the small base rate. |

### 2b. Support Ticket Rate

| H0 | Support ticket rate is equal in Control and Treatment |
|---|---|
| H1 | Support ticket rate differs between Control and Treatment |
| Test | Two-proportion Z-test, two-tailed, α = 0.05 |
| Result | Control 14.78% vs Treatment 24.79%; Z = 4.69; **p < 0.0001** |
| Decision | **Reject H0.** Treatment users contact support significantly more often. This is a real risk signal — the new onboarding may be driving engagement at the cost of confusion or friction. |

### 2c. Engagement Score

| H0 | Mean engagement score is equal in Control and Treatment |
|---|---|
| H1 | Mean engagement score differs between Control and Treatment |
| Test | Welch's two-sample t-test (unequal variance), two-tailed, α = 0.05 |
| Result | Control mean 57.03 vs Treatment mean 62.94; **p < 0.0001** |
| Decision | **Reject H0.** Treatment users are more engaged — a positive guardrail result that supports the conversion story. |

### 2d. Revenue per Converted User

| H0 | Mean revenue per converted user is equal in Control and Treatment |
|---|---|
| H1 | Mean revenue per converted user differs between Control and Treatment |
| Test | Welch's two-sample t-test (unequal variance), two-tailed, α = 0.05 |
| Result | Control mean \$1,630.10 vs Treatment mean \$770.41; **p = 0.0791** |
| Decision | Fail to reject H0 at α = 0.05, but the gap is large and directionally concerning (marginal/borderline result) — Treatment converters are generating notably less revenue each, partly driven by a higher share of Free-plan converters in Treatment (see segment analysis). This warrants continued monitoring even though it isn't "statistically significant" by the strict 0.05 cutoff. |

### 2e. Days to Convert

| H0 | Mean days-to-convert is equal in Control and Treatment |
|---|---|
| H1 | Mean days-to-convert differs between Control and Treatment |
| Test | Welch's two-sample t-test (unequal variance), two-tailed, α = 0.05 |
| Result | Control mean 8.86 days vs Treatment mean 6.40 days; **p = 0.0083** |
| Decision | **Reject H0.** Treatment users convert faster — a positive operational signal (faster time-to-value). |

---

## 3. Connection to the Business Decision

The hypotheses above are not run in isolation — they form a single decision framework:

1. The North Star test confirms the campaign **works** (conversion lift is real and large).
2. The guardrail tests confirm whether that lift is **safe to scale**:
   - Engagement and days-to-convert guardrails are **green** (both improve).
   - Support ticket rate is a **red flag** (statistically significant increase in support burden).
   - Refund rate and revenue-per-converted-user are **borderline** — not significant at α = 0.05, but directionally worse for Treatment and worth a closer look (see `outputs/recommendation_memo.md` for how this shapes the final recommendation).

This is why the recommendation is not a simple "launch" — see the recommendation memo for the full risk-weighted decision.
