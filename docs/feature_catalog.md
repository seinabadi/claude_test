# Feature catalog

Candidate model inputs computed from each asset's own daily OHLCV, calendar and options data, grouped by family; each is a function of data available at or before the close of day t.

## Targets (what to forecast)

Pick one primary label; the others work as auxiliary tasks or sanity checks. C, O, H, L, V = close, open, high, low, volume; sigma(t) = trailing daily volatility known at t.

| Target | Definition | Notes |
| --- | --- | --- |
| Next-day return | ln(C(t+1) / C(t)) | Default regression label |
| Next-day direction | sign of next-day return; or 3 classes with a dead zone of 0.1 to 0.25 × sigma(t) | Classification; flag near-zero days |
| Volatility-scaled return | next-day return / sigma(t) | Stabilises variance across regimes |
| Open-to-close return | ln(C(t+1) / O(t+1)) | Matches a trade entered at the open |
| Overnight return | ln(O(t+1) / C(t)) | Gap forecast; decide before the open |
| Excess return vs sector ETF | next-day return − beta × sector ETF return | Relative-value or market-neutral trading |
| Excess return vs SPY | next-day return − beta × SPY return | Beta-hedged alpha |
| Next-day range | ln(H(t+1) / L(t+1)) | Volatility and position-sizing forecast |
| Next-day realized volatility | absolute next-day return, or sum of squared intraday returns | Intraday data gives a far cleaner label |
| Multi-day forward return | ln(C(t+h) / C(t)) for h = 2, 5, 10, 21 | Less noise than the 1-day label |
| Triple-barrier label | +1, −1 or 0 by first touch of profit target, stop or time limit (barriers scaled by sigma) | Matches how stops actually trade |
| Meta-label | 1 if the primary model's trade was profitable, else 0 | Filters or sizes trades from a primary signal |

## Returns and momentum

Compute on split- and dividend-adjusted closes; k is the lookback in trading days.

| Feature | Definition | Windows k |
| --- | --- | --- |
| Log return | ln(C(t) / C(t−k)) | 1, 2, 3, 5, 10, 21, 42, 63, 126, 252 |
| Lagged daily returns | r(t−1) to r(t−10) as separate columns | 10 lags |
| Overnight return | ln(O(t) / C(t−1)), plus lags and 5- and 21-day sums | 1, 5, 21 |
| Intraday return | ln(C(t) / O(t)), plus lags and 5- and 21-day sums | 1, 5, 21 |
| Overnight minus intraday | cumulative overnight return − cumulative intraday return | 21, 63, 252 |
| Momentum 12-1 | ln(C(t−21) / C(t−252)) | fixed |
| Momentum 6-1 | ln(C(t−21) / C(t−126)) | fixed |
| Short-term reversal | negative of the return over the last 5 and 21 days | 5, 21 |
| Return acceleration | r(k) − r(k) measured k days earlier | 5, 21 |
| Volatility-scaled return | r(k) / (sigma(21) × sqrt(k)) | 1, 5, 21, 63 |
| Rolling Sharpe | mean(r) / std(r) × sqrt(252) | 21, 63, 126, 252 |
| Trend strength | ln(C(t) / C(t−252)) / (sigma(252) × sqrt(252)) | 252 |
| Time-series momentum sign | sign of r(k) | 21, 63, 126, 252 |
| Up-day ratio | share of positive daily returns in window | 5, 10, 21, 63 |
| Streak | count of consecutive up (or down) closes, signed | current |
| Up-day vs down-day mean | average r on up days minus average r on down days | 21, 63 |
| Max and min daily return | max(r), min(r) in window | 5, 21, 63 |
| Distance to window high | C(t) / max(H over k) − 1 | 21, 63, 252 |
| Distance to window low | C(t) / min(L over k) − 1 | 21, 63, 252 |
| Days since window high or low | trading days elapsed | 63, 252 |
| Return percentile | percentile rank of r(1) within trailing window | 63, 252 |
| Relative return vs sector ETF | r(k) of asset − r(k) of its sector ETF | 1, 5, 21, 63, 252 |
| Relative return vs SPY | r(k) of asset − r(k) of SPY | 1, 5, 21, 63, 252 |
| Beta-adjusted residual return | r(k) − beta(63) × r(k) of SPY | 1, 5, 21 |
| Relative strength rank | rank of r(k) among the 11 assets, 0 to 1 | 5, 21, 63, 252 |
| Momentum of relative strength | 5-day change in relative return vs SPY | 21, 63 |
| Cumulative residual momentum | sum of daily residuals vs SPY and sector | 63, 126, 252 |
| Return autocorrelation | corr(r(t), r(t−1)) over window | 21, 63, 252 |
| Sign agreement | product of sign(r(1)) and sign(r(5)) | fixed |

## Trend and moving averages

Express every moving average as a distance from price (ratio − 1) or a slope, never as a level.

| Feature | Definition | Parameters |
| --- | --- | --- |
| Price vs SMA | C / SMA(k) − 1 | k = 5, 10, 20, 50, 100, 200 |
| Price vs EMA | C / EMA(k) − 1 | k = 5, 10, 20, 50, 100, 200 |
| MA crossover spread | SMA(short) / SMA(long) − 1 | 5/20, 10/50, 20/100, 50/200 |
| Cross flags and age | 1 if short MA above long MA; trading days since last cross | same pairs |
| MA slope | SMA(k) / SMA(k) five days earlier − 1 | k = 20, 50, 200 |
| Count of MAs above | number of the six SMAs the close sits above, 0 to 6 | fixed |
| MACD | line, signal and histogram, each divided by C | 12, 26, 9 |
| PPO | (EMA12 − EMA26) / EMA26 | 12, 26, 9 |
| ADX, +DI, −DI | trend strength and direction | 14, 28 |
| Aroon up, down, oscillator | days since window high and low | 25 |
| Parabolic SAR | C / SAR − 1 and side flag | 0.02, 0.2 |
| Bollinger %B | (C − SMA20) / (upper − lower) | 20, 2 sigma |
| Bollinger bandwidth | (upper − lower) / SMA20 and its 252-day percentile | 20, 2 sigma |
| Keltner position | (C − EMA20) / (2 × ATR10) | 20, 10 |
| Donchian position | (C − min L) / (max H − min L) | 20, 55, 100 |
| Donchian breakout flags | C above window high or below window low | 20, 55 |
| Linear regression slope | slope of ln C on time, annualized | 20, 63, 126 |
| Regression R-squared | fit quality of the same regression | 20, 63, 126 |
| Price vs regression line | residual of C from the fit, in sigma units | 20, 63 |
| Ichimoku | distances of C to tenkan, kijun, senkou A and B; cloud thickness; above-cloud flag | 9, 26, 52 |
| Hull, KAMA, TEMA, DEMA, VWMA | C / MA − 1 for each adaptive average | 20, 50 |
| Supertrend | side flag and distance to the line | 10, 3 |
| Vortex | VI+ and VI− | 14 |
| Choppiness index | 0 to 100, high means range-bound | 14 |
| Rolling VWAP distance | C / VWAP(k) − 1, VWAP from (H + L + C) / 3 and volume | 5, 20 |
| Efficiency ratio | abs(C(t) − C(t−k)) / sum of abs daily changes | 10, 20 |
| Trend regime flag | 1 if C above SMA200, SMA50 above SMA200 and SMA200 slope positive | fixed |

## Oscillators

| Feature | Definition | Parameters |
| --- | --- | --- |
| RSI | relative strength index, plus its 5-day change | 2, 7, 14, 21 |
| RSI extremes | overbought and oversold flags; days since RSI above 70 or below 30 | 14 |
| Stochastic %K, %D | fast and slow | 14, 3, 3 |
| Stochastic RSI | stochastic applied to RSI | 14, 14, 3, 3 |
| Williams %R | position of C in the window range | 14, 28 |
| CCI | commodity channel index | 20 |
| MFI | money flow index, volume-weighted RSI | 14 |
| Ultimate oscillator | three-window weighted | 7, 14, 28 |
| TRIX | rate of change of triple-smoothed EMA | 15 |
| CMO | Chande momentum oscillator | 14 |
| DPO | detrended price oscillator, divided by C | 20 |
| Awesome oscillator | SMA5 − SMA34 of the midpoint, divided by C | 5, 34 |
| Elder ray | bull power (H − EMA13) / C and bear power (L − EMA13) / C | 13 |
| KST | know sure thing | 10, 15, 20, 30 |
| Coppock curve | long-term momentum | 14, 11, 10 |
| TSI | true strength index | 25, 13 |
| RVI | relative vigor index | 10 |
| Fisher transform | of the median price | 10 |
| Connors RSI | composite of RSI(3), streak RSI(2) and percent rank(100) | fixed |
| Price z-score | (C − SMA(k)) / std(C over k) | 20, 63 |
| Close percentile | percentile rank of C in the trailing window | 20, 63, 252 |
| Divergence flag | price makes a new 20-day high while RSI14 does not, and the inverse | 20 |

## Volatility and range

| Feature | Definition | Windows |
| --- | --- | --- |
| Realized volatility | std of daily log returns, annualized | 5, 10, 21, 63, 126, 252 |
| EWMA volatility | exponentially weighted std | lambda 0.94, 0.97 |
| GARCH(1,1) forecast | one-day-ahead conditional vol, refit on a rolling window | 500 |
| HAR-RV components | mean of squared returns over 1, 5 and 22 days | 1, 5, 22 |
| Parkinson volatility | from ln(H / L) | 10, 21 |
| Garman-Klass volatility | from O, H, L, C | 10, 21 |
| Rogers-Satchell volatility | drift-independent OHLC estimator | 10, 21 |
| Yang-Zhang volatility | handles overnight gaps | 10, 21 |
| Overnight share of variance | variance of overnight returns / total variance | 21, 63 |
| ATR | average true range | 5, 14, 21 |
| Normalized ATR | ATR / C | 14 |
| ATR percentile | rank of ATR14 over the trailing 252 days | 252 |
| True range ratio | true range of day t / ATR14 | 1 |
| Daily range | ln(H / L), plus its 5- and 21-day mean | 1, 5, 21 |
| Range expansion | range of day t / mean range over 20 days | 20 |
| Narrow range flags | NR4, NR7: narrowest range of the last 4 or 7 days | 4, 7 |
| Volatility ratio | sigma(5) / sigma(21); sigma(21) / sigma(63); sigma(21) / sigma(252) | pairs |
| Volatility term structure slope | regression slope of sigma across 5, 21, 63, 252 | fixed |
| Volatility percentile | rank of sigma(21) over trailing 252 and 504 days | 252, 504 |
| Volatility z-score | (sigma(21) − mean sigma(21)) / std over 252 | 252 |
| Volatility of volatility | std of rolling sigma(21) | 63 |
| Volatility change | sigma(21) / sigma(21) five days earlier − 1 | 5 |
| Downside semi-deviation | std of negative returns only | 21, 63 |
| Upside semi-deviation | std of positive returns only | 21, 63 |
| Up-down vol ratio | upside / downside semi-deviation | 63 |
| Absolute return lags | abs(r(t−1)) to abs(r(t−5)) | 5 lags |
| Squared return lags | squares of r(t−1) to r(t−5) | 5 lags |
| Bollinger squeeze | 1 when the Bollinger bands sit inside the Keltner channel | 20 |
| Chaikin volatility | rate of change of EMA of H − L | 10, 10 |
| Mass index | sum of EMA ratios of the range | 9, 25 |
| Large-move count | days with abs(r) above 2 sigma in window | 21, 63 |
| Days since last large move | trading days since abs(r) above 2 sigma | current |
| Gap volatility | std of overnight returns | 21, 63 |
| Implied minus realized | IV30 − realized sigma(21); see the options section | fixed |

## Volume and liquidity

| Feature | Definition | Windows |
| --- | --- | --- |
| Log volume | ln(V) | 1 |
| Relative volume | V / mean V over window | 5, 10, 21, 63 |
| Volume z-score | (V − mean V) / std V | 21, 63 |
| Volume percentile | rank of V over the trailing window | 63, 252 |
| Volume spike flag | V above 2 × mean V(21) | 21 |
| Volume trend | slope of ln V over window; mean V(5) / mean V(21) − 1 | 21 |
| Dollar volume | ln(C × V) and its relative versions | 1, 21 |
| Volume-weighted return | r(1) × V / mean V(21) | 21 |
| Up minus down volume | (V on up days − V on down days) / total V | 10, 21 |
| OBV | on-balance volume as OBV / mean V(21), and its 10-day slope | 10, 21 |
| Accumulation/distribution line | cumulative CLV × V, normalized like OBV | 21 |
| Chaikin money flow | sum(CLV × V) / sum(V) | 20 |
| Chaikin oscillator | EMA3 − EMA10 of the A/D line, divided by mean V | 3, 10 |
| Force index | EMA of (C − C(t−1)) × V, divided by C × mean V | 2, 13 |
| Ease of movement | midpoint move / (V / range) | 14 |
| Volume price trend | cumulative r × V, normalized | 21 |
| PVO | (EMA12 − EMA26 of V) / EMA26 | 12, 26, 9 |
| Klinger oscillator | volume force EMA34 − EMA55 | 34, 55, 13 |
| Positive and negative volume index | PVI and NVI as ratios to their 255-day EMA | 255 |
| Volume-return correlation | corr(abs r, V) over window | 21, 63 |
| Price-volume divergence flag | price up over 5 days while the volume trend falls, and the inverse | 5, 21 |
| Amihud illiquidity | mean(abs(r) / dollar volume) | 21, 63 |
| Turnover | V / shares outstanding | 1, 21 |
| Roll spread estimator | 2 × sqrt(−cov(r(t), r(t−1))) when the covariance is negative | 21, 63 |
| Corwin-Schultz spread | bid-ask spread estimate from daily highs and lows | 1, 21 |
| Kyle lambda proxy | regression slope of r on signed dollar volume | 63 |
| Short sale volume ratio | FINRA daily short volume / total volume, and its 21-day mean | 1, 21 |
| Same-weekday volume ratio | V / mean V on the same weekday over 12 weeks | 60 |

## Price action and gaps

| Feature | Definition | Windows |
| --- | --- | --- |
| Candle body | (C − O) / O | 1, and lags 1 to 3 |
| Body to range | abs(C − O) / (H − L) | 1 |
| Upper shadow | (H − max(O, C)) / (H − L) | 1 |
| Lower shadow | (min(O, C) − L) / (H − L) | 1 |
| Close location value | (C − L) / (H − L), plus its 5-day mean | 1, 5 |
| Close vs prior range | 1 above the prior H, −1 below the prior L, 0 inside | 1 |
| Open gap | ln(O / C(t−1)); gap / ATR14; lags | 1 to 3 |
| Gap fill flag | 1 if price traded back to C(t−1) during day t | 1 |
| Gap count | gaps larger than 1 ATR in window | 21, 63 |
| Inside day, outside day | flags from H and L vs the prior day | 1 |
| New high flags | C above max H over window | 5, 10, 20, 50, 252 |
| New low flags | C below min L over window | 5, 10, 20, 50, 252 |
| Days since new high or low | trading days elapsed | 20, 50, 252 |
| Higher-high and higher-low counts | days in window with a higher high or higher low | 5, 10 |
| Swing pivot distance | C / last zigzag pivot − 1, pivot threshold 3 to 5% | current |
| Support and resistance proximity | distance to the nearest local min and max of the last 63 days, in ATR units | 63 |
| Fibonacci position | (C − L63) / (H63 − L63) and distance to the nearest 38.2, 50 or 61.8 level | 63 |
| Pivot point distances | C vs classic P, R1, R2, S1, S2 from the prior day | 1 |
| Round number distance | distance of C to the nearest multiple of 5, 10, 50, 100, divided by C | 1 |
| Mean-reversion distance | (C − SMA20) / ATR14 | 20 |
| Breakout with volume | C above the Donchian 20 high and V above 1.5 × mean V(21) | 20 |
| Wide-range reversal | range above 2 × ATR and close in the opposite third from the open | 1 |
| Close-above-open streak | consecutive days with C above O, signed | current |
| Candlestick patterns | TA-Lib CDL functions, 61 pattern flags (doji, hammer, engulfing, stars and others) | 1 |
| Three-day pattern code | ordinal code of the sign sequence of the last 3 returns, 8 states | 3 |
| Range vol ratio | Parkinson sigma / close-to-close sigma | 21 |

## Statistical and risk features

| Feature | Definition | Windows |
| --- | --- | --- |
| Beta to SPY | regression slope of r on the SPY return | 21, 63, 126, 252 |
| Beta to sector ETF | same vs the asset's sector ETF | 63, 252 |
| Beta change | beta(63) − beta(252) | fixed |
| Downside beta | beta measured on SPY down days only | 252 |
| Correlation to SPY | rolling correlation | 21, 63, 252 |
| Correlation to sector ETF | rolling correlation | 21, 63 |
| Correlation to 10-year yield and DXY | rolling correlation of returns | 63 |
| Mean pairwise correlation | across the 11 assets | 21, 63 |
| Idiosyncratic volatility | std of residuals from the SPY and sector regression | 21, 63 |
| Idiosyncratic share | idiosyncratic variance / total variance | 63 |
| Residual momentum | sum of regression residuals | 21, 63, 252 |
| Skewness | of daily returns | 21, 63, 252 |
| Kurtosis | of daily returns | 21, 63, 252 |
| Max drawdown | from the running peak within window | 21, 63, 252 |
| Current drawdown | C / running max over window − 1 | 252, full history |
| Drawdown duration | days since the running max | 252 |
| Max run-up | mirror of drawdown | 63, 252 |
| Ulcer index | root mean square of drawdowns | 14, 63 |
| Calmar and Sortino | rolling return / max drawdown; return / downside deviation | 252 |
| Value at risk | empirical 5% and 1% quantiles of returns | 63, 252 |
| Expected shortfall | mean return below the VaR quantile | 63, 252 |
| Autocorrelation | lags 1, 2 and 5 of r and of abs(r) | 63, 252 |
| Variance ratio | var(r(5)) / (5 × var(r(1))) | 252 |
| Hurst exponent | rescaled range or DFA on log price | 126, 252 |
| Sample entropy | of the return series | 63 |
| Jump count | days with abs(r) above 3 sigma | 63, 252 |
| Fractionally differenced price | FFD with d chosen for stationarity, 0.3 to 0.6 | full history |
| Regime state | 2- or 3-state HMM on returns and vol; state probabilities | fit rolling |
| PCA factor loadings | loadings on the first 3 components of the 11-asset return panel | 63, 252 |
| Cross-sectional z-scores | each feature standardized across the 11 assets on day t | daily |
| Rolling rank of features | percentile of a feature over its own trailing history | 252 |

## Calendar and event timing

Encode cyclical fields as sine and cosine pairs or one-hot; flags are 0 or 1; countdowns in trading days.

| Feature | Definition |
| --- | --- |
| Day of week | Monday to Friday |
| Day of month, week of month, month, quarter | cyclical encodings |
| Turn of month | last trading day and first 3 trading days of the month |
| First and last day of week, month, quarter, year | flags |
| Days to and since month end and quarter end | countdowns |
| Holiday adjacency | day before and after a market holiday; half-day session; days in the trading week (4 or 5) |
| Options expiration | monthly OpEx (third Friday); days to OpEx; quad witching (third Friday of Mar, Jun, Sep, Dec); weekly expiry Friday |
| Index rebalance | S&P quarterly rebalance (quad witching day); Russell reconstitution (late June); MSCI semi-annual (May, Nov) |
| FOMC | decision day, day before, day after, days to next meeting, minutes release day, Fed blackout window |
| Macro release days | CPI, PPI, PCE, NFP (first Friday), retail sales, GDP, ISM manufacturing (first business day), ISM services, JOLTS, claims (Thursday) |
| Earnings season | weeks 2 to 6 after quarter end; share of S&P 500 reporting that week |
| Own earnings | days to next report, days since last, report-day flag, before-open or after-close timing (stocks) |
| Ex-dividend | ex-date flag, days to ex-date, dividend / price |
| Treasury auctions | 2-, 5-, 7-, 10- and 30-year auction days (rate-sensitive assets) |
| EIA and rig count | Wednesday crude inventories, Thursday gas storage, Friday rig count (XOM, NEE) |
| Seasonal windows | January; Santa rally (last 5 and first 2 trading days); May to October; September; December tax-loss window |
| Presidential cycle | year 1 to 4 of the term; election day; midterm flag |
| Daylight saving changes | March and November transition weeks |
| Time trend | trading-day index since the start of the sample; use with care |
| Days since last big move | trading days since abs(r) above 2 sigma |

## Stock-only features (the 11 single stocks)

Align each item to the date it became public, then forward-fill; lag quarterly data by at least one day after the filing.

| Family | Features |
| --- | --- |
| Earnings | EPS surprise (actual − consensus) / abs(consensus); revenue surprise; earnings-day return and gap; post-earnings drift over 5, 21, 63 days; pre-earnings 5-day run-up; guidance raised or cut flag; call-transcript sentiment and its change |
| Analyst estimates | consensus rating mean; upgrades and downgrades over 5 and 21 days; price target / C − 1; EPS revisions over 1 and 3 months (up minus down count, percent change); estimate dispersion / mean |
| Valuation | trailing and forward P/E; EV/EBITDA; P/B; P/S; PEG; dividend yield; FCF yield; earnings yield − 10-year yield; each as a z-score vs own 5-year history and vs the sector median |
| Growth | revenue YoY and QoQ; EPS YoY; gross, operating and net margin with YoY change; FCF margin |
| Balance sheet and quality | ROE, ROA, ROIC; debt / equity; net debt / EBITDA; interest coverage; accruals ratio; asset growth; capex / sales; R&D / sales; Piotroski F-score; Altman Z; Beneish M |
| Capital actions | buyback yield; share count change YoY; dividend growth; buyback announcement flag; secondary offering flag; split flag |
| Size and membership | log market cap; market cap rank within sector; S&P 500 weight and its change; index inclusion or removal event |
| Ownership | institutional ownership %; 13F net change (quarterly); insider net buys over 21 and 63 days (Form 4); insider buy / sell ratio |
| Short interest | short interest % of float; days to cover; change since the last report; borrow fee; utilization; FINRA daily short volume ratio |
| Credit | 5-year CDS spread and its change; bond yield spread to Treasuries; rating changes |
| News and attention | article count over 1 and 5 days; news sentiment score (FinBERT or vendor); abnormal news flag; Google Trends index; social mentions and sentiment (X, Reddit, StockTwits) |
| Filings text | 10-K and 10-Q similarity to the prior year; risk-factor section length change; 8-K count |
| Corporate events | M&A announcement; CEO change; litigation or regulatory news; product approval or launch (LLY) |

## ETF-only features (the 11 sector SPDRs)

| Family | Features |
| --- | --- |
| Flows | daily net creations and redemptions in dollars and as % of AUM; 5-, 21- and 63-day cumulative flow; flow z-score over 252 days; daily shares-outstanding change (public flow proxy) |
| Size | log AUM; 21-day AUM change; ETF AUM / sector market cap |
| NAV | premium or discount to NAV and its z-score; NAV return vs price return (tracking residual) |
| Holdings breadth | share of holdings above 50- and 200-day SMA; share of holdings up on the day; advance-decline line of the holdings; new highs minus new lows among holdings |
| Holdings dispersion | cross-sectional std of holdings' daily returns; mean pairwise correlation of the top 20 holdings; top-10 weight concentration (HHI); largest holding weight |
| Cap vs equal weight | cap-weighted sector return − equal-weighted sector return (XLK vs RSPT and so on); 21-day spread |
| Implied return check | weighted sum of constituent returns vs the ETF return; residual as a data-quality and arbitrage feature |
| Relative volume | ETF volume / mean 21; ETF volume / SPY volume |
| Sector weight | sector weight in the S&P 500 and its 63-day change |
| Rotation | sector return rank among the 11 over 5, 21 and 63 days; rank change; share of sectors beating SPY |
| Leveraged ETF pressure | sum over leveraged and inverse ETFs on the sector of leverage × AUM × daily return (end-of-day rebalancing flow proxy) |
| Short interest | ETF short interest % of shares; change since the last report |
| Options | put/call volume and OI ratios on the ETF; ATM IV and skew (see the options section) |
| Rebalance timing | days to the quarterly sector rebalance; GICS reclassification event flag |

## Options-implied features (the asset's own listed options)

Source: an end-of-day options chain or a vendor surface (ORATS, CBOE DataShop, OptionMetrics); every stock and ETF here has liquid options.

| Feature | Definition | Variants |
| --- | --- | --- |
| ATM implied volatility | interpolated at-the-money IV | 7, 30, 60, 90, 180 days |
| IV rank and percentile | IV30 relative to its trailing 252-day range and distribution | 252 |
| IV change | IV30 change over 1, 5 and 21 days | 1, 5, 21 |
| Variance risk premium | IV30 − realized sigma(21); also IV30 − GARCH forecast | fixed |
| IV term structure | IV90 / IV30 − 1; IV7 / IV30 − 1; inversion flag | fixed |
| Skew | IV(25-delta put) − IV(25-delta call); risk reversal; its 252-day percentile | 30, 90 days |
| Smile curvature | butterfly: mean of the 25-delta wings − ATM | 30 days |
| Implied skewness and kurtosis | model-free moments from the option surface | 30 days |
| Expected move | IV30 × sqrt(1 / 252) × C; straddle price / C | 1 day, nearest expiry |
| Put/call volume ratio | daily; 5- and 21-day means; z-score | 1, 5, 21 |
| Put/call open interest ratio | total put OI / call OI | 1 |
| Options volume | total contracts; options volume / mean 21; options notional / stock dollar volume | 1, 21 |
| Open interest change | total OI change, split by calls and puts | 1, 5 |
| Max pain | max-pain strike for the nearest expiry, as distance from C | nearest expiry |
| Call wall and put wall | strikes with the largest call and put OI, distance from C | nearest monthly |
| Gamma exposure | net dealer gamma from OI × gamma, in dollars per 1% move; distance to the zero-gamma level | all expiries |
| OI concentration | share of OI at the nearest expiry; days to that expiry | fixed |
| Implied earnings move | IV of the expiry straddling earnings vs the next expiry (stocks) | per event |
| Implied correlation | ETF IV vs weighted constituent IVs (sector ETFs) | 30 days |
| Vol surface PCA | first three components of the surface: level, slope, curvature | daily |
| Delta-hedged return proxy | straddle return minus delta-hedged P&L | 21 days |

## Rules for using the list

1. Every feature at row t uses only data available at the close of day t. Lag fundamentals to the filing date plus one day, macro to its release timestamp, options to the same-day close.
2. Adjust prices for splits and dividends with a total-return series. Splits in this set: AMZN 20:1 (Jun 2022), GOOGL 20:1 (Jul 2022), WMT 3:1 (Feb 2024), NEE 4:1 (Oct 2020); GE spun off GE HealthCare (Jan 2023) and GE Vernova (Apr 2024), so its history needs spin-off adjustment.
3. Use returns, ratios, distances and ranks, never raw price or volume levels. Fractional differentiation keeps memory while making a series stationary.
4. Scale with trailing-only statistics: rolling z-scores over 252 days, or cross-sectional ranks across the 11 assets each day. Never fit a scaler on the full sample.
5. Validate with walk-forward or purged k-fold cross-validation with an embargo of at least the label horizon. Random k-fold on daily data leaks.
6. Expect weak signals: a next-day direction model at AUC 0.53 to 0.56 out of sample is doing well. Check calibration and trade only above a confidence threshold.
7. Many features above are near-duplicates. Start with 50 to 80 covering each family, then prune by permutation importance or SHAP measured under the purged CV.
8. Evaluate the strategy on net returns with spread and commission, against buy-and-hold and a 50/200 crossover baseline.
9. Limit the search: every extra feature set, horizon and threshold tried inflates the best backtest. Report the deflated Sharpe ratio or the number of trials.
10. Record lineage for each feature: source, download date, adjustment method, and the first date the data was actually available.
