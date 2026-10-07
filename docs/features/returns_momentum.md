# Returns, momentum and relative strength

Back to the [catalog index](../feature_catalog.md) for notation. All returns are log returns on the adjusted close unless stated. $k$ is the lookback in trading days; $r^{M}$ and $r^{S}$ are the SPY and sector-ETF returns.

## Raw and lagged returns

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| $k$-day log return | Cumulative log price change over the window; the basic momentum (long $k$) or reversal (short $k$) input. | $r^{(k)}_t = \ln(C_t / C_{t-k})$ | $k = 1, 2, 3, 5, 10, 21, 42, 63, 126, 252$ | A: Jegadeesh (1990); Jegadeesh and Titman (1993) |
| Lagged daily returns | Each of the last ten daily returns as its own column so the model can learn its own short-horizon filter instead of a fixed window. | $r_{t-j}$ | $j = 1, \dots, 10$ | D: generic; cf. mom1m in Gu, Kelly and Xiu (2020) |
| Simple return | Arithmetic return, used where the model consumes percentages or for exact compounding checks. | $R^{(k)}_t = C_t / C_{t-k} - 1 = e^{r^{(k)}_t} - 1$ | $k = 1, 5, 21$ | D: derived |
| Overnight return | Close-to-open gap return, plus lags and trailing sums; overnight and intraday returns have different drivers and different persistence (Lou, Polk and Skouras 2019). | $o_t = \ln(O_t / C_{t-1})$; $\sum_{i=0}^{k-1} o_{t-i}$ | lags 1 to 3; sums $k = 5, 21$ | A: Lou, Polk and Skouras (2019) |
| Intraday return | Open-to-close return, plus lags and trailing sums. | $c_t = \ln(C_t / O_t)$; $\sum_{i=0}^{k-1} c_{t-i}$ | lags 1 to 3; sums $k = 5, 21$ | A: Lou, Polk and Skouras (2019) |
| Overnight minus intraday | Difference of cumulative overnight and intraday returns; a persistent positive gap signals a "tug of war" between overnight and day-session traders. | $\sum_{i=0}^{k-1} o_{t-i} - \sum_{i=0}^{k-1} c_{t-i}$ | $k = 21, 63, 252$ | A: Lou, Polk and Skouras (2019) |
| Overnight share of return | Fraction of the window's total return earned overnight. | $\sum_{i<k} o_{t-i} \big/ \sum_{i<k} \lvert r_{t-i} \rvert$ | $k = 63, 252$ | D: derived from Lou, Polk and Skouras (2019) |
| Average overnight and intraday return | Lou, Polk and Skouras (2019): the past year's mean overnight return predicts future overnight returns positively and intraday returns negatively, and vice versa; use as separate inputs for the overnight and open-to-close targets. | $\mu_{252}(o)_t$; $\mu_{252}(c)_t$ | 252 | A: Lou, Polk and Skouras (2019) |
| Nominal price | Unadjusted share price level (Miller and Scholes 1982); tick-size and round-lot effects. | $\ln P_t$ | 1 | A: Miller and Scholes (1982); weak |
| Close-to-high and close-to-low returns | Where today closed relative to today's extremes, in return units. | $\ln(C_t/H_t)$, $\ln(C_t/L_t)$ | 1 | D: derived |
| Return relative to typical price | Close against the day's typical price; positive when the close sits in the upper part of the day. | $\ln(C_t / TP_t)$ | 1 | D: derived |

## Momentum

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Momentum 12-1 | Classic Jegadeesh-Titman momentum: the return from 12 months ago to 1 month ago, skipping the most recent month to avoid short-term reversal. | $MOM^{12,1}_t = \ln(C_{t-21} / C_{t-252})$ | fixed | A: Jegadeesh and Titman (1993); Asness, Moskowitz and Pedersen (2013) |
| Momentum 6-1 | Intermediate-horizon version. | $MOM^{6,1}_t = \ln(C_{t-21} / C_{t-126})$ | fixed | A: Jegadeesh and Titman (1993) |
| Intermediate momentum | Novy-Marx (2012): the return from months 12 to 7 predicts better than months 6 to 2. | $\ln(C_{t-126} / C_{t-252})$ | fixed | A: Novy-Marx (2012) |
| Other skip-month momenta | Jegadeesh-Titman 3-1 and 9-1 variants. | $\ln(C_{t-21}/C_{t-63})$; $\ln(C_{t-21}/C_{t-189})$ | fixed | A: Jegadeesh and Titman (1993) |
| Long-term reversal | De Bondt and Thaler (1985): returns over years 2 to 3 and 2 to 5 reverse. | $\ln(C_{t-252}/C_{t-756})$; $\ln(C_{t-252}/C_{t-1260})$ | fixed | A: De Bondt and Thaler (1985) |
| Momentum change | Gettleman and Marks (2006): six-month return minus the prior six-month return. | $\ln(C_{t-21}/C_{t-126}) - \ln(C_{t-126}/C_{t-252})$ | fixed | A: Gettleman and Marks (2006) |
| Sector momentum level | Moskowitz and Grinblatt (1999): the asset's own sector ETF 12-1 return as a predictor in its own right, not only relative to the asset. | $\ln(C^S_{t-21}/C^S_{t-252})$; also $\ln(C^S_t / C^S_{t-252})$ | fixed | A: Moskowitz and Grinblatt (1999) |
| Annual return seasonality | Heston and Sadka (2008): the return in the same calendar month in prior years predicts this month's return. Daily analogue: the 21-day return centred one, two to five, and six to ten years ago. | $seas^{1}_t = \ln(C_{t-242}/C_{t-263})$; $seas^{2..5}_t = \frac{1}{4}\sum_{y=2}^{5}\ln(C_{t-252y+10}/C_{t-252y-11})$; $seas^{6..10}_t$ analogously; non-annual version: mean 21-day return over the same years excluding the annual windows (negative predictor) | 1, 5, 10 years | A: Heston and Sadka (2008) |
| Same-weekday seasonality | Keloharju, Linnainmaa and Nyberg (2016): mean return on the same weekday over the last year minus the mean on other weekdays. | $\mu(r \mid \text{same weekday})_{252} - \mu(r \mid \text{other weekdays})_{252}$ | 252 | A: Keloharju, Linnainmaa and Nyberg (2016) |
| Price delay | Hou and Moskowitz (2005): share of return variation explained by lagged market returns; slow incorporation of information predicts higher returns. | $D1 = 1 - \dfrac{R^2_{restricted}}{R^2_{full}}$, full: $r_t = a + \sum_{j=0}^{5} b_j r^M_{t-j} + \varepsilon$, restricted: $b_{1..5} = 0$ | 252 | A: Hou and Moskowitz (2005) |
| Momentum excluding last week | Skips only the last 5 days; useful at daily frequency where the reversal is a week, not a month. | $\ln(C_{t-5} / C_{t-k})$ | $k = 63, 126, 252$ | D: daily variant of Jegadeesh and Titman (1993) |
| Time-series momentum sign | Moskowitz-Ooi-Pedersen trend signal: sign of the trailing return. | $\operatorname{sgn}(r^{(k)}_t)$ | $k = 21, 63, 126, 252$ | A: Moskowitz, Ooi and Pedersen (2012) |
| Volatility-scaled return | Trailing return in units of its expected noise; makes windows and assets comparable. | $r^{(k)}_t / \big(\sigma_{21}(t)\sqrt{k}\big)$ | $k = 1, 5, 21, 63$ | A: Moskowitz, Ooi and Pedersen (2012); Barroso and Santa-Clara (2015) |
| Rolling Sharpe ratio | Annualized mean over standard deviation of daily returns; a risk-adjusted momentum measure. | $SR_k(t) = \dfrac{\mu_k(r)_t}{\operatorname{std}_k(r)_t}\sqrt{252}$ | $k = 21, 63, 126, 252$ | D: derived |
| Trend strength (t-statistic) | One-year return divided by its standard error; the $t$-stat of the mean daily return. | $TS_t = \dfrac{\ln(C_t / C_{t-252})}{\sigma_{252}(t)\sqrt{252}}$ | 252 | D: derived; cf. Moskowitz, Ooi and Pedersen (2012) |
| Return acceleration | Change in a $k$-day return against the same return measured $k$ days earlier; positive when momentum is building. | $r^{(k)}_t - r^{(k)}_{t-k}$ | $k = 5, 21$ | D: short-horizon variant of Gettleman and Marks (2006) |
| Momentum of momentum | Slope of the $k$-day return over the last 5 days. | $r^{(k)}_t - r^{(k)}_{t-5}$ | $k = 21, 63$ | D: derived |
| Information discreteness (frog in the pan) | Da, Gurun and Warachka (2014): momentum that arrives in many small moves continues more than momentum delivered in a few jumps. | $PRET_t = \ln(C_{t-21}/C_{t-252})$; $ID_t = \operatorname{sgn}(PRET_t)\,\big(\%\text{neg} - \%\text{pos}\big)$ over the same window; continuous-momentum score $PRET_t \times (-ID_t)$ | formation 252 minus 21; more negative ID means more continuous | A: Da, Gurun and Warachka (2014) |
| Momentum gap | Spread between the top and bottom of the trailing path: how far the asset is from its best and worst points. | $\ln(\max_k C / C_t) - \ln(C_t / \min_k C)$ | $k = 63, 252$ | D: derived |
| 52-week-high ratio | George and Hwang (2004): nearness to the 52-week high predicts continuation independently of past returns. | $C_t / \max_{252}(C)_t$ | 252 | A: George and Hwang (2004) |
| 52-week-high recency | Bhootra and Hur (2013): how recently the high was set adds to the ratio. | $1 - (t - \arg\max_{252} C)/252$ | 252 | A: Bhootra and Hur (2013) |
| Up-day ratio | Share of positive daily returns in the window; a discreteness and trend-quality measure. | $\frac{1}{k}\sum_{i=0}^{k-1}\mathbb{1}[r_{t-i} > 0]$ | $k = 5, 10, 21, 63$ | D: component of Da, Gurun and Warachka (2014) |
| Streak | Signed count of consecutive closes in the same direction. | $S_t = \operatorname{sgn}(r_t)\,(\lvert S_{t-1} \rvert + 1)$ if $\operatorname{sgn}(r_t) = \operatorname{sgn}(r_{t-1})$, else $\operatorname{sgn}(r_t)$; 0 on an unchanged close | current | D: derived; cf. Connors RSI |
| Up-day vs down-day mean | Average return on up days minus the average on down days; asymmetry of the move sizes. | $\mu(r \mid r > 0)_k - \mu(r \mid r < 0)_k$ over the last $k$ days | $k = 21, 63$ | D: derived |
| Max and min daily return | Largest and smallest daily return in the window; MAX is Bali, Cakici and Whitelaw's (2011) lottery-demand predictor (high MAX, low future return). | $\max_k(r)_t$, $\min_k(r)_t$; MAX minus MIN spread $\max_{21} r - \lvert \min_{21} r \rvert$ | $k = 5, 21, 63$ | A: Bali, Cakici and Whitelaw (2011) |
| MAX5 and volatility-adjusted MAX5 | Bali, Brown, Murray and Tang (2017): mean of the five largest returns in the month; Asness et al. (2020) divide by volatility (betting against lottery). | $MAX5_t = \frac{1}{5}\sum_{j=1}^{5} r_{(j)}$, the five largest in 21 days; $MAX5_t / \sigma_{21}(t)$ | 21 | A: Bali, Brown, Murray and Tang (2017); Asness et al. (2020) |
| Return percentile | Where today's return sits in the trailing distribution of daily returns. | $\operatorname{prank}_k(r)_t$ | $k = 63, 252$ | D: derived |
| Sum of signs | Net count of up minus down days; a slow trend vote. | $\sum_{i=0}^{k-1} \operatorname{sgn}(r_{t-i})$ | $k = 10, 21$ | D: derived |
| Sign agreement | Whether the 1-day and 5-day returns point the same way. | $\operatorname{sgn}(r_t)\,\operatorname{sgn}(r^{(5)}_t)$ | fixed | D: derived |
| Linear-decay weighted return | Kakushadze's `decay_linear`: recency-weighted average of daily returns. | $\dfrac{\sum_{i=0}^{k-1}(k-i)\,r_{t-i}}{k(k+1)/2}$ | $k = 10, 21$ | D: Kakushadze (2016) operator |
| Time-series rank of return | Kakushadze's `ts_rank`: rank of the latest $k$-day return against its own history. | $\operatorname{prank}_{252}(r^{(k)})_t$ | $k = 21, 63$ | D: Kakushadze (2016) operator |

## Reversal and path shape

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Short-term reversal | Negative of the recent return; short horizons mean-revert (Jegadeesh 1990, Lehmann 1990). | $-r^{(k)}_t$ | $k = 1, 5, 21$ | A: Jegadeesh (1990); Lehmann (1990) |
| Distance to window high | Percentage below the highest high of the window. | $C_t / \max_k(H)_t - 1$ | $k = 21, 63, 252$ | A: George and Hwang (2004) for 252 days; shorter windows untested |
| Distance to window low | Percentage above the lowest low of the window. | $C_t / \min_k(L)_t - 1$ | $k = 21, 63, 252$ | D: mirror of George and Hwang (2004) |
| Days since window high or low | Trading days elapsed since the window's maximum close and minimum close (Kakushadze's `ts_argmax`, `ts_argmin`). | $t - \arg\max_{s \in (t-k, t]} C_s$; $t - \arg\min_{s \in (t-k, t]} C_s$ | $k = 63, 252$ | A: Bhootra and Hur (2013) |
| Position in window range | Where the close sits between the window's low and high, 0 to 1. | $(C_t - \min_k(L)_t) / (\max_k(H)_t - \min_k(L)_t)$ | $k = 21, 63, 252$ | C: Lane (1984) stochastic on long windows |
| Return autocorrelation | Lag-1 autocorrelation of daily returns over the window; negative values indicate mean reversion, positive values trending. | $\rho_1 = \operatorname{corr}_k(r_t, r_{t-1})$ | $k = 21, 63, 252$ | B: Lo and MacKinlay (1988) |
| Partial reversal | Return today against yesterday's return, in units of volatility; captures one-day overreaction. | $-r_{t-1} / \sigma_{21}(t)$ | fixed | D: derived from Jegadeesh (1990) |
| Path length | Sum of absolute daily returns; the "distance travelled" in the window regardless of direction. | $PL_k(t) = \sum_{i=0}^{k-1} \lvert r_{t-i} \rvert$ | $k = 21, 63$ | D: derived |
| Path efficiency | Net displacement over path length (Kaufman's efficiency ratio on returns), 0 to 1; near 1 means a straight-line trend. | $ER_k(t) = \lvert r^{(k)}_t \rvert / PL_k(t)$ | $k = 10, 21, 63$ | C: Kaufman (1995) |

## Relative strength

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Relative return vs sector ETF | Asset return minus its sector ETF return over the same window. | $r^{(k)}_t - r^{S,(k)}_t$ | $k = 1, 5, 21, 63, 252$ | A: Moskowitz and Grinblatt (1999); Asness, Porter and Stevens (2000) |
| Relative return vs SPY | Asset return minus the market return. | $r^{(k)}_t - r^{M,(k)}_t$ | $k = 1, 5, 21, 63, 252$ | D: market-adjusted variant of momentum |
| Beta-adjusted residual return | Return not explained by the market with a trailing beta. | $r^{(k)}_t - \beta^{M}_{63}(t)\, r^{M,(k)}_t$ | $k = 1, 5, 21$ | A: Blitz, Huij and Martens (2011) |
| Cumulative residual momentum | Blitz, Huij and Martens (2011): sum of regression residuals against SPY and the sector ETF, scaled by their standard deviation. | $\dfrac{\sum_{i=0}^{k-1} \varepsilon_{t-i}}{\operatorname{std}_k(\varepsilon)}$, $\varepsilon_s = r_s - \hat a - \hat\beta^M r^M_s - \hat\beta^S r^S_s$ from a 252-day regression | $k = 63, 126, 252$ | A: Blitz, Huij and Martens (2011) |
| Relative strength rank | Rank of the asset's $k$-day return among the 11 assets. | $\operatorname{xrank}(r^{(k)})_t$ | $k = 5, 21, 63, 252$ | A: Jegadeesh and Titman (1993); noisy with 22 assets |
| Momentum of relative strength | Five-day change in the relative return against SPY. | $\Delta_5\big(r^{(k)} - r^{M,(k)}\big)_t$ | $k = 21, 63$ | D: derived |
| Relative strength line slope | Slope of the log price ratio against SPY over the window, annualized. | OLS slope of $\ln(C_s / C^{M}_s)$ on $s$ over the last $k$ days, times 252 | $k = 21, 63$ | D: derived |
| Relative strength ratio vs sector, z-scored | Price ratio to the sector ETF against its own trailing mean. | $z_{63}\big(\ln(C / C^S)\big)_t$ | 63 | D: derived |
| Sector-relative sign agreement | Whether the asset and its sector moved the same way today and over 5 days. | $\operatorname{sgn}(r_t)\operatorname{sgn}(r^S_t)$; $\operatorname{sgn}(r^{(5)}_t)\operatorname{sgn}(r^{S,(5)}_t)$ | fixed | D: derived |
| Beat-the-market count | Days in the window the asset outperformed SPY. | $\frac{1}{k}\sum_{i<k}\mathbb{1}[r_{t-i} > r^M_{t-i}]$ | $k = 21, 63$ | D: derived |

## Formulaic alphas (Kakushadze 2016)

"101 Formulaic Alphas" (arXiv 1601.00991) expresses short-horizon signals in a small operator vocabulary over daily data. The operators are useful building blocks on their own; the alphas below use only OHLCV and are written verbatim (with `returns` = daily simple return, `vwap` $\approx TP_t$, `adv20` = 20-day mean dollar volume).

| Operator | Meaning | Formula | Evidence |
| --- | --- | --- | --- |
| `rank(x)` | Cross-sectional percentile rank on day $t$; with 22 assets, substitute `ts_rank(x, 252)`. | $\operatorname{xrank}(x)_t$ | D: Kakushadze (2016) operator |
| `delay(x, d)`, `delta(x, d)` | Lag and $d$-day difference. | $x_{t-d}$; $x_t - x_{t-d}$ | D: Kakushadze (2016) operator |
| `correlation(x, y, d)`, `covariance(x, y, d)` | Time-series correlation and covariance over $d$ days. | $\operatorname{corr}_d(x, y)_t$ | D: Kakushadze (2016) operator |
| `ts_min`, `ts_max`, `ts_argmin`, `ts_argmax`, `ts_rank` | Rolling extremes, the day index (0 = today) on which they occurred, and the rolling percentile rank. | $\min_d$, $\max_d$, $t - \arg\max$, $\operatorname{prank}_d$ | D: Kakushadze (2016) operator |
| `sum`, `product`, `stddev` | Rolling sum, product and standard deviation. | | D: Kakushadze (2016) operator |
| `decay_linear(x, d)` | Linearly weighted average with weights $d, d-1, \dots, 1$. | $WMA_d(x)$ | D: Kakushadze (2016) operator |
| `signedpower(x, a)` | Sign-preserving power. | $\operatorname{sgn}(x)\lvert x \rvert^a$ | D: Kakushadze (2016) operator |
| `scale(x)` | Rescale cross-sectionally so that $\sum \lvert x \rvert = 1$. | $x / \sum_i \lvert x_i \rvert$ | D: Kakushadze (2016) operator |
| `indneutralize(x, g)` | Subtract the group mean (sector). | $x - \bar x_g$ | D: Kakushadze (2016) operator |

| Alpha | Expression | Reads as | Evidence |
| --- | --- | --- | --- |
| 1 | `rank(ts_argmax(signedpower((returns < 0) ? stddev(returns, 20) : close, 2), 5)) - 0.5` | Recency of the largest squared close or vol over 5 days | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 2 | `-correlation(rank(delta(log(volume), 2)), rank((close - open) / open), 6)` | Volume changes against intraday return | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 3 | `-correlation(rank(open), rank(volume), 10)` | Price-volume decorrelation | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 6 | `-correlation(open, volume, 10)` | Same without ranks | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 9 | `(0 < ts_min(delta(close, 1), 5)) ? delta(close, 1) : ((ts_max(delta(close, 1), 5) < 0) ? delta(close, 1) : -delta(close, 1))` | Follow a 5-day monotone trend, otherwise fade yesterday | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 12 | `sign(delta(volume, 1)) * (-delta(close, 1))` | Reversal gated by the volume change | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 13 | `-rank(covariance(rank(close), rank(volume), 5))` | Rank covariance of price and volume | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 23 | `((sum(high, 20) / 20) < high) ? -delta(high, 2) : 0` | Fade new highs above the 20-day mean high | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 26 | `-ts_max(correlation(ts_rank(volume, 5), ts_rank(high, 5), 5), 3)` | Peak recent rank correlation | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 41 | `sqrt(high * low) - vwap` | Geometric mid against VWAP | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 42 | `rank(vwap - close) / rank(vwap + close)` | Close relative to VWAP | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 46 | `(0.25 < ((delay(close, 20) - delay(close, 10)) / 10 - (delay(close, 10) - close) / 10)) ? -1 : (((delay(close, 20) - delay(close, 10)) / 10 - (delay(close, 10) - close) / 10) < 0) ? 1 : -(close - delay(close, 1))` | Reversal gated by trend deceleration | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 49 | `(((delay(close, 20) - delay(close, 10)) / 10 - (delay(close, 10) - close) / 10) < -0.1) ? 1 : -(close - delay(close, 1))` | Same with a different threshold | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 53 | `-delta(((close - low) - (high - close)) / (close - low), 9)` | 9-day change in close location | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 54 | `-((low - close) * open^5) / ((low - high) * close^5)` | Body-to-range ratio | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 55 | `-correlation(rank((close - ts_min(low, 12)) / (ts_max(high, 12) - ts_min(low, 12))), rank(volume), 6)` | Stochastic position against volume | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 60 | `-(2 * scale(rank(((close - low) - (high - close)) / (high - low) * volume)) - scale(rank(ts_argmax(close, 10))))` | CLV-weighted volume against recency of the high | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
| 101 | `(close - open) / ((high - low) + 0.001)` | Normalized body | D: Kakushadze (2016); backtested on thousands of stocks cross-sectionally, not on 22 names |
