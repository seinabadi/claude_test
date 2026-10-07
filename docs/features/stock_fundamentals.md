# Stock-only features (the 11 single stocks)

Back to the [catalog index](../feature_catalog.md). Align each item to the date it became public (filing or announcement timestamp), then forward-fill; lag quarterly financials by at least one trading day after the filing. Quantities: $P_t$ = unadjusted price, $N_t$ = shares outstanding, $MC_t = P_t N_t$ = market cap, $EV_t = MC_t + \text{debt}_t - \text{cash}_t$ (plus preferred and minority interest), TTM = trailing twelve months, NTM = next twelve months consensus, $y^{10}_t$ = 10-year Treasury yield.

Many fundamental ratios move slowly, so express each as (a) its level, (b) its z-score against the stock's own 5-year history, $z_{1260}(x)_t$, and (c) its difference from the sector median on day $t$.

## Earnings events

| Feature | Definition | Formula | Notes |
| --- | --- | --- | --- |
| EPS surprise | Reported EPS against the consensus estimate just before the report, scaled by the consensus. | $SUR_q = (EPS^{act}_q - EPS^{cons}_q) / \lvert EPS^{cons}_q \rvert$ | use the last pre-report consensus |
| Standardized unexpected earnings (SUE) | Seasonal-random-walk surprise scaled by the history of such surprises; the classic post-earnings-announcement-drift predictor (Foster, Olsen and Shevlin 1984; Bernard and Thomas 1989). | $SUE_q = \dfrac{(EPS_q - EPS_{q-4}) - \hat\mu}{\hat\sigma}$, $\hat\mu, \hat\sigma$ the mean and std of $EPS_{s} - EPS_{s-4}$ over the prior 8 quarters | 8 quarters |
| Price-scaled analyst surprise | Livnat and Mendenhall (2006): surprise against the median consensus scaled by price, which stays well behaved when the consensus is near zero. | $SUE^A_q = (EPS^{act}_q - \operatorname{median}\,EPS^{cons}_q) / P_{q-1}$ | |
| Three-day announcement return | Kishore et al. (2008): market-adjusted return over the window around the report; predicts continued drift. | $EAR_q = \sum_{d=-1}^{+1}(r_d - r^M_d)$ around the reaction day | per event |
| Abnormal announcement volume | Lerman, Livnat and Mendenhall (2007): volume around the report against normal volume; high attention announcements drift more. | $AEAVOL_q = \dfrac{\frac{1}{3}\sum_{d=-1}^{+1} V_d}{\frac{1}{20}\sum_{d=-30}^{-11} V_d} - 1$ | per event |
| Consecutive earnings increases | Barth, Elliott and Finn (1999): number of consecutive quarters with year-over-year EPS growth, up to 8. | $\sum_{j=0}^{7}\prod_{i \le j}\mathbb{1}[EPS_{q-i} > EPS_{q-i-4}]$ | 8 quarters |
| Expected announcement window | Frazzini and Lamont (2007): a flag for the week in which the company reported one year ago, usable before the date is confirmed; stocks earn a premium in expected announcement windows, more so with high past announcement volume. | $EA_t = \mathbb{1}[\text{report occurred in the same week one year earlier}]$; $EA_t \times AEAVOL_{q-4}$ | |
| Revenue surprise | Same construction on revenue. | $(REV^{act}_q - REV^{cons}_q) / REV^{cons}_q$ | |
| Earnings-day return and gap | Abnormal return on the reaction day and the overnight gap into it. | $r_{E} - \beta^{M} r^{M}_{E}$; $o_{E}$ | reaction day is the report day or the next, by timing |
| Post-earnings drift | Cumulative abnormal return since the last report; drift continues in the direction of the surprise. | $\sum_{s = E+1}^{t} (r_s - \beta^M r^M_s)$ for $t - E \in \{5, 21, 63\}$ | |
| Pre-earnings run-up | Return over the 5 days before the next report. | $r^{(5)}_t$ when $N^E_{to}(t) \le 1$ | |
| Earnings announcement premium | Average abnormal return over past announcement windows; stocks with high premia keep earning them (Frazzini and Lamont 2007). | $\frac{1}{8}\sum_{q} \big(r^{(3)}_{E_q + 1} - r^{M,(3)}_{E_q + 1}\big)$ over the last 8 reports | |
| Guidance change | Management raised, maintained or cut guidance at the last report. | $+1, 0, -1$ | from the release or transcript |
| Call sentiment | Sentiment score of the earnings-call transcript (FinBERT or Loughran-McDonald word lists) and its change from the prior call. | $S^{call}_q = \dfrac{\#pos - \#neg}{\#pos + \#neg}$; $\Delta S^{call}_q = S^{call}_q - S^{call}_{q-1}$ | |
| Earnings volatility | Standard deviation of the last 8 quarterly EPS values, scaled by their mean. | $\operatorname{std}(EPS_{q-7..q}) / \lvert \mu(EPS_{q-7..q}) \rvert$ | an earnings-quality measure |

## Analyst estimates

| Feature | Definition | Formula | Notes |
| --- | --- | --- | --- |
| Consensus rating | Mean broker recommendation (1 = strong buy to 5 = sell) and its 21-day change. | $\bar{R}_t$; $\Delta_{21}\bar R_t$ | |
| Upgrades and downgrades | Net count of rating changes over the window. | $\#up_{(t-k, t]} - \#down_{(t-k, t]}$ | $k = 5, 21$ |
| Price target premium | Consensus 12-month target against the current price. | $PT_t / P_t - 1$ | |
| EPS revision ratio | Net share of analysts revising NTM EPS up over the window. | $\dfrac{\#up - \#down}{\#estimates}$ over 1 and 3 months | the "earnings revisions" factor |
| EPS revision magnitude | Percentage change in the consensus NTM EPS. | $EPS^{NTM}_t / EPS^{NTM}_{t-k} - 1$ | $k = 21, 63$ |
| Price-scaled revision sum | Chan, Jegadeesh and Lakonishok (1996): six-month sum of consensus forecast changes scaled by price. | $\sum_{m=1}^{6}(F_m - F_{m-1})/P_{m-1}$ | 6 months |
| Estimate dispersion | Disagreement among analysts (Diether, Malloy and Scherbina 2002: high dispersion, low return). | $\operatorname{std}(EPS^{est}_i) / \lvert \mu(EPS^{est}_i) \rvert$ | |
| Long-term growth estimate | Consensus 3- to 5-year EPS growth rate and its change. | $g^{LT}_t$; $\Delta_{63} g^{LT}_t$ | |
| Days since last revision | Trading days since any analyst changed an estimate. | count | staleness |

## Valuation

| Feature | Definition | Formula | Notes |
| --- | --- | --- | --- |
| Trailing and forward P/E | Price per unit of trailing or expected earnings; use the inverse (earnings yield) to keep negatives well behaved. | $P_t / EPS^{TTM}_t$; $P_t / EPS^{NTM}_t$; $EY_t = EPS^{TTM}_t / P_t$ | |
| EV / EBITDA | Enterprise value against operating cash earnings; capital-structure neutral. | $EV_t / EBITDA^{TTM}_t$ | |
| Price to book | Market value against book equity; the inverse is the value factor's book-to-market. | $P_t N_t / BE_t$; $BM_t = BE_t / MC_t$ | |
| Price to sales | Market cap against TTM revenue. | $MC_t / REV^{TTM}_t$ | |
| PEG | Forward P/E divided by expected growth. | $(P_t / EPS^{NTM}_t) / (100\, g^{LT}_t)$ | |
| Dividend yield | TTM dividends per share over price. | $DPS^{TTM}_t / P_t$ | |
| Free-cash-flow yield | TTM free cash flow over market cap. | $FCF^{TTM}_t / MC_t$ | |
| Shareholder yield | Dividends plus net buybacks over market cap. | $(DIV^{TTM}_t + BB^{TTM}_t) / MC_t$ | |
| Earnings yield minus 10-year yield | The "Fed model" spread: equity earnings yield against the risk-free rate. | $EY_t - y^{10}_t$ | |
| Valuation z-score | Each multiple against the stock's own 5-year history. | $z_{1260}(x)_t$ | |
| Valuation vs sector | Each multiple minus the sector ETF's median constituent multiple. | $x_t - \operatorname{median}_{sector}(x)_t$ | |
| Price to 52-week-high valuation drift | Change in forward P/E over 63 days decomposed into price change and estimate change. | $\Delta_{63}\ln(P/EPS^{NTM}) = \Delta_{63}\ln P - \Delta_{63}\ln EPS^{NTM}$ | separates multiple expansion from earnings |

## Growth and profitability

| Feature | Definition | Formula | Notes |
| --- | --- | --- | --- |
| Revenue growth | Year-over-year and quarter-over-quarter revenue change. | $REV_q / REV_{q-4} - 1$; $REV_q / REV_{q-1} - 1$ | |
| EPS growth | Year-over-year EPS change. | $EPS_q / EPS_{q-4} - 1$ | |
| Gross, operating and net margin | Profitability at each level and its year-over-year change. | $GM = (REV - COGS)/REV$; $OM = EBIT/REV$; $NM = NI/REV$; $\Delta_{4q} M$ | |
| FCF margin | Free cash flow over revenue. | $FCF^{TTM} / REV^{TTM}$ | |
| Gross profitability | Novy-Marx (2013): gross profits over total assets, a strong quality predictor. | $GP/A = (REV - COGS) / TA$ | |
| Return on equity, assets, invested capital | Profitability on each capital base. | $ROE = NI^{TTM}/BE$; $ROA = NI^{TTM}/TA$; $ROIC = NOPAT^{TTM}/(BE + \text{debt} - \text{cash})$ | |
| Operating leverage | Sensitivity of operating income to revenue. | $\Delta\ln EBIT^{TTM} / \Delta\ln REV^{TTM}$ over 4 quarters | |

## Balance sheet and quality

| Feature | Definition | Formula | Notes |
| --- | --- | --- | --- |
| Leverage | Debt to equity and net debt to EBITDA. | $D/BE$; $(D - \text{cash})/EBITDA^{TTM}$ | |
| Interest coverage | EBIT over interest expense. | $EBIT^{TTM} / INT^{TTM}$ | |
| Accruals | Sloan (1996): earnings not backed by cash flow predict lower returns. | $ACC = (NI - CFO)/TA_{avg}$ | |
| Asset growth | Cooper, Gulen and Schill (2008): fast asset growth predicts lower returns. | $TA_q / TA_{q-4} - 1$ | |
| Net operating assets | Hirshleifer et al. (2004): bloated balance sheets. | $NOA = (OA - OL)/TA_{t-1}$ | |
| Capex intensity and R&D intensity | Investment rates. | $CAPEX^{TTM}/REV^{TTM}$; $R\&D^{TTM}/REV^{TTM}$ | |
| Piotroski F-score | Nine binary quality tests on profitability, leverage and efficiency, summed, 0 to 9. | $F = \sum_{i=1}^{9} \mathbb{1}[\text{test}_i]$ (ROA > 0, CFO > 0, $\Delta$ROA > 0, CFO > NI, $\Delta$leverage < 0, $\Delta$current ratio > 0, no new shares, $\Delta$GM > 0, $\Delta$asset turnover > 0) | |
| Altman Z | Bankruptcy-risk composite. | $Z = 1.2\frac{WC}{TA} + 1.4\frac{RE}{TA} + 3.3\frac{EBIT}{TA} + 0.6\frac{MC}{TL} + 1.0\frac{REV}{TA}$ | |
| Ohlson O-score | Ohlson (1980) bankruptcy probability from nine accounting ratios; high distress risk predicts lower returns. | $O = -1.32 - 0.407\ln TA + 6.03\frac{TL}{TA} - 1.43\frac{WC}{TA} + 0.0757\frac{CL}{CA} - 1.72\,\mathbb{1}[TL > TA] - 2.37\frac{NI}{TA} - 1.83\frac{FFO}{TL} + 0.285\,\mathbb{1}[NI < 0 \text{ two years}] - 0.521\frac{NI_t - NI_{t-1}}{\lvert NI_t \rvert + \lvert NI_{t-1} \rvert}$ | |
| Quality minus junk composite | Asness, Frazzini and Pedersen (2019): average of profitability, growth and safety z-scores, each a z-score of ranked component ratios. | $QMJ = z\big(z(\text{prof}) + z(\text{growth}) + z(\text{safety})\big)$ with profitability from GP/A, ROE, ROA, CFO/A, GM, accruals; growth their 5-year changes; safety from low beta, low leverage, low O-score, low earnings volatility | quarterly |
| Mispricing composites | Stambaugh and Yuan (2017): average percentile rank across the performance anomalies (momentum, gross profitability, ROA, distress, issuance) and the management anomalies (accruals, NOA, asset growth, investment, issuance). | $MISP^{perf}_t$, $MISP^{mgmt}_t$ as mean ranks in $[0, 1]$ | monthly |
| Beneish M-score | Earnings-manipulation probability from eight accounting indexes. | $M = -4.84 + 0.92\,DSRI + 0.528\,GMI + 0.404\,AQI + 0.892\,SGI + 0.115\,DEPI - 0.172\,SGAI + 4.679\,TATA - 0.327\,LVGI$ | $M > -1.78$ flags manipulation risk |
| Cash ratio | Cash and equivalents over total assets. | $\text{cash}/TA$ | |
| Effective tax rate change | Year-over-year change; a transitory-earnings signal. | $\Delta_{4q}(TAX/PBT)$ | |

## Capital actions, size and ownership

| Feature | Definition | Formula | Notes |
| --- | --- | --- | --- |
| Buyback yield | Net share repurchases over market cap. | $-(N_t - N_{t-252})\,P_t / MC_t$ | |
| Share count change | Year-over-year change in shares outstanding (negative = buybacks). | $N_t / N_{t-252} - 1$ | |
| Dividend growth | Year-over-year change in dividends per share. | $DPS^{TTM}_t / DPS^{TTM}_{t-252} - 1$ | |
| Event flags | Buyback authorization, secondary offering, split, dividend increase or cut announced today. | 0/1 columns | |
| Log market cap and sector rank | Size, its rank within the sector, and the industry-adjusted size of Asness, Porter and Stevens (2000). | $\ln MC_t$; $\operatorname{rank}_{sector}(MC_t)$; $\ln MC_t - \overline{\ln MC}_{sector}$ | |
| S&P 500 weight | Index weight and its 63-day change. | $w_t = MC^{float}_t / \sum_j MC^{float}_{j,t}$; $\Delta_{63} w_t$ | |
| Institutional ownership | Share of float held by 13F filers and its quarterly change. | $IO_q$; $\Delta IO_q$ | 45-day filing lag |
| Insider net buying | Net open-market insider purchases from Form 4 over the window, in shares over float and as a buy/sell count ratio. | $\sum (\text{buys} - \text{sells}) / N^{float}$; $\#buys / \#sells$ | $k = 21, 63$ |
| Passive ownership | Share of float held by index funds; a flow-sensitivity measure. | $PO_q$ | |

## Short interest and borrow

| Feature | Definition | Formula | Notes |
| --- | --- | --- | --- |
| Short interest ratio | Shares short over float; high and rising short interest predicts lower returns (Rapach, Ringgenberg and Zhou 2016 at the aggregate level). | $SI_t = \text{shares short}_t / N^{float}_t$ | bi-monthly FINRA report, 8-day lag |
| Days to cover | Shares short over average daily volume. | $DTC_t = \text{shares short}_t / \mu_{21}(V)_t$ | |
| Short interest change | Change since the prior report and its z-score. | $\Delta SI_t$; $z_{24}(SI)$ over 24 reports | |
| Borrow fee and utilization | Cost to borrow (annualized) and share of lendable supply on loan. | $fee_t$; $util_t$ | daily from securities-lending data |
| Daily short volume ratio | FINRA daily short-sale volume over total volume and its 21-day mean. | $SVR_t = V^{short}_t / V_t$; $\mu_{21}(SVR)_t$ | |

## Credit

| Feature | Definition | Formula | Notes |
| --- | --- | --- | --- |
| CDS spread | 5-year CDS spread in bp and its 5- and 21-day change. | $CDS_t$; $\Delta_k CDS_t$ | |
| Bond spread | Yield of the on-the-run senior bond minus the matched Treasury. | $y^{bond}_t - y^{UST}_t$ | |
| Rating change | Upgrade or downgrade flag and notches. | $\pm n$ | |
| Distance to default | Merton-model distance to default from equity value, equity vol and debt. | $DD_t = \dfrac{\ln(A_t / D_t) + (\mu - \sigma_A^2/2)T}{\sigma_A \sqrt T}$ with $A_t, \sigma_A$ solved from the equity Black-Scholes relation | $T = 1$ year |

## News, attention and text

| Feature | Definition | Formula | Notes |
| --- | --- | --- | --- |
| News volume | Count of articles over 1 and 5 days and its z-score against the trailing 63 days. | $n_t$; $z_{63}(n)_t$ | |
| News sentiment | Mean article sentiment (FinBERT class probabilities or vendor score) and its 5-day mean. | $\bar s_t$; $\mu_5(\bar s)_t$ | |
| Abnormal news flag | News count more than 3 standard deviations above its trailing mean. | $\mathbb{1}[z_{63}(n)_t > 3]$ | |
| Google Trends | Weekly search-volume index for the ticker and its change (Da, Engelberg and Gao 2011). | $GT_w$; $\Delta GT_w$ | weekly, forward-filled |
| Social mentions and sentiment | Counts and net sentiment from X, Reddit and StockTwits. | $m_t$; $(\#bull - \#bear)/(\#bull + \#bear)$ | |
| Filing text similarity | Cosine similarity of the 10-K or 10-Q to the prior year's (Cohen, Malloy and Nguyen 2020: changes predict lower returns). | $\cos(\mathbf{v}_q, \mathbf{v}_{q-4})$ on TF-IDF vectors | |
| Risk-factor length change | Change in word count of Item 1A. | $\ln(W_q / W_{q-4})$ | |
| 8-K count | Current-report filings over the window. | count over 21 days | |
| Corporate event flags | M&A announcement, CEO change, litigation or regulatory action, product approval or launch. | 0/1 columns | per company watch list |
