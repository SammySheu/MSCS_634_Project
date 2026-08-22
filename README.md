# MSCS 634 — Project Deliverable 4

## Final Insights, Recommendations, and Presentation
### Factors Associated with Five-Year Home Price Growth and Housing Price Prediction in California

**Name:** Chun-Shuo Sheu
**Course:** MSCS 634 — Big Data and Data Mining
**Deliverable:** 4 of 4 — Final Insights, Recommendations, and Presentation

---

## What this deliverable contains

| Artifact | Path |
|---|---|
| Consolidated notebook (the whole project in one narrative pass) | `notebooks/Project_Deliverable_4.ipynb` |
| Comprehensive written report, 15 sections | `report/REPORT.md` · `report/REPORT.pdf` |
| Presentation deck, 7 slides, no code | `slides/slides.html` · `slides/slides.pdf` |
| Timed narration script (907 words ≈ 6:23) | `slides/SPEAKER_NOTES.md` |
| 17 figures | `figures/` |
| The dataset D1 built, and D2's 2024 forecast frame | `data/processed/` |

> **Still to do: the recording.** The deck, the script and the timing are ready; the 5–7 minute
> video itself has to be recorded live. `slides/SPEAKER_NOTES.md` ends with the setup steps.

D4 introduces **no new data and no new modeling technique.** Everything here was computed in
D1–D3; this deliverable assembles it into one argument, adds the ethics analysis, and presents it.

---

## The two research questions

1. **Which factors are associated with five-year home price growth in California?**
2. **Given current conditions, what will homes be worth in five years (2029)?**

## The answers, in one paragraph

Cross-sectional ZIP-code features **cannot** predict the *level* of California home-price growth.
On an unseen base year every model — Ridge, Random Forest, Gradient Boosting — posts a **negative
test R²**, losing to a constant, and 65–81% of that error is a single level shift: the models
predicted a boom-era average that 2020 did not deliver. But their **ranking survives**. After each
year's statewide median is subtracted from the target, Random Forest and Gradient Boosting reach
**test R² ≈ 0.12 with Spearman ≈ 0.41** on a year they never saw. The features that carry that
signal are **affordability** (price-to-income, price-to-rent, mortgage-payment-to-income — all
negative) and **urban scale** (population, positive). Four methods with essentially no shared
assumptions — penalized regression, logistic regression, K-Means that never saw the outcome, and
Apriori — independently place affordability at the centre. The median California ZIP code is
forecast at **\$723k (2024) → ≈\$1.02M (2029), +39.5%**, with an 80% interval of **\$700k–\$976k**
that sits *entirely below* the point estimate, because the last time this model met an unseen year
it over-predicted by 23 points.

---

## Headline results

### Regression — notebook §4, report §5

| Model | CV RMSE | Val R² (2019) | **Test R² (2020)** | Test R², relative target |
|---|---|---|---|---|
| Baseline (mean) | 22.33 | −0.27 | **−1.88** | −0.01 |
| Ridge | 17.76 | −0.22 | **−4.51** | −0.02 |
| Random Forest | 18.06 | −0.05 | **−2.85** | **+0.12** |
| Gradient Boosting | 17.71 | −0.00 | **−3.23** | **+0.12** |

Mean realized growth: train 47.96% → val 38.15% → **test 24.31%**. That collapse *is* the
negative R².

### Classification — notebook §5, report §6

| | Absolute label | Within-year label |
|---|---|---|
| Positive rate, 2020 test | 2.0% (30 of 1,528) | 25.0% by construction |
| Majority-baseline accuracy | **98.04%** | 75.0% |
| Tuned RF test F1 | 0.089 | **0.406** |
| Logistic test F1 | 0.023 | **0.440** |
| Tuned RF test ROC-AUC | 0.793 | 0.719 |

**Accuracy is never used as a headline metric** — a classifier that says "no ZIP code is ever
high-growth" scores 98%.

### Clustering — notebook §6, report §7 — five market types, ranked by relative growth

| Segment | n (2020) | Median value | Median income | vs. own-year median | Top-quartile rate |
|---|---|---|---|---|---|
| Affordable Working-Class Markets | 254 | \$284k | \$46k | **+6.0 pts** | 39.4% |
| Affluent Growing Suburbs | 366 | \$822k | \$106k | −0.1 pts | 25.2% |
| Mid-Market Suburban California | 659 | \$463k | \$68k | −1.4 pts | 19.2% |
| Elite Coastal & Tech Markets | 120 | \$1.70M | \$159k | −5.8 pts | 22.2% |
| **Price-Detached Urban Markets** | 129 | \$1.06M | \$68k | **−15.8 pts** | 12.4% |

Price-Detached ranks **last in all five** five-year windows. ANOVA F = 27.49, p = 4.6 × 10⁻²².

### Association rules — notebook §7, report §8

26 rules survive (support ≥ 0.05, confidence ≥ 0.50, lift ≥ 1.20), lift 1.52–1.87.
`PriceToIncome` appears in **9 of 9** low-growth rules and **7 of 17** high-growth rules, with
opposite direction — but the tails are **not** symmetric:

- `{PriceToIncome=High} → {FutureGrowth=Low}` — lift **1.414**, alone, over 1,177 ZIP-years.
- `{PriceToIncome=Low} → {FutureGrowth=High}` — lift only **1.225**, and needs low income plus high
  unemployment before lift reaches 1.87.

*Unaffordability alone predicts underperformance. Affordability alone does not predict
outperformance.*

### The synthesis — notebook §8, report §9

Classified by a rule stated before the answer was looked at — sign-consistency across all five base
years, Ridge |coefficient| rank, and grouped (collinearity-safe) permutation importance:

| Category | Variables | Evidence |
|---|---|---|
| **Primary** | `mortgage_to_income_ratio`, `price_to_income_ratio`, `price_to_rent_ratio`, `population` | Ridge |coef| ranks 1, 2, 3, 6 of 13; sign-consistent in all five base years; the only two blocks with positive grouped permutation importance on both held-out years |
| **Secondary** | `home_value_1yr_growth_pct`, `bachelor_degree_pct`, `unemployment_rate`, `home_value_3yr_growth_pct`, `median_gross_rent`, `population_growth_pct` | Large in one method but **sign flips across base years**, or sign-stable yet tiny |
| **Weak** | `rent_to_income_ratio`, `current_home_value`, `median_household_income` | Low and unstable in both methods |
| **Excluded** | `mortgage_rate`, `estimated_monthly_mortgage` | Constant within a base year — impurity rank 2 of 15, **permutation importance exactly 0.000** on the test year |

---

## Ethical considerations — summary

The full treatment is report §12 (notebook §10.3). The three points that matter most:

1. **ZIP code is a well-documented proxy for race and class in the United States.** A model keyed on
   ZIP-level income, education and home value can reproduce racial patterns with **no race variable
   anywhere in it**. Removing race from the feature list does not remove race from the model; it
   removes the ability to measure what the model is doing.
2. **Historical housing data carries the imprint of redlining.** The target itself — realized
   appreciation — is an outcome shaped by decades of discriminatory lending. A model trained on
   those outcomes learns the pattern discrimination produced and projects it forward.
3. **This model must never be used for lending, insurance, or tenant screening.** Predicting that a
   ZIP code will underperform and then acting on it by withholding credit is *precisely the
   mechanism of redlining, restated in the language of machine learning.* Legitimate uses are
   descriptive and aggregate.

Also covered: aggregation is not anonymity (and the ecological fallacy), non-random missingness as
a **fairness** problem rather than a data problem, ACS sampling error, ZHVI as an index rather than
a transaction record, and forecast fragility.

---

## Known limitations

1. **Overlapping outcome windows** — 7,487 rows are **not** 7,487 independent observations.
   Mitigated by temporal splits and `GroupKFold` everywhere; plain `KFold` appears nowhere.
2. **All five outcome windows contain the pandemic housing boom.** The model can only learn "what
   predicted growth during a boom." No amount of cross-validation fixes this.
3. **The 2029 forecast requires extrapolation** — training rates span 3.11–4.55%, the 2024 input is
   6.72%, outside the range for 100% of rows. Reported with and without the rate features.
4. **The mortgage rate is national** — five effective data points, confounded with `base_year`.
5. **ZIP ≠ ZCTA**, and the ACS ZCTA vintage changes in 2020.
6. **ACS 5-year estimates** are averages with sampling error and a built-in lag.
7. **ZORI is 78% missing, non-randomly** — ACS median gross rent is used instead.
8. **ZHVI is an index, not a transaction record.**
9. **Listings coverage is non-random and correlated with the outcome** — covered ZIPs have ~4× the
   median population and grew ~10 points faster.
10. **Clusters describe a continuum, not natural kinds** — mean silhouette ≈ 0.23.
11. **Association is not causation.** Nothing here identifies a causal effect.

---

## Reproducing

```bash
pip install -r requirements.txt

# The two processed inputs are committed:
#   data/processed/ca_zip_housing_panel_clean.csv     (D1's output, 7,487 x 28)
#   data/processed/ca_zip_forecast_frame_2024.csv     (D2's output, 1,543 ZIPs)
# To rebuild them from raw sources, see Deliverable 1's and Deliverable 2's READMEs.

python scripts/make_deliverable_4_notebook.py
jupyter nbconvert --execute --inplace notebooks/Project_Deliverable_4.ipynb

# Report and slides (needs Google Chrome for PDF export)
python scripts/build_documents.py
```

The notebook is **self-contained**: it imports nothing from `scripts/`, loads only the two CSVs
above, and writes all 17 figures into `figures/`. Every model is seeded with `random_state=42`, so
the run is reproducible; runtime is about three minutes.

---

## Repository layout

```
notebooks/Project_Deliverable_4.ipynb      the consolidated deliverable (run top to bottom)
scripts/make_deliverable_4_notebook.py     regenerates the notebook from source
scripts/build_documents.py                 renders report/ and slides/ to PDF
report/REPORT.md, REPORT.html, REPORT.pdf  the 15-section written report
slides/slides.html, slides.pdf             7-slide deck, no code
slides/SPEAKER_NOTES.md                    timed narration script
data/processed/                            the panel, the forecast frame, the 2029 forecast
figures/                                   17 figures, all regenerated by the notebook
```

## Figures

| File | Content |
|---|---|
| `fig01_target_distribution.png` | Target pooled and by base year — the regime shift |
| `fig02_affordability.png` | Price-to-income vs. growth, and the within-year check |
| `fig03_correlation_heatmap.png` | Full correlation matrix, target boxed |
| `fig04_target_correlations.png` | Every feature's r with the target |
| `fig05_outliers.png` | Flagged outliers at both extremes, and flag counts by year |
| `fig06_model_comparison.png` | CV / validation / test RMSE across four regressors |
| `fig07_regime_shift.png` | Residual distributions, predicted vs. actual by year, the rate gap |
| `fig08_relative_target.png` | What removing the macro level does to test R² |
| `fig09_feature_importance.png` | Ridge coefficients, per-feature permutation, grouped permutation |
| `fig10_label_prevalence.png` | Positive class collapsing 42.6% → 2.0% |
| `fig11_ranking_vs_calibration.png` | Probability drift, decile lift, both labels compared |
| `fig12_confusion_roc.png` | Confusion matrices and ROC curves, within-year label |
| `fig13_pca_clusters.png` | PCA projection with loadings, and the same space by outcome |
| `fig14_cluster_growth.png` | Segment growth by base year, relative growth, top-quartile rate |
| `fig15_association_rules.png` | Top rules by lift, and the support–confidence scatter |
| `fig16_synthesis.png` | Primary / secondary / weak classification, and block importance |
| `fig17_forecast.png` | 2029 forecast in context, by affordability quartile, top 12 |

---

## The four deliverables

| | Scope | Repo |
|---|---|---|
| D1 | Data collection, cleaning, exploration → the 7,487-row panel | `MSCS_634_ProjectDeliverable_1` |
| D2 | Regression, evaluation, the 2029 forecast | `MSCS_634_ProjectDeliverable_2` |
| D3 | Classification, clustering, pattern mining | `MSCS_634_ProjectDeliverable_3` |
| **D4** | **Consolidation, insights, ethics, presentation** | **`MSCS_634_Project`** |
