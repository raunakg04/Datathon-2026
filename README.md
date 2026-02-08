# NHIS Access-to-Care Analysis — Datathon 2026

An end-to-end analysis of the CDC National Health Interview Survey (NHIS) Adult Summary Health Statistics dataset, identifying access barriers, forecasting trends, and surfacing the populations most at risk of falling through the cracks.

---

## The Challenge

The NHIS dataset contains 26,208 rows of health statistics across 54 topics and ~75 demographic subgroups, spanning 2019–2024. We were tasked with addressing three core questions:

1. **Subgroup-specific prediction**: For specific health topics and subgroups, predict estimated prevalence of delayed care and projected unmet needs for the next year.
2. **Dataset-wide trends**: Predict trends in delayed or missed care across the entire dataset over the next year.
3. **At-risk identification**: Identify subgroups most at risk of falling through the cracks using clustering or anomaly detection techniques.

---

## Our Solutions

### Idea 1: Subgroup-Specific Forecasts

Rather than arbitrarily choosing a single topic and subgroup, we used a **composite friction score** to data-drivenly identify the 6 subgroups with the highest access barriers in 2024. For each, we trained a weighted linear regression model (WLS, weights = 1/se^2) to forecast 2025 estimates across all 4 access measures with calibrated prediction intervals.

**Key results**:
- 298 unique time series modeled across all (measure, GROUP, SUBGROUP) combinations
- 97.3% prediction interval coverage (target: 95%) using t-distribution quantiles with leverage correction
- Deep-dive forecasts for 6 high-friction subgroups: Uninsured, Disabled, Medium/Small Metro, With Functioning Difficulties, Female, Small MSA

**Figure**: `figures/deepdive_forecasts.png` (24 panels: 6 subgroups x 4 measures)

### Idea 2: Dataset-Wide Delayed/Missed Care Trends

We produced three complementary views:
- **Cost-barrier overlay**: All 3 cost-barrier measures on the same axes with 2025 forecasts, revealing they move in lockstep (r = 0.94–0.98)
- **Composite delayed-care index**: Averaged across measures for a single robust trend line
- **Subgroup dot-whisker plots**: 2025 forecast + interval for every subgroup, showing inequality spread

**Key findings**:
- Cost barriers dropped 1.3–1.7 pp in 2020 (COVID lockdown effect), then rebounded at +0.47 pp/year
- By 2025, all three cost barriers are projected to exceed pre-pandemic (2019) levels
- Subgroup dispersion is widening — the gap between best-off and worst-off populations is growing

**Figures**: `figures/overall_delayed_care_forecast.png`, `figures/subgroup_forecast_dotwhisker.png`

### Idea 3: At-Risk Subgroup Identification

We implemented a three-layer detection system:
1. **K-Means clustering** (k=4, validated with silhouette=0.275, Davies-Bouldin=0.862) partitions subgroups by access profile
2. **Isolation Forest** (200 trees) independently flags anomalous subgroups without circular logic
3. **Composite risk score** combines IF anomaly z-score with friction z-score

**Top 5 at-risk subgroups**:

| Rank | Subgroup | Risk Score | Why Anomalous |
|------|----------|-----------|--------------|
| 1 | Uninsured (under 65) | 5.84 | Extreme outlier on all cost barriers |
| 2 | Bisexual | 3.50 | Disproportionate barriers vs peers |
| 3 | No HS diploma | 2.45 | Compounding low literacy + low income |
| 4 | With disability | 1.93 | High barriers despite insurance |
| 5 | <100% FPL | 1.86 | Poverty concentrates all barriers |

**Figures**: `figures/cluster_pca.png`, `figures/at_risk_subgroups.png`, `figures/cluster_validation.png`

---

## Methodology — Why It's Superior

### 1. Precision Weighting, Not Guessing

Every ESTIMATE in the NHIS dataset has a confidence interval encoding survey precision. We derived standard errors from these CIs and used them as regression weights (1/se^2), ensuring more precisely measured observations have proportionally greater influence. A basic analysis would ignore this information entirely, treating all data points as equally reliable.

### 2. Calibrated Uncertainty, Not Just Point Predictions

We produce 95% prediction intervals using t-distribution quantiles (not the z = 1.96 normal approximation, which would achieve only ~43% coverage with n = 6). Our 97.3% empirical coverage means decision-makers can trust our intervals for resource planning. We also include leverage correction — intervals are automatically wider for extrapolation beyond observed years.

### 3. Honest Baseline Comparison

We transparently report that naive (last-value) forecasts achieve lower MAE (1.005 pp) than our WLS model (1.574 pp). Rather than hiding this, we explain that these metrics are near-random-walk series where persistence forecasts are optimal — and that our model's value lies in calibrated uncertainty, trend identification, and extrapolation capability that naive forecasts cannot provide.

### 4. Independent Anomaly Detection

Using centroid distance from K-Means clusters as an anomaly score creates circular logic (high-friction subgroups pull centroids, giving them low distances). We solve this with Isolation Forest — an independent ensemble method that scores anomalies without reference to cluster assignments. Cluster validity is confirmed with quantitative metrics (silhouette + Davies-Bouldin), not subjective elbow plots.

### 5. Barrier Syndrome Discovery

Our correlation analysis reveals that cost-related barriers are not independent — they correlate at r = 0.94–0.98. This "barrier syndrome" insight means a single well-targeted intervention (insurance expansion) addresses all barriers simultaneously, rather than requiring four separate programs. This finding fundamentally changes the policy calculus.

### 6. COVID Disruption as Evidence

Rather than treating 2020 as "an anomaly to exclude," we quantified the disruption (cost barriers dropped 1.3–1.7 pp) and used it as a natural experiment. The paradoxical drop reveals that barriers are demand-driven (people who don't seek care don't encounter cost barriers), and the post-2021 rebound (+0.4 pp/year) confirms they are structural, not transient. Sensitivity analysis confirms including 2020 yields comparable forecast quality.

---

## Project Structure

```
Datathon-2026/
  Access_to_Care_Dataset.csv    # Input data (NHIS Adult Summary Health Statistics)
  analysis.ipynb                # Main analysis notebook (28 cells, fully executable)
  figures/                      # 25 generated PNG visualizations
    *_trend.png (x4)            # Per-measure trend lines with CI
    *_subgroups.png (x4)        # Top-10 subgroup bar charts with CI
    *_spread.png (x4)           # Estimate spread box-plots
    friction_heatmap.png        # Composite friction score heatmap
    friction_ranked.png         # Top 15 friction-ranked subgroups
    inequity_gaps.png           # Inequity gap trends (4 pairs)
    barrier_correlation.png     # Pearson correlation matrix
    forecast_examples.png       # Total-population WLS forecasts
    deepdive_forecasts.png      # 6 subgroups x 4 measures deep-dive
    overall_delayed_care_forecast.png  # National cost-barrier overlay
    subgroup_forecast_dotwhisker.png   # Dot-whisker 2025 forecasts
    cluster_validation.png      # Silhouette + DB + elbow
    cluster_pca.png             # PCA scatter (3 panels)
    at_risk_subgroups.png       # Top 15 at-risk bar chart
    residual_diagnostics.png    # Residual analysis
    elbow_plot.png              # K-Means elbow plot
  report/
    submission.md               # Final report (rubric-aligned)
  README.md                     # This file
```

## Requirements

- Python 3.8+
- pandas
- numpy
- matplotlib
- scikit-learn

No additional packages required. The notebook is self-contained and fully reproducible.

## Running the Analysis

```bash
jupyter notebook analysis.ipynb
# Or execute non-interactively:
jupyter nbconvert --to notebook --execute analysis.ipynb
```

All figures are saved to `figures/` and the report is generated to `report/submission.md` automatically.
