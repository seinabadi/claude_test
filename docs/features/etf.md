# ETF-only features (the 11 sector SPDRs)

Back to the [catalog index](../feature_catalog.md). Quantities: $AUM_t$ = net assets, $N_t$ = ETF shares outstanding, $NAV_t$ = net asset value per share, $P_t$ = ETF closing price, $w_{i,t}$ = weight of holding $i$, $r_{i,t}$ = daily return of holding $i$. Holdings and shares outstanding are published daily by the sponsor.

| Feature | Definition | Formula | Parameters | Evidence |
| --- | --- | --- | --- | --- |
| Daily net flow | Dollar creations minus redemptions, inferred from the change in shares outstanding at NAV. | $F_t = (N_t - N_{t-1})\, NAV_{t-1}$ | | A: Brown, Davies and Ringgenberg (2021) |
| Flow as a share of AUM | Flow scaled by fund size. | $f_t = F_t / AUM_{t-1}$ | | A: Brown, Davies and Ringgenberg (2021) |
| Cumulative flow | Summed flow share over the window. | $\sum_{i=0}^{k-1} f_{t-i}$ | $k = 5, 21, 63$ | A: Brown, Davies and Ringgenberg (2021) |
| Flow z-score | Today's flow against its trailing distribution. | $z_{252}(f)_t$ | 252 | D: derived |
| Flow-return correlation | Whether flows chase returns over the window. | $\operatorname{corr}_{63}(f_t, r_{t-1})$ | 63 | D: derived |
| Log AUM and AUM change | Fund size and its growth. | $\ln AUM_t$; $AUM_t / AUM_{t-21} - 1$ | | D: derived |
| ETF AUM over sector market cap | How much of the sector is held through this ETF. | $AUM_t / \sum_i MC_{i,t}$ | | A: Ben-David, Franzoni and Moussawi (2018), ETF ownership raises volatility |
| Premium or discount to NAV | Price against NAV; large deviations signal arbitrage pressure or stale NAV. | $pd_t = P_t / NAV_t - 1$; $z_{252}(pd)_t$ | | A: Petajisto (2017) |
| Tracking residual | ETF price return minus NAV return. | $\ln(P_t/P_{t-1}) - \ln(NAV_t/NAV_{t-1})$ | | D: derived |
| Implied-return check | Weighted constituent return minus the ETF return; a data-quality check and a short-horizon arbitrage signal. | $\sum_i w_{i,t-1} r_{i,t} - r_t$ | | D: data-quality check |
| Breadth above moving averages | Share of holdings closing above their 50- and 200-day SMA. | $\frac{1}{n}\sum_i \mathbb{1}[C_{i,t} > SMA_k(C_i)_t]$ | $k = 50, 200$ | C: Zweig (1986) breadth indicators |
| Share of holdings up | Fraction of constituents with a positive return today. | $\frac{1}{n}\sum_i \mathbb{1}[r_{i,t} > 0]$ | | C: breadth indicator |
| Advance-decline line | Cumulative advances minus declines among holdings, divided by holdings count. | $ADL_t = ADL_{t-1} + \frac{\#adv_t - \#dec_t}{n}$ | and its 21-day slope | C: breadth indicator |
| McClellan oscillator | Difference of EMAs of net advances; a breadth momentum oscillator. | $EMA_{19}(\#adv - \#dec)_t - EMA_{39}(\#adv - \#dec)_t$, divided by $n$ | 19, 39 | C: McClellan (1970) |
| New highs minus new lows | Constituents at a 63-day high minus at a 63-day low, over $n$. | $(\#NH_t - \#NL_t)/n$ | 63, 252 | C: breadth indicator |
| Return dispersion | Cross-sectional standard deviation of constituent returns; high dispersion means stock picking matters and index correlation is low. | $\operatorname{std}_i(r_{i,t})$; weighted version $\sqrt{\sum_i w_i (r_{i,t} - r_t)^2}$ | and its 21-day mean | A: Stivers and Sun (2010) |
| Mean pairwise correlation | Average correlation among the top-20 holdings over the window. | $\frac{2}{m(m-1)}\sum_{i<j}\operatorname{corr}_k(r_i, r_j)_t$ | $k = 21, 63$ | A: Pollet and Wilson (2010) at the market level |
| Concentration | Herfindahl index of weights and the largest weight. | $HHI_t = \sum_i w_{i,t}^2$; $\max_i w_{i,t}$ | | D: derived |
| Effective number of holdings | Inverse Herfindahl. | $1 / HHI_t$ | | D: derived |
| Cap-weighted minus equal-weighted return | Spread between the SPDR and its equal-weight twin (XLK vs RSPT and so on); positive when mega caps lead. | $r^{cw}_t - r^{ew}_t$; $\sum_{i<21}$ | 1, 21 | A: Plyakha, Uppal and Vilkov (2012) |
| Top-holding contribution | Share of today's ETF return explained by the largest holding. | $w_{1,t-1} r_{1,t} / r_t$ when $\lvert r_t \rvert > 0.1\%$ | | D: derived |
| Relative volume | ETF volume against its own mean and against SPY volume. | $V_t / \mu_{21}(V)_t$; $V_t / V^{SPY}_t$ | | D: derived; cf. Gervais, Kaniel and Mingelgrin (2001) |
| Sector weight in S&P 500 | The sector's index weight and its 63-day change; rising weight attracts passive flow. | $W^{sec}_t$; $\Delta_{63} W^{sec}_t$ | | D: derived |
| Sector return rank | Rank of the ETF's return among the 11 sector ETFs over the window, and its change. | $\operatorname{xrank}(r^{(k)})_t$; $\Delta_5$ | $k = 5, 21, 63$ | A: Moskowitz and Grinblatt (1999) |
| Sectors beating SPY | Share of the 11 sectors outperforming SPY over the window; a breadth measure of the market. | $\frac{1}{11}\sum_j \mathbb{1}[r^{(k)}_{j,t} > r^{M,(k)}_t]$ | $k = 21$ | D: derived |
| Sector rotation momentum | Spread between cyclical and defensive baskets. | $\frac{1}{3}(r_{XLK} + r_{XLY} + r_{XLI}) - \frac{1}{3}(r_{XLP} + r_{XLU} + r_{XLV})$ over 21 days | | A: Moskowitz and Grinblatt (1999); Conover, Jensen, Johnson and Mercer (2008) |
| Leveraged-ETF rebalancing pressure | End-of-day flow the leveraged and inverse funds on the sector must trade: a fund with leverage $\lambda$ and assets $A$ must buy $A\lambda(\lambda - 1) r_t$ of the index after a move $r_t$ (Cheng and Madhavan 2009). | $\sum_{f} A_{f,t-1}\, \lambda_f (\lambda_f - 1)\, r_t$, scaled by the sector's dollar volume | includes $\lambda = -1, -2, -3, 2, 3$ | A: Cheng and Madhavan (2009); Tuzun (2013) |
| ETF short interest | Shares short over shares outstanding and its change; ETF shorts are often hedges, so interpret with the options data. | $SI_t = \text{short}_t / N_t$; $\Delta SI_t$ | | A: Karmaziene and Sokolovski (2022) |
| Put/call ratios on the ETF | Volume and open-interest ratios. | see [options.md](options.md) | | see options file |
| Days to sector rebalance | Trading days to the quarterly S&P rebalance and a GICS reclassification flag. | $N^{reb}_{to}(t)$; flag | | D: derived |
| Constituent earnings share | Share of the ETF's weight reporting earnings this week. | $\sum_{i \in \text{reporting}} w_{i,t}$ | | D: derived |
| Weighted constituent short interest | Weight-averaged short interest of the holdings. | $\sum_i w_{i,t} SI_{i,t}$ | | D: aggregation of an A-grade stock feature |
| Weighted constituent revisions | Weight-averaged EPS revision ratio of the holdings. | $\sum_i w_{i,t} REV_{i,t}$ | | D: aggregation of an A-grade stock feature |
| Implied correlation | ETF implied variance against the weighted constituent implied variances; see the options file. | $\rho^{imp}_t = \dfrac{\sigma^2_{ETF} - \sum_i w_i^2 \sigma_i^2}{\sum_{i \ne j} w_i w_j \sigma_i \sigma_j}$ | 30-day IVs | A: Driessen, Maenhout and Vilkov (2009) |
