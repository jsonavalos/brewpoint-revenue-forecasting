# ADS-505 - Maven Roasters (BrewPoint Analytics)
End-to-end workflow for forecasting next-hour café revenue, staffing recommendations, and inventory/menu insights across three NYC stores.

## 1) Project Structure
- `notebooks/` ADS 505 Final Project
- `deliverables/`
  - **Modeling:** `global_compare.csv`, `model_compare_per_store.csv`, `model_winners_per_store.csv`, `tuning_table.csv`, `walkforward_metrics.csv`
  - **Explainability:** `driver_importance_per_store.csv`
  - **Staffing:** `staffing_suggestions_by_hour.csv`, `staffing_summary_by_hour.csv`, `staffing_summary_by_day.csv`, `staffing_schedule_blocks.csv`, `staffing_summary_sensitivity.csv`
  - **Inventory/Menu:** `top_sellers_by_store_daypart.csv`, `attach_rates_category_pairs.csv`, `size_mix_by_store.csv`, `month_trend_by_item_store.csv`, `expected_units_by_family.csv`, `baseline_hourly_revenue_by_store.csv`
  - **Visuals:** `heatmap_orders_<store>.png`, `slide_tables/per_store_winners.png`, `slide_tables/global_compare.png`
  - **QA:** `feature_dictionary.csv`, `residuals_by_hour.csv`, `residuals_by_store.csv`
- This `README.md`

## 2) Repro Steps (clean run)
1. **Open notebook** and run the **BOOTSTRAP** cell to load/prepare `df`.
2. Run **feature engineering** → lags/rollings, cyclical hour/day, holiday, store×hour, OHE, scaling.
3. Build `store_hour` with **t+1** target and do **time-aware 70/15/15** split.
4. Run **model comparison** (global + per-store: Ridge, DecisionTree, RandomForest, HistGBR).
5. Run **hyperparameter tuning** with `PredefinedSplit` (HistGBR & RF).
6. Train **final per-store HistGBR**; export **winners** and **driver_importance**.
7. Generate **staffing_suggestions** (+ summaries & schedule blocks).
8. Generate **what sells & when** assets + **expected_units_by_family**.
9. Run **walk-forward backtest** and **residual diagnostics**.
10. Run the “**Save All Deliverables**” cell.

## 3) Modeling Choices
- **Per-store HistGradientBoosting** (HGB) wins: Test MAE ≈ $49–$53 (~11 orders/hr).
- Baselines: Ridge, DecisionTree; Advanced: RandomForest, HistGBR; (XGBoost optional).
- **Time-aware CV** via `PredefinedSplit` (Train vs Val), Test held out.

## 4) Staffing Policy (operational)
If `(forecast_orders − baseline_orders) ≥ threshold` for **2 consecutive hours** ⇒ **+1 barista**;  
if `≤ −threshold` for **2 hours** ⇒ **−1 barista`.  
Threshold = store’s **MAE_test_orders** (~11 orders/hr).

## 5) Inventory & Menu
- Convert forecasts → expected units by family per hour using **attach rates** (fallback to presence/share).  
- Safety: **+20%** in 7-10a & 11–14; **+10%** off-peak.  
- Promote **top sellers by daypart**; use **attach pairs** for POS prompts; use **month trends** for seasonal features.

## 6) QA / Leakage Guard
- Features are **t or earlier** for predicting **t+1**.  
- See `feature_dictionary.csv` for availability tags.  
- Residuals by hour/store confirm no extreme hotspots.

## 7) Next Iteration (optional)
- Add **weather** + enriched **holiday** features; re-fit per-store HGB.  
- SHAP visualizations.  
- Item-level basket if density permits.

## 8) Contacts
BrewPoint Analytics (ADS 505 Team): Alanis Perez, Jason Avalos Morfin, Paola Rodriguez

