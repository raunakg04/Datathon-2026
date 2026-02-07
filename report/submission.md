# NHIS Access-to-Care Analysis — Submission Report

## 1. Overview

This analysis examines the CDC National Health Interview Survey (NHIS) Adult Summary
Health Statistics dataset (2019-2024) to identify barriers, inequities, and trends in
healthcare access among U.S. adults aged 18 and older.

## 2. Methods

### 2.1 Preprocessing

1. Loaded 26208 rows from `Access_to_Care_Dataset.csv`.
2. Normalized string columns (strip whitespace) and coerced numeric fields.
3. Created three tiers:
   - `df_all` (26208 rows) — all records
   - `df_numeric` (23839 rows) — non-null ESTIMATE
   - `df_clean` (23609 rows) — non-null ESTIMATE **and** uncertainty (CI or SE); flagged rows excluded
4. Derived standard error from confidence intervals: `se = (UCI − LCI) / (2 × 1.96)`
   since the STANDARD_ERROR column is empty for all rows in this dataset.

### 2.2 Classification

Rows were classified as **access_barrier** or **condition_prevalence** via case-insensitive
keyword search over `TOPIC + SUBTOPIC + TAXONOMY + ESTIMATE_TYPE`. Keywords:
`delay`, `did not get`, `unmet`, `cost`, `afford`, `uninsured`, `usual place of care`,
`Healthcare access and quality`, `Health insurance`.

### 2.3 Selected Access Measures

- **Has a usual place of care among adults** (Percent of population, crude)
- **Delayed getting medical care due to cost among adults** (Percent of population, crude)
- **Did not get needed medical care due to cost** (Percent of population, crude)
- **Did not take medication as prescribed to save money** (Percent of population, crude)

### 2.4 Composite Access Friction Score

For each selected measure, year, and demographic group:
1. Z-scored ESTIMATE values within `(measure, year, CLASSIFICATION | GROUP)`.
2. Averaged z-scores across measures per `(SUBGROUP, year)`.
Higher friction → subgroup experiences disproportionately worse access.

### 2.5 Inequity Gaps

Dynamically detected subgroup pairs:
- Age extremes: 18-34 years vs 65 years and older
- Income: <100% FPL vs ≥200% FPL
- Sex: Male vs Female
- Insurance: Uninsured vs Private

For each pair, computed absolute gap per year with pooled-SE confidence intervals.

### 2.6 Forecasting

Weighted linear regression (`weights = 1/se²`) on series with ≥ 5 years.

Back-test on 298 series (hold-out-last-year):
- **MAE**: 1.574 percentage points
- **95 % prediction-interval coverage**: 97.3 %


## 3. Key Findings

### 3.1 Top Friction Subgroups (Latest Year)

|   Rank | SUBGROUP                                       |   TIME_PERIOD |   friction_score |
|-------:|:-----------------------------------------------|--------------:|-----------------:|
|      1 | Medium and small metro                         |          2024 |         0.828501 |
|      2 | Uninsured                                      |          2024 |         0.745151 |
|      3 | With functioning difficulties                  |          2024 |         0.707107 |
|      4 | With disability                                |          2024 |         0.707107 |
|      5 | Female                                         |          2024 |         0.707107 |
|      6 | Small MSA                                      |          2024 |         0.669765 |
|      7 | South                                          |          2024 |         0.641665 |
|      8 | Employed - Part-time                           |          2024 |         0.631477 |
|      9 | Native Hawaiian or Other Pacific Islander only |          2024 |         0.620125 |
|     10 | Some college                                   |          2024 |         0.611405 |
|     11 | Divorced or separated                          |          2024 |         0.606356 |
|     12 | Medium social vulnerability                    |          2024 |         0.594563 |
|     13 | Black and White                                |          2024 |         0.593211 |
|     14 | Bisexual                                       |          2024 |         0.573614 |
|     15 | 45-64 years                                    |          2024 |         0.559478 |

### 3.2 Trend Highlights

- Per-measure trend plots are saved in `figures/` showing national-level trajectories.
- Box-plots reveal year-over-year dispersion across demographic subgroups.

### 3.3 Inequity Gaps

- Gap plots in `figures/inequity_gaps.png` show how disparities between advantaged
  and disadvantaged subgroups evolve from 2019 to 2024.

### 3.4 Forecasts

- Forecast examples for total-population series are in `figures/forecast_examples.png`.

## 4. Limitations

1. **Derived SE**: All standard errors are back-calculated from 95 % CIs; this may
   slightly differ from the survey-design-based SE.
2. **Short time series**: Only 6 years (2019-2024) limits trend reliability and
   forecast precision.
3. **Crude estimates**: The dataset provides unadjusted ("crude") percentages; age-
   adjusted comparisons may differ.
4. **Survey design effects**: NHIS uses a complex sample design; aggregated estimates
   may not capture within-stratum variability.
5. **FLAG column**: Only a small number of rows carried data-quality flags and were
   excluded; the vast majority of estimates are unflagged.
6. **Linear model assumption**: Forecasts assume linear trends, which may not hold for
   structural changes (e.g., policy shifts, pandemics).

## 5. Figures

All figures are saved to the `figures/` directory:
- `*_trend.png` — total population trend lines with CI
- `*_subgroups.png` — top subgroup bar charts
- `*_spread.png` — estimate-spread box plots
- `friction_heatmap.png` — composite friction heatmap
- `friction_ranked.png` — ranked friction table
- `inequity_gaps.png` — gap trends over time
- `forecast_examples.png` — forecast examples with prediction intervals
