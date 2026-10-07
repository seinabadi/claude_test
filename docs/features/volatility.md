# Volatility and range

Back to the [catalog index](../feature_catalog.md) for notation. Daily volatility is annualized by $\sqrt{252}$. Range estimators use $u_t = \ln(H_t/O_t)$, $d_t = \ln(L_t/O_t)$, $c_t = \ln(C_t/O_t)$ and $o_t = \ln(O_t/C_{t-1})$. All variance estimators below are averaged over $n$ days and reported as the square root.

## Close-to-close and model-based volatility

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Realized volatility | Sample standard deviation of daily log returns, annualized. Use the demeaned (sample) version; the zero-mean version $\sqrt{\frac{1}{n}\sum r^2}$ is slightly more efficient at short windows. | $\sigma_n(t) = \sqrt{\dfrac{1}{n-1}\sum_{i=0}^{n-1}(r_{t-i} - \mu_n(r)_t)^2}\,\sqrt{252}$ | $n = 5, 10, 21, 63, 126, 252$ |
| Realized variance components (HAR-RV) | Corsi (2009): daily, weekly and monthly averages of squared returns, the regressors of the HAR model. | $RV^{(d)}_t = r_t^2$; $RV^{(w)}_t = \frac{1}{5}\sum_{i<5} r_{t-i}^2$; $RV^{(m)}_t = \frac{1}{22}\sum_{i<22} r_{t-i}^2$ | 1, 5, 22 |
| HAR-RV forecast | One-step forecast from the HAR regression fit on a rolling window (in logs or square roots for stability). With daily data only, $RV^{(d)}_t = r_t^2$ is noisy; a per-day range estimator (Parkinson or Yang-Zhang) is a better daily proxy. | $\widehat{RV}_{t+1} = \hat\beta_0 + \hat\beta_d RV^{(d)}_t + \hat\beta_w RV^{(w)}_t + \hat\beta_m RV^{(m)}_t$ | fit on 1000 days |
| Volatility surprise | Realized variance today against what the HAR model predicted yesterday. | $RV^{(d)}_t - \widehat{RV}_{t \mid t-1}$, divided by $\widehat{RV}_{t \mid t-1}$ | |
| HAR extensions (regressors) | Andersen-Bollerslev-Diebold HAR-RV-J adds the jump component; Patton-Sheppard HAR-RS splits the daily term into positive and negative semivariance; Corsi-Reno LHAR adds a leverage term. Each extra regressor is a feature in its own right. | $J_t = \max(RV_t - BV_t, 0)$; $RS^{\pm}_t$; $r_t\mathbb{1}[r_t < 0]$ (daily, weekly and monthly means) | 1, 5, 22 |
| MIDAS volatility | Ghysels, Santa-Clara and Valkanov (2006): a long weighted sum of past squared or absolute returns with hump-shaped Beta-polynomial weights; distinct from EWMA because the weight profile is flexible. | $\hat\sigma^2_t = \sum_{k=0}^{K-1} w_k(\theta_1, \theta_2)\, x_{t-k}$, $w_k \propto (k/K)^{\theta_1 - 1}(1 - k/K)^{\theta_2 - 1}$, $x \in \{r^2, \lvert r \rvert, \ln^2(H/L)\}$ | $K = 250$; $(\theta_1, \theta_2) = (1, 5)$ and $(1, 20)$ |
| EWMA volatility | RiskMetrics exponentially weighted variance; $\lambda = 0.94$ is the RiskMetrics daily setting (half-life 11 days), $0.97$ is their monthly setting used here simply as a slower filter. | $\hat\sigma^2_t = \lambda\hat\sigma^2_{t-1} + (1-\lambda) r_t^2$, seeded with the sample variance of the first 25 returns; feature $\sqrt{252\,\hat\sigma^2_t}$ | $\lambda = 0.94, 0.97$ |
| GARCH(1,1) forecast | Bollerslev (1986): one-day-ahead conditional variance from a GARCH(1,1) refit on a rolling window; captures clustering with mean reversion to the long-run variance. | $\hat\sigma^2_{t+1} = \omega + \alpha\,\varepsilon_t^2 + \beta\,\hat\sigma^2_t$, $\varepsilon_t = r_t - \mu$; long-run $\bar\sigma^2 = \omega/(1 - \alpha - \beta)$; $h$-step $\hat\sigma^2_{t+h} = \bar\sigma^2 + (\alpha + \beta)^{h-1}(\hat\sigma^2_{t+1} - \bar\sigma^2)$; 21-day cumulative $\sum_{h=1}^{21}\hat\sigma^2_{t+h}$ for comparison with IV30 | QMLE, refit every 21 days on 500 to 1000 days |
| GARCH persistence and half-life | How long shocks last. | $\hat\alpha + \hat\beta$; $\ln 0.5 / \ln(\hat\alpha + \hat\beta)$ | |
| GJR-GARCH leverage term | Asymmetric GARCH: extra variance after negative shocks. | $\hat\sigma^2_{t+1} = \omega + (\alpha + \gamma\,\mathbb{1}[\varepsilon_t < 0])\varepsilon_t^2 + \beta\hat\sigma^2_t$; feature $\hat\gamma$ and the forecast | same |
| GARCH standardized residual | Today's return in units of the GARCH conditional volatility: a surprise measure. | $\varepsilon_t / \hat\sigma_t$ | |
| Volatility forecast ratio | GARCH forecast against trailing realized vol: whether the model expects vol to rise. | $\hat\sigma_{t+1}\sqrt{252} / \sigma_{21}(t)$ | |

## Range-based estimators

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Parkinson | Parkinson (1980): uses the high-low range; about 5 times more efficient than close-to-close under zero drift and no gaps. | $\sigma^{P}_n = \sqrt{\dfrac{1}{4 n \ln 2}\sum_{i<n}\big[\ln(H_{t-i}/L_{t-i})\big]^2}$ | $n = 10, 21$ |
| Garman-Klass | Garman and Klass (1980): adds the open-close move to the range; this is the authors' recommended practical form, which nearly every library implements (the original minimum-variance form uses coefficients 0.511, 0.019, 0.383 with a cross term). | $\sigma^{GK}_n = \sqrt{\dfrac{1}{n}\sum_{i<n}\Big[\tfrac{1}{2}(u - d)^2_{t-i} - (2\ln 2 - 1)\,c^2_{t-i}\Big]}$, $2\ln 2 - 1 \approx 0.3863$ | 10, 21 |
| Meilijson | Meilijson (2011): unbiased zero-drift OHLC estimator with slightly lower variance than Garman-Klass (efficiency 7.73 vs 7.4). | Reflect so the body is non-negative: if $c < 0$ use $(c', u', d') = (-c, -d, -u)$. $\hat\sigma^2_1 = 2[(u' - c')^2 + d'^2]$, $\hat\sigma^2_2 = c'^2$, $\hat\sigma^2_3 = 2(u' - c' - d')c'$, $\hat\sigma^2_4 = -(u' - c')d'/(2\ln 2 - 5/4)$; $\hat\sigma^2_M = 0.2735\,\hat\sigma^2_1 + 0.1604\,\hat\sigma^2_2 + 0.3652\,\hat\sigma^2_3 + 0.2009\,\hat\sigma^2_4$, averaged over $n$ days | 10, 21 |
| Rogers-Satchell | Rogers and Satchell (1991): drift-independent. | $\sigma^{RS}_n = \sqrt{\dfrac{1}{n}\sum_{i<n}\big[u(u - c) + d(d - c)\big]_{t-i}}$ | 10, 21 |
| Yang-Zhang | Yang and Zhang (2000): combines overnight variance, open-to-close variance and Rogers-Satchell; handles gaps and drift, the most efficient daily estimator. Several web sources put $k$ on the wrong terms; the paper's weights are $1$ on overnight, $k$ on open-to-close, $1-k$ on Rogers-Satchell. | $\sigma^2_{o} = \frac{1}{n-1}\sum(o_{t-i} - \bar o)^2$; $\sigma^2_{c} = \frac{1}{n-1}\sum(c_{t-i} - \bar c)^2$; $\sigma^2_{YZ} = \sigma^2_o + k\,\sigma^2_c + (1 - k)\,(\sigma^{RS}_n)^2$, $k = \dfrac{0.34}{1.34 + (n+1)/(n-1)}$ | 10, 21 |
| Garman-Klass-Yang-Zhang | Garman-Klass with an explicit overnight term. | $\sigma^2 = \frac{1}{n}\sum\big[o^2 + \tfrac{1}{2}(u - d)^2 - (2\ln 2 - 1)c^2\big]$ | 10, 21 |
| Range-based vs close-based ratio | Parkinson over close-to-close: above 1 means intraday reversals, below 1 means trending days. | $\sigma^P_{21}/\sigma_{21}$ | 21 |
| Overnight share of variance | Share of total variance that arrives overnight. | $\dfrac{\operatorname{var}_n(o)}{\operatorname{var}_n(o) + \operatorname{var}_n(c)}$ | 21, 63 |
| Gap volatility | Standard deviation of overnight returns, annualized. | $\operatorname{std}_n(o)\sqrt{252}$ | 21, 63 |
| Intraday volatility | Standard deviation of open-to-close returns. | $\operatorname{std}_n(c)\sqrt{252}$ | 21, 63 |

## True range and ATR

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| ATR | Wilder's average true range; the range including gaps, Wilder-smoothed. | $ATR_k(t) = RMA_k(TR)_t$ | $k = 5, 14, 21$ |
| Normalized ATR | ATR as a share of price, comparable across assets. | $NATR_t = ATR_{14}(t)/C_t$ | 14 |
| ATR percentile and z-score | ATR against its own one-year history. | $\operatorname{prank}_{252}(NATR)_t$; $z_{252}(NATR)_t$ | 252 |
| True range ratio | Today's true range against the ATR; a range-expansion measure. | $TR_t / ATR_{14}(t-1)$ | 1 |
| ATR change | Short ATR against long ATR. | $ATR_5(t)/ATR_{21}(t) - 1$ | 5, 21 |
| Daily log range | High-low log range and its trailing means. | $\ln(H_t/L_t)$; $\mu_5$, $\mu_{21}$ | 1, 5, 21 |
| Range expansion | Today's range against the 20-day mean range. | $(H_t - L_t)/\mu_{20}(H - L)_t$ | 20 |
| Range percentile | Rank of today's range in the trailing window. | $\operatorname{prank}_{63}(\ln(H/L))_t$ | 63 |
| Narrow-range flags | NR4 and NR7, see price action. | $\mathbb{1}[(H - L)_t = \min_n(H - L)_t]$ | 4, 7 |
| Chaikin volatility | Rate of change of the EMA of the range. | $CV_t = 100\,\Big(\dfrac{EMA_{10}(H - L)_t}{EMA_{10}(H - L)_{t-10}} - 1\Big)$ | 10, 10 |
| Mass index | Dorsey: sum of the ratio of a single to a double EMA of the range; a "reversal bulge" above 27. | $MI_t = \sum_{i<25}\dfrac{EMA_9(H - L)_{t-i}}{EMA_9(EMA_9(H - L))_{t-i}}$ | 9, 25 |
| Ulcer index | Root mean square of percentage drawdowns from the window high; a downside-only volatility. | $UI_k(t) = \sqrt{\dfrac{1}{k}\sum_{i<k}\Big(100\,\dfrac{C_{t-i} - \max_{k}(C)_{t-i}}{\max_k(C)_{t-i}}\Big)^2}$ | 14, 63 |

| Elder's market thermometer | Larger of the outside-the-prior-bar moves of the high and the low; spikes mark panic or breakouts. | $Therm_t = \max(\lvert H_t - H_{t-1} \rvert, \lvert L_{t-1} - L_t \rvert)$ (the larger one only); $Therm_t/C_t$; $Therm_t/EMA_{20}(Therm)$ | 20 |
| Price distance | Total path a bar travelled including the gap, not just its range. | $PD_t = 2(H_t - L_t) + \lvert O_t - C_{t-1} \rvert - \lvert C_t - O_t \rvert$; $PD_t/C_t$; $PD_t/ATR_{14}$ | 14 |
| Robust scale and z-score | Median absolute deviation of returns and the robust z-score of today's return. | $MAD_k = \operatorname{median}\lvert r_i - \operatorname{median}_k(r) \rvert$; $z^{rob}_t = (r_t - \operatorname{median}_k(r))/(1.4826\,MAD_k)$ | 21, 63 |
| Keltner and Donchian width | Channel widths relative to price, with their one-year percentile; range-based regime gauges alongside Bollinger bandwidth. | $2 m\,ATR_{10}/EMA_{20}(C)$; $(\max_{20} H - \min_{20} L)/C_t$; $\operatorname{prank}_{252}$ of each | 20, $m = 2$ |

## Volatility regimes and dynamics

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Volatility ratios | Short against long realized vol; above 1 means vol is rising. | $\sigma_5/\sigma_{21}$; $\sigma_{21}/\sigma_{63}$; $\sigma_{21}/\sigma_{252}$ | pairs |
| Volatility term-structure slope | Slope of realized vol across windows. | OLS slope of $\sigma_k$ on $\ln k$ for $k \in \{5, 21, 63, 252\}$ | fixed |
| Volatility percentile | Rank of current 21-day vol over one and two years. | $\operatorname{prank}_{252}(\sigma_{21})_t$; $\operatorname{prank}_{504}(\sigma_{21})_t$ | 252, 504 |
| Volatility z-score | 21-day vol against its one-year mean and std. | $z_{252}(\sigma_{21})_t$ | 252 |
| Log volatility change | Change in 21-day vol over 5 days. | $\ln(\sigma_{21}(t)/\sigma_{21}(t-5))$ | 5 |
| Volatility of volatility | Standard deviation of the log 21-day vol series. | $\operatorname{std}_{63}(\ln\sigma_{21})_t$ | 63 |
| Volatility autocorrelation | Persistence of absolute returns. | $\operatorname{corr}_{63}(\lvert r_t \rvert, \lvert r_{t-1} \rvert)$ | 63 |
| Volatility regime flag | Low, normal or high vol by percentile thresholds. | $-1$ if $\operatorname{prank}_{252}(\sigma_{21}) < 0.2$; $+1$ if $> 0.8$; else 0 | 252 |
| Bollinger squeeze | Bollinger bands inside the Keltner channel: volatility compression (Carter's TTM squeeze). | $\mathbb{1}[SMA_{20} + 2\operatorname{std}_{20}(C) < EMA_{20} + 1.5\,ATR_{20} \wedge SMA_{20} - 2\operatorname{std}_{20}(C) > EMA_{20} - 1.5\,ATR_{20}]$ | 20, 1.5 |
| Squeeze duration | Consecutive days in a squeeze. | count | |
| Bollinger bandwidth percentile | Bandwidth against its one-year range. | $\operatorname{prank}_{252}(BW)_t$ | 252 |
| Large-move count | Days with $\lvert r \rvert > 2\sigma$ in the window. | $\sum_{i<k}\mathbb{1}[\lvert r_{t-i} \rvert > 2\sigma_{21}(t-i-1)]$ | 21, 63 |
| Days since last large move | Trading days since the last 2-sigma day. | count | current |
| Jump count | Days with $\lvert r \rvert > 3\sigma$. | analogous | 63, 252 |

## Asymmetry and partial-variance measures

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Downside semi-deviation | Downside deviation around a target of zero with the full-window denominator (the Sortino convention), not the dispersion of the down-day subsample. | $\sigma^-_n = \sqrt{\dfrac{1}{n}\sum_{i<n}\min(r_{t-i}, 0)^2}\sqrt{252}$ | 21, 63 |
| Upside semi-deviation | Same for positive returns. | $\sigma^+_n = \sqrt{\dfrac{1}{n}\sum_{i<n}\max(r_{t-i}, 0)^2}\sqrt{252}$ | 21, 63 |
| Up-down vol ratio | Upside over downside semi-deviation; above 1 means the big moves have been up. | $\sigma^+_{63}/\sigma^-_{63}$ | 63 |
| Realized semivariance share | Barndorff-Nielsen, Kinnebrock and Shephard (2010): share of variance from negative returns. | $RS^-_n/(RS^-_n + RS^+_n)$ with $RS^{\pm}_n = \sum_{i<n} r_{t-i}^2\mathbb{1}[\pm r_{t-i} > 0]$ | 21, 63 |
| Signed jump variation | Patton and Sheppard (2015): positive minus negative semivariance; "bad" volatility (negative SJ) predicts higher future volatility. | $SJ_n = RS^+_n - RS^-_n$; $SJ^+ = SJ\,\mathbb{1}[SJ > 0]$, $SJ^- = SJ\,\mathbb{1}[SJ < 0]$ | 5, 21 |
| Relative signed jump | Bollerslev, Li and Zhao (2020): signed jump variation scaled by total variance, computed over the past week; high RSJ predicts lower returns. | $RSJ_n = \dfrac{RS^+_n - RS^-_n}{RS^+_n + RS^-_n}$ | 5, 21 |
| Realized skewness and kurtosis | Amaya, Christoffersen, Jacobs and Vasquez (2015): non-demeaned higher moments of returns; realized skewness predicts lower next-week returns, realized kurtosis higher. Intraday version uses the $M$ five-minute returns of a day, averaged over the week; the daily-data version uses the last $k$ daily returns. | $RSkew = \dfrac{\sqrt{M}\sum_j r_j^3}{(\sum_j r_j^2)^{3/2}}$; $RKurt = \dfrac{M\sum_j r_j^4}{(\sum_j r_j^2)^2}$ (replace $M$ by $k$ and $r_j$ by daily $r_{t-i}$ for the daily proxy) | $k = 21$; demeaned sample versions in [statistical_risk.md](statistical_risk.md) |
| Bipower variation and jumps | Barndorff-Nielsen and Shephard (2004): a jump-robust variance estimate from products of adjacent absolute returns; the gap to realized variance measures the jump contribution. A cleaner jump proxy than counting 3-sigma days. | $BV_n = \dfrac{\pi}{2}\cdot\dfrac{n}{n-1}\sum_{i<n-1}\lvert r_{t-i} \rvert\lvert r_{t-i-1} \rvert$; $RV_n = \sum_{i<n} r^2_{t-i}$; $J_n = \max(RV_n - BV_n, 0)$; relative jump $RJ_n = J_n / RV_n$ | 21, 63 |
| Intraday jump statistics (if 5-minute data is available) | Huang-Tauchen jump test and the median realized variance of Andersen, Dobrev and Schaumburg (2012). | $Z_t = \dfrac{RJ_t}{\sqrt{((\pi/2)^2 + \pi - 5)\,\frac{1}{M}\max(1, TQ_t/BV_t^2)}}$; $MedRV = \dfrac{\pi}{6 - 4\sqrt 3 + \pi}\cdot\dfrac{M}{M-2}\sum \operatorname{med}(\lvert r_{j-1} \rvert, \lvert r_j \rvert, \lvert r_{j+1} \rvert)^2$; realized quarticity $RQ = \frac{M}{3}\sum r_j^4$ | daily, from intraday bars |
| Absolute and squared return lags | Recent magnitudes as separate columns. | $\lvert r_{t-j} \rvert$, $r_{t-j}^2$ | $j = 1, \dots, 5$ |
| Leverage-effect correlation | Correlation between returns and subsequent changes in volatility; negative for equities. | $\operatorname{corr}_{126}(r_t, \ln\sigma_{5}(t+5) - \ln\sigma_5(t))$ computed only with data up to $t$ (so the latest usable pair ends at $t$) | 126 |
| Implied minus realized | Variance risk premium proxy: 30-day implied vol minus 21-day realized vol; see the options file. | $IV^{30}_t - \sigma_{21}(t)$ | fixed |
