# Trend and moving averages

Back to the [catalog index](../feature_catalog.md) for notation. Express every moving average as a distance from price ($C_t / MA_t - 1$), a spread between two averages, or a slope, never as a level. Recursive averages are seeded with the SMA of the first full window and warmed up on at least $5k$ bars.

## Moving-average definitions

| Average | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| SMA | Equal-weighted mean of the last $k$ closes. | $SMA_k(C)_t = \frac{1}{k}\sum_{i=0}^{k-1} C_{t-i}$ | $k = 5, 10, 20, 50, 100, 200$ | C: Brock, Lakonishok and LeBaron (1992); not robust in Sullivan, Timmermann and White (1999) |
| EMA | Exponentially weighted mean; reacts faster than the SMA with the same nominal length. | $EMA_k(C)_t = \alpha C_t + (1-\alpha) EMA_k(C)_{t-1}$, $\alpha = 2/(k+1)$ | same $k$ | C: Appel (1979); same evidence as SMA |
| WMA | Linearly weighted mean with weights $k, k-1, \dots, 1$. | $WMA_k(C)_t = \dfrac{\sum_{i=0}^{k-1}(k-i) C_{t-i}}{k(k+1)/2}$ | $k = 10, 20$ | C: standard; TA-Lib WMA |
| DEMA | Double EMA (Mulloy 1994): removes much of the EMA's lag. | $DEMA_k = 2\,EMA_k(C) - EMA_k(EMA_k(C))$ | $k = 20, 50$ | C: Mulloy (1994) |
| TEMA | Triple EMA. | $TEMA_k = 3\,EMA_k - 3\,EMA_k(EMA_k) + EMA_k(EMA_k(EMA_k))$ | $k = 20, 50$ | C: Mulloy (1994) |
| TRIMA | Triangular MA, an SMA of an SMA (center-weighted, weights $1, 2, \dots, 2, 1$). | even $k$: $SMA_{k/2+1}(SMA_{k/2}(C))$; odd $k$: $SMA_{(k+1)/2}(SMA_{(k+1)/2}(C))$ | $k = 20$ | C: TA-Lib TRIMA |
| Hull MA | Hull (2005): very low lag using WMAs of different lengths. | $HMA_k = WMA_{\lfloor\sqrt k\rfloor}\big(2\,WMA_{\lfloor k/2 \rfloor}(C) - WMA_k(C)\big)$ | $k = 20, 50$ | C: Hull (2005) |
| KAMA | Kaufman's adaptive MA: smoothing speed set by the efficiency ratio so it is fast in trends and slow in chop. | $ER_t = \dfrac{\lvert C_t - C_{t-n} \rvert}{\sum_{i=0}^{n-1}\lvert C_{t-i} - C_{t-i-1} \rvert}$; $SC_t = \big(ER_t (\tfrac{2}{f+1} - \tfrac{2}{s+1}) + \tfrac{2}{s+1}\big)^2$; $KAMA_t = KAMA_{t-1} + SC_t (C_t - KAMA_{t-1})$ | $n = 10$, fast $f = 2$, slow $s = 30$ | C: Kaufman (1995) |
| ZLEMA | Zero-lag EMA (Ehlers and Way): EMA of a de-lagged price, as implemented by Tulip, pandas-ta and Pine. | $lag = \lfloor (k-1)/2 \rfloor$; $ZLEMA_k = EMA_k\big(2C_t - C_{t-lag}\big)$ | $k = 20$ | C: Ehlers and Way (2010) |
| T3 | Tillson's T3: a six-fold EMA with volume factor $v$. | $GD(x) = EMA_k(x)(1+v) - EMA_k(EMA_k(x))\,v$; $T3 = GD(GD(GD(C)))$ | $k = 5$, $v = 0.7$ | C: Tillson (1998) |
| VWMA | Volume-weighted MA; where most shares traded. | $VWMA_k = \dfrac{\sum_{i<k} C_{t-i} V_{t-i}}{\sum_{i<k} V_{t-i}}$ | $k = 20, 50$ | C: standard |
| ALMA | Arnaud Legoux MA: Gaussian-weighted window with offset. | $w_i = \exp\big(-\frac{(i - m)^2}{2 s^2}\big)$, $m = \lfloor o(k-1) \rfloor$, $s = k/\sigma$; $ALMA = \sum_i w_i C_{t-k+1+i} / \sum_i w_i$ | $k = 9$, $o = 0.85$, $\sigma = 6$ | C: Legoux (2009) |
| VIDYA | Chande's variable-index dynamic average: EMA whose $\alpha$ scales with the CMO. | $VIDYA_t = \alpha \lvert CMO_n(t) / 100 \rvert\, C_t + (1 - \alpha \lvert CMO_n(t)/100 \rvert)\, VIDYA_{t-1}$, $\alpha = 2/(k+1)$ | $k = 14$, $n = 9$ | C: Chande (1992) |
| FRAMA | Ehlers' fractal adaptive MA: $\alpha$ from the fractal dimension of the window. | $D_t = \dfrac{\ln(N_1 + N_2) - \ln N_3}{\ln 2}$ with $N_1, N_2$ the half-window ranges over $k/2$ and $N_3$ the full range, each divided by its length; $\alpha_t = e^{-4.6 (D_t - 1)}$ clipped to $[0.01, 1]$; $FRAMA_t = \alpha_t C_t + (1-\alpha_t) FRAMA_{t-1}$ | $k = 16$ | C: Ehlers (2005) |
| McGinley Dynamic | Self-adjusting average that speeds up in fast markets. | $MD_t = MD_{t-1} + \dfrac{C_t - MD_{t-1}}{k\,(C_t / MD_{t-1})^4}$ | $k = 10$ to $14$ | C: McGinley (1997) |
| Ehlers SuperSmoother | Two-pole Butterworth-style low-pass filter with a sharper roll-off than an EMA; Ehlers' standard pre-smoother. | $a = e^{-\sqrt 2\pi/k}$, $b = 2a\cos(\sqrt 2\pi/k)$, $c = 1 - b + a^2$; $SSF_t = \tfrac{c}{2}(C_t + C_{t-1}) + b\,SSF_{t-1} - a^2 SSF_{t-2}$ | $k = 20$ | C: Ehlers (2013) |
| MAMA and FAMA | Ehlers' MESA adaptive MA: EMA whose $\alpha$ is set by the rate of change of the Hilbert-transform phase; FAMA follows at half speed and the spread is a low-whipsaw trend signal. | $\Delta\phi_t = \max(\phi_{t-1} - \phi_t, 1)$ (degrees), $\alpha_t = \operatorname{clip}(f/\Delta\phi_t, s, f)$; $MAMA_t = \alpha_t MP_t + (1 - \alpha_t)MAMA_{t-1}$; $FAMA_t = \tfrac{\alpha_t}{2}MAMA_t + (1 - \tfrac{\alpha_t}{2})FAMA_{t-1}$; features $C/MAMA - 1$, $(MAMA - FAMA)/C$ | $f = 0.5$, $s = 0.05$ | C: Ehlers (2001) |
| Hilbert instantaneous trendline | TA-Lib HT_TRENDLINE: average over the current dominant cycle length, then a 4-3-2-1 smooth; a near-zero-lag adaptive MA. | $iT_t = \frac{1}{DCP}\sum_{i<DCP} MP_{t-i}$ with $DCP$ the rounded dominant cycle period; $TL_t = (4iT_t + 3iT_{t-1} + 2iT_{t-2} + iT_{t-3})/10$; feature $C_t/TL_t - 1$ | fixed | C: Ehlers (2001); TA-Lib HT_TRENDLINE |
| Holt-Winters MA and channel | Triple-exponential (level, trend, acceleration) smoother; the channel adds an exponentially weighted forecast-error variance, a model-based alternative to Bollinger bands. | $F_t = (1 - n_a)(F_{t-1} + V_{t-1} + \tfrac{1}{2}A_{t-1}) + n_a C_t$; $V_t = (1 - n_b)(V_{t-1} + A_{t-1}) + n_b(F_t - F_{t-1})$; $A_t = (1 - n_c)A_{t-1} + n_c(V_t - V_{t-1})$; $HWMA_t = F_t + V_t + \tfrac{1}{2}A_t$; $var_t = (1 - n_d)var_{t-1} + n_d(C_{t-1} - HWMA_{t-1})^2$; position $(C_t - HWMA_t)/\sqrt{var_{t-1}}$ | $n_a = 0.2$, $n_b = n_c = n_d = 0.1$ | B: Holt (1957), Winters (1960) as a forecaster; untested as a signal |
| Jurik MA (JMA) | Adaptive, phase-controlled smoother whose speed depends on a 65-bar relative-volatility estimate; very low lag and noise. Use the pandas-ta recursion. | library recursion; features $C/JMA - 1$ and its 5-day slope | $k = 7$, phase 0 | C: Jurik Research; pandas-ta jma |

## Price-to-average distances and spreads

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Price vs SMA | Percentage distance of the close from each SMA; the simplest trend and mean-reversion input. | $C_t / SMA_k(C)_t - 1$ | $k = 5, 10, 20, 50, 100, 200$ | C: Brock, Lakonishok and LeBaron (1992); Han, Yang and Zhou (2013) find MA timing works in high-volatility stocks |
| Price vs EMA | Same for EMAs. | $C_t / EMA_k(C)_t - 1$ | same | C: as SMA |
| Price vs adaptive averages | Same for HMA, KAMA, TEMA, DEMA, VWMA, T3, ALMA. | $C_t / MA_t - 1$ | $k = 20, 50$ | C: as SMA |
| Distance in ATR units | Distance scaled by volatility instead of price. | $(C_t - SMA_k(C)_t) / ATR_{14}(t)$ | $k = 20, 50, 200$ | D: derived |
| MA crossover spread | Relative gap between a short and a long average; the sign is the classic crossover signal and the size its conviction. | $SMA_{s}(C)_t / SMA_{l}(C)_t - 1$ | $(s, l) = (5, 20), (10, 50), (20, 100), (50, 200)$ | C: Brock, Lakonishok and LeBaron (1992); Sullivan, Timmermann and White (1999) negative out of sample |
| Cross flag and age | Whether the short average is above the long one, and trading days since the last cross. | $\mathbb{1}[SMA_s > SMA_l]$; $t - \max\{u \le t : \operatorname{sgn}(SMA_s - SMA_l)_u \ne \operatorname{sgn}(SMA_s - SMA_l)_{u-1}\}$ | same pairs | C: as crossover |
| MA slope | Percentage change of the average over 5 days, an estimate of the trend's velocity. | $SMA_k(C)_t / SMA_k(C)_{t-5} - 1$ | $k = 20, 50, 200$ | D: derived |
| MA slope sign streak | Consecutive days the SMA slope has had the same sign. | signed count | $k = 50$ | D: derived |
| MA curvature | Change of the slope: acceleration of the trend. | $\Delta_5\big(SMA_k(C)_t / SMA_k(C)_{t-5} - 1\big)$ | $k = 20, 50$ | D: derived |
| Count of MAs above | Number of the six SMAs the close sits above, 0 to 6; a trend-alignment score. | $\sum_{k} \mathbb{1}[C_t > SMA_k(C)_t]$ | fixed | D: derived; cf. Neely, Rapach, Tu and Zhou (2014) MA ensembles at the market level |
| MA alignment | Whether the averages are stacked in trend order. | $\mathbb{1}[SMA_{20} > SMA_{50} > SMA_{200}]$, $-1$ for the reverse order, else 0 | fixed | D: derived |
| Guppy MMA spread | Separation between the short-term and long-term EMA groups (Guppy multiple moving average). | $\dfrac{\frac{1}{6}\sum_{k \in \{3,5,8,10,12,15\}} EMA_k - \frac{1}{6}\sum_{k \in \{30,35,40,45,50,60\}} EMA_k}{C_t}$ | fixed | C: Guppy (2004) |
| Trend regime flag | Combined long-term trend filter. | $\mathbb{1}[C_t > SMA_{200} \wedge SMA_{50} > SMA_{200} \wedge SMA_{200,t} > SMA_{200,t-20}]$ | fixed | D: derived |
| Deviation-rate z-score (MADR) | Price-to-SMA percentage deviation standardized by its own rolling distribution. | $rate_t = 100(C_t/SMA_k - 1)$; $MADR_t = (rate_t - SMA_{2k}(rate)_t)/\operatorname{std}_{2k}(rate)_t$ | $k = 21$ | C: technical library MADR |
| Technical-ratings consensus | TradingView-style vote: $+1$ for each moving average the close is above and each oscillator in a buy state, $-1$ for sells, divided by the count; a cheap ensemble feature. | $\dfrac{\#buys - \#sells}{n}$ over SMA and EMA 10, 20, 30, 50, 100, 200 and RSI, Stochastic, CCI, ADX, AO, Momentum, MACD, %R, Bull/Bear power, UO rules | fixed | C: TradingView; cf. Neely, Rapach, Tu and Zhou (2014) |
| Alligator and Gator | Bill Williams: three Wilder-smoothed median-price averages (jaw 13, teeth 8, lips 5); their ordering and spread separate trending from sleeping markets. Use the unshifted values (the chart versions are plotted forward). | $(Lips - Teeth)/C_t$, $(Teeth - Jaw)/C_t$ with $Jaw = RMA_{13}(MP)$, $Teeth = RMA_8(MP)$, $Lips = RMA_5(MP)$; order flag $\mathbb{1}[Lips > Teeth > Jaw]$ | 13, 8, 5 | C: Williams (1995) |
| TTM trend | Close against the mean of the last six mid prices, as a bar colour and its run length. | $\operatorname{sgn}\big(C_t - \tfrac{1}{6}\sum_{i<6} MP_{t-i}\big)$; consecutive same-sign count | 6 | C: Carter (2005) |
| Correlation trend indicator | Pearson correlation of the close with time: a signed, bounded trend-quality measure (the signed square root of $R^2$). The Spearman version is the rank correlation index. | $CTI_t = \operatorname{corr}(C_{t-k+1..t}, 0..k-1)$; $RCI_t = 100\big(1 - \frac{6\sum d_i^2}{k(k^2 - 1)}\big)$, $d_i$ the rank difference between price and time | $k = 12$; RCI $k = 9$ | C: pandas-ta cti |
| Vertical horizontal filter | Ratio of the window's close range to the path length of closes; a trend-versus-range gauge like the efficiency ratio but using the window extremes. | $VHF_t = \dfrac{\max_k C - \min_k C}{\sum_{i<k}\lvert C_{t-i} - C_{t-i-1} \rvert}$ | $k = 28$ | C: White (1991) |

## MACD family

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| MACD line | Difference of a fast and a slow EMA, divided by price for comparability. | $MACD_t = (EMA_{12}(C)_t - EMA_{26}(C)_t) / C_t$ | 12, 26 | C: Appel (1979); Chong and Ng (2008) mixed |
| MACD signal and histogram | EMA of the MACD line and the gap between them; the histogram turning is an early momentum signal. | $SIG_t = EMA_9(MACD)_t$; $HIST_t = MACD_t - SIG_t$ | 9 | C: Appel (1979) |
| MACD histogram slope | Change of the histogram over 3 days. | $\Delta_3 HIST_t$ | | D: derived |
| PPO | Percentage price oscillator: MACD normalized by the slow EMA (so the level is comparable across assets without dividing by price). | $PPO_t = 100\,\dfrac{EMA_{12}(C)_t - EMA_{26}(C)_t}{EMA_{26}(C)_t}$; signal $EMA_9(PPO)$; histogram | 12, 26, 9 | C: StockCharts PPO |
| APO | Absolute price oscillator, divided by price. | $(EMA_{12} - EMA_{26})/C_t$ | | C: TA-Lib APO |
| Zero-lag MACD | MACD built from DEMAs. | $(DEMA_{12} - DEMA_{26}) / C_t$ | | C: derived from Mulloy (1994) |

## Directional movement and trend strength

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| +DM, −DM | Wilder's directional movement: today's upward or downward range extension, only the larger of the two counts. | $up = H_t - H_{t-1}$, $dn = L_{t-1} - L_t$; $+DM_t = up\,\mathbb{1}[up > dn \wedge up > 0]$; $-DM_t = dn\,\mathbb{1}[dn > up \wedge dn > 0]$ | | C: Wilder (1978) |
| +DI, −DI | Smoothed directional movement as a share of smoothed true range. | $+DI_t = 100\,\dfrac{RMA_k(+DM)_t}{RMA_k(TR)_t}$; $-DI_t$ likewise | $k = 14, 28$ | C: Wilder (1978) |
| DX and ADX | Directional index and its Wilder-smoothed average; trend strength regardless of direction (above 25 is trending). | $DX_t = 100\,\dfrac{\lvert +DI_t - (-DI_t) \rvert}{+DI_t + (-DI_t)}$; $ADX_t = RMA_k(DX)_t$ | $k = 14$ | C: Wilder (1978) |
| ADXR | Average of the current ADX and the ADX $k$ days ago. | $(ADX_t + ADX_{t-k})/2$ | 14 | C: Wilder (1978) |
| DI spread | Net direction. | $(+DI_t - (-DI_t))/100$ | 14 | D: derived |
| ADX slope | Whether trend strength is rising. | $\Delta_5 ADX_t$ | | D: derived |
| Aroon up and down | Share of the window elapsed since the highest high and the lowest low; 100 means the extreme was today. | $AroonUp_t = 100\,\dfrac{k - (t - \arg\max_{(t-k, t]} H)}{k}$; $AroonDown_t$ with $\arg\min L$ | $k = 25$ | C: Chande (1995) |
| Aroon oscillator | Up minus down. | $AroonUp_t - AroonDown_t$ | 25 | C: Chande (1995) |
| Vortex indicator | Ratio of positive and negative vortex movement to true range (Botes and Siepman 2010). | $VM^+_t = \lvert H_t - L_{t-1} \rvert$, $VM^-_t = \lvert L_t - H_{t-1} \rvert$; $VI^{\pm}_t = \dfrac{\sum_{i<k} VM^{\pm}_{t-i}}{\sum_{i<k} TR_{t-i}}$ | $k = 14$ | C: Botes and Siepman (2010) |
| Choppiness index | Dreiss: log ratio of summed true range to the window's range, 0 to 100; high values mean sideways. | $CHOP_t = 100\,\dfrac{\log_{10}\big(\sum_{i<k} TR_{t-i} / (\max_k H - \min_k L)\big)}{\log_{10} k}$ | $k = 14$ | C: Dreiss (1990s) |
| Efficiency ratio | Kaufman: net price change over the sum of absolute daily changes, 0 to 1. | $ER_k(t) = \dfrac{\lvert C_t - C_{t-k} \rvert}{\sum_{i=0}^{k-1}\lvert C_{t-i} - C_{t-i-1} \rvert}$ | $k = 10, 20$ | C: Kaufman (1995) |
| Trend intensity index | Share of the window's closes above the moving average, weighted by distance. | $TII_t = 100\,\dfrac{\sum_{i<k/2}\max(C_{t-i} - SMA_k, 0)}{\sum_{i<k/2}\lvert C_{t-i} - SMA_k \rvert}$ | $k = 60$ | C: Pee (2002) |
| Random walk index | Whether the move exceeds what a random walk would produce. | $RWI^{high}_t = \max_{2 \le n \le k}\dfrac{H_t - L_{t-n}}{ATR_n \sqrt n}$; $RWI^{low}_t = \max_n \dfrac{H_{t-n} - L_t}{ATR_n \sqrt n}$ | $k = 14$ | C: Poulos (1991) |

## Channels and bands

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Bollinger %B | Position of the close within the Bollinger bands, 0 at the lower band and 1 at the upper. | $M = SMA_{20}(C)$, $U = M + 2\operatorname{std}_{20}(C)$, $Lo = M - 2\operatorname{std}_{20}(C)$; $\%B_t = (C_t - Lo_t)/(U_t - Lo_t)$ | 20, 2 (population std in TA-Lib) | C: Bollinger (2001); Lento, Gradojevic and Wright (2007) find no profitability |
| Bollinger bandwidth | Band width relative to the middle band; a volatility-regime measure; its 252-day percentile flags squeezes. | $BW_t = (U_t - Lo_t)/M_t$; $\operatorname{prank}_{252}(BW)_t$ | 20, 2 | C: Bollinger (2001) |
| Bollinger band walk | Consecutive days closing above the upper or below the lower band. | signed count | | C: Bollinger (2001) |
| Keltner position | Position of the close within the Keltner channel (EMA plus or minus a multiple of ATR). | $KC_t = (C_t - EMA_{20}(C)_t) / (2\,ATR_{10}(t))$ | 20, 10, 2 | C: Keltner (1960); Raschke |
| Donchian position | Position of the close within the window's high-low range. | $(C_t - \min_k L_t)/(\max_k H_t - \min_k L_t)$ | $k = 20, 55, 100$ | C: Donchian (1960); Turtle rules |
| Donchian breakout flags | Close above the prior window's highest high or below its lowest low (the turtle rule). | $\mathbb{1}[C_t > \max_{k}(H)_{t-1}]$; $\mathbb{1}[C_t < \min_k(L)_{t-1}]$ | $k = 20, 55$ | C: Donchian (1960); cf. Hurst, Ooi and Pedersen (2017) for a century of trend-following evidence across asset classes |
| Donchian width | Range of the channel relative to price. | $(\max_k H - \min_k L)/C_t$ | 20 | D: derived |
| Price channel midline distance | Distance from the mid of the Donchian channel. | $C_t / \big((\max_k H + \min_k L)/2\big) - 1$ | 20 | D: derived |
| Standard-error channel position | Position relative to a regression line plus or minus 2 standard errors. | $(C_t - \hat C_t) / (2\, SE_k)$, $\hat C_t$ the fitted value, $SE_k$ the residual standard error | $k = 20, 63$ | C: standard-error channel |
| Acceleration bands | Headley: bands from the daily range around an SMA. | $U = SMA_{20}\big(H(1 + 4\frac{H - L}{H + L})\big)$, $Lo = SMA_{20}\big(L(1 - 4\frac{H-L}{H+L})\big)$; position of $C$ within | 20 | C: Headley (2002) |
| Envelope position | Fixed percentage envelope around the SMA. | $(C_t - SMA_{20})/(0.025\,SMA_{20})$ | 20, 2.5% | C: standard |

## Regression and curve fitting

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Linear regression slope | Slope of log price on time, annualized; a smoothed trend velocity. | $\hat b_k(t) = \dfrac{\sum_{i=0}^{k-1}(x_i - \bar x)(\ln C_{t-k+1+i} - \overline{\ln C})}{\sum_i (x_i - \bar x)^2}$, $x_i = i$; feature $= 252\,\hat b$ | $k = 20, 63, 126$ | C: Clenow (2015) |
| Regression R-squared | Share of price variance explained by the time trend; trend quality. | $R^2_k = 1 - \dfrac{\sum_i \hat\varepsilon_i^2}{\sum_i (\ln C_i - \overline{\ln C})^2}$ | same | C: Clenow (2015) |
| Slope times R-squared | Combined trend velocity and quality (Clenow's momentum score). | $252\,\hat b_k \cdot R^2_k$ | 63, 126 | C: Clenow (2015) |
| Price vs regression line | Residual of today's log close from the fit, in residual-sigma units. | $(\ln C_t - \widehat{\ln C}_t) / SE_k$ | $k = 20, 63$ | D: derived |
| Regression forecast | Fitted value extrapolated one day ahead against the close. | $\widehat{\ln C}_{t+1} - \ln C_t$ | 20 | C: TA-Lib TSF |
| Quadratic trend coefficient | Second-order coefficient of a quadratic fit of log price on time; curvature of the trend. | $\hat c$ in $\ln C_i = a + b x_i + c x_i^2 + \varepsilon_i$ over the window | 63 | D: derived |
| Linear regression angle | Slope in degrees (TA-Lib LINEARREG_ANGLE). | $\arctan(\hat b)\, 180/\pi$ on raw price | 14 | C: TA-Lib LINEARREG_ANGLE |
| Time-series forecast (TSF) | TA-Lib's one-step linear forecast, as a distance from price; the forecast error against yesterday's forecast is the Chande forecast oscillator. | $TSF_t = \hat a + \hat b k$; $(TSF_t/C_t - 1)$; $FOSC_t = 100\,(C_t - TSF_{t-1})/C_t$ | 14; FOSC 9 to 14 | C: TA-Lib TSF; Chande forecast oscillator |
| Regression t-statistic | Slope over its standard error: trend significance. | $\hat b_k / SE(\hat b_k)$ with $SE$ from the residual standard error $\sqrt{\sum\hat\varepsilon^2/(k-2)}$ | 20, 63 | D: derived; cf. Lopez de Prado (2020) trend scanning |

## Ichimoku, SAR and Supertrend

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Ichimoku lines | Mid-points of the high-low range over three windows. | $Tenkan_t = (\max_9 H + \min_9 L)/2$; $Kijun_t = (\max_{26} H + \min_{26} L)/2$; $SenkouA_t = (Tenkan_{t-26} + Kijun_{t-26})/2$; $SenkouB_t = (\max_{52} H_{t-26} + \min_{52} L_{t-26})/2$ | 9, 26, 52 (the cloud plotted at $t$ was computed 26 days ago, so it is known) | C: Hosoda (1969) |
| Ichimoku distances | Close relative to each line and the cloud. | $C_t/Tenkan_t - 1$, $C_t/Kijun_t - 1$, $C_t/SenkouA_t - 1$, $C_t/SenkouB_t - 1$ | | C: Hosoda (1969) |
| Cloud thickness and side | Cloud width relative to price, and whether price is above, inside or below the cloud. | $(SenkouA_t - SenkouB_t)/C_t$; $+1$ if $C_t > \max(A, B)$, $-1$ if below $\min(A, B)$, else 0 | | C: Hosoda (1969) |
| Chikou confirmation | Today's close against the close 26 days ago (the lagging span, evaluated without look-ahead). | $\mathbb{1}[C_t > C_{t-26}]$ | 26 | C: Hosoda (1969) |
| Parabolic SAR | Wilder's stop-and-reverse: a trailing stop that accelerates toward price. | Long: $SAR_{t+1} = SAR_t + AF_t (EP_t - SAR_t)$, $EP$ = highest high since the trend began, $AF$ starts at 0.02, rises by 0.02 on each new $EP$ to a cap of 0.20; $SAR_{t+1}$ is capped at $\min(L_t, L_{t-1})$ (the two prior bars); reverse to short when $L_{t+1} \le SAR_{t+1}$, setting $SAR = EP$, $EP = L_{t+1}$, $AF = 0.02$ (mirror rules for short) | 0.02, 0.02, 0.20 (TA-Lib and StockCharts agree) | C: Wilder (1978) |
| SAR features | Distance to the SAR and the side. | $C_t / SAR_t - 1$; $\mathbb{1}[\text{long}]$; days in the current trend | | C: Wilder (1978) |
| Supertrend | ATR-based trailing band that flips side when price closes through it. | $U_t = MP_t + m\,ATR_k(t)$, $Lo_t = MP_t - m\,ATR_k(t)$; final $U^f_t = U_t$ if $U_t < U^f_{t-1}$ or $C_{t-1} > U^f_{t-1}$ else $U^f_{t-1}$; final $Lo^f_t = Lo_t$ if $Lo_t > Lo^f_{t-1}$ or $C_{t-1} < Lo^f_{t-1}$ else $Lo^f_{t-1}$; trend flips up when $C_t > U^f_{t}$ and down when $C_t < Lo^f_{t}$; $ST_t = Lo^f_t$ in an uptrend, $U^f_t$ in a downtrend | $k = 10$, $m = 3$ | C: Seban (2000s) |
| Supertrend features | Side flag, distance to the line and days since the last flip. | $\pm 1$; $C_t / ST_t - 1$; count | | C: Seban (2000s) |
| Chandelier exit distance | Distance of the close from a trailing stop hung from the window high (long) or low (short), and the side. | $Long_t = \max_{22} H - m\,ATR_{22}$, $Short_t = \min_{22} L + m\,ATR_{22}$; side $+1$ if $C_t > Long_{t-1}$, $-1$ if $C_t < Short_{t-1}$, else carried; $(C_t - Long_t)/ATR_{22}$ | 22, $m = 3$ (pandas-ta 2) | C: LeBeau (1990s) |
| ATR trailing stop | Ratcheting stop that only moves in the trade's favour; the volatility-stop line. | if $C_t > SMA_{20}(C)$: $S_t = \max(C_t - m\,ATR_{14}, S_{t-1})$; else $S_t = \min(C_t + m\,ATR_{14}, S_{t-1})$, reset on a side flip; feature $(C_t - S_t)/ATR_{14}$ | $m = 3$ | C: Wilder (1978) volatility stop |
| Profit maximizer (PMAX) | Supertrend applied to a moving average instead of the mid price, which removes most whipsaws. | Supertrend recursion with $MA_t = EMA_{12}(C)$ in place of $MP_t$ and $MA_{t-1}$ in place of $C_{t-1}$ in the carry rules | 10, 3, 12 | C: technical library PMAX |
| Gann HiLo activator and SSL channel | Trailing lines that flip between an average of highs and an average of lows depending on which side the close is on. | HiLo: if $C_t > SMA_{13}(H)_{t-1}$ use $SMA_{21}(L)_t$ (long); if $C_t < SMA_{21}(L)_{t-1}$ use $SMA_{13}(H)_t$ (short); else carry. SSL: state $hlv_t = +1$ if $C_t > SMA_{10}(H)$, $-1$ if $C_t < SMA_{10}(L)$, else carried; features side and $(C_t - line_t)/ATR_{14}$ | 13, 21; 10 | C: Krausz (1998) |

## Rolling VWAP and anchored averages

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Rolling VWAP distance | Close against the volume-weighted average typical price over the window. | $VWAP_k(t) = \dfrac{\sum_{i<k} TP_{t-i} V_{t-i}}{\sum_{i<k} V_{t-i}}$; $C_t / VWAP_k(t) - 1$ | $k = 5, 20, 63$ | C: Berkowitz, Logue and Noser (1988) define VWAP as a benchmark; untested as a signal |
| Anchored VWAP distance | VWAP since an anchor (start of month, quarter, year, last earnings date). | same sum starting at the anchor | per anchor | C: Shannon (2008) |
| VWAP bands position | Close within one volume-weighted standard deviation of the rolling VWAP. | $\dfrac{C_t - VWAP_k}{\sqrt{\sum_{i<k} V_{t-i}(TP_{t-i} - VWAP_k)^2 / \sum_{i<k} V_{t-i}}}$ | 20 | C: standard |

## Price transforms and synthetic bars

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Alternative price series | TA-Lib AVGPRICE, MEDPRICE, TYPPRICE, WCLPRICE: single-price series that can replace the close in any feature above; the weighted close double-weights the close. | $AVG_t = (O + H + L + C)/4$; $MED_t = MP_t$; $TYP_t = TP_t$; $WCL_t = (H + L + 2C)/4$; derived $C_t/WCL_t - 1$ and $\ln(TP_t/TP_{t-1})$ | | C: TA-Lib price transforms |
| Heikin-Ashi body and streak | Smoothed synthetic candles; the HA body sign and its run length are a classic trend filter. | $HAC_t = (O_t + H_t + L_t + C_t)/4$; $HAO_t = (HAO_{t-1} + HAC_{t-1})/2$; $HAH_t = \max(H_t, HAO_t, HAC_t)$, $HAL_t = \min(L_t, HAO_t, HAC_t)$; features $(HAC_t - HAO_t)/C_t$, colour streak, shadow shares | | C: Valcu (2004) |
| Z-scored OHLC | Each of open, high and low standardized over the window, complementing the close z-score. | $z_k(X)_t$ for $X \in \{O, H, L\}$ | 30 | D: pandas-ta cdl_z |
