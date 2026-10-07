# Feature catalog

Candidate model inputs for a next-day forecasting model on 11 US single stocks and the 11 Select Sector SPDR ETFs. Every feature is a function of data available at or before the close of trading day $t$, computed from the asset's own daily OHLCV, calendar, fundamental, flow and options data. External series (indexes, rates, macro) are listed separately in [Indexes and external data](indexes_and_external_data.md).

The catalog is split into one file per family. Each row gives the feature, a plain-language definition of what it measures and why it might carry information, the exact formula, the default parameters, and an evidence grade with its source (see [Evidence grades](#evidence-grades)). Formulas use the shared notation below. Row counts are table rows; most rows define a family of columns (one per window or parameter), so the catalog describes roughly 2,000 candidate columns from about 790 feature definitions.

| File | Family | Rows |
| --- | --- | --- |
| this file | Targets and shared notation | 16 |
| [returns_momentum.md](features/returns_momentum.md) | Returns, momentum, reversal, seasonality, relative strength, formulaic alphas | 88 |
| [trend.md](features/trend.md) | Moving averages, trend-following overlays, channels, regression, price transforms | 96 |
| [oscillators.md](features/oscillators.md) | Bounded and unbounded momentum oscillators | 59 |
| [volatility.md](features/volatility.md) | Realized and range-based volatility, ATR, vol regimes | 63 |
| [volume_liquidity.md](features/volume_liquidity.md) | Volume, money flow, liquidity and spread estimators | 62 |
| [price_action.md](features/price_action.md) | Candles, gaps, breakouts, pivots, patterns | 63 |
| [statistical_risk.md](features/statistical_risk.md) | Beta, correlation, higher moments, drawdown, memory, entropy, time-series shape statistics, structural breaks, market state, regimes | 117 |
| [calendar.md](features/calendar.md) | Calendar, seasonality and scheduled-event timing | 29 |
| [stock_fundamentals.md](features/stock_fundamentals.md) | Earnings, estimates, valuation, quality, ownership, short interest, text (stocks only) | 84 |
| [etf.md](features/etf.md) | Flows, NAV, holdings breadth and dispersion, rotation (ETFs only) | 34 |
| [options.md](features/options.md) | Implied volatility, skew, term structure, positioning, model-free moments | 45 |
| [references.md](features/references.md) | Full citations for every source named in the Evidence columns | |

## Shared notation

| Symbol | Meaning |
| --- | --- |
| $O_t, H_t, L_t, C_t, V_t$ | Open, high, low, close and share volume on trading day $t$, split- and dividend-adjusted (total-return series) |
| $r_t = \ln(C_t / C_{t-1})$ | Daily close-to-close log return |
| $r_t^{(k)} = \ln(C_t / C_{t-k}) = \sum_{i=0}^{k-1} r_{t-i}$ | $k$-day log return |
| $o_t = \ln(O_t / C_{t-1})$, $c_t = \ln(C_t / O_t)$ | Overnight (close-to-open) and intraday (open-to-close) log returns, so $r_t = o_t + c_t$ |
| $TP_t = (H_t + L_t + C_t)/3$, $MP_t = (H_t + L_t)/2$ | Typical price and median (mid) price |
| $TR_t = \max(H_t - L_t,\ \lvert H_t - C_{t-1} \rvert,\ \lvert L_t - C_{t-1} \rvert)$ | True range |
| $SMA_k(x)_t = \frac{1}{k}\sum_{i=0}^{k-1} x_{t-i}$ | Simple moving average over the last $k$ days |
| $EMA_k(x)_t = \alpha x_t + (1-\alpha)\,EMA_k(x)_{t-1}$, $\alpha = 2/(k+1)$ | Exponential moving average, seeded with $SMA_k$ on the first full window |
| $RMA_k(x)_t = \frac{1}{k} x_t + (1 - \frac{1}{k})\,RMA_k(x)_{t-1}$ | Wilder's smoothing (an EMA with $\alpha = 1/k$), seeded with $SMA_k$ |
| $WMA_k(x)_t = \frac{\sum_{i=0}^{k-1}(k-i)\,x_{t-i}}{k(k+1)/2}$ | Linearly weighted moving average, most weight on the latest value |
| $\mu_k(x)_t$, $\operatorname{std}_k(x)_t$ | Rolling sample mean and sample standard deviation (divisor $k-1$) over the last $k$ days |
| $\sigma_k(t) = \operatorname{std}_k(r)_t$ | Trailing $k$-day daily volatility; annualized as $\sigma_k \sqrt{252}$ |
| $\max_k(x)_t$, $\min_k(x)_t$ | Rolling maximum and minimum over the last $k$ days, inclusive of $t$ |
| $z_k(x)_t = (x_t - \mu_k(x)_t)/\operatorname{std}_k(x)_t$ | Rolling z-score of $x$ against its own trailing window |
| $\operatorname{prank}_k(x)_t$ | Percentile rank of $x_t$ within $\{x_{t-k+1}, \dots, x_t\}$, in $[0, 1]$ |
| $\operatorname{xrank}(x)_t$ | Cross-sectional rank of $x_t$ among the 11 assets on day $t$, rescaled to $[0, 1]$ |
| $\operatorname{corr}_k(x, y)_t$, $\operatorname{cov}_k(x, y)_t$ | Rolling Pearson correlation and covariance over $k$ days |
| $\beta_k(x \mid y)_t = \operatorname{cov}_k(x, y)_t / \operatorname{var}_k(y)_t$ | Rolling OLS slope of $x$ on $y$ |
| $\mathbb{1}[\cdot]$, $\operatorname{sgn}(\cdot)$ | Indicator function (1 if true, else 0) and sign function |
| $r^{M}_t$, $r^{S}_t$ | Daily log return of SPY (market) and of the asset's own sector ETF |
| $\Delta_k x_t = x_t - x_{t-k}$ | $k$-day difference (Kakushadze's `delta`); $x_{t-k}$ is `delay` |

Conventions: every rolling statistic uses only $\{x_{t-k+1}, \dots, x_t\}$. Where a library (TA-Lib, pandas-ta) differs from the original author's definition, the row says which one is written. Indicators expressed in price units are always divided by $C_t$ or by $ATR$ so that they are comparable across assets and time.

## Targets (what to forecast)

Pick one primary label; the others work as auxiliary tasks, sanity checks or sizing inputs. $h$ is the horizon in trading days and $\sigma_t$ is the trailing volatility known at $t$ (default $\sigma_{21}(t)$).

| Target | Definition | Formula | Notes |
| --- | --- | --- | --- |
| Next-day return | Log change in the adjusted close from today's close to tomorrow's; the default regression label. | $y_t = r_{t+1} = \ln(C_{t+1}/C_t)$ | Use log returns so multi-day labels add |
| Next-day direction | Sign of tomorrow's return; optionally three classes with a dead zone so that noise days near zero are not forced into up or down. | $y_t = \operatorname{sgn}(r_{t+1})$; 3-class: $+1$ if $r_{t+1} > \delta\sigma_t$, $-1$ if $r_{t+1} < -\delta\sigma_t$, else $0$ | $\delta \in [0.1, 0.25]$; weight classes by inverse frequency |
| Volatility-scaled return | Tomorrow's return in units of today's known volatility; stabilises the label variance across calm and turbulent regimes. | $y_t = r_{t+1} / \sigma_t$ | Also the natural target for a vol-targeted strategy |
| Open-to-close return | Return earned by a trade entered at tomorrow's open and closed at tomorrow's close. | $y_t = c_{t+1} = \ln(C_{t+1}/O_{t+1})$ | Matches execution at the open |
| Overnight return | Gap from today's close to tomorrow's open; the decision must be made before the open. | $y_t = o_{t+1} = \ln(O_{t+1}/C_t)$ | Driven by news and global markets |
| Excess return vs sector ETF | Return beyond what the sector ETF explains, using a trailing beta; for relative-value or sector-neutral trading. | $y_t = r_{t+1} - \beta^{S}_{63}(t)\, r^{S}_{t+1}$ | $\beta$ estimated on data up to $t$ |
| Excess return vs SPY | Beta-hedged alpha against the market. | $y_t = r_{t+1} - \beta^{M}_{63}(t)\, r^{M}_{t+1}$ | Same estimation rule |
| Next-day range | Tomorrow's high-low log range; a volatility and position-sizing label. | $y_t = \ln(H_{t+1}/L_{t+1})$ | Always positive; model $\ln y$ |
| Next-day realized volatility | Magnitude of tomorrow's move; with intraday data, the root of the summed squared 5-minute returns. | Daily proxy $\lvert r_{t+1} \rvert$; intraday $RV_{t+1} = \sqrt{\sum_{j} r_{t+1,j}^2}$ | Intraday RV is far less noisy |
| Multi-day forward return | Return over the next $h$ days; less noise than a one-day label at the cost of overlapping observations. | $y_t = r^{(h)}_{t+h} = \ln(C_{t+h}/C_t)$ | $h = 2, 5, 10, 21$; purge $h$ days in CV |
| Forward Sharpe-like return | Forward return divided by the realized volatility over the same horizon; rewards smooth moves. | $y_t = \ln(C_{t+h}/C_t) / \big(\operatorname{std}(r_{t+1..t+h}) \sqrt{h}\big)$ | $h = 5, 21$ |
| Triple-barrier label | $+1$, $-1$ or $0$ according to which of a profit-taking barrier, a stop-loss barrier or a vertical time barrier is touched first; mirrors how a position with stops is actually closed. | Barriers at $C_t e^{+a\sigma_t}$ and $C_t e^{-b\sigma_t}$, time limit $h$; label by the first path touch using $H$ and $L$ | $a = b = 2$, $h = 5$ to $10$; from Lopez de Prado (2018) |
| Meta-label | 1 if the primary model's proposed trade (side known) ended profitable, else 0; a second model then sizes or filters trades. | $y_t = \mathbb{1}[\operatorname{side}_t \cdot r^{(h)}_{t+h} > 0]$ | Train on the primary model's out-of-sample signals |
| Trend-scaling label | Lopez de Prado's trend label: sign of the $t$-statistic of a regression of price on time over the forward window with the highest $\lvert t \rvert$. | $y_t = \operatorname{sgn}\big(\hat t_{\hat h}\big)$, $\hat h = \arg\max_{h \in H} \lvert \hat t_h \rvert$, $\hat t_h$ = $t$-stat of the slope of $\ln C$ on $\{1..h\}$ over $t+1..t+h$ | $H = \{5, \dots, 20\}$ |
| Forward max favorable and adverse excursion | Best and worst paths over the next $h$ days; inputs for stop and target placement. | $MFE = \ln(\max_{1 \le i \le h} H_{t+i} / C_t)$, $MAE = \ln(\min_{1 \le i \le h} L_{t+i} / C_t)$ | $h = 5, 10$ |
| Forward drawdown flag | Whether the path falls more than $d$ before the horizon; a risk-aware auxiliary target. | $y_t = \mathbb{1}[\min_{1 \le i \le h} \ln(L_{t+i}/C_t) < -d\,\sigma_t]$ | $d = 2$, $h = 10$ |

## Evidence grades

Every row in the family files carries an **Evidence** column: a grade and the source that defines the feature or documents its use. Full citations are in [references.md](features/references.md). The grades are:

| Grade | Meaning | Rows |
| --- | --- | --- |
| **A** | A peer-reviewed paper documents that the feature predicts returns (or volatility, where stated) out of sample, usually in a monthly cross-section of US stocks. | 188 |
| **B** | A validated estimator or model from econometrics or statistics. The source proves it measures what it claims (efficiency, consistency), not that it predicts returns. | 91 |
| **C** | A classical technical indicator with a named author, book or library source. Definitions are reliable; the academic evidence on profitability is mixed to negative. | 225 |
| **D** | A generic time-series descriptor or a construction derived for this catalog (ratios, z-scores, percentiles, flags, windows). Standard feature-engineering practice, not a published finding. | 235 |

One row cross-references another file. Counts by family:

| Family | A | B | C | D |
| --- | --- | --- | --- | --- |
| Stock fundamentals | 69 | 5 | 0 | 10 |
| Returns and momentum | 30 | 1 | 2 | 55 |
| Statistical and risk | 25 | 44 | 3 | 45 |
| Options | 20 | 5 | 10 | 10 |
| Calendar | 15 | 1 | 0 | 13 |
| ETF | 13 | 0 | 5 | 15 |
| Volume and liquidity | 9 | 10 | 25 | 18 |
| Volatility | 5 | 24 | 10 | 24 |
| Price action | 2 | 0 | 39 | 22 |
| Trend | 0 | 1 | 79 | 16 |
| Oscillators | 0 | 0 | 52 | 7 |

What the grades imply for this project:

1. **Grade A evidence was produced on a different problem.** Almost all of it comes from monthly or weekly cross-sectional sorts over hundreds or thousands of stocks spanning decades. Nothing here has been shown to work on next-day returns of 22 liquid large caps and sector ETFs. Expect effect sizes to be far smaller at the daily horizon and in a universe this narrow.
2. **Published effects decay.** McLean and Pontiff (2016) find anomaly returns fall by about a third after publication and more than half after academic publication plus trading. Harvey, Liu and Zhu (2016) argue most published factors fail a multiple-testing hurdle; Hou, Xue and Zhang (2020) fail to replicate the majority of 452 anomalies at conventional significance; Jensen, Kelly and Pedersen (2023) are more optimistic but still find modest post-publication returns. Treat Grade A as "worth testing first", not "known to work".
3. **Technical indicators have the weakest record.** Brock, Lakonishok and LeBaron (1992) found moving-average and range-breakout rules profitable on the Dow through 1986; Sullivan, Timmermann and White (1999) showed the result does not survive data-snooping adjustment or the post-1986 sample. Park and Irwin (2007) survey 95 studies: positive results concentrate in older or less rigorous work. Marshall, Young and Rose (2006) find candlestick patterns have no value on Dow stocks. The honest case for Grade C rows is that they are cheap nonlinear transforms of recent prices that a model may use or ignore.
4. **Grade B measures are reliable as measurements.** Range-based volatility estimators, GARCH and HAR, variance ratios, structural-break tests and spread estimators do what they claim. Volatility forecasts in particular are well validated and feed directly into position sizing and the volatility-scaled targets; their value as return predictors is indirect.
5. **Grade D rows are hypotheses.** Nobody has shown that, for example, the five-day change in the 21-day Sharpe ratio predicts anything. They are included because tree models can exploit such transforms cheaply, and they should be the first candidates to prune.

A defensible first model uses the Grade A rows that are computable from the available data, plus a handful of Grade B volatility measures and the calendar flags, and adds Grade C and D rows only when a purged cross-validation shows they help.

## Rules for using the list

1. Every feature at row $t$ uses only data available at the close of day $t$. Lag fundamentals to the filing date plus one day, macro to its release timestamp, options to the same-day close.
2. Adjust prices for splits and dividends with a total-return series. Splits in this set: AMZN 20:1 (Jun 2022), GOOGL 20:1 (Jul 2022), WMT 3:1 (Feb 2024), NEE 4:1 (Oct 2020); GE spun off GE HealthCare (Jan 2023) and GE Vernova (Apr 2024), so its history needs spin-off adjustment. Use unadjusted prices only for features that depend on the actual traded level (round numbers, strike distances, nominal gaps).
3. Use returns, ratios, distances and ranks, never raw price or volume levels. Fractional differentiation keeps memory while making a series stationary.
4. Scale with trailing-only statistics: rolling z-scores over 252 days, or cross-sectional ranks across the 11 assets each day. Never fit a scaler on the full sample.
5. Validate with walk-forward or purged k-fold cross-validation with an embargo of at least the label horizon. Random k-fold on daily data leaks.
6. Expect weak signals: a next-day direction model at AUC 0.53 to 0.56 out of sample is doing well. Check calibration and trade only above a confidence threshold.
7. Many features are near-duplicates (every oscillator is a function of a few lagged returns). Start with 50 to 80 covering each family, then prune by permutation importance, SHAP, or clustered feature importance measured under the purged CV.
8. Evaluate the strategy on net returns with spread and commission, against buy-and-hold and a 50/200 crossover baseline.
9. Limit the search: every extra feature set, horizon and threshold tried inflates the best backtest. Report the deflated Sharpe ratio or the number of trials.
10. Record lineage for each feature: source, download date, adjustment method, and the first date the data was actually available.
11. Recursive indicators (EMA, RMA, ADX, SAR, Supertrend, KAMA) depend on their seed. Warm them up on at least $5k$ bars before the first row used for training so that the seed does not leak into the features.
12. When an indicator's denominator can be zero ($H_t = L_t$, zero volume, zero variance), define the feature as 0 or carry the prior value forward, and add a flag column; do not drop the row.
