# What Drives Google Play App Installs?

An exploratory and regression analysis of Google Play Store app listings in python (pandas, statsmodels). The goal is to find which app attributes are associated with higher install counts, and how much each one explains.

**Notebook:** `apps_pricing_analysis.ipynb`  |  **Status:** work in progress

## Question

Which observable app attributes (rating, price, free vs. paid, category) are associated with higher install counts, and how much of the variation in installs do they explain?

## Data

- Google Play Store app listings (`googleplaystore.csv`) — [add source link here]
- Fields used: `App`, `Category`, `Rating`, `Reviews`, `Installs`, `Type`, `Price`
- **8,195 apps** have the fields needed for modeling (apps with no rating are excluded)

## Cleaning

| Step | Why |
|---|---|
| Removed the malformed row | One row in the file has shifted columns (non-numeric install and price values) |
| Parsed `Installs` (e.g. `"10,000+"`) and `Price` (e.g. `"$4.99"`) from text to numbers | Needed for modeling |
| Removed duplicate apps, keeping the listing with the most reviews | Some apps appear several times |
| Used `log(1 + installs)` as the outcome | Installs are heavily right-skewed, and 15 apps have 0 installs, so `log(installs)` is undefined |

## Method

1. Exploratory step: correlations and a category-level summary (app count, median installs, median rating, share of paid apps)
2. Fit a sequence of nested OLS regressions on `log(1 + installs)` using `statsmodels`
3. Compare models with adjusted R² and F-tests for nested models
4. Convert coefficients to percentage effects with `exp(coef) − 1`

## Results

| Model | Adjusted R² |
|---|---|
| Rating + Price | 0.010 |
| + Category (32 dummies vs. FAMILY) | 0.102 |
| Rating + Type (free/paid) + Category | 0.167 |
| + Type × Category interaction | 0.170 |

**Findings**

- **Rating and price alone explain about 1% of the variation in installs.** Both are statistically significant (p < 0.001), but the effect on explanatory power is small. Holding price fixed, +1 rating point is associated with about **+78%** installs (95% CI: roughly +53% to +106%). Each extra $1 of price is associated with about **−1.2%** installs.
- **Category matters far more.** Adding it lifts adjusted R² from 0.010 to 0.102 (joint F-test p ≈ 5 × 10⁻¹⁵⁴). Compared with FAMILY apps, the typical (geometric-mean) install count is much higher for ENTERTAINMENT, PHOTOGRAPHY and GAME, and much lower for MEDICAL, EVENTS and BUSINESS.
- **Free vs. paid explains more than the dollar amount.** Replacing linear price with a free/paid indicator raises adjusted R² from 0.102 to 0.167.
- **The Type × Category interaction is significant but small** (F-test p = 0.0003; adjusted R² gain of 0.003).

**Takeaway:** which category an app is in, and whether it charges at all, tell you much more about installs than its rating or exact price.

## Limitations

- `Installs` is reported in bins (10,000+, 500,000+, ...), so the outcome is a lower bound, not an exact count. OLS on its log is an approximation.
- Association, not causation. Popular apps may attract better ratings, and the reverse.
- Large category effects (e.g. ENTERTAINMENT, 87 apps) come from small groups and heavy-tailed data.
- The Type × Category design matrix is rank-deficient (some categories have no paid apps), so interaction estimates are not uniquely determined.
- Apps without a rating are dropped. Reviews, Size and Content Rating are not yet modeled.

## Next steps

- Residual diagnostics and robust standard errors
- Add Content Rating, Size and app age
- Collapse rare categories to fix the rank-deficient interaction model
- Try an ordinal model, since installs are binned

## Reproduce

```terminal
pip install pandas numpy statsmodels jupyter
```

Put `googleplaystore.csv` in the repo folder, update the file path in the first cell, and run `apps_pricing_analysis.ipynb` top to bottom (Python 3.12).
