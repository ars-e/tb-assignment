# Assumptions & Additional Commentary

## Data and preprocessing

* **Trading hours.** Data restricted to NSE session **09:15–15:30 IST** using the **XNSE** calendar. Out-of-session points are dropped.
* **Missing values.** We only forward-filled **small gaps (≤3 min)** to smooth occasional ticks; **large gaps (overnights/holidays)** were *not* filled. This preserves regime jumps at opens.
* **Index alignment.** Duplicate timestamps were removed; series sorted strictly by time. No resampling beyond calendar enforcement.
* **Units check.** We treat **TTE in days**; all models use $V_t = \text{spread}_t \cdot \text{TTE}_t^{0.7}$ per the assignment.

## Trading & P/L construction

* **Spread.** $X_t=\text{IV}_{BN,t}-\text{IV}_{N,t}$. We did **not** change the definition across models; when we standardize/estimate means we always do so on $X_t$.
* **Value series.** $V_t=X_t\cdot \text{TTE}_t^{0.7}$. If the spread definition were to change (e.g., time-varying β), we would recompute $V_t$ accordingly before P/L.
* **Execution timing.** Signals are evaluated at **t−1**; positions apply from **t** (one-bar delay → no look-ahead).
* **PnL mode.** **Delta P/L:** $\text{PnL}_t=\text{position}_{t-1}\cdot \Delta V_t$. This keeps all models comparable.
* **Direction & hysteresis.** Enter when $|z|>z_{\text{in}}$ (default 2.0), exit when $|z|<z_{\text{out}}$ (default 1.0). No take-profit/stop outside of this banding.
* **Hold across days.** We **do not force EOD flat** (medium-frequency horizon); an optional flag can close at 15:30 without materially changing results.
* **Costs.** Base results assume **0 bp**; robustness checks include **0.5 bp per flip**. No slippage/impact modeled—see “Limitations”.

## Model-specific assumptions

* **Baseline rolling z (W=390 min).** Assumes a *locally* stationary mean/variance over \~1 trading day. Sensitive to window choice; we cross-checked 360–1950 min.
* **EWMA z (half-life 375 min).** Assumes **exponentially decaying memory** is a better proxy for time-varying mean/variance than a hard window; captures volatility clustering explicitly.
* **Ornstein–Uhlenbeck (OU) s-score (L≈5 days).** Assumes the spread is **mean-reverting** with **continuous-time OU dynamics**. We estimate a rolling AR(1) with intercept to obtain $\mu_t$, $\phi_t$, residual σ, and use the **stationary variance** to scale the score. This is a principled standardization and implicitly adapts to changes in reversion speed.
* **Kalman mean tracker.** Models the **latent mean** of the spread as a random walk. Assumes measurement noise $R$ is roughly constant over the estimation window; **q\_scale** controls how quickly the mean can drift. With light regularization, it can over-react (we observed over-trading).

## Risk management & parameterization

* **Thresholds.** $z_{\text{in}}=2$, $z_{\text{out}}=1$ are canonical stat-arb levels; we verified nearby choices do not change conclusions.
* **Windows.** Chosen to match **intraday (≈1d)** and **weekly (≈5d)** dynamics: W=390 (baseline), EWMA half-life ≈1d, OU lookback ≈5d.
* **Min-hold.** Not required for the main results; adding **10–15 minutes** to OU slightly trims micro-flips with minimal P/L impact.
* **Capital scaling.** Results are in **“ΔV units”** (model P/L proxy). Converting to monetary P/L would require mapping to **real option exposures** (e.g., ATM straddles), margin, and financing.

## Robustness checks (what we did)

* **Costs & turnover.** 0 to 0.5 bp shows **negligible impact** on Baseline/EWMA/OU; high-churn Kalman is more sensitive.
* **Parameter stability.** Small nudges (±1 day window / half-life; OU lookback 2–8 days) preserve the model ranking: **OU highest P/L**, **Baseline highest Sharpe**, **EWMA in between**.
* **Train/test sanity (earlier run).** A simple time split and walk-forward confirmed OU/EWMA remain competitive OOS.


## Why our recommended model

* **OU s-score** delivered **highest absolute P/L** with **Sharpe > 2**, **acceptable drawdown (\~−4.5)**, and **lowest turnover** among top performers—well-suited to the assignment’s 30-minute to multi-day horizon.
* **Baseline** remains the **Sharpe/DD** champion and a strong benchmark.
* **EWMA** offers a **simpler, lower-churn** alternative with P/L close to baseline.

