# Recommendation Memo: Onboarding & Activation Campaign

**To:** Leadership
**From:** Business Analytics
**Re:** Should the new onboarding ("Treatment") experience be launched to all users?
**Files referenced:** `outputs/experiment_summary.xlsx`, `analysis/experiment_analysis.xlsx`, `analysis/hypothesis_test_notes.md`, `outputs/kpi_tree.png`

---

## Executive Summary

The new onboarding campaign (Treatment) **more than doubles the paid conversion rate** (3.19% → 7.04%, p = 0.0011) and improves engagement and time-to-convert. However, it also drives a **statistically significant increase in support ticket volume** (+10.0 points, p < 0.0001) and shows **directionally weaker revenue per converted user** (-\$859.69, p = 0.079, borderline). Refund rate is also directionally higher in Treatment, though not statistically significant.

**Recommendation: Launch only for selected segments**, while continuing to monitor support burden and revenue quality before a full rollout. See Section 7 for the full reasoning.

---

## 1. North Star Metric

**Paid conversion rate** (`converted_to_paid`, i.e., the share of signed-up users who convert to a paying customer) is the North Star metric.

- It is the clearest, least gameable signal that a user found real value during onboarding.
- Funnel metrics (landing page visits, trial starts, onboarding completion) are *supporting* metrics — they are leading indicators on the path to conversion, but a user can complete every onboarding step and still never pay, so they don't represent business outcome on their own.
- Paid conversion connects directly to revenue growth: more conversions, holding price constant, means more recurring revenue and a healthier acquisition funnel.
- **Risk of optimizing blindly:** conversion rate alone says nothing about the *quality* of the customers gained. A campaign could spike conversions by being aggressive, oversimplifying onboarding, or setting unrealistic expectations — producing customers who churn quickly, refund, or burden support. This is exactly the pattern guardrails are designed to catch (see Section 5).

---

## 2. KPI Tree

See `outputs/kpi_tree.png` / `screenshots/kpi_tree_preview.png`.

The North Star (Paid Conversion Rate) breaks into three primary drivers, each with two sub-drivers, plus four guardrails monitored alongside:

- **Acquisition & Activation Funnel** → Landing Page Visit Rate, Trial Start Rate
- **Onboarding Experience Quality** → Onboarding Completion Rate, Time-to-Value / Engagement Score
- **Revenue Quality & Retention Readiness** → Avg Revenue per Converted User, Days to Convert
- **Guardrails:** Refund Rate, Support Ticket Rate, Engagement Score Quality, Revenue per Converted User

---

## 3. Experiment Result Summary

| Metric | Control | Treatment | Relative Lift | P-value |
|---|---|---|---|---|
| Landing page visit rate | 63.62% | 72.39% | +13.8% | 0.0004 |
| Trial start rate | 25.07% | 29.01% | +15.7% | 0.0970 |
| Onboarding completion rate | 15.65% | 21.13% | +35.0% | 0.0083 |
| **Paid conversion rate (North Star)** | **3.19%** | **7.04%** | **+120.9%** | **0.0011** |
| Avg revenue per user | \$51.97 | \$54.25 | +4.4% | 0.9107 |
| Avg revenue per converted user | \$1,630.10 | \$770.41 | -52.7% | 0.0791 |
| Refund rate (guardrail) | 0.00% | 0.42% | n/a | 0.0874 |
| Support ticket rate (guardrail) | 14.78% | 24.79% | +67.7% | <0.0001 |
| Avg engagement score | 57.03 | 62.94 | +10.4% | <0.0001 |
| Avg days to convert (guardrail) | 8.86 days | 6.40 days | -27.8% | 0.0083 |

Every funnel stage improves in Treatment, and the lift compounds down the funnel — the conversion lift is not driven by one isolated stage.

---

## 4. Hypothesis Test Interpretation

The primary test (two-proportion Z-test on paid conversion rate) gives **Z = 3.264, p = 0.0011** — well below the α = 0.05 threshold, so we reject the null hypothesis of no difference. The lift is both statistically significant and practically large (conversion rate more than doubles), which is rare — many experiments produce a significant but commercially trivial lift; this is not one of those cases.

Full test framework, formulas, and all five secondary tests are documented in `analysis/hypothesis_test_notes.md`, with evidence in `screenshots/hypothesis_test_output.png`.

---

## 5. Guardrail Analysis

Per Task 8, conversion improvement alone is not sufficient grounds to launch. Four guardrails were evaluated:

| Guardrail | Result | Risk? |
|---|---|---|
| **Support ticket rate** | 14.78% → 24.79% (p < 0.0001) | **Yes — significant risk.** Treatment users need meaningfully more support, which has direct cost implications and may signal confusing UX in the new flow. |
| **Refund rate** | 0.00% → 0.42% (p = 0.087) | **Possible risk, not yet significant.** Off a near-zero base, this is the kind of metric that can become significant with more data or at full scale — worth a guarded eye, not yet a blocker. |
| **Avg revenue per converted user** | \$1,630 → \$770 (p = 0.079, borderline) | **Possible risk.** Treatment is converting more *lower-value* users (disproportionately on the Free plan — see segment analysis), which dilutes revenue per converted user even as total conversions rise. |
| **Engagement score** | 57.03 → 62.94 (p < 0.0001) | **No risk — improvement.** Treatment users are measurably more engaged. |
| **Days to convert** | 8.86 → 6.40 days (p = 0.0083) | **No risk — improvement.** Faster time-to-value is a positive operational signal. |

**Net guardrail read:** two guardrails clearly improve, one is a confirmed risk (support tickets), and two are borderline risks worth watching (refunds, revenue per converted user). This is a mixed picture, not a clean "all clear."

---

## 6. Segment-Level Insight

The conversion lift holds in **every** segment examined (region, device type, plan type) — there is no segment where Treatment underperforms Control, which is reassuring for a broad rollout. However:

- **Plan type matters most for the revenue story.** Free-plan users see the largest conversion lift (3.06% → 9.29%), while Premium and Basic lift more modestly. This is good for top-of-funnel growth but explains the lower average revenue per converted user — Treatment is pulling in more low-tier converters.
- **Device type:** Mobile shows the strongest conversion lift (2.58% → 7.41%); Desktop the weakest relative lift, though still positive.
- **Region:** North and South show the largest absolute lifts; West shows the smallest, though still statistically meaningful as part of the pooled result.

No segment shows a guardrail breach severe enough to suggest the campaign should be withheld entirely from that segment — but the Free-plan / mobile pattern reinforces that the new converters are a different (lower ARPU, higher support-need) mix than Control's converters.

---

## 7. Final Recommendation: **Launch Only for Selected Segments**

We recommend a **phased, segment-targeted launch** rather than an immediate full rollout or an outright "do not launch":

- **Launch now** for segments where the lift is strong and guardrail exposure is most acceptable — particularly **Mobile** and **Free-plan** acquisition, where the conversion gains are largest and the funnel-stage data is most convincing.
- **Hold full Desktop / Premium rollout** briefly while the support-ticket driver is diagnosed — Premium customers generating more tickets is a higher-cost risk per user than Free-plan tickets.
- **Do not yet claim a full, unconditional launch** company-wide, because the support ticket increase is statistically confirmed and the revenue-per-converted-user decline, while not yet significant, is large enough (-52.7%) to warrant a short confirmatory monitoring window rather than being dismissed.

This is a middle path between "Launch" and "Continue testing": the core hypothesis is proven, but the guardrail picture is mixed enough that an unconditional full launch would be acting on the upside evidence while ignoring the support-cost evidence.

---

## 8. Risks and Limitations

- **Sample size on rare events:** Refund counts are very small (0 vs 3), so the refund-rate test is underpowered; a true effect could exist that this sample can't yet detect.
- **Revenue outliers:** Four high-value converters were identified via IQR analysis (retained, not removed, as they appear genuine) — they inflate Control's average revenue per converted user and partly explain the gap with Treatment; a median-based or trimmed-mean comparison is a useful follow-up.
- **Support ticket cause is unknown:** This analysis identifies *that* tickets increased, not *why*. It could be confusion, or simply more engaged users asking more questions — these have very different cost/risk implications.
- **30-day window:** All outcome metrics are measured at 30 days; downstream retention/churn beyond that window is not observed in this dataset.
- **Observational segment cuts:** Segment-level results are descriptive comparisons within an already-randomized experiment; segment sample sizes (e.g., Tablet, n≈111) are smaller and estimates there are noisier.

---

## 9. Next Steps

1. Launch Treatment to Mobile and Free-plan segments; hold a control slice (e.g., 10%) for ongoing monitoring.
2. Open a fast-follow investigation into the support ticket driver (ticket category tagging, qualitative review of a ticket sample).
3. Re-run the refund-rate and revenue-per-converted-user tests after 4-6 more weeks of data / a larger sample to resolve the two borderline guardrails.
4. Track 60-90 day retention and churn for both groups once available, to confirm the engagement and conversion gains translate into durable revenue.
5. Re-evaluate full rollout to Desktop/Premium once the support driver is understood and guardrail metrics stabilize.
