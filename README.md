# Ultimate Technologies Inc. Take-Home Challenge

Complete analysis of all three parts of the provided interview challenge.

## Reproduce
Place `logins.json` and `ultimate_data_challenge.json` **one directory above** `analysis.py`, then run:

```bash
pip install -r requirements.txt
python analysis.py
```

Or open `Ultimate_Challenge.ipynb` in Jupyter and execute its cells (the notebook runs `analysis.py` and displays the figures). The raw JSON files are deliberately not included in the submission archive, consistent with the challenge instructions.

## Part 1 — Login demand
- 93,142 login events from 1970-01-01 20:12:16 to 1970-04-13 18:57:38; 0 invalid timestamps.
- 15-minute bins include 407 empty bins (filled with zero).
- The most active hours by average 15-minute volume include 22:00, 01:00 and 23:00. Weekend activity is elevated: Saturday 13.46, Sunday 12.62 versus Monday 6.21.
- There are 877 repeated timestamp values; these were retained because simultaneous logins can legitimately share timestamps. Timestamps have no timezone information; the first and last days are incomplete.

## Part 2 — Experiment design
**Primary metric:** proportion of eligible active driver partners who complete at least one revenue trip in *each* city during a pre-specified 7-day measurement window (cross-city service rate). The denominator is all randomized eligible drivers, including those who do not cross, to avoid post-treatment selection bias.

**Design:** randomize eligible drivers 1:1, stratifying by home city, baseline cross-city activity and driving intensity. Treatment receives automatic toll reimbursement for eligible crossings; control remains on current policy. Run a baseline measurement and a 4-week experiment, with the first week designated ramp-up and the final 3 weeks for the primary comparison. Communicate the offer clearly and instrument actual crossings, trips, driver-hours, toll reimbursements, and city/time-of-day. Guardrails: net contribution margin after reimbursements, completed trips per online hour, wait time, cancellations, and driver earnings.

**Statistical analysis:** estimate intention-to-treat difference in cross-city service proportions with a two-sided two-proportion z-test and 95% confidence interval, plus logistic regression adjusted for pre-specified strata and baseline behavior. Calculate sample size using baseline rate, minimum detectable effect, alpha=0.05, power=0.80 before launch. Analyze driver-level outcomes, not individual trips, because repeated trips from one driver are correlated. For frequent outcomes use a regression with robust standard errors. Check for interference/spillovers: drivers share a marketplace and treatment may change control-driver opportunities. If strong, consider geographic/time cluster randomization or a switchback experiment, with appropriate clustered inference.

**Decision rule:** recommend rollout only if the confidence interval excludes zero in the positive direction, the effect is practically meaningful, and incremental contribution margin and rider-service guardrails are acceptable. Report absolute and relative lift, cost per incremental cross-city driver, and heterogeneity by city/time. Do not treat nonsignificance as proof of no effect; report power and confidence intervals. Monitor fraudulent or unnecessary crossings, carryover effects, seasonality, and toll reimbursement errors.

## Part 3 — Rider retention
- Observed retention: 18,804 / 50,000 = 37.61%. The snapshot is defined as the latest last-trip date (2014-07-01); retained means a trip on or after 2014-06-01 (inclusive).
- Missing values: driver rating by rider 8,122, rider rating by driver 201, phone 396. There are 8 exact duplicate rows, but no rider ID exists to distinguish identical users from repeated records, so records are retained.
- Model features: city, phone, avg_dist, surge_pct, avg_surge, trips_in_first_30_days, ultimate_black_user, weekday_pct. Exclude last_trip_date (defines the label), signup_date (not needed for a first-month baseline), and both rating columns because they describe *all trips* and could contain future information. Note that the documentation says some other aggregate features may also describe all trips; the model is an exploratory benchmark, not a production-safe first-month prediction without point-in-time validation.
- 80/20 stratified train/test split (seed 42). Impute missing numeric/categorical values inside a pipeline, one-hot encode categories, compare logistic regression and Random Forest.

| Model | ROC-AUC | Accuracy | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|
| Logistic regression | 0.753 | 0.715 | 0.662 | 0.493 | 0.565 |
| Random Forest | 0.842 | 0.776 | 0.741 | 0.621 | 0.676 |

Best held-out model: **random_forest**. Highest held-out permutation importance (ROC-AUC loss): city, weekday_pct, phone, surge_pct. City retention: Astapor 25.6%, King's Landing 62.8%, Winterfell 35.2%. These are associations, not causal effects.

**Business recommendations:** pilot targeted re-engagement for low-predicted-retention segments; prioritize rider experience and supply quality in low-retention cities; test incentives and differentiated onboarding with randomized trials. Measure incremental retained riders, contribution margin, and treatment fairness, not just model scores. Retrain and validate on later signup cohorts; use a strict point-in-time feature store and temporal holdout before deployment. The observed 30-day activity label is a proxy for retention and does not guarantee sustained future activity.

## Files
- `analysis.py`: full reproducible analysis and chart generation.
- `Ultimate_Challenge.ipynb`: notebook wrapper with executive narrative and generated plots.
- `Ultimate_Challenge_Report.pdf`: presentation-ready report.
- `metrics.json`, `permutation_importance.csv`, and PNG figures: outputs.
