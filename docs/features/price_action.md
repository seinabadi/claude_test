# Price action and gaps

Back to the [catalog index](../feature_catalog.md) for notation. Range-normalized quantities are undefined when $H_t = L_t$; set them to 0 and add a flag. Round-number and pivot distances use unadjusted prices because traders see those levels; everything else uses adjusted prices.

## Candle anatomy

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Candle body | Open-to-close change, the day-session return in price terms. | $body_t = (C_t - O_t)/O_t$ | lags 0 to 3 | D: derived |
| Body to range | Share of the day's range covered by the body; near 0 is a doji, near 1 a marubozu. | $\lvert C_t - O_t \rvert / (H_t - L_t)$ | 1 | C: Nison (1991) |
| Upper shadow | Upper wick as a share of the range; selling into strength. | $(H_t - \max(O_t, C_t))/(H_t - L_t)$ | 1 | C: Nison (1991) |
| Lower shadow | Lower wick as a share of the range; buying on weakness. | $(\min(O_t, C_t) - L_t)/(H_t - L_t)$ | 1 | C: Nison (1991) |
| Close location value | Where the close sits in the range, $-1$ to 1, and its 5-day mean. | $CLV_t = \dfrac{(C_t - L_t) - (H_t - C_t)}{H_t - L_t}$; $\mu_5(CLV)$ | 1, 5 | C: Chaikin; internal bar strength (practitioner) |
| Open location value | Where the open sat in the range. | $(O_t - L_t)/(H_t - L_t)$ | 1 | D: derived |
| Range relative to ATR | Today's range as a multiple of average range. | $(H_t - L_t)/ATR_{14}(t)$ | 14 | D: derived |
| Body relative to ATR | Body size in ATR units. | $(C_t - O_t)/ATR_{14}(t)$ | 14 | D: derived |
| Candle color streak | Consecutive days with $C > O$ (positive) or $C < O$ (negative). | signed count | current | D: derived |
| Three-day sign code | Ordinal code of the signs of the last three daily returns, 8 states. | $4\,\mathbb{1}[r_t > 0] + 2\,\mathbb{1}[r_{t-1} > 0] + \mathbb{1}[r_{t-2} > 0]$ | 3 | D: derived |
| Average body and wick ratios | Mean body-to-range and wick shares over the window: how decisive sessions have been. | $\mu_k(\lvert C - O \rvert/(H - L))$ | $k = 5, 21$ | D: derived |

## Gaps

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Open gap | Overnight gap as a return and in ATR units, plus lags. | $o_t = \ln(O_t/C_{t-1})$; $(O_t - C_{t-1})/ATR_{14}(t-1)$ | lags 0 to 3 | D: derived; cf. Lou, Polk and Skouras (2019) |
| Gap type | Full gap up (open above the prior high), partial gap up (above the prior close only), and mirror for down; a 5-state code. | $+2$ if $O_t > H_{t-1}$; $+1$ if $H_{t-1} \ge O_t > C_{t-1}$; $-1$, $-2$ mirrored; 0 if $O_t = C_{t-1}$ | 1 | C: StockCharts gap classification |
| Gap fill flag | Price traded back to the prior close during the day. | $\mathbb{1}[L_t \le C_{t-1}]$ for gap-ups, $\mathbb{1}[H_t \ge C_{t-1}]$ for gap-downs | 1 | C: practitioner |
| Gap fill fraction | Share of the gap retraced by the close. | $(O_t - C_t)/(O_t - C_{t-1})$ clipped to $[-1, 2]$ | 1 | D: derived |
| Gap continuation | Whether the day session continued in the gap's direction. | $\operatorname{sgn}(o_t)\operatorname{sgn}(c_t)$ | 1 | D: derived |
| Gap count | Gaps larger than one ATR in the window. | $\sum_{i<k}\mathbb{1}[\lvert O_{t-i} - C_{t-i-1} \rvert > ATR_{14}(t-i-1)]$ | $k = 21, 63$ | D: derived |
| Unfilled gap distance | Distance to the nearest unfilled gap level within the last 63 days (gap edges attract price). | $\min_g \lvert C_t - G_g \rvert / C_t$ over unfilled gap edges $G_g$ | 63 | C: practitioner |
| Days since last gap | Trading days since the last gap larger than one ATR. | count | current | D: derived |

## Range relationships and breakouts

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Close vs prior range | Close above the prior high ($+1$), below the prior low ($-1$) or inside (0). | $\mathbb{1}[C_t > H_{t-1}] - \mathbb{1}[C_t < L_{t-1}]$ | 1 | D: derived |
| Inside day | Today's range entirely inside yesterday's; compression. | $\mathbb{1}[H_t \le H_{t-1} \wedge L_t \ge L_{t-1}]$ | 1 | C: Crabel (1990) |
| Outside day | Today's range engulfs yesterday's; expansion. | $\mathbb{1}[H_t > H_{t-1} \wedge L_t < L_{t-1}]$ | 1 | C: Crabel (1990) |
| NR4 and NR7 | Narrowest range of the last 4 or 7 days (Crabel); volatility contraction precedes expansion. | $\mathbb{1}[(H_t - L_t) = \min_{n}(H - L)_t]$ | $n = 4, 7$ | C: Crabel (1990) |
| Inside-day plus NR4 | Crabel's ID/NR4 setup. | $\mathbb{1}[\text{inside}_t \wedge NR4_t]$ | | C: Crabel (1990) |
| Wide-range bar | Range above twice the ATR. | $\mathbb{1}[(H_t - L_t) > 2\,ATR_{14}(t)]$ | 14 | C: Crabel (1990) |
| Wide-range reversal | Wide range with a close in the opposite third of the range from the open. | $WRB_t \wedge \lvert CLV_t - OLV_t \rvert$ large with opposite signs, where $OLV$ is the open location in $[-1,1]$ | 1 | C: practitioner |
| New high and new low flags | Close at the highest close or above the highest high of the window. | $\mathbb{1}[C_t > \max_k(H)_{t-1}]$; $\mathbb{1}[C_t < \min_k(L)_{t-1}]$ | $k = 5, 10, 20, 50, 252$ | A: George and Hwang (2004) for the 252-day window; shorter windows are Donchian (C) |
| Days since new high or low | Trading days since the last window high and low. | count | $k = 20, 50, 252$ | A: Bhootra and Hur (2013) |
| Higher-high and higher-low counts | Days in the window with a higher high or higher low than the day before (trend structure). | $\sum_{i<k}\mathbb{1}[H_{t-i} > H_{t-i-1}]$; $\sum_{i<k}\mathbb{1}[L_{t-i} > L_{t-i-1}]$ | $k = 5, 10$ | D: derived |
| Breakout with volume | Donchian breakout confirmed by volume. | $\mathbb{1}[C_t > \max_{20}(H)_{t-1} \wedge V_t > 1.5\,\mu_{21}(V)_t]$ | 20 | C: practitioner |
| Failed breakout | Close back inside the channel within 3 days of a breakout. | $\mathbb{1}[\exists j \le 3: \text{breakout}_{t-j} \wedge C_t < \max_{20}(H)_{t-j-1}]$ | 20 | C: practitioner |
| Range position of the week and month | Where the close sits in the current week's and month's range so far. | $(C_t - \min L_{wk})/(\max H_{wk} - \min L_{wk})$ | | D: derived |
| Range vol ratio | Range-based volatility against close-to-close volatility; a high ratio means intraday swings without net progress. | $\sigma^{Park}_{21}(t)/\sigma_{21}(t)$ | 21 | D: derived |
| Mean-reversion distance | Close against the 20-day SMA in ATR units. | $(C_t - SMA_{20}(C)_t)/ATR_{14}(t)$ | 20, 14 | D: derived |

## Levels: pivots, swings, retracements

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Classic pivot points | Floor-trader levels from yesterday's OHLC; distance of the close to each. | $P = TP_{t-1}$; $R_1 = 2P - L_{t-1}$, $S_1 = 2P - H_{t-1}$; $R_2 = P + (H_{t-1} - L_{t-1})$, $S_2 = P - (H_{t-1} - L_{t-1})$; $R_3 = H_{t-1} + 2(P - L_{t-1})$, $S_3 = L_{t-1} - 2(H_{t-1} - P)$; feature $(C_t - X)/C_t$ | 1 | C: floor-trader pivots; Person (2004) |
| Fibonacci pivots | Pivot plus Fibonacci multiples of yesterday's range. | $R_1 = P + 0.382\,(H - L)_{t-1}$, $R_2 = P + 0.618\,(H-L)_{t-1}$, $R_3 = P + (H-L)_{t-1}$; $S$ mirrored | 1 | C: practitioner |
| Camarilla pivots | Tight intraday levels. | $R_n = C_{t-1} + k_n (H - L)_{t-1}$, $S_n = C_{t-1} - k_n (H-L)_{t-1}$ with $k = (1.1/12, 1.1/6, 1.1/4, 1.1/2)$ | 1 | C: Scott (1989) |
| Woodie pivot | Pivot weighted toward the close. | $P = (H_{t-1} + L_{t-1} + 2\,C_{t-1})/4$ | 1 | C: practitioner |
| DeMark pivot | Pivot that depends on the open-close relation. | $X = H + 2L + C$ if $C < O$; $2H + L + C$ if $C > O$; $H + L + 2C$ if $C = O$ (all at $t-1$); $P = X/4$, $R_1 = X/2 - L_{t-1}$, $S_1 = X/2 - H_{t-1}$ | 1 | C: DeMark (1994) |
| Nearest pivot distance | Distance to the closest of the classic levels, in ATR units, and which one. | $\min_X \lvert C_t - X \rvert / ATR_{14}$ | 1 | D: derived |
| Zigzag pivot distance | Distance from the last confirmed swing pivot: a pivot high is confirmed once price has fallen at least $\theta$ from the running maximum (mirror for lows). Use only pivots confirmed by day $t$; the plotted zigzag repaints and leaks. | $C_t / Z_t - 1$, $Z_t$ the last confirmed pivot price; also days since confirmation | $\theta = 3\%, 5\%$ | C: standard zigzag |
| Swing structure | Whether the last two swing highs and lows are rising (uptrend), falling or mixed. | $+1$ if $ZH_1 > ZH_2 \wedge ZL_1 > ZL_2$; $-1$ if both lower; else 0 | $\theta = 5\%$ | C: Dow theory |
| Support and resistance proximity | Distance to the nearest local minimum and maximum of the last 63 days (a local extreme is a high or low not exceeded within $\pm 5$ days), in ATR units. | $\min_j (C_t - S_j)/ATR_{14}$ over supports below; $\min_j (R_j - C_t)/ATR_{14}$ over resistances above | 63, 5 | C: Osler (2000) finds predictive content in published support and resistance levels |
| Level touch count | Number of times price came within 0.5 ATR of the nearest level in the window (tested levels matter more). | count | 63 | D: derived |
| Fibonacci position | Position within the 63-day range and distance to the nearest retracement level. | $f_t = (C_t - \min_{63} L)/(\max_{63} H - \min_{63} L)$; $\min_{\ell \in \{0.236, 0.382, 0.5, 0.618, 0.786\}} \lvert f_t - \ell \rvert$ | 63 | C: practitioner; no evidence |
| Round-number distance | Distance of the unadjusted close to the nearest multiple of 5, 10, 50 and 100 dollars. | $\min_m \lvert P_t - m\,\rho \rvert / P_t$ for $\rho \in \{5, 10, 50, 100\}$ | 1 | C: Harris (1991); Osler (2003) order clustering at round numbers |
| Prior-day and prior-week levels | Distance to yesterday's high, low and close and to last week's high and low. | $(C_t - X)/ATR_{14}$ | 1, 5 | C: practitioner |
| Monthly and yearly open distance | Distance from the first open of the month and of the year. | $C_t / O_{month} - 1$; $C_t / O_{year} - 1$ | | D: derived |
| Volume-profile point of control | Price level with the most volume over the window (from daily TP and V binned at 0.5 ATR), and the distance to it. | $POC_t = \arg\max_{bin}\sum_{i<k} V_{t-i}\mathbb{1}[TP_{t-i} \in bin]$; $C_t/POC_t - 1$ | 63 | C: Steidlmayer (1984) |

## Candlestick patterns

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| TA-Lib pattern flags | The 61 CDL functions (doji, dragonfly and gravestone doji, hammer, inverted hammer, hanging man, shooting star, marubozu, spinning top, engulfing, harami and harami cross, piercing, dark cloud cover, morning and evening star, three white soldiers, three black crows, three inside/outside, abandoned baby, kicking, tasuki gap, hikkake, belt hold, and others), each in $\{-100, -80, 0, 80, 100\}$; divide by 100. TA-Lib's thresholds: long body = average body over 10 bars, short body = that average, doji body = $0.1\times$ average range over 10, very short shadow = $0.1\times$ average range, long shadow = the bar's own body, near = $0.2\times$ average range over 5, far = $0.6\times$ average range over 5. | library output | 1 | C: Nison (1991); Marshall, Young and Rose (2006) find no profitability |
| Doji | Body smaller than 10% of the range. | $\mathbb{1}[\lvert C_t - O_t \rvert \le 0.1\,(H_t - L_t)]$ | 1 | C: Nison (1991); Marshall, Young and Rose (2006) |
| Hammer / hanging man | Small body at the top, lower shadow at least twice the body, little upper shadow; after a decline it is a hammer, after an advance a hanging man. | $\mathbb{1}[\text{lower}_t \ge 2\lvert C_t - O_t \rvert \wedge \text{upper}_t \le 0.1(H_t - L_t)]$ times $-\operatorname{sgn}(r^{(5)}_{t-1})$ | 1 | C: Nison (1991); Marshall, Young and Rose (2006) |
| Shooting star / inverted hammer | Mirror of the above with a long upper shadow. | analogous | 1 | C: Nison (1991); Marshall, Young and Rose (2006) |
| Engulfing | Today's body engulfs yesterday's with opposite color. | bullish: $\mathbb{1}[C_{t-1} < O_{t-1} \wedge C_t > O_t \wedge O_t \le C_{t-1} \wedge C_t \ge O_{t-1}]$; bearish mirrored | 1 | C: Nison (1991); Marshall, Young and Rose (2006) |
| Harami | Today's body inside yesterday's body with opposite color. | $\mathbb{1}[\max(O_t, C_t) \le \max(O_{t-1}, C_{t-1}) \wedge \min(O_t, C_t) \ge \min(O_{t-1}, C_{t-1})]$ times $-\operatorname{sgn}(C_{t-1} - O_{t-1})$ | 1 | C: Nison (1991); Marshall, Young and Rose (2006) |
| Piercing / dark cloud | Open beyond yesterday's extreme and close past yesterday's midpoint. | piercing: $\mathbb{1}[C_{t-1} < O_{t-1} \wedge O_t < L_{t-1} \wedge C_t > (O_{t-1} + C_{t-1})/2 \wedge C_t < O_{t-1}]$ | 1 | C: Nison (1991); Marshall, Young and Rose (2006) |
| Morning / evening star | Three-bar reversal: long body, small gapped body, long body back into the first. | TA-Lib CDLMORNINGSTAR, CDLEVENINGSTAR with penetration 0.3 | 3 | C: Nison (1991); Marshall, Young and Rose (2006) |
| Three soldiers / three crows | Three consecutive long bodies in the same direction, each opening inside the prior body and closing beyond it. | TA-Lib CDL3WHITESOLDIERS, CDL3BLACKCROWS | 3 | C: Nison (1991); Marshall, Young and Rose (2006) |
| Pattern count | Number of bullish minus bearish TA-Lib patterns firing today. | $\sum_p \operatorname{sgn}(CDL_p(t))$ | 1 | D: derived |

## Structure, imbalance and flag decay

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Williams fractals | Five-bar swing highs and lows; a fractal at $t-2$ is only confirmable at $t$, so use the lagged flag and the distance to the last confirmed fractal. | $FH_{t-2} = \mathbb{1}[H_{t-2} > \max(H_{t-4}, H_{t-3}, H_{t-1}, H_t)]$; $FL$ mirrored; features: days since the last fractal high and low, $C_t/FH^{level} - 1$ | 2 bars each side | C: Williams (1995) |
| Fair value gap | Three-bar imbalance: bar $t-2$'s high below bar $t$'s low (bullish) or its low above bar $t$'s high (bearish), with a large middle bar; unfilled gaps attract price. | $FVG^{+}_t = \mathbb{1}[L_t > H_{t-2} \wedge \lvert C_{t-1} - O_{t-1} \rvert > 1.5\,ATR_{14}]$; $FVG^-$ mirrored; gap size $/ATR_{14}$; days since the last unfilled gap | 14, 1.5 | C: practitioner (smart money concepts) |
| Z-body doji | Body relative to the rolling standard deviation of bodies rather than to the range. | $\mathbb{1}[\lvert C_t - O_t \rvert < 0.1\,\operatorname{std}_{10}(\lvert C - O \rvert)]$; $z_{10}(C - O)_t$ | 10 | C: pandas-ta cdl_doji |
| Decayed event flags | Standard way to turn sparse flags (patterns, breakouts, gaps, fractals) into dense features: linear or exponential decay since the last event. | linear $D_t = \max(x_t, D_{t-1} - 1/k, 0)$; exponential $D_t = \max(x_t, D_{t-1}(1 - 1/k), 0)$ | $k = 5, 10$ | D: Tulip decay; feature-engineering device |
