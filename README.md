# GYTS Global Tobacco Indicators Pipeline

A Jupyter notebook pipeline that harmonizes Global Youth Tobacco Survey (GYTS) microdata from dozens of countries into a single pooled dataset, computes survey-weighted prevalence estimates for four tobacco-related indicators among 13–15 year olds, and fits a logistic regression on smoking susceptibility.

## What it does

1. **Recodes** each country's GYTS file into four standardized binary indicators.
2. **Pools** the recoded files into one global dataset (about 364,000 student records across 98 country-level surveys from 2012 to 2023, six WHO regions).
3. **Estimates** design-based prevalence and 95% confidence intervals for every country-year and indicator, using Taylor linearization with the survey strata and PSUs.
4. **Models** predictors of smoking susceptibility with a logistic regression.
5. **Plots** country, regional, and odds-ratio figures.

## Indicators

All indicators are restricted to students aged 13–15 (`CR1` coded 3–5, or 13–15 in files that use raw ages). Each is coded 1 = yes, 0 = no, and missing when the underlying item is absent or unanswered.

| Indicator | Meaning | Source variable(s) | Coding |
|-----------|---------|--------------------|--------|
| `CSMK` | Current cigarette smoking | `CR7` | 0 if response is 1, else 1 |
| `SHS` | Secondhand smoke exposure at home | `CR19` | 0 if response is 1, else 1 |
| `PROADV` | Exposure to pro-tobacco advertising | `CR35` | 1 if response is 2, 0 if response is 3 |
| `SUSCEP` | Susceptibility to smoking, among never-smokers only (`CR5 == 2`) | `CR39`, `CR40` | 0 if both are 1 ("definitely not"), else 1 |

Please check the item wording and response codes against the GYTS codebook for each survey year before reusing these definitions, since questionnaires vary across countries and years.

## Setup

The notebook was developed on Python 3.14 (Windows).

```bash
pip install pandas numpy samplics statsmodels matplotlib seaborn
```

Note that `samplics` is archived and prints a `FutureWarning` recommending [`svy`](https://pypi.org/project/svy/) as its successor.

**Data location:** the first cell defines `DATA_DIR = Path("GYTS_CSVs")`, a folder next to the notebook, and `CLEANED_DIR = DATA_DIR / "CLEANED"` for outputs. Put your GYTS CSVs in `GYTS_CSVs/` or change `DATA_DIR` to point elsewhere. Every read and write in the notebook uses these two variables, so run the first cell before any others.

## Running the notebook

Open `gyts-pipeline-clean.ipynb` and run the cells in order. The main stages are:

| Stage | What happens |
|-------|--------------|
| Prototype | Loads Bangladesh 2013 and builds and checks the indicators on one country |
| Batch recode | Defines `recode_gyts()` and loops over every `GYTS_*.csv`, assigning country, year, and WHO region |
| Cleanup | Fixes years parsed as 0 from filenames and assigns regions that were missing from the mapping |
| Pooling | Concatenates all frames and saves `global_pooled.csv` |
| Survey estimates | `weighted_estimate()` uses `samplics.TaylorEstimator` with `single_psu=skip` for strata with one PSU, and saves `survey_estimates.csv` |
| Regression | Builds complete cases, fits the logit, and saves odds ratios |
| Figures | Writes three PNGs |

The notebook redefines `recode_gyts()` and `weighted_estimate()` more than once as the logic was refined. The later definition in each case is the one that produced the saved outputs.

## Outputs

Written to `CLEANED_DIR` (`GYTS_CSVs/CLEANED/` by default):

| File | Contents |
|------|----------|
| `Bangladesh_2013_clean.csv` | Single-country prototype output |
| `global_pooled.csv` | Pooled, harmonized microdata |
| `survey_estimates.csv` | Prevalence, 95% CI, n, and CV per country, year, and indicator |
| `logistic_regression_results.csv` | Odds ratios, CIs, and p-values |
| `fig1_csmk_by_country.png` | Current smoking prevalence by country, colored by WHO region |
| `fig2_heatmap_by_region.png` | Regional mean prevalence for all four indicators |
| `fig3_odds_ratios.png` | Forest-style plot of regression odds ratios |

## Results snapshot

Simple unweighted-by-population means of country-level estimates per WHO region are in `fig2_heatmap_by_region.png`. Logistic regression of susceptibility among never-smokers (n = 179,938 complete cases):

| Predictor | Odds ratio | 95% CI |
|-----------|-----------|--------|
| Female (vs male) | 0.92 | 0.90–0.95 |
| Secondhand smoke at home | 1.37 | 1.33–1.41 |
| Pro-tobacco advertising | 1.56 | 1.52–1.60 |
