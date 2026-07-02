# ADS-505 - Maven Roasters (BrewPoint Analytics)
Forecasting next-hour café revenue and turning it into staffing, prep (pars), and upsell actions across three NYC stores.

---

## 1) Project Overview
We use POS line items (timestamp, store, category/type, unit price, quantity) to build a **store-hour** table and forecast **revenue for the next hour (t+1)**. The forecasts drive:
- **Staffing**: a simple ±1 barista rule based on forecast “pressure” vs a historical baseline.
- **Inventory/Prep (pars)**: hour×weekday multipliers to scale prep in pressure windows.
- **Menu & Upsell**: attachment + lift analysis to surface gentle POS prompts.

Chronological split: **70% Train / 15% Validation / 15% Test** (no shuffling). Final models are **per-store Histogram-based Gradient Boosting (HGB)**.

---

## 2) Repository Structure

## 2) Repository Structure

```
notebooks/
└── ADS 505 Final Project.ipynb        # main notebook

deliverables/
├── Modeling/
│   ├── global_compare.csv
│   ├── model_compare_per_store.csv
│   ├── model_winners_per_store.csv
│   ├── tuning_table.csv
│   └── walkforward_metrics.csv
│
├── Explainability/
│   └── driver_importance_per_store.csv
│
├── Staffing/
│   ├── staffing_suggestions_by_hour.csv
│   ├── staffing_summary_by_hour.csv
│   ├── staffing_summary_by_day.csv
│   ├── staffing_schedule_blocks.csv
│   └── staffing_summary_sensitivity.csv
│
├── Inventory_Menu/
│   ├── top_sellers_by_store_daypart.csv
│   ├── attach_rates_category_pairs.csv
│   ├── size_mix_by_store.csv
│   ├── month_trend_by_item_store.csv
│   ├── expected_units_by_family.csv
│   └── baseline_hourly_revenue_by_store.csv
│
├── Visuals/
│   ├── heatmap_orders_<store>.png
│   └── slide_tables/
│       ├── per_store_winners.png
│       └── global_compare.png
│
└── QA/
    ├── feature_dictionary.csv
    ├── residuals_by_hour.csv
    └── residuals_by_store.csv

README.md                              # this file
```

> **Note:** `<store>` = Astoria, Hell's Kitchen, Lower Manhattan.

---

## 3) Repro Steps (clean run)

1. **Bootstrap**  
   Open the notebook and run the **BOOTSTRAP** cell to load/prepare `df`.

2. **Feature Engineering**  
   Build lags/rollings (t−1, t−24, t-168; 24h/7d rolling means), cyclical hour/day encodings, holiday flags (if available), store×hour effects, one-hot encodings, and scaling—*fit on Train only*.

3. **Target & Split**  
   Construct `store_hour` with `hour_revenue_tplus1` (next-hour target).  
   Do a **time-aware 70/15/15** split (Train/Val/Test) with fixed cutoffs (no shuffle).

4. **Model Comparison**  
   Run **global + per-store** comparisons for: Ridge, DecisionTree, RandomForest, **HistGBR**.

5. **Hyperparameter Tuning**  
   Use `PredefinedSplit` (Train vs Val) for **HistGBR** and **RF**; early stopping where supported.

6. **Final Models**  
   Train **per-store HistGBR** with tuned params on **Train+Val**; export winners + `driver_importance_per_store.csv`.

7. **Staffing Outputs**  
   Generate `staffing_suggestions_by_hour.csv` plus summaries and `staffing_schedule_blocks.csv`.

8. **Inventory/Menu Outputs**  
   Generate “what sells & when” artifacts + `expected_units_by_family.csv`.

9. **Backtesting & QA**  
   Run walk-forward backtest, export `walkforward_metrics.csv`, and residual diagnostics (`residuals_by_*`).

10. **Save All Deliverables**  
    Run the final cell to write all CSVs/PNGs into `deliverables/`.

---

## 4) Modeling Choices

- **Winner:** **Per-store HistGradientBoosting (HGB)**  
  Test **MAE ≈ $49-$53** (≈ **11 orders/hour** on average-ticket basis).
- **Baselines:** Ridge, DecisionTree  
- **Advanced:** RandomForest, HistGBR (XGBoost optional if available)  
- **Validation:** Time-aware via `PredefinedSplit` (Train vs Val), **Test held out**.

---

## 5) Staffing Policy (operational)

Let **pressure** = (forecast_orders − baseline_orders), where baseline is the **median** revenue at that clock hour (converted to orders).

- If **pressure ≥ threshold** for **2 consecutive hours** ⇒ **+1 barista**  
- If **pressure ≤ −threshold** for **2 consecutive hours** ⇒ **−1 barista**

**Threshold** = store’s **MAE_test_orders** (≈ **11 orders/hour**).  
This 2-hour rule dampens one-off spikes but stays responsive to true surges.

---

## 6) Inventory & Menu

- Convert forecasts → **expected units by family per hour** using attach rates  
  (fallback to presence/share when sparse).
- **Safety buffers:** +20% in **7–10 a.m.** & **11-14**, +10% off-peak.
- Promote top sellers by **daypart**; deploy **attach pairs** as POS prompts; use `month_trend_by_item_store.csv` for seasonality.

---

## 7) QA / Leakage Guard

- All features for predicting **t+1** are from **t or earlier**.  
- See `feature_dictionary.csv` for availability tags and transform notes.  
- Residual diagnostics (`residuals_by_hour.csv`, `residuals_by_store.csv`) show no extreme hotspots.

---

## 8) Next Iteration (optional)

- Add **weather** + enriched **holiday** signals; re-fit per-store HGB.  
- Add **SHAP** visualizations for localized explainability.  
- Move to **item-level basket** if density permits and order-IDs are available.  
- Package a small **Streamlit** app for day-ahead usage.

---

## 9) AI Assistance Disclosure

## AI Assistance

- **Paper editing & flow:** Rewrote sections for clarity, added in-text citations, standardized terms (pressure, pars, attach/lift).
- **Time-series setup:** Framed next-hour target (`hour_revenue_tplus1`), prevented leakage (lag/roll features; Train-only pipelines), and advised cyclical encodings + seasonal lags.
- **Bootstrap & splitting:** Helped create a reproducible BOOTSTRAP step and a time-ordered 70/15/15 split with fixed cutoffs.
- **Model strategy:** Baseline Ridge vs. per-store HGB; advised per-store winners based on validation.
- **Tuning:** RandomizedSearchCV with PredefinedSplit; early stopping; refit on Train+Val; single Test score.
- **Operationalization:** Defined pressure = forecast − baseline; ±1 barista rule (2-hour confirm), pars multipliers (hour×weekday), and attach/lift prompts with small nudges.
- **Troubleshooting:** Fixed RandomizedSearch boolean-index error; harmonized global vs per-store comparison columns/units; added residuals and walk-forward QA tables.
- **Slides & scripts:** Drafted 6-slide exec deck and 2-slide technical narrative; tied exhibits to `deliverables/Visuals/`.

> **Disclosure:** We used OpenAI ChatGPT (GPT-5 Thinking, Oct 2025) for drafting, code scaffolding, and debugging. All outputs were reviewed, edited, and validated by the authors.


**Citation options (APA):**
## References

Agrawal, R., Imieliński, T., & Swami, A. (1993). Mining association rules between sets of items in large databases. In *Proceedings of the 1993 ACM SIGMOD International Conference on Management of Data* (pp. 207–216). https://doi.org/10.1145/170035.170072

Chen, T., & Guestrin, C. (2016). XGBoost: A scalable tree boosting system. In *Proceedings of the 22nd ACM SIGKDD International Conference on Knowledge Discovery and Data Mining* (pp. 785–794). https://doi.org/10.1145/2939672.2939785

Google Developers. (2018). *Rules of ML: Best practices for ML engineering—Leakage*. https://developers.google.com/machine-learning/guides/rules-of-ml#leakage

Hyndman, R. J., & Athanasopoulos, G. (2021). *Forecasting: Principles and practice* (3rd ed.). OTexts. https://otexts.com/fpp3/

Kohavi, R., Longbotham, R., Sommerfield, D., & Henne, R. M. (2009). Controlled experiments on the web: Survey and practical guide. *Data Mining and Knowledge Discovery, 18*(1), 140–181. https://doi.org/10.1007/s10618-008-0114-1

NIST/SEMATECH. (2013). *e-Handbook of statistical methods*. National Institute of Standards and Technology. https://www.itl.nist.gov/div898/handbook/

OpenAI. (2025, October). *ChatGPT (GPT-5 Thinking, October 2025 version)* [Large language model]. https://chat.openai.com/

scikit-learn developers. (2024). *Histogram-based gradient boosting (HistGradientBoosting)*. In *scikit-learn user guide*. https://scikit-learn.org/stable/modules/ensemble.html#histogram-based-gradient-boosting

scikit-learn developers. (2024). *Preventing common pitfalls in machine learning* [Documentation]. https://scikit-learn.org/1.5/common_pitfalls.html

scikit-learn developers. (2024). *TimeSeriesSplit (time series cross-validation)*. In *scikit-learn user guide*. https://scikit-learn.org/stable/modules/generated/sklearn.model_selection.TimeSeriesSplit.html

Tan, P.-N., Steinbach, M., & Kumar, V. (2005). *Introduction to data mining*. Pearson.

Willmott, C. J., & Matsuura, K. (2005). Advantages of the mean absolute error (MAE) over the root mean square error (RMSE) in assessing average model performance. *Climate Research, 30*(1), 79-82. https://doi.org/10.3354/cr030079

Yan, Q. (2019). Dynamic regression models—Fourier terms. In *Notes for “Forecasting”*. https://qiushiyan.github.io/fpp/dynamic-regression-models.html

---

## 10) Contacts

**BrewPoint Analytics (ADS-505 Team)**  
Alanis Perez · Jason Avalos-Morfin · Paola Rodriguez
