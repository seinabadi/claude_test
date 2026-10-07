# Returns, momentum and relative strength

Back to the [catalog index](../feature_catalog.md) for notation. All returns are log returns on the adjusted close unless stated. $k$ is the lookback in trading days; $r^{M}$ and $r^{S}$ are the SPY and sector-ETF returns.

## Raw and lagged returns

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| $k$-day log return | Cumulative log price change over the window; the basic momentum (long $k$) or reversal (short $k$) input. | $r^{(k)}_t = \ln(C_t / C_{t-k})$ | $k = 1, 2, 3, 5, 10, 21, 42, 63, 126, 252$ |
| Lagged daily returns | Each of the last ten daily returns as its own column so the model can learn its own short-horizon filter instead of a fixed window. | $r_{t-j}$ | $j = 1, \dots, 10$ |
| Simple return | Arithmetic return, used where the model consumes percentages or for exact compounding checks. | $R^{(k)}_t = C_t / C_{t-k} - 1 = e^{r^{(k)}_t} - 1$ | $k = 1, 5, 21$ |
| Overnight return | Close-to-open gap return, plus lags and trailing sums; overnight and intraday returns have different drivers and different persistence (Lou, Polk and Skouras 2019). | $o_t = \ln(O_t / C_{t-1})$; $\sum_{i=0}^{k-1} o_{t-i}$ | lags 1 to 3; sums $k = 5, 21$ |
| Intraday return | Open-to-close return, plus lags and trailing sums. | $c_t = \ln(C_t / O_t)$; $\sum_{i=0}^{k-1} c_{t-i}$ | lags 1 to 3; sums $k = 5, 21$ |
| Overnight minus intraday | Difference of cumulative overnight and intraday returns; a persistent positive gap signals a "tug of war" between overnight and day-session traders. | $\sum_{i=0}^{k-1} o_{t-i} - \sum_{i=0}^{k-1} c_{t-i}$ | $k = 21, 63, 252$ |
| Overnight share of return | Fraction of the window's total return earned overnight. | $\sum_{i<k} o_{t-i} \big/ \sum_{i<k} \lvert r_{t-i} \rvert$ | $k = 63, 252$ |
| Average overnight and intraday return | Lou, Polk and Skouras (2019): the past year's mean overnight return predicts future overnight returns positively and intraday returns negatively, and vice versa; use as separate inputs for the overnight and open-to-close targets. | $\mu_{252}(o)_t$; $\mu_{252}(c)_t$ | 252 |
| Nominal price | Unadjusted share price level (Miller and Scholes 1982); tick-size and round-lot effects. | $\ln P_t$ | 1 |
| Close-to-high and close-to-low returns | Where today closed relative to today's extremes, in return units. | $\ln(C_t/H_t)$, $\ln(C_t/L_t)$ | 1 |
| Return relative to typical price | Close against the day's typical price; positive when the close sits in the upper part of the day. | $\ln(C_t / TP_t)$ | 1 |

## Momentum

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Momentum 12-1 | Classic Jegadeesh-Titman momentum: the return from 12 months ago to 1 month ago, skipping the most recent month to avoid short-term reversal. | $MOM^{12,1}_t = \ln(C_{t-21} / C_{t-252})$ | fixed |
| Momentum 6-1 | Intermediate-horizon version. | $MOM^{6,1}_t = \ln(C_{t-21} / C_{t-126})$ | fixed |
| Intermediate momentum | Novy-Marx (2012): the return from months 12 to 7 predicts better than months 6 to 2. | $\ln(C_{t-126} / C_{t-252})$ | fixed |
| Other skip-month momenta | Jegadeesh-Titman 3-1 and 9-1 variants. | $\ln(C_{t-21}/C_{t-63})$; $\ln(C_{t-21}/C_{t-189})$ | fixed |
| Long-term reversal | De Bondt and Thaler (1985): returns over years 2 to 3 and 2 to 5 reverse. | $\ln(C_{t-252}/C_{t-756})$; $\ln(C_{t-252}/C_{t-1260})$ | fixed |
| Momentum change | Gettleman and Marks (2006): six-month return minus the prior six-month return. | $\ln(C_{t-21}/C_{t-126}) - \ln(C_{t-126}/C_{t-252})$ | fixed |
| Sector momentum level | Moskowitz and Grinblatt (1999): the asset's own sector ETF 12-1 return as a predictor in its own right, not only relative to the asset. | $\ln(C^S_{t-21}/C^S_{t-252})$; also $\ln(C^S_t / C^S_{t-252})$ | fixed |
| Annual return seasonality | Heston and Sadka (2008): the return in the same calendar month in prior years predicts this month's return. Daily analogue: the 21-day return centred one, two to five, and six to ten years ago. | $seas^{1}_t = \ln(C_{t-242}/C_{t-263})$; $seas^{2..5}_t = \frac{1}{4}\sum_{y=2}^{5}\ln(C_{t-252y+10}/C_{t-252y-11})$; $seas^{6..10}_t$ analogously; non-annual version: mean 21-day return over the same years excluding the annual windows (negative predictor) | 1, 5, 10 years |
| Same-weekday seasonality | Keloharju, Linnainmaa and Nyberg (2016): mean return on the same weekday over the last year minus the mean on other weekdays. | $\mu(r \mid \text{same weekday})_{252} - \mu(r \mid \text{other weekdays})_{252}$ | 252 |
| Price delay | Hou and Moskowitz (2005): share of return variation explained by lagged market returns; slow incorporation of information predicts higher returns. | $D1 = 1 - \dfrac{R^2_{restricted}}{R^2_{full}}$, full: $r_t = a + \sum_{j=0}^{5} b_j r^M_{t-j} + \varepsilon$, restricted: $b_{1..5} = 0$ | 252 |
| Momentum excluding last week | Skips only the last 5 days; useful at daily frequency where the reversal is a week, not a month. | $\ln(C_{t-5} / C_{t-k})$ | $k = 63, 126, 252$ |
| Time-series momentum sign | Moskowitz-Ooi-Pedersen trend signal: sign of the trailing return. | $\operatorname{sgn}(r^{(k)}_t)$ | $k = 21, 63, 126, 252$ |
| Volatility-scaled return | Trailing return in units of its expected noise; makes windows and assets comparable. | $r^{(k)}_t / \big(\sigma_{21}(t)\sqrt{k}\big)$ | $k = 1, 5, 21, 63$ |
| Rolling Sharpe ratio | Annualized mean over standard deviation of daily returns; a risk-adjusted momentum measure. | $SR_k(t) = \dfrac{\mu_k(r)_t}{\operatorname{std}_k(r)_t}\sqrt{252}$ | $k = 21, 63, 126, 252$ |
| Trend strength (t-statistic) | One-year return divided by its standard error; the $t$-stat of the mean daily return. | $TS_t = \dfrac{\ln(C_t / C_{t-252})}{\sigma_{252}(t)\sqrt{252}}$ | 252 |
| Return acceleration | Change in a $k$-day return against the same return measured $k$ days earlier; positive when momentum is building. | $r^{(k)}_t - r^{(k)}_{t-k}$ | $k = 5, 21$ |
| Momentum of momentum | Slope of the $k$-day return over the last 5 days. | $r^{(k)}_t - r^{(k)}_{t-5}$ | $k = 21, 63$ |
| Information discreteness (frog in the pan) | Da, Gurun and Warachka (2014): momentum that arrives in many small moves continues more than momentum delivered in a few jumps. | $PRET_t = \ln(C_{t-21}/C_{t-252})$; $ID_t = \operatorname{sgn}(PRET_t)\,\big(\%\text{neg} - \%\text{pos}\big)$ over the same window; continuous-momentum score $PRET_t \times (-ID_t)$ | formation 252 minus 21; more negative ID means more continuous |
| Momentum gap | Spread between the top and bottom of the trailing path: how far the asset is from its best and worst points. | $\ln(\max_k C / C_t) - \ln(C_t / \min_k C)$ | $k = 63, 252$ |
| 52-week-high ratio | George and Hwang (2004): nearness to the 52-week high predicts continuation independently of past returns. | $C_t / \max_{252}(C)_t$ | 252 |
| 52-week-high recency | Bhootra and Hur (2013): how recently the high was set adds to the ratio. | $1 - (t - \arg\max_{252} C)/252$ | 252 |
| Up-day ratio | Share of positive daily returns in the window; a discreteness and trend-quality measure. | $\frac{1}{k}\sum_{i=0}^{k-1}\mathbb{1}[r_{t-i} > 0]$ | $k = 5, 10, 21, 63$ |
| Streak | Signed count of consecutive closes in the same direction. | $S_t = \operatorname{sgn}(r_t)\,(\lvert S_{t-1} \rvert + 1)$ if $\operatorname{sgn}(r_t) = \operatorname{sgn}(r_{t-1})$, else $\operatorname{sgn}(r_t)$; 0 on an unchanged close | current |
| Up-day vs down-day mean | Average return on up days minus the average on down days; asymmetry of the move sizes. | $\mu(r \mid r > 0)_k - \mu(r \mid r < 0)_k$ over the last $k$ days | $k = 21, 63$ |
| Max and min daily return | Largest and smallest daily return in the window; MAX is Bali, Cakici and Whitelaw's (2011) lottery-demand predictor (high MAX, low future return). | $\max_k(r)_t$, $\min_k(r)_t$; MAX minus MIN spread $\max_{21} r - \lvert \min_{21} r \rvert$ | $k = 5, 21, 63$ |
| MAX5 and volatility-adjusted MAX5 | Bali, Brown, Murray and Tang (2017): mean of the five largest returns in the month; Asness et al. (2020) divide by volatility (betting against lottery). | $MAX5_t = \frac{1}{5}\sum_{j=1}^{5} r_{(j)}$, the five largest in 21 days; $MAX5_t / \sigma_{21}(t)$ | 21 |
| Return percentile | Where today's return sits in the trailing distribution of daily returns. | $\operatorname{prank}_k(r)_t$ | $k = 63, 252$ |
| Sum of signs | Net count of up minus down days; a slow trend vote. | $\sum_{i=0}^{k-1} \operatorname{sgn}(r_{t-i})$ | $k = 10, 21$ |
| Sign agreement | Whether the 1-day and 5-day returns point the same way. | $\operatorname{sgn}(r_t)\,\operatorname{sgn}(r^{(5)}_t)$ | fixed |
| Linear-decay weighted return | Kakushadze's `decay_linear`: recency-weighted average of daily returns. | $\dfrac{\sum_{i=0}^{k-1}(k-i)\,r_{t-i}}{k(k+1)/2}$ | $k = 10, 21$ |
| Time-series rank of return | Kakushadze's `ts_rank`: rank of the latest $k$-day return against its own history. | $\operatorname{prank}_{252}(r^{(k)})_t$ | $k = 21, 63$ |

## Reversal and path shape

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Short-term reversal | Negative of the recent return; short horizons mean-revert (Jegadeesh 1990, Lehmann 1990). | $-r^{(k)}_t$ | $k = 1, 5, 21$ |
| Distance to window high | Percentage below the highest high of the window. | $C_t / \max_k(H)_t - 1$ | $k = 21, 63, 252$ |
| Distance to window low | Percentage above the lowest low of the window. | $C_t / \min_k(L)_t - 1$ | $k = 21, 63, 252$ |
| Days since window high or low | Trading days elapsed since the window's maximum close and minimum close (Kakushadze's `ts_argmax`, `ts_argmin`). | $t - \arg\max_{s \in (t-k, t]} C_s$; $t - \arg\min_{s \in (t-k, t]} C_s$ | $k = 63, 252$ |
| Position in window range | Where the close sits between the window's low and high, 0 to 1. | $(C_t - \min_k(L)_t) / (\max_k(H)_t - \min_k(L)_t)$ | $k = 21, 63, 252$ |
| Return autocorrelation | Lag-1 autocorrelation of daily returns over the window; negative values indicate mean reversion, positive values trending. | $\rho_1 = \operatorname{corr}_k(r_t, r_{t-1})$ | $k = 21, 63, 252$ |
| Partial reversal | Return today against yesterday's return, in units of volatility; captures one-day overreaction. | $-r_{t-1} / \sigma_{21}(t)$ | fixed |
| Path length | Sum of absolute daily returns; the "distance travelled" in the window regardless of direction. | $PL_k(t) = \sum_{i=0}^{k-1} \lvert r_{t-i} \rvert$ | $k = 21, 63$ |
| Path efficiency | Net displacement over path length (Kaufman's efficiency ratio on returns), 0 to 1; near 1 means a straight-line trend. | $ER_k(t) = \lvert r^{(k)}_t \rvert / PL_k(t)$ | $k = 10, 21, 63$ |

## Relative strength

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Relative return vs sector ETF | Asset return minus its sector ETF return over the same window. | $r^{(k)}_t - r^{S,(k)}_t$ | $k = 1, 5, 21, 63, 252$ |
| Relative return vs SPY | Asset return minus the market return. | $r^{(k)}_t - r^{M,(k)}_t$ | $k = 1, 5, 21, 63, 252$ |
| Beta-adjusted residual return | Return not explained by the market with a trailing beta. | $r^{(k)}_t - \beta^{M}_{63}(t)\, r^{M,(k)}_t$ | $k = 1, 5, 21$ |
| Cumulative residual momentum | Blitz, Huij and Martens (2011): sum of regression residuals against SPY and the sector ETF, scaled by their standard deviation. | $\dfrac{\sum_{i=0}^{k-1} \varepsilon_{t-i}}{\operatorname{std}_k(\varepsilon)}$, $\varepsilon_s = r_s - \hat a - \hat\beta^M r^M_s - \hat\beta^S r^S_s$ from a 252-day regression | $k = 63, 126, 252$ |
| Relative strength rank | Rank of the asset's $k$-day return among the 11 assets. | $\operatorname{xrank}(r^{(k)})_t$ | $k = 5, 21, 63, 252$ |
| Momentum of relative strength | Five-day change in the relative return against SPY. | $\Delta_5\big(r^{(k)} - r^{M,(k)}\big)_t$ | $k = 21, 63$ |
| Relative strength line slope | Slope of the log price ratio against SPY over the window, annualized. | OLS slope of $\ln(C_s / C^{M}_s)$ on $s$ over the last $k$ days, times 252 | $k = 21, 63$ |
| Relative strength ratio vs sector, z-scored | Price ratio to the sector ETF against its own trailing mean. | $z_{63}\big(\ln(C / C^S)\big)_t$ | 63 |
| Sector-relative sign agreement | Whether the asset and its sector moved the same way today and over 5 days. | $\operatorname{sgn}(r_t)\operatorname{sgn}(r^S_t)$; $\operatorname{sgn}(r^{(5)}_t)\operatorname{sgn}(r^{S,(5)}_t)$ | fixed |
| Beat-the-market count | Days in the window the asset outperformed SPY. | $\frac{1}{k}\sum_{i<k}\mathbb{1}[r_{t-i} > r^M_{t-i}]$ | $k = 21, 63$ |

## Formulaic alphas (Kakushadze 2016)

"101 Formulaic Alphas" (arXiv 1601.00991) expresses short-horizon signals in a small operator vocabulary over daily data. The operators are useful building blocks on their own; the alphas below use only OHLCV and are written verbatim (with `returns` = daily simple return, `vwap` $\approx TP_t$, `adv20` = 20-day mean dollar volume).

| Operator | Meaning | Formula |
| --- | --- | --- |
| `rank(x)` | Cross-sectional percentile rank on day $t$; with 22 assets, substitute `ts_rank(x, 252)`. | $\operatorname{xrank}(x)_t$ |
| `delay(x, d)`, `delta(x, d)` | Lag and $d$-day difference. | $x_{t-d}$; $x_t - x_{t-d}$ |
| `correlation(x, y, d)`, `covariance(x, y, d)` | Time-series correlation and covariance over $d$ days. | $\operatorname{corr}_d(x, y)_t$ |
| `ts_min`, `ts_max`, `ts_argmin`, `ts_argmax`, `ts_rank` | Rolling extremes, the day index (0 = today) on which they occurred, and the rolling percentile rank. | $\min_d$, $\max_d$, $t - \arg\max$, $\operatorname{prank}_d$ |
| `sum`, `product`, `stddev` | Rolling sum, product and standard deviation. | |
| `decay_linear(x, d)` | Linearly weighted average with weights $d, d-1, \dots, 1$. | $WMA_d(x)$ |
| `signedpower(x, a)` | Sign-preserving power. | $\operatorname{sgn}(x)\lvert x \rvert^a$ |
| `scale(x)` | Rescale cross-sectionally so that $\sum \lvert x \rvert = 1$. | $x / \sum_i \lvert x_i \rvert$ |
| `indneutralize(x, g)` | Subtract the group mean (sector). | $x - \bar x_g$ |

| Alpha | Expression | Reads as |
| --- | --- | --- |
| 1 | `rank(ts_argmax(signedpower((returns < 0) ? stddev(returns, 20) : close, 2), 5)) - 0.5` | Recency of the largest squared close or vol over 5 days |
| 2 | `-correlation(rank(delta(log(volume), 2)), rank((close - open) / open), 6)` | Volume changes against intraday return |
| 3 | `-correlation(rank(open), rank(volume), 10)` | Price-volume decorrelation |
| 6 | `-correlation(open, volume, 10)` | Same without ranks |
| 9 | `(0 < ts_min(delta(close, 1), 5)) ? delta(close, 1) : ((ts_max(delta(close, 1), 5) < 0) ? delta(close, 1) : -delta(close, 1))` | Follow a 5-day monotone trend, otherwise fade yesterday |
| 12 | `sign(delta(volume, 1)) * (-delta(close, 1))` | Reversal gated by the volume change |
| 13 | `-rank(covariance(rank(close), rank(volume), 5))` | Rank covariance of price and volume |
| 23 | `((sum(high, 20) / 20) < high) ? -delta(high, 2) : 0` | Fade new highs above the 20-day mean high |
| 26 | `-ts_max(correlation(ts_rank(volume, 5), ts_rank(high, 5), 5), 3)` | Peak recent rank correlation |
| 41 | `sqrt(high * low) - vwap` | Geometric mid against VWAP |
| 42 | `rank(vwap - close) / rank(vwap + close)` | Close relative to VWAP |
| 46 | `(0.25 < ((delay(close, 20) - delay(close, 10)) / 10 - (delay(close, 10) - close) / 10)) ? -1 : (((delay(close, 20) - delay(close, 10)) / 10 - (delay(close, 10) - close) / 10) < 0) ? 1 : -(close - delay(close, 1))` | Reversal gated by trend deceleration |
| 49 | `(((delay(close, 20) - delay(close, 10)) / 10 - (delay(close, 10) - close) / 10) < -0.1) ? 1 : -(close - delay(close, 1))` | Same with a different threshold |
| 53 | `-delta(((close - low) - (high - close)) / (close - low), 9)` | 9-day change in close location |
| 54 | `-((low - close) * open^5) / ((low - high) * close^5)` | Body-to-range ratio |
| 55 | `-correlation(rank((close - ts_min(low, 12)) / (ts_max(high, 12) - ts_min(low, 12))), rank(volume), 6)` | Stochastic position against volume |
| 60 | `-(2 * scale(rank(((close - low) - (high - close)) / (high - low) * volume)) - scale(rank(ts_argmax(close, 10))))` | CLV-weighted volume against recency of the high |
| 101 | `(close - open) / ((high - low) + 0.001)` | Normalized body |
