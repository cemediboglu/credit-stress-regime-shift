# Quantitative Post-Mortem: The Naive Forecast Trap & Structural Regime Shifts in US Credit Spread Modeling

An end-to-end quantitative research project exposing the statistical pitfalls of financial time series modeling—transitioning from autoregressive persistence traps to early-warning classification under structural macro regime shifts.

---

## 1. Macroeconomic Radar (The Triad)
This framework analyzes US corporate credit market dynamics using three key financial series:

* **`HY_Spread` (Target):** ICE BofA US High Yield Index Option-Adjusted Spread (FRED: `BAMLH0A0HYM2`). Measures the risk premium required by investors to hold non-investment grade corporate debt over risk-free US Treasuries.
* **`Yield_Curve_10Y2Y` (Leading Indicator):** 10-Year minus 2-Year US Treasury yield spread (FRED: `T10Y2Y`). Inversions (negative spreads) serve as historical indicators of impending economic contraction.
* **`VIX` (Concurrent Risk Sentiment):** CBOE Volatility Index (`^VIX`). Serves as a market proxy for short-term implied volatility, equity panic, and liquidity constraints.

---

## 2. Data Pipeline & Integrity Constraints
* **Coverage Constraints:** While a 10-year horizon was queried, upstream API limitations on the free continuous FRED OAS feed yielded an effective, dense observation window spanning **3 years (~755 trading days)**.
* **Calendar Synchronization:** Mismatched trading holidays between bond markets (FRED) and equity options (Yahoo Finance) were resolved via forward-fill (`ffill()`) to maintain temporal alignment without lookahead bias.
* **Stationarity Verification:** The target series was evaluated using the Augmented Dickey-Fuller (ADF) test, confirming statistical stationarity ($p = 0.0099 < 0.05$).

---

## 3. Empirical Experiments & Modeling Insights

### Experiment A: The Naive Regression Trap (Level Forecasting)
* **Design:** Chronological 80/20 train/test split with zero data leakage, using lagged features ($t-1, t-5, t-10$) and 30-day rolling statistics (Moving Average, Volatility, Z-Score).
* **Finding:** Yielded low error metrics ($MAE = 0.0423$ bps, $RMSE = 0.0571$ bps). However, feature importance was overwhelmingly dominated by `HY_Spread_lag1` ($86.66\%$).
* **Diagnostic:** The gradient boosted tree defaulted to an autoregressive persistence strategy ($y_t \approx y_{t-1}$), producing an illusion of predictive power while remaining blind to structural market turning points.

### Experiment B: Early Warning Classification (Spike Detection)
* **Design:** Reframed the target into a binary classification problem predicting daily spread widening exceeding the 80th percentile of positive changes (Stress Spikes).
* **Finding:** Successfully eliminated the `lag1` dominance, elevating leading indicators such as `HY_Spread_ZScore_lag1`, `Yield_Curve`, and `VIX`.
* **Diagnostic:** Revealed out-of-sample performance degradation caused by extreme class imbalance ($7.9\%$ positive rate) and structural shifts across the test window.

---

## 4. Key Takeaways & Market Reality: The Non-Stationarity Dilemma

* **Historical Data Is Not a Static Oracle:** Statistical correlations established during the training window (2023–2025) detached during the 2026 test period. Relationships learned in stable phases can fail under altered macro and liquidity conditions.
* **Daily Noise vs. Macro Signal:** 1-day forward forecast horizons are dominated by idiosyncratic corporate events and high-frequency noise rather than structural economic momentum.
* **Institutional Model Governance:** Production-grade credit risk models cannot operate as static systems. Robust deployment requires walk-forward rolling retraining, wider forward horizons (5- to 20-day momentum), and regime-detection overlays to account for shifting market environments.

---

## 5. Setup & Reproduction
```bash
python -m venv .venv
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
pip install -r requirements.txt
