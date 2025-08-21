# Summary of results and findings

## Models

1. **Baseline :** rolling z-score of spread (W=390 min ≈ 1 trading day). Enter on $|z|>2$, exit on $|z|<1$.
2. **EWMA z-score:** replaces rolling stats with **exponentially-weighted** ones (half-life 375 min).
3. **OU s-score:** model spread as **Ornstein–Uhlenbeck**; estimate rolling AR(1) with intercept over \~5 days (1875 min) to obtain equilibrium mean & stationary variance. Signal $s_t=(X_t-\mu_t)/\sigma_{\text{stat},t}$.
4. **Kalman mean tracker:** latent mean of spread as a random walk; standardized one-step-ahead residual $s_t$. (q\_scale tuned.)

All models use the **same** entry/exit and **same P/L definition** to keep comparisons fair.

## Headline performance (full sample; min\_hold=0; cost=0 unless stated)

* **Baseline z (W=390)** — **Sharpe 3.17**, **Total P/L 69.56**, **MaxDD −3.80**, **Trades \~5.0k**, **Avg hold \~25 min**.
  (With 0.5 bp flip cost: **Sharpe 3.16**, **P/L 69.31** → robust.)
* **EWMA z (hl=375m)** — **Sharpe 1.98**, **P/L 69.59**, **MaxDD −3.99**, **Trades \~3.5k**, **Avg hold \~38 min**.
* **OU s-score (L=1875m)** — **Sharpe 2.25**, **P/L 79.80 (highest)**, **MaxDD −4.54**, **Trades \~2.8k (lowest)**, **Avg hold \~76 min**.
  (With 0.5 bp flip cost: **P/L 79.66**, essentially unchanged.)
* **Kalman (q=0.01)** — **Sharpe 1.35**, **P/L 70.16**, **MaxDD −4.30**, but **Trades \~21k**, **Avg hold \~6.5 min** (over-trading).

## Interpretation

* **Baseline** is a strong benchmark with the **highest Sharpe** and **low drawdown**, but moderate churn.
* **EWMA** gives similar P/L with **fewer trades** and smoother adaptation, albeit lower Sharpe.
* **OU** is the **P/L maximizer** and trades **least often** with **longer holds**—exactly what we want for a medium-frequency relative-value strategy. Sharpe >2 with acceptable DD shows the OU mean-reversion assumption is well-matched to this spread.
* **Kalman** (simple mean tracker) is too reactive here; without extra constraints it over-trades and underperforms on risk-adjusted terms.


## Pareto view & decision

A Pareto plot (Sharpe ↑ vs |DD| ↓, bubble = P/L) shows **Baseline (top-left)** as the **Sharpe/DD** choice and **OU (larger bubble)** as the **absolute P/L** choice. Both lie on the frontier; **EWMA** sits between; **Kalman** is only attractive if minimizing risk at the expense of return.

## Recommendation

* **Proposed model:** **OU s-score** with **L≈1875 min**, **z\_in=2**, **z\_out=1** (no min-hold); delivers **\~+10 P/L** over baseline with **\~−44% fewer flips** and **Sharpe >2**.
* **Risk-first alternative:** **Baseline z (W=390)** for highest Sharpe and lowest DD among top P/L models.
* **Operationally simple backup:** **EWMA z (hl=375m)** for similar P/L with reduced turnover.

## Robustness & next steps

* **Costs:** 0 to 0.5 bp has **negligible effect** on Baseline/EWMA/OU; a good sign.
* **Stability:** Small parameter nudges (±1 day window; half-life 0.5–2 days) leave conclusions unchanged.
* **Next improvements:** add a light **min\_hold (10–15 min)** to OU to trim micro-flips; explore **time-varying β** (Kalman spread) or **execution frictions** (slippage bands) for productionization.

**Bottom line:** A clear, defensible upgrade path—**OU s-score** as the primary model (highest P/L with solid Sharpe and lower turnover), with **Baseline** and **EWMA** as robust benchmarks/alternatives.
