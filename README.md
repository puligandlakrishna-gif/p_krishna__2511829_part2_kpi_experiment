# p_krishna_2511829_part2_kpi_experiment

## Business Context

A subscription-based digital product company launched a new onboarding and activation campaign to improve user conversion and early engagement. Users were randomly split into two groups:

- **Control:** existing onboarding experience
- **Treatment:** new campaign experience

**Business problem statement:**
- **Decision to be made:** Whether to launch the new ("Treatment") onboarding experience to all users.
- **Who it impacts:** All new signups going forward (product/growth team owns the rollout; support team absorbs any change in ticket volume; finance/revenue teams are affected by changes in conversion mix and revenue per converted user).
- **Metric that should improve:** Paid conversion rate (the North Star metric — see below).
- **Risks to monitor:** Refund rate, support ticket rate, revenue per converted user, and engagement score — i.e., whether any conversion gain comes at the cost of lower-quality customers or higher operating cost.
- **Evidence required before recommending:** A statistically valid comparison of Control vs Treatment on the North Star metric (hypothesis test), plus a check of guardrail metrics and segment-level consistency, before any launch recommendation is made.

This is summarized further in `outputs/recommendation_memo.md`.

## Dataset Description

- **File:** `data/campaign_experiment_data.xlsx`
- **Sheet `experiment_data`:** 1,408 raw rows (1,400 unique users after removing 8 exact duplicate rows), one row per user, with: `user_id`, `signup_date`, `experiment_group` (Control/Treatment), `region`, `device_type`, `traffic_source`, `plan_type`, funnel flags (`visited_landing_page`, `started_trial`, `completed_onboarding`, `converted_to_paid`), `revenue_30d`, `support_tickets_30d`, `refund_requested`, `days_to_convert`, `engagement_score`.
- **Sheet `business_context`:** plain-text description of the experiment design, provided by the business.

## North Star Metric

**Paid conversion rate** (`converted_to_paid`) — the share of signed-up users who become paying customers.

- **Why North Star:** it is the cleanest signal of real value delivered; funnel steps (landing page visit, trial start, onboarding completion) are *leading* indicators, but only conversion reflects an actual business outcome (revenue-generating customer).
- **Why other metrics are supporting, not North Star:** funnel-stage metrics can improve without ever producing a paying customer; revenue and engagement metrics describe the *quality* of conversions, not whether conversion itself improved.
- **Connection to growth:** more conversions at a stable or improving price point directly grows recurring revenue and funds further acquisition spend.
- **Risk of optimizing blindly:** chasing conversion rate alone can reward tactics that produce low-value, high-churn, or high-support-cost customers — exactly why guardrails are tracked alongside it (see below).

## KPI Tree Summary

See `outputs/kpi_tree.png` (full diagram) and `screenshots/kpi_tree_preview.png`.

- **North Star:** Paid Conversion Rate
- **Driver 1 — Acquisition & Activation Funnel:** Landing Page Visit Rate, Trial Start Rate
- **Driver 2 — Onboarding Experience Quality:** Onboarding Completion Rate, Time-to-Value / Engagement Score
- **Driver 3 — Revenue Quality & Retention Readiness:** Avg Revenue per Converted User, Days to Convert
- **Guardrails:** Refund Rate, Support Ticket Rate, Engagement Score Quality, Revenue per Converted User

## Experiment Analysis Approach

All cleaning and analysis is done with **live Excel formulas** in `analysis/experiment_analysis.xlsx` (verified by recalculating with LibreOffice headless and cross-checking against an independent Python/pandas/scipy analysis — all values match):

1. **`raw_data`** — unmodified copy of the source data.
2. **`data_quality_checks`** — missing values, group counts, duplicate user IDs, invalid binary value checks, revenue outlier (IQR) check, and a note on segment balance.
3. **`cleaned_data`** — duplicates removed (1,408 → 1,400 rows), missing `device_type` / `traffic_source` filled as `"Unknown"`; includes helper columns used by t-test formulas.
4. **`segment_balance`** — % share of region / device type / traffic source / plan type within each group, confirming random assignment was balanced.
5. **`metrics_summary`** — Control vs Treatment for all core funnel, revenue, and guardrail metrics.
6. **`segment_metrics`** — conversion rate, refund rate, and engagement score broken out by region, device type, and plan type.
7. **`hypothesis_testing`** — two-proportion Z-tests (conversion, refund rate, support ticket rate) and Welch's t-tests (engagement score, revenue per converted user), all as live formulas.

### Data Quality Checks (Task 4) — handling summary

| Check | Finding | Handling |
|---|---|---|
| Missing values | `device_type` (18), `traffic_source` (24), `engagement_score` (14) missing; `days_to_convert` missing for all 1,336 non-converters (expected, not an error) | Categorical blanks filled as `"Unknown"`; numeric blanks excluded via `AVERAGE`-style formulas which ignore blanks; `days_to_convert` NaNs are logically correct and left as-is |
| Group counts | Control 693 / Treatment 715 (raw); 690 / 710 after dedup | Used post-dedup counts for all analysis |
| Duplicate user IDs | 8 fully duplicated rows (4 user_ids appearing twice, identical in every column) | Removed via "keep first occurrence" in `cleaned_data` |
| Invalid binary values | None — all of `visited_landing_page`, `started_trial`, `completed_onboarding`, `converted_to_paid`, `refund_requested` contain only 0/1 | No action needed; verified with `COUNTIFS` checks |
| Outliers in revenue | 4 converted users flagged via IQR method (> Q3 + 1.5×IQR) | Retained (appear to be genuine high-value customers, not data entry errors); flagged for awareness in revenue interpretation |
| Segment distribution across groups | Region, device type, traffic source, and plan type shares are all within ~1-4 percentage points between Control and Treatment | Confirms randomization was balanced; no segment-specific reweighting needed |

## Hypothesis Test Summary

Full detail in `analysis/hypothesis_test_notes.md`; evidence in `screenshots/hypothesis_test_output.png`.

- **Primary test:** Two-proportion Z-test on paid conversion rate. H0: no difference between groups. Two-tailed, α = 0.05.
  **Result: Z = 3.264, p = 0.0011 → reject H0.** Treatment's conversion rate (7.04%) is significantly higher than Control's (3.19%), a +120.9% relative lift.
- **Guardrail tests:** support ticket rate (p < 0.0001, significant increase — risk), engagement score (p < 0.0001, significant improvement), days to convert (p = 0.0083, significant improvement), refund rate (p = 0.087, not significant but directionally worse), revenue per converted user (p = 0.079, borderline, directionally worse).

## Guardrail Metrics Considered

1. **Refund rate** — Control 0.00% vs Treatment 0.42% (p = 0.087, not significant, but worth monitoring off a near-zero base).
2. **Support ticket rate** — Control 14.78% vs Treatment 24.79% (p < 0.0001, **significant risk**).
3. **Revenue per converted user** — Control \$1,630 vs Treatment \$770 (p = 0.079, borderline risk; driven partly by a higher share of Free-plan converters in Treatment).
4. **Engagement score** and **days to convert** — both improve significantly in Treatment (positive guardrails, no risk).

## Final Recommendation

**Launch only for selected segments** (Mobile and Free-plan acquisition show the strongest, safest lift), while holding a short monitoring window before a full Desktop/Premium rollout, pending investigation of the support-ticket driver and confirmation on the borderline refund-rate / revenue-per-converted-user guardrails. Full reasoning, risks, and next steps are in `outputs/recommendation_memo.md`.

## Assumptions and Limitations

- Users were assumed to be randomly and independently assigned to Control/Treatment; the segment balance check supports this.
- The 8 duplicate rows were treated as data-entry duplication (not as two genuinely repeated user actions) and removed via "keep first occurrence."
- Revenue outliers (4 converted users) were treated as genuine rather than erroneous and retained in all analysis.
- All outcome metrics reflect a 30-day observation window; no longer-term retention/churn data is available in this dataset.
- Segment-level results, especially for smaller segments (e.g., Tablet, n≈111), are descriptive and based on smaller samples than the overall test, so should be treated as directional rather than independently conclusive.
- The two-proportion Z-test and Welch's t-test assume the standard large-sample approximations underlying those tests; with very small counts (e.g., 0 vs 3 refunds), p-values for that specific guardrail should be treated cautiously.

## Screenshots Included

- `screenshots/kpi_tree_preview.png` — KPI tree diagram
- `screenshots/summary_metrics.png` — Control vs Treatment metrics summary table (from `outputs/experiment_summary.xlsx`)
- `screenshots/hypothesis_test_output.png` — primary hypothesis test (two-proportion Z-test) output (from `analysis/experiment_analysis.xlsx`)

```