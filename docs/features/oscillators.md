# Oscillators

Back to the [catalog index](../feature_catalog.md) for notation. Oscillators are bounded or de-trended transforms of recent price changes; they are highly collinear with lagged returns, so include a handful per model, plus their 5-day change and their extremes.

## RSI family

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| RSI (Wilder) | Relative strength index: Wilder-smoothed average gain over average loss, mapped to 0 to 100. Above 70 is overbought, below 30 oversold. | $U_t = \max(C_t - C_{t-1}, 0)$, $D_t = \max(C_{t-1} - C_t, 0)$; $RS_t = RMA_k(U)_t / RMA_k(D)_t$; $RSI_t = 100 - \dfrac{100}{1 + RS_t}$ | $k = 2, 7, 14, 21$ | C: Wilder (1978); Chong and Ng (2008) mixed |
| Cutler's RSI | Same with simple averages (no recursion). | $RS_t = SMA_k(U)_t / SMA_k(D)_t$ | 14 | C: Cutler |
| RSI change | Five-day change in RSI; momentum of momentum. | $\Delta_5 RSI_t$ | 14 | D: derived |
| RSI extremes | Overbought and oversold flags and days since the last extreme. | $\mathbb{1}[RSI_t > 70]$, $\mathbb{1}[RSI_t < 30]$; $t - \max\{s : RSI_s > 70\}$ | 14 | C: Wilder (1978) |
| RSI centered | RSI rescaled to $[-1, 1]$ for models that prefer symmetric inputs. | $(RSI_t - 50)/50$ | | D: derived |
| Stochastic RSI | Stochastic oscillator applied to the RSI series; faster and more extreme than RSI. | $StochRSI_t = \dfrac{RSI_t - \min_n(RSI)_t}{\max_n(RSI)_t - \min_n(RSI)_t}$; $\%K = SMA_3(StochRSI)$, $\%D = SMA_3(\%K)$ | RSI 14, $n = 14$, 3, 3 | C: Chande and Kroll (1994) |
| Connors RSI | Composite of a short RSI, an RSI of the up/down streak, and the percentile rank of today's return. | $streak_t = streak_{t-1} + 1$ if $C_t > C_{t-1}$ and $streak_{t-1} > 0$ (else 1), $streak_{t-1} - 1$ if $C_t < C_{t-1}$ and $streak_{t-1} < 0$ (else $-1$), 0 if unchanged; $CRSI_t = \tfrac{1}{3}\big(RSI_3(C)_t + RSI_2(streak)_t + 100\,\operatorname{prank}_{100}(R^{(1)})_t\big)$ | 3, 2, 100 | C: Connors Research (2012) |
| RSX (Jurik RSI) | RSI computed through three cascaded two-stage exponential filters on the price change and its absolute value; same scale as RSI with far less noise. | $f_{18} = 3/(k+2)$, $f_{20} = 1 - f_{18}$; each stage $a_t = f_{20}a_{t-1} + f_{18}x_t$, $b_t = f_{18}a_t + f_{20}b_{t-1}$, out $= 1.5a - 0.5b$, cascaded three times on $100\,\Delta C$ and on $\lvert 100\,\Delta C \rvert$; $RSX_t = \operatorname{clip}\big(50(v_{14}/v_{20} + 1), 0, 100\big)$ | 14 | C: Jurik Research |
| QQE | Quantitative qualitative estimation: smoothed RSI with an ATR-of-RSI trailing band; a trend-following reading of RSI. | $rs_t = EMA_5(RSI_{14})$; $dar_t = 4.236\,EMA_{27}(EMA_{27}(\lvert rs_t - rs_{t-1} \rvert))$; trailing band from $rs \pm dar$ with Supertrend-style carry rules; features $rs_t - QQE_t$ and the side | 14, 5, 4.236 | C: trading-platform community indicator; no author |
| Laguerre RSI | Ehlers: RSI built from a four-stage Laguerre filter, very smooth. | $L0 = (1-\gamma)C_t + \gamma L0_{t-1}$; $L1 = -\gamma L0_t + L0_{t-1} + \gamma L1_{t-1}$; $L2, L3$ likewise; $CU = \sum \max(L_{i} - L_{i+1}, 0)$, $CD = \sum \max(L_{i+1} - L_i, 0)$; $LRSI = CU/(CU + CD)$ | $\gamma = 0.5$ | C: Ehlers (2004) |
| RSI divergence flag | Price makes a new 20-day high while RSI does not (bearish), and the inverse. | $\mathbb{1}[C_t = \max_{20} C \wedge RSI_t < \max_{20} RSI]$; mirror for lows | 20 | C: Wilder (1978) divergence rule |
| Dynamic momentum index | RSI whose length varies inversely with volatility. | $k_t = \operatorname{clip}\big(\lfloor 14 / (\operatorname{std}_5(C)_t / SMA_{10}(\operatorname{std}_5(C))_t) \rfloor, 5, 30\big)$; $DMI_t = RSI_{k_t}(C)_t$ | 14, 5, 10 | C: Chande and Kroll (1994) |

## Stochastic family

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Fast stochastic %K | Position of the close within the window's high-low range, 0 to 100. | $\%K_t = 100\,\dfrac{C_t - \min_k(L)_t}{\max_k(H)_t - \min_k(L)_t}$ | $k = 14$ | C: Lane (1984) |
| Slow stochastic %K, %D | Smoothed versions; the slow %K is the SMA3 of fast %K, %D the SMA3 of that. | $\%K^{slow} = SMA_3(\%K)$; $\%D = SMA_3(\%K^{slow})$ | 14, 3, 3 | C: Lane (1984) |
| Stochastic spread | %K minus %D, the crossover signal. | $\%K^{slow}_t - \%D_t$ | | D: derived |
| Williams %R | Inverse of the stochastic: position of the close from the window high, $-100$ to 0. | $\%R_t = -100\,\dfrac{\max_k(H)_t - C_t}{\max_k(H)_t - \min_k(L)_t}$ | $k = 14, 28$ | C: Williams (1979) |
| Stochastic momentum index | Blau: close relative to the mid of the range, double-smoothed and scaled, $-100$ to 100. | $d_t = C_t - (\max_k H + \min_k L)/2$; $hl_t = \max_k H - \min_k L$; $SMI_t = 100\,\dfrac{EMA_s(EMA_s(d))_t}{EMA_s(EMA_s(hl))_t / 2}$ | $k = 13$, $s = 25$ then 2 | C: Blau (1993) |
| Stochastic of a longer window | Same %K over a quarter and a year, as a position-in-range feature. | $\%K$ with $k = 63, 252$ | 63, 252 | D: derived |
| KDJ J-line | Slow stochastic with the J divergence line, which overshoots 0 to 100 on strong moves. | $K = RMA_3(\%K_9)$, $D = RMA_3(K)$, $J = 3K - 2D$ | 9, 3 | C: Lane (1984) variant |

## Rate-of-change oscillators

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Rate of change and momentum | Percentage and absolute change over $k$ days (TA-Lib ROC, ROCP, ROCR, MOM). | $ROC_k = 100\,(C_t/C_{t-k} - 1)$; $MOM_k = (C_t - C_{t-k})/C_t$ | $k = 10, 21$ | C: standard; evidence for momentum is in the returns file |
| CMO | Chande momentum oscillator: net gains over total movement, $-100$ to 100 (Chande's plain sums give $2\,RSI^{Cutler} - 100$; TA-Lib uses Wilder-smoothed sums, giving $2\,RSI - 100$). | $CMO_t = 100\,\dfrac{\sum_{i<k} U_{t-i} - \sum_{i<k} D_{t-i}}{\sum_{i<k} U_{t-i} + \sum_{i<k} D_{t-i}}$ | $k = 14$ | C: Chande (1994) |
| TRIX | One-day percentage rate of change of a triple-smoothed EMA; filters out short cycles. | $T_t = EMA_k(EMA_k(EMA_k(C)))_t$; $TRIX_t = 100\,(T_t/T_{t-1} - 1)$; signal $EMA_9(TRIX)$ | $k = 15$ | C: Hutson (1983) |
| TSI | True strength index: double-EMA-smoothed price change over double-smoothed absolute change, $-100$ to 100. | $m_t = C_t - C_{t-1}$; $TSI_t = 100\,\dfrac{EMA_s(EMA_r(m))_t}{EMA_s(EMA_r(\lvert m \rvert))_t}$; signal $EMA_7(TSI)$ (StockCharts; pandas-ta uses 13) | $r = 25$, $s = 13$ | C: Blau (1991) |
| KST | Pring's know sure thing: weighted sum of four smoothed rates of change. | $KST_t = \sum_{j=1}^{4} j \cdot SMA_{n_j}(ROC_{k_j})_t$ with $k = (10, 15, 20, 30)$, $n = (10, 10, 10, 15)$; signal $SMA_9(KST)$ | fixed | C: Pring (1992) |
| Coppock curve | Long-term momentum: WMA of the sum of two rates of change. | $Coppock_t = WMA_{10}(ROC_{14} + ROC_{11})_t$ | 14, 11, 10 (monthly in the original; use the same numbers on daily bars) | C: Coppock (1962) |
| Percentage price oscillator of momentum | PPO applied to the 21-day return series (momentum of momentum). | $PPO(r^{(21)})$ | 12, 26 | D: derived |
| Ultimate oscillator | Williams: weighted average of buying pressure over true range across three windows, 0 to 100. | $BP_t = C_t - \min(L_t, C_{t-1})$; $A_k = \sum_{i<k} BP_{t-i} / \sum_{i<k} TR_{t-i}$; $UO_t = 100\,\dfrac{4 A_7 + 2 A_{14} + A_{28}}{7}$ | 7, 14, 28 | C: Williams (1985) |
| SMI ergodic oscillator | TSI minus its signal line (the TSI histogram), as implemented in pandas-ta with fast and slow EMAs. | $TSI^{5,20}_t - EMA_5(TSI^{5,20})_t$ | 5, 20, 5 | C: Blau (1995) |
| Relative momentum index | RSI on $m$-day changes instead of one-day changes; smoother and more trend-like. | $u_t = \max(C_t - C_{t-m}, 0)$, $d_t = \max(C_{t-m} - C_t, 0)$; $RMI_t = 100 - \dfrac{100}{1 + EMA_k(u)/EMA_k(d)}$ | $k = 20$, $m = 5$ | C: Altman (1993) |
| Schaff trend cycle | MACD passed twice through a stochastic, 0 to 100; faster than MACD. | $M = EMA_{23} - EMA_{50}$; $S_1 = 100\,\dfrac{M - \min_{10} M}{\max_{10} M - \min_{10} M}$; $F_1 = EMA_{0.5}(S_1)$ (with $\alpha = 0.5$); $S_2$ = stochastic of $F_1$; $STC = EMA_{0.5}(S_2)$ | 23, 50, 10 | C: Schaff (2008) |

## Price-level oscillators

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| CCI | Commodity channel index: typical price against its SMA, scaled by mean absolute deviation; $\pm 100$ marks extremes. | $CCI_t = \dfrac{TP_t - SMA_k(TP)_t}{0.015\,\cdot\, \frac{1}{k}\sum_{i<k}\lvert TP_{t-i} - SMA_k(TP)_t \rvert}$ | $k = 20$ | C: Lambert (1980) |
| DPO | Detrended price oscillator: price against an SMA shifted back half a window, divided by price (removes the trend to expose the cycle). | $DPO_t = \big(C_{t - (k/2 + 1)} - SMA_k(C)_t\big) / C_t$ (TA-Lib and StockCharts align it this way so it uses no future data) | $k = 20$ | C: standard; StockCharts DPO |
| Awesome oscillator | Williams: difference of a 5- and a 34-period SMA of the mid price, divided by price. | $AO_t = (SMA_5(MP)_t - SMA_{34}(MP)_t)/C_t$ | 5, 34 | C: Williams (1995) |
| Accelerator oscillator | AO minus its own SMA5. | $AC_t = AO_t - SMA_5(AO)_t$ | 5 | C: Williams (1995) |
| Elder ray | Bull and bear power: high and low relative to a 13-day EMA. | $Bull_t = (H_t - EMA_{13}(C)_t)/C_t$; $Bear_t = (L_t - EMA_{13}(C)_t)/C_t$ | 13 | C: Elder (1993) |
| Price z-score | Close against its rolling mean in standard deviations. | $z_k(C)_t = (C_t - SMA_k(C)_t)/\operatorname{std}_k(C)_t$ | $k = 20, 63$ | D: derived; cf. Bollinger %B |
| Close percentile | Rank of today's close in the trailing window. | $\operatorname{prank}_k(C)_t$ | $k = 20, 63, 252$ | D: derived |
| Pretty good oscillator | Distance from the SMA in ATR units. | $PGO_t = (C_t - SMA_k(C)_t)/EMA_k(TR)_t$ | $k = 14$ | C: Johnson |
| Qstick | Chande and Kroll: average candle body; net buying pressure within sessions. | $Qstick_t = SMA_k(C - O)_t / C_t$ | $k = 8$ | C: Chande and Kroll (1994) |
| Balance of power | Where the close sits relative to the open within the day's range. | $BOP_t = (C_t - O_t)/(H_t - L_t)$; $SMA_{14}(BOP)$ | 1, 14 | C: Livshin (2001) |
| Psychological line | Share of up days in the window, 0 to 100. | $PSY_t = 100\,\frac{1}{k}\sum_{i<k}\mathbb{1}[C_{t-i} > C_{t-i-1}]$ | $k = 12$ | C: standard |
| Relative vigor index | Dorsey: close-minus-open relative to the range, both symmetrically smoothed over four bars and then averaged. | $num_t = \tfrac{1}{6}\big[(C-O)_t + 2(C-O)_{t-1} + 2(C-O)_{t-2} + (C-O)_{t-3}\big]$; $den_t$ same on $(H-L)$; $RVI_t = \sum_{i<k} num_{t-i} / \sum_{i<k} den_{t-i}$; signal $= \tfrac{1}{6}(RVI_t + 2RVI_{t-1} + 2RVI_{t-2} + RVI_{t-3})$ | $k = 10$ | C: Ehlers (2002) |
| Fisher transform | Ehlers: maps the normalized mid-price position to a near-Gaussian variable so turning points are sharper. | $x_t = 0.33 \cdot 2\Big(\dfrac{MP_t - \min_k MP}{\max_k MP - \min_k MP} - 0.5\Big) + 0.67\,x_{t-1}$, clipped to $\pm 0.999$; $F_t = 0.5\ln\dfrac{1 + x_t}{1 - x_t} + 0.5\,F_{t-1}$ | $k = 10$ | C: Ehlers (2002) |
| Inverse Fisher of RSI | Compresses RSI into $[-1, 1]$ with sharp extremes. | $y = 0.1\,(RSI_5 - 50)$; $IFT = \dfrac{e^{2y} - 1}{e^{2y} + 1}$ | 5 | C: Ehlers (2004) |
| Center of gravity | Ehlers: weighted mean position of recent prices. | $COG_t = -\dfrac{\sum_{i=0}^{k-1}(i+1)\,C_{t-i}}{\sum_{i<k} C_{t-i}}$ | $k = 10$ | C: Ehlers (2002) |
| Relative volatility index and inertia | Dorsey: the RSI formula applied to the rolling standard deviation instead of the price change; the share of volatility occurring on up days. Inertia is its linear-regression smoothing. | $s_t = \operatorname{std}_{14}(C)$; $u_t = s_t\mathbb{1}[C_t > C_{t-1}]$, $d_t = s_t\mathbb{1}[C_t < C_{t-1}]$; $RVI^{vol}_t = 100\,\dfrac{EMA_{14}(u)}{EMA_{14}(u) + EMA_{14}(d)}$; $Inertia_t = LINREG_{20}(RVI^{vol})$ | 14, 20 | C: Dorsey (1993) |
| TD Sequential setup count | DeMark: consecutive closes above (below) the close four bars earlier; a count of 9 is an exhaustion setup. | $up_t = (up_{t-1} + 1)\mathbb{1}[C_t > C_{t-4}]$; $dn_t = (dn_{t-1} + 1)\mathbb{1}[C_t < C_{t-4}]$; features $up_t - dn_t$ capped at $\pm 13$, flags $up_t = 9$, $dn_t = 9$ | lag 4 | C: DeMark (1994) |
| True momentum oscillator | Sum of close-versus-open signs over the window, triple smoothed. | $S_t = \sum_{i<k}\operatorname{sgn}(C_{t-i} - O_{t-i})$; $main = EMA_3(EMA_5(S))$; feature $main/k$ and $main_t - main_{t-k}$ | $k = 14$ | C: pandas-ta tmo |
| BRAR sentiment ratios | Buying versus selling pressure measured against the open (AR) and against the prior close (BR). | $AR_t = 100\,\dfrac{\sum_{i<k}(H - O)}{\sum_{i<k}(O - L)}$; $BR_t = 100\,\dfrac{\sum_{i<k}\max(H - C_{t-1}, 0)}{\sum_{i<k}\max(C_{t-1} - L, 0)}$ | $k = 26$ | C: pandas-ta brar |
| Squeeze momentum | The momentum histogram of Carter's TTM squeeze (the on/off flag is in the volatility file), plus the three-level squeeze state. | $mom_t = LINREG_{20}\big(C - \tfrac{1}{2}(\tfrac{1}{2}(\max_{20} H + \min_{20} L) + SMA_{20}(C))\big)/C_t$; state 0 to 3 from Bollinger inside Keltner at multipliers 2.0, 1.5, 1.0 | 20 | C: Carter (2005) |
| Traders dynamic index | RSI inside its own Bollinger-style band with two SMAs: RSI position relative to its recent volatility. | $rsi = RSI_{13}$; $(rsi - SMA_{34}(rsi))/(1.6185\,\operatorname{std}_{34}(rsi))$; $SMA_2(rsi) - SMA_7(rsi)$ | 13, 2, 7, 34 | C: Malone (2000s) |
| TKE composite | Equal-weight average of seven oscillators (RSI, %K, UO, MFI, %R + 100, ROC ratio, CCI); a cheap ensemble. | $TKE_t = \tfrac{1}{7}(RSI + \%K + UO + MFI + (\%R + 100) + ROCR100 + CCI)$ at length 14; signal $EMA_5$ | 14 | C: technical library TKE |
| Reflex and Trendflex | Ehlers (2020): deviation of a SuperSmoothed price from its $k$-bar linear trend (Reflex) or from its own $k$-bar mean (Trendflex), normalized by an exponentially weighted mean square; near-zero-lag cycle and trend oscillators. | $F = SSF_{20}(C)$; Reflex: $slope_t = (F_{t-k} - F_t)/k$, $S_t = \tfrac{1}{k}\sum_{j=1}^{k-1}(F_t - F_{t-j} + j\,slope_t)$; Trendflex: $S_t = \tfrac{1}{k}\sum_{j=1}^{k-1}(F_t - F_{t-j})$; $MS_t = 0.04 S_t^2 + 0.96 MS_{t-1}$; out $= S_t/\sqrt{MS_t}$ | $k = 20$ | C: Ehlers (2020) |
| Even better sine wave | Ehlers: high-pass filter then SuperSmoother, normalized by a 3-bar RMS; a bounded $[-1, 1]$ cycle indicator. | $a_1 = (1 - \sin(2\pi/k))/\cos(2\pi/k)$; $HP_t = \tfrac{1}{2}(1 + a_1)(C_t - C_{t-1}) + a_1 HP_{t-1}$; $F = SSF_{10}(HP)$; $Wave_t = \dfrac{\mu_3(F)_t}{\sqrt{\mu_3(F^2)_t}}$ | $k = 40$ | C: Ehlers (2013) |
| Chande Kroll stop distance | Distance to a volatility-based trailing stop. | $stop^{long}_t = \max_q\big(\max_p H - x\,ATR_p\big)$; feature $(C_t - stop^{long}_t)/C_t$ | $p = 10$, $x = 1$, $q = 9$ | C: Chande and Kroll (1994) |
| Ehlers' Hilbert dominant cycle | TA-Lib HT_DCPERIOD: estimated dominant cycle length in bars (6 to 50) from a Hilbert transform of the mid price; short periods flag choppy regimes. | Smooth $= (4MP_t + 3MP_{t-1} + 2MP_{t-2} + MP_{t-3})/10$; Hilbert operator $HT[x]_t = (0.0962x_t + 0.5769x_{t-2} - 0.5769x_{t-4} - 0.0962x_{t-6})(0.075\,Period_{t-1} + 0.54)$; in-phase and quadrature components $I, Q$ from it; $Period_t = 360/\arctan(Im/Re)$ clamped to $[0.67, 1.5]\times Period_{t-1}$ and to $[6, 50]$, then smoothed $0.2/0.8$ and $0.33/0.67$; also its 21-day change | warm up 63 bars | C: Ehlers (2001); TA-Lib HT_DCPERIOD |
| Hilbert phase and phasor | TA-Lib HT_DCPHASE and HT_PHASOR: where in the cycle price sits (0 to 360 degrees) and the in-phase and quadrature components; encode the phase as sine and cosine. | $\phi_t$ from $\arctan(Re/Im)$ of the dominant-cycle DFT of the smoothed price with the half-bar lead $360/Period_t$; $\sin\phi_t$, $\cos\phi_t$, $\sqrt{I_t^2 + Q_t^2}/C_t$ | fixed | C: Ehlers (2001); TA-Lib HT_DCPHASE, HT_PHASOR |
| Hilbert trend mode | TA-Lib HT_TRENDMODE: 1 if the series is in trend mode, 0 if in cycle mode. | TA-Lib algorithm | fixed | C: Ehlers (2001); TA-Lib HT_TRENDMODE |
| Sine wave indicator | TA-Lib HT_SINE: sine and lead sine of the dominant cycle phase; crossovers mark cycle turns. | $\sin(\phi_t)$, $\sin(\phi_t + 45^{\circ})$ with $\phi$ the Hilbert phase | fixed | C: Ehlers (2001); TA-Lib HT_SINE |
