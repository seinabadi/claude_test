# Volume and liquidity

Back to the [catalog index](../feature_catalog.md) for notation. Volume is never used as a level: always relative to its own history, in dollar terms relative to history, or as a signed money-flow accumulation normalized by average volume. $DV_t = C_t V_t$ is dollar volume and $CLV_t = \dfrac{(C_t - L_t) - (H_t - C_t)}{H_t - L_t}$ is the close location value in $[-1, 1]$.

## Volume activity

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Log volume and log dollar volume | Levels in logs, for use only after z-scoring within the asset. | $\ln V_t$; $\ln DV_t$ | |
| Relative volume | Today's volume against its trailing mean. | $RV_k(t) = V_t / \mu_k(V)_t$ | $k = 5, 10, 21, 63$ |
| Volume z-score | Standardized volume. | $z_k(V)_t$ | $k = 21, 63$ |
| Volume percentile | Rank of today's volume in the trailing window. | $\operatorname{prank}_k(V)_t$ | $k = 63, 252$ |
| Volume spike flag | Volume more than twice its 21-day mean. | $\mathbb{1}[V_t > 2\mu_{21}(V)_t]$ | 21 |
| Volume trend | Slope of log volume on time over the window, and the ratio of short to long mean volume. | OLS slope of $\ln V$ over $k$ days; $\mu_5(V)_t/\mu_{21}(V)_t - 1$ | 21 |
| Volume oscillator | Percentage gap between a fast and a slow volume average (TradingView uses EMA 5 and 10). | $VO_t = 100\,\dfrac{SMA_5(V)_t - SMA_{20}(V)_t}{SMA_{20}(V)_t}$ | 5, 20 |
| PVO | Percentage volume oscillator: MACD applied to volume. | $PVO_t = 100\,\dfrac{EMA_{12}(V)_t - EMA_{26}(V)_t}{EMA_{26}(V)_t}$; signal $EMA_9$; histogram | 12, 26, 9 |
| Volume RSI | RSI applied to volume: share of recent volume that came on up days. | RSI formula with $U_t = V_t\mathbb{1}[C_t > C_{t-1}]$, $D_t = V_t\mathbb{1}[C_t < C_{t-1}]$ | 14 |
| Same-weekday relative volume | Volume against the mean volume on the same weekday over the last 12 weeks; removes the weekday pattern. | $V_t / \frac{1}{12}\sum_{j=1}^{12} V_{t - 5j}$ | 60 |
| Volume autocorrelation | Persistence of volume. | $\operatorname{corr}_{63}(\ln V_t, \ln V_{t-1})$ | 63 |
| Volume-weighted return | Today's return weighted by relative volume: moves on heavy volume carry more information. | $r_t \cdot V_t / \mu_{21}(V)_t$ | 21 |
| Volume-return correlation | Correlation of absolute returns with volume (the volume-volatility relation). | $\operatorname{corr}_k(\lvert r \rvert, V)_t$ | $k = 21, 63$ |
| Signed volume correlation | Correlation of signed returns with relative volume: positive when up days have the heavier volume. | $\operatorname{corr}_k(r, V/\mu_{21}(V))_t$ | 21, 63 |
| Up minus down volume | Net volume on up days as a share of total volume. | $\dfrac{\sum_{i<k} V_{t-i}\operatorname{sgn}(r_{t-i})}{\sum_{i<k} V_{t-i}}$ | $k = 10, 21$ |
| Price-volume divergence flag | Price rises over 5 days while the volume trend falls, and the inverse. | $\mathbb{1}[r^{(5)}_t > 0 \wedge \mu_5(V)_t < \mu_{21}(V)_t]$; mirror | 5, 21 |
| Volume at new highs | Relative volume on days that set a 63-day high (confirmation of breakouts). | $RV_{21}(t)\,\mathbb{1}[C_t = \max_{63} C]$ | 63 |
| Price-volume rank | Blau: sum of the ranks of the price change and the volume change, 1 to 4 (1 = price up on rising volume). | $PVR_t = 1$ if $\Delta C > 0, \Delta V > 0$; 2 if $\Delta C > 0, \Delta V \le 0$; 3 if $\Delta C \le 0, \Delta V \le 0$; 4 if $\Delta C \le 0, \Delta V > 0$ | 1 |

## Accumulation and money flow

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| On-balance volume | Granville: running sum of signed volume; normalized by average volume and summarized by its slope. | $OBV_t = OBV_{t-1} + V_t\operatorname{sgn}(C_t - C_{t-1})$; features $(OBV_t - OBV_{t-k})/\mu_{21}(V)_t$ and the OLS slope of $OBV / \mu_{21}(V)$ over 10 days | 10, 21 |
| OBV divergence | Price at a 20-day high while OBV is not. | $\mathbb{1}[C_t = \max_{20} C \wedge OBV_t < \max_{20} OBV]$ | 20 |
| Accumulation/distribution line | Chaikin: cumulative close-location-weighted volume. | $ADL_t = ADL_{t-1} + CLV_t V_t$; features normalized like OBV | 21 |
| Chaikin money flow | Mean CLV weighted by volume over the window, $-1$ to 1. | $CMF_t = \dfrac{\sum_{i<k} CLV_{t-i} V_{t-i}}{\sum_{i<k} V_{t-i}}$ | $k = 20$ |
| Chaikin oscillator | MACD of the A/D line, divided by average volume. | $CO_t = (EMA_3(ADL)_t - EMA_{10}(ADL)_t)/\mu_{21}(V)_t$ | 3, 10 |
| Money flow index | Volume-weighted RSI on typical price, 0 to 100. | $MF_t = TP_t V_t$; $PMF_k = \sum_{i<k} MF_{t-i}\mathbb{1}[TP_{t-i} > TP_{t-i-1}]$, $NMF_k$ for down days; $MFI_t = 100 - \dfrac{100}{1 + PMF_k/NMF_k}$ | $k = 14$ |
| Force index | Elder: price change times volume, EMA-smoothed; divided by price times average volume. | $FI_t = (C_t - C_{t-1}) V_t$; feature $EMA_k(FI)_t / (C_t \mu_{21}(V)_t)$ | $k = 2, 13$ |
| Ease of movement | Arms: how far price moves per unit of volume relative to the range (scaled so values are readable). | $EMV_t = \dfrac{MP_t - MP_{t-1}}{V_t / (H_t - L_t)} \times \text{scale}$; feature $SMA_{14}(EMV)/C_t$ with scale $10^8$ | 14 |
| Volume price trend | Cumulative percentage change weighted by volume. | $VPT_t = VPT_{t-1} + V_t (C_t/C_{t-1} - 1)$; normalized like OBV | 21 |
| Negative volume index | Fosback: cumulates price changes only on days when volume fell (informed trading on quiet days); as a ratio to its 255-day EMA. | $NVI_t = NVI_{t-1}\big(1 + (C_t/C_{t-1} - 1)\mathbb{1}[V_t < V_{t-1}]\big)$; feature $NVI_t / EMA_{255}(NVI)_t - 1$ | 255 |
| Positive volume index | Same on rising-volume days. | $PVI_t = PVI_{t-1}\big(1 + (C_t/C_{t-1} - 1)\mathbb{1}[V_t > V_{t-1}]\big)$ | 255 |
| Klinger volume oscillator | Volume force (signed, trend- and range-adjusted volume) through two EMAs. | $trend_t = \operatorname{sgn}(HLC_t - HLC_{t-1})$ with $HLC = H + L + C$; $dm_t = H_t - L_t$; $cm_t = cm_{t-1} + dm_t$ if $trend_t = trend_{t-1}$ else $dm_{t-1} + dm_t$; $VF_t = V_t \cdot \lvert 2\,dm_t/cm_t - 1 \rvert \cdot trend_t \cdot 100$; $KVO_t = EMA_{34}(VF)_t - EMA_{55}(VF)_t$, signal $EMA_{13}(KVO)$; divide by $\mu_{21}(V)$ | 34, 55, 13 |
| Twiggs money flow | CMF with true-range-based CLV and Wilder smoothing (robust to gaps). | $TRH = \max(H_t, C_{t-1})$, $TRL = \min(L_t, C_{t-1})$; $AD_t = V_t \dfrac{(C_t - TRL) - (TRH - C_t)}{TRH - TRL}$; $TMF_t = RMA_{21}(AD)_t / RMA_{21}(V)_t$ | 21 |
| Williams accumulation/distribution | Williams: cumulative true-range-based price change. | $WAD_t = WAD_{t-1} + (C_t - \min(L_t, C_{t-1}))$ if $C_t > C_{t-1}$; $+ (C_t - \max(H_t, C_{t-1}))$ if $C_t < C_{t-1}$; else $+0$; normalize by $C_t$ | |
| Volume-weighted MACD | MACD on the VWMA instead of the EMA. | $(VWMA_{12} - VWMA_{26})/C_t$ | 12, 26 |
| Money flow ratio | Dollar volume on up days over dollar volume on down days. | $\dfrac{\sum_{i<k} DV_{t-i}\mathbb{1}[r_{t-i} > 0]}{\sum_{i<k} DV_{t-i}\mathbb{1}[r_{t-i} < 0]}$ | 21 |

| Volume flow indicator | Markos Katsanos: money-flow sum where volume is capped and only counted when the typical-price change exceeds a volatility cutoff; a robust OBV and CMF hybrid. | $inter_t = \ln TP_t - \ln TP_{t-1}$, $cut_t = 0.2\,\operatorname{std}_{30}(inter)\,C_t$; $vc_t = \min(V_t, 2.5\,SMA_k(V)_{t-1})$; $vcp_t = +vc_t$ if $TP_t - TP_{t-1} > cut_t$, $-vc_t$ if $< -cut_t$, else 0; $VFI_t = \sum_{i<k} vcp_{t-i}/SMA_k(V)_{t-1}$ | $k = 130$ |
| Volume-price confirmation indicator | Dormeier: product of the VWMA-versus-SMA spread, their short ratio and a volume multiplier; positive when price rises on heavy volume. | $VPC = VWMA_{20} - SMA_{20}$; $VPR = VWMA_5/SMA_5$; $VM = SMA_5(V)/SMA_{20}(V)$; $VPCI_t = VPC \cdot VPR \cdot VM / C_t$ | 5, 20 |
| Time segmented volume | Sum of volume times price change: OBV weighted by the size of the move. | $TSV_t = \sum_{i<k} V_{t-i}(C_{t-i} - C_{t-i-1})$, divided by $C_t\,\mu_{21}(V)$; ratio to $SMA_{10}(TSV)$ | $k = 18$ |
| Market facilitation index | Bill Williams: range per unit of volume; with the volume change it gives four states (green, fade, fake, squat). | $MFI^{W}_t = (H_t - L_t)/V_t$; $MFI^W_t/SMA_{21}(MFI^W)$; state from $(\operatorname{sgn}\Delta MFI^W, \operatorname{sgn}\Delta V)$ | 21 |
| Value area position | Rolling volume profile: whether the close sits inside the 70% value area around the point of control. | $VA$ = smallest set of price bins around $POC$ holding 70% of the window's volume; $(C_t - VA_{low})/(VA_{high} - VA_{low})$; $\mathbb{1}[C_t \in VA]$ | 63, 10 to 24 bins |

## Liquidity and transaction-cost proxies

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Amihud illiquidity | Amihud (2002): absolute return per dollar traded; price impact of a dollar of volume. | $ILLIQ_k(t) = \dfrac{1}{k}\sum_{i<k}\dfrac{\lvert r_{t-i} \rvert}{DV_{t-i}}$, usually times $10^6$ and logged | $k = 21, 63$ |
| Amihud z-score and change | Illiquidity against its own history. | $z_{252}(\ln ILLIQ_{21})_t$; $\ln(ILLIQ_{21,t}/ILLIQ_{21,t-21})$ | |
| Turnover | Volume over shares outstanding (or ETF shares outstanding). | $TO_t = V_t / N_t$; $\mu_{21}(TO)$ | 1, 21 |
| Turnover and dollar-volume variability | Chordia, Subrahmanyam and Anshuman (2001): variability of trading activity predicts lower returns (GKX `std_turn`, `std_dolvol`). | $\operatorname{std}_{21}(TO)_t$; $\operatorname{std}_{21}(\ln DV)_t$; coefficient of variation $\operatorname{std}_{126}(TO)/\mu_{126}(TO)$ | 21, 126 |
| High-volume return premium | Gervais, Kaniel and Mingelgrin (2001): a day in the top decile of the last 50 days' volume predicts higher returns over the next 20 to 100 days; bottom decile the opposite. | $\mathbb{1}[V_t \ge Q_{0.9}(V_{t-49..t})]$; $\mathbb{1}[V_t \le Q_{0.1}(V_{t-49..t})]$ | 50 |
| Volume change | Kakushadze's `delta(log(volume), d)`. | $\ln V_t - \ln V_{t-d}$ | $d = 1, 2, 5$ |
| Rank correlation of price and volume | Kakushadze alphas 3, 6, 13, 44: short-window correlation between price-level ranks and volume ranks, negated; captures whether volume is chasing price. | $-\operatorname{corr}_d\big(\operatorname{prank}_{252}(x), \operatorname{prank}_{252}(V)\big)_t$ for $x \in \{O, H, C, r\}$ | $d = 5, 10, 20$ |
| Dollar volume relative to market | Share of SPY dollar volume. | $DV_t / DV^{SPY}_t$ | |
| Roll spread | Roll (1984): effective spread implied by the negative autocovariance of price changes. | $S^{Roll}_t = 2\sqrt{-\operatorname{cov}_k(\Delta C_t, \Delta C_{t-1})}$ when the covariance is negative, else 0; divide by $C_t$ | $k = 21, 63$ |
| Roll noise and impact | Companions of the Roll model: the fundamental (efficient-price) variance and the spread per dollar traded. | $\sigma^2_u = \operatorname{var}_k(\Delta C) + 2\operatorname{cov}_k(\Delta C_t, \Delta C_{t-1})$; $S^{Roll}_t / \mu_k(DV)_t$ | 21, 63 |
| Becker-Parkinson volatility | High-low volatility estimate from the Corwin-Schultz $\beta, \gamma$ terms (Lopez de Prado 2018, ch. 19). | $\sigma^{HL} = \dfrac{(2^{-1/2} - 1)\sqrt{\beta}}{k_2(3 - 2\sqrt 2)} + \sqrt{\dfrac{\gamma}{k_2^2(3 - 2\sqrt 2)}}$, $k_2 = \sqrt{8/\pi}$ | 1, 21 |
| Corwin-Schultz spread | Corwin and Schultz (2012): bid-ask spread from the ratio of two-day to one-day high-low ranges. | $\beta_t = \sum_{j=0}^{1}\big[\ln(H_{t-j}/L_{t-j})\big]^2$; $\gamma_t = \big[\ln\big(\max(H_t, H_{t-1})/\min(L_t, L_{t-1})\big)\big]^2$; $\alpha_t = \dfrac{\sqrt{2\beta_t} - \sqrt{\beta_t}}{3 - 2\sqrt 2} - \sqrt{\dfrac{\gamma_t}{3 - 2\sqrt 2}}$; $S_t = \dfrac{2(e^{\alpha_t} - 1)}{1 + e^{\alpha_t}}$, negative values set to 0; report $\mu_{21}(S)$. Overnight adjustment: if $O_t > H_{t-1}$ shift $H_t, L_t$ down by $O_t - H_{t-1}$; if $O_t < L_{t-1}$ shift them up by $L_{t-1} - O_t$ | 1, 21 |
| Abdi-Ranaldo spread | Abdi and Ranaldo (2017): spread from the close against the mid of the day's range, robust to low-frequency data. | $\eta_t = (\ln H_t + \ln L_t)/2$; $S^2_t = \max\big(4(\ln C_t - \eta_t)(\ln C_t - \eta_{t+1}), 0\big)$; $S_t = \sqrt{\mu_{21}(S^2)}$ (use $\eta_{t}$ and $\eta_{t-1}$ with $C_{t-1}$ to avoid look-ahead) | 21 |
| Kyle's lambda | Kyle (1985): price impact per unit of signed order flow, estimated by regressing price changes on tick-rule-signed volume. | $\lambda_t$ = OLS slope of $\Delta C_s$ on $b_s V_s$ over the window, $b_s = \operatorname{sgn}(\Delta C_s)$ carried forward when $\Delta C_s = 0$; also its $t$-statistic and $\lambda\,\mu_{21}(V)/C_t$ | 63 |
| Amihud lambda (regression form) | Slope version of the Amihud ratio. | OLS slope of $\lvert r_s \rvert$ on $DV_s$ over the window | 63 |
| Amihud-Hasbrouck lambda | Hasbrouck (2009): square-root price impact. | OLS slope of $r_s$ on $\operatorname{sgn}(r_s)\sqrt{DV_s}$ | 63 |
| Zero-return days | Share of days with zero close-to-close change (Lesmond, Ogden and Trzcinka 1999 illiquidity proxy); near zero for these liquid names. | $\frac{1}{k}\sum_{i<k}\mathbb{1}[C_{t-i} = C_{t-i-1}]$ | 63 |
| High-low spread ratio | Range relative to the average spread estimate. | $\ln(H_t/L_t) / \mu_{21}(S^{CS})_t$ | 21 |
| Daily short volume ratio | FINRA short-sale volume over total volume and its mean. | $SVR_t = V^{short}_t / V_t$; $\mu_{21}(SVR)_t$; $z_{63}(SVR)_t$ | 1, 21 |
| Off-exchange share | Share of volume reported to TRF (dark pools and internalizers). | $V^{TRF}_t / V_t$ | 1, 21 |
| Closing-auction share | Share of the day's volume in the closing auction; high on rebalance days. | $V^{close}_t / V_t$ | 1 |
| Block-trade count | Trades above 10,000 shares (from tick data) relative to the mean. | $\#blocks_t / \mu_{21}(\#blocks)$ | 21 |
| Order-flow imbalance proxy | Tick-rule signed volume (Lee-Ready) from intraday data, or the daily BVC bulk-volume classification. | $OFI_t = \dfrac{V^{buy}_t - V^{sell}_t}{V_t}$; BVC: $V^{buy}_t = V_t\,\Phi\big(r_t / \sigma_{21}(t)\big)$ | 1, 5 |
| VPIN | Easley, Lopez de Prado and O'Hara (2012): volume-synchronized probability of informed trading over the last $n$ volume buckets. | $VPIN = \dfrac{\sum_{b=1}^{n}\lvert V^{buy}_b - V^{sell}_b \rvert}{n\,V_{bucket}}$ | $n = 50$ buckets of $\mu_{21}(V)/50$ shares |
