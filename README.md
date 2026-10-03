# Medical Insurance Cost Analysis

A regression analysis of individual medical insurance charges, exploring which demographic and behavioral factors drive healthcare costs and by how much.

## Overview

This project models annual medical insurance charges using the well-known `insurance.csv` dataset (1,338 observations, introduced by Brett Lantz in *Machine Learning with R* and widely used on Kaggle). The goal is not just predictive accuracy but interpretability: the analysis is built step by step, from single-predictor regressions to a final model whose coefficients can be read and explained, and it is validated with a full diagnostic pass rather than taken at face value.

The response variable (`charges`) is strongly right-skewed, so the analysis works on `sqrt(charges)` rather than the raw dollar scale — a compromise chosen deliberately over the log scale, which would have compressed the high-cost tail too aggressively for what is, in an insurance context, the part of the distribution that matters most.

## Dataset

| Variable | Type | Description |
|---|---|---|
| `age` | numeric | Age of the primary beneficiary (18–64) |
| `sex` | categorical | `female` / `male` |
| `bmi` | numeric | Body mass index |
| `children` | numeric | Number of dependents covered |
| `smoker` | categorical | `yes` / `no` |
| `region` | categorical | US residential region (4 levels) |
| `charges` | numeric | Individual medical costs billed by insurance (response) |

No missing values; all 1,338 rows are used as-is.

## Methodology

The analysis follows a cumulative, chapter-by-chapter build-up:

1. **Exploratory analysis** — univariate and bivariate distributions, identifying `smoker` as the factor that separates costs most sharply.
2. **Response transformation** — comparing `charges`, `sqrt(charges)`, and `log(charges)` on skewness, tail behavior, and interpretability; `sqrt(charges)` is selected.
3. **Simple regressions** — one predictor at a time, ranked by R² (`smoker` ≈ 0.57, `age` ≈ 0.17, the rest much lower).
4. **Forward selection** (AIC and BIC) over the main effects.
5. **Interaction screening** — every numeric × categorical pair tested with an F-test; only `age:smoker` and `bmi:smoker` prove significant.
6. **Final multiple model** — `sqrt(charges) ~ sex + children + region + age*smoker + bmi*smoker`, compared against the additive model via ANOVA.
7. **Residual diagnostics** — normality (Shapiro-Wilk, Jarque-Bera) and heteroscedasticity (Breusch-Pagan) checks, followed by a Weighted Least Squares (WLS) re-fit.
8. **Quantile regression** — checking whether effects are stable across the cost distribution (10th to 90th percentile), not just at the mean.

## Key Findings

- **Smoking status is the dominant driver**, but its effect isn't just a fixed offset: it *changes the slope* of both `age` and `bmi`.
- Among non-smokers, `bmi` has almost no effect on cost; among smokers, each extra BMI point adds substantially more.
- Among smokers, cost grows *more slowly* with age than among non-smokers — smokers start from a higher baseline that grows less steeply.
- The final model explains about **83% of the variance** in `sqrt(charges)` (adjusted R²), with low multicollinearity (VIF/GVIF close to 1 throughout).
- Residuals show mild, borderline heteroscedasticity and heavy tails (expected, given a handful of very high-cost individuals) — WLS is used as the reference fit, though it only partially corrects the variance structure. With n = 1,338, the OLS estimator remains asymptotically valid regardless.
- Quantile regression confirms the interaction effects hold across the distribution, with `bmi:smoker` peaking around the third quartile and `children` mattering more for already high-cost individuals.

## Repository Contents

```
insurance-regression/
├── README.md
├── Progetto_finale.pdf   # full compiled report (Italian)
└── LICENSE
```

This repository contains the finished report only — the original R/R Markdown source used to produce it is not available, so there is no code to run here. The report itself documents every step (code excerpts, tables, and figures included) in full detail.

## Tools

R, with `ggplot2`, `dplyr`/tidyverse, `MASS` (stepwise AIC/BIC selection), `car` (VIF), `lmtest` (Breusch-Pagan), `moments` (skewness, Jarque-Bera), `quantreg`, and `knitr` for the report itself.

## Limitations

The data is observational and cross-sectional, so associations should not be read as causal effects. The predictor set is limited to six demographic/behavioral variables and excludes clinical history, diet, and physical activity — `smoker` likely absorbs some of that unobserved risk. The dataset is US-specific and not directly transferable to other healthcare systems.

## Author

Antonio Verde

## License

MIT — see [LICENSE](./LICENSE).
