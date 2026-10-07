# Statistical and risk features

Back to the [catalog index](../feature_catalog.md) for notation. $r^M$ is the SPY return, $r^S$ the sector-ETF return, $r^f$ the daily risk-free rate (3-month T-bill / 252). Regressions are OLS over the trailing window ending at $t$.

## Market exposure

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Beta to SPY | Slope of the asset's returns on the market's; systematic exposure. | $\beta^M_k(t) = \dfrac{\operatorname{cov}_k(r, r^M)_t}{\operatorname{var}_k(r^M)_t}$ | $k = 21, 63, 126, 252$ |
| Beta to sector ETF | Same against the sector. | $\beta^S_k(t)$ | 63, 252 |
| Frazzini-Pedersen beta | Frazzini and Pedersen (2014): correlation from 5 years of overlapping 3-day returns, volatilities from 1 year of daily returns, shrunk toward 1; low-beta assets earn higher alpha (betting against beta). | $\rho = \operatorname{corr}_{1260}\big(\sum_{j=0}^{2} r_{t-j}, \sum_{j=0}^{2} r^M_{t-j}\big)$; $\hat\beta^{TS} = \rho\,\sigma_{252}(r)/\sigma_{252}(r^M)$; $\hat\beta^{BAB} = 0.6\,\hat\beta^{TS} + 0.4$ | 252, 1260 |
| Beta squared | Nonlinear beta term (Fama and MacBeth 1973). | $(\beta^M_{252})^2$ | 252 |
| Dimson beta | Dimson (1979): sum of lead, contemporaneous and lag market betas; captures stale-price or delayed reaction. | $\hat b_{-1} + \hat b_0 + \hat b_{+1}$ from $r_t = a + b_{-1}r^M_{t-1} + b_0 r^M_t + b_{+1}r^M_{t+1} + \varepsilon$ over the window (the lead term uses data through $t$, so the last usable row is $t-1$) | 21, 63 |
| Long-run correlation | Asness, Frazzini, Gormsen and Pedersen (2020), betting against correlation: 5-year correlation of 3-day returns with the market; low correlation earns higher alpha. | $\operatorname{corr}_{1260}\big(\sum_{j<3} r_{t-j}, \sum_{j<3} r^M_{t-j}\big)$ | 1260 |
| Beta change | Short beta minus long beta. | $\beta^M_{63}(t) - \beta^M_{252}(t)$ | fixed |
| Downside and upside beta | Ang, Chen and Xing (2006): beta measured on days the market was below (above) its mean. | $\beta^- = \dfrac{\operatorname{cov}(r, r^M \mid r^M < \mu^M)}{\operatorname{var}(r^M \mid r^M < \mu^M)}$ over 252 days; $\beta^+$ analogously | 252 |
| Beta asymmetry | Downside minus upside beta. | $\beta^- - \beta^+$ | 252 |
| Correlation to SPY and sector | Rolling return correlations. | $\operatorname{corr}_k(r, r^M)_t$; $\operatorname{corr}_k(r, r^S)_t$ | 21, 63, 252 |
| Correlation change | Short minus long correlation to SPY. | $\operatorname{corr}_{21} - \operatorname{corr}_{252}$ | fixed |
| Correlation to rates and the dollar | Rolling correlation to the daily change in the 10-year yield and to the DXY return. | $\operatorname{corr}_{63}(r, \Delta y^{10})$; $\operatorname{corr}_{63}(r, r^{DXY})$ | 63 |
| Mean pairwise correlation | Average correlation across the 11 assets; a market-wide risk-on/off gauge. | $\dfrac{2}{m(m-1)}\sum_{i<j}\operatorname{corr}_k(r_i, r_j)_t$ | 21, 63 |
| Co-skewness | Harvey and Siddique (2000): sensitivity to squared market returns; negative co-skewness earns a premium. | $\dfrac{E[\varepsilon_t (r^M_t - \mu^M)^2]}{\sqrt{E[\varepsilon_t^2]}\,E[(r^M_t - \mu^M)^2]}$, $\varepsilon$ the CAPM residual | 252 |
| Co-kurtosis | Sensitivity to cubed market returns. | $\dfrac{E[\varepsilon_t (r^M_t - \mu^M)^3]}{\sqrt{E[\varepsilon_t^2]}\,E[(r^M_t - \mu^M)^2]^{3/2}}$ | 252 |
| Idiosyncratic volatility | Ang, Hodrick, Xing and Zhang (2006) use Fama-French three-factor residuals over one month; here a market-plus-sector two-factor variant. High IVOL predicts lower returns. | $\varepsilon_s = r_s - \hat a - \hat\beta^M r^M_s - \hat\beta^S r^S_s$; $IVOL_k = \operatorname{std}_k(\varepsilon)\sqrt{252}$; relative version $IVOL_{21} / \sigma_{21}(r^M)$ | 21, 63, 252 |
| Idiosyncratic share | Share of variance not explained by the factors. | $1 - R^2$ of the same regression | 63 |
| Residual momentum | Sum of residuals scaled by their std (see returns file). | $\sum_{i<k}\varepsilon_{t-i}/\operatorname{std}_k(\varepsilon)$ | 21, 63, 252 |
| Idiosyncratic skewness | Boyer, Mitton and Vorkink (2010): skewness of the residuals; lottery preference predicts lower returns. | sample skewness of $\varepsilon$ over the window | 252 |
| Rolling alpha | Intercept of the market regression, annualized. | $252\,\hat a_k$ | 63, 252 |
| Tracking error to sector | Standard deviation of the return difference. | $\operatorname{std}_{63}(r - r^S)\sqrt{252}$ | 63 |

## Distribution shape

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Skewness | Sample skewness of daily returns with bias correction; negative values mean a fat left tail. | $g_1 = \dfrac{\sqrt{n(n-1)}}{n-2}\cdot\dfrac{\frac{1}{n}\sum(r_i - \bar r)^3}{\big(\frac{1}{n}\sum(r_i - \bar r)^2\big)^{3/2}}$ | 21, 63, 252 |
| Excess kurtosis | Sample excess kurtosis with bias correction; tail heaviness. | $g_2 = \dfrac{n-1}{(n-2)(n-3)}\Big[(n+1)\dfrac{\frac{1}{n}\sum(r_i - \bar r)^4}{\big(\frac{1}{n}\sum(r_i - \bar r)^2\big)^2} - 3(n-1)\Big]$ | 21, 63, 252 |
| Quantile-based skew | Robust skew from quartiles (Bowley). | $\dfrac{Q_{0.75} + Q_{0.25} - 2 Q_{0.5}}{Q_{0.75} - Q_{0.25}}$ | 63 |
| Expected idiosyncratic skewness proxy | Lagged skewness and idiosyncratic volatility, the inputs Boyer et al. use. | $g_1^{(252)}$, $IVOL_{63}$ | |
| Jarque-Bera statistic | Normality test statistic from skew and kurtosis. | $JB = \dfrac{n}{6}\Big(g_1^2 + \dfrac{g_2^2}{4}\Big)$ | 63, 252 |
| Historical VaR | Empirical quantile of daily returns. | $VaR_\alpha = -Q_\alpha(r_{t-n+1..t})$ | $\alpha = 1\%, 5\%$; $n = 63, 252$ |
| Parametric and Cornish-Fisher VaR | Gaussian VaR and a skew- and kurtosis-adjusted version. | $VaR^{G}_\alpha = -(\mu + z_\alpha\sigma)$; $z^{CF}_\alpha = z_\alpha + \frac{(z_\alpha^2 - 1)g_1}{6} + \frac{(z_\alpha^3 - 3z_\alpha)g_2}{24} - \frac{(2z_\alpha^3 - 5z_\alpha)g_1^2}{36}$ | 63, 252 |
| Expected shortfall | Mean return in the tail beyond VaR. | $ES_\alpha = -E[r \mid r \le Q_\alpha]$ | 5%, 63 and 252 |
| Tail ratio | Right-tail quantile over left-tail quantile. | $Q_{0.95}/\lvert Q_{0.05} \rvert$ | 252 |
| Hill tail index | Heaviness of the left tail from the $m$ most negative returns. | $\hat\xi = \dfrac{1}{m}\sum_{j=1}^{m}\ln\dfrac{\lvert r_{(j)} \rvert}{\lvert r_{(m+1)} \rvert}$, $r_{(j)}$ the $j$-th most negative | $m = 25$, $n = 504$ |
| Gain-loss ratio | Average gain over average loss. | $\dfrac{\mu(r \mid r > 0)}{\lvert \mu(r \mid r < 0) \rvert}$ | 63 |
| Omega ratio | Probability-weighted gains over losses relative to a threshold of zero. | $\Omega = \dfrac{\sum \max(r_i, 0)}{\sum \max(-r_i, 0)}$ | 63, 252 |

## Drawdown and path risk

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Current drawdown | Distance below the running maximum close within the window (or full history). | $DD_t = C_t / \max_k(C)_t - 1$ | 252, full |
| Max drawdown | Worst peak-to-trough decline within the window. | $MDD_k = \min_{s \in (t-k, t]}\big(C_s / \max_{u \le s} C_u - 1\big)$ | 21, 63, 252 |
| Drawdown duration | Trading days since the running maximum. | $t - \arg\max_{(t-k, t]} C$ | 252 |
| Time under water share | Share of the window spent more than 5% below the running high. | $\frac{1}{k}\sum\mathbb{1}[DD_s < -0.05]$ | 252 |
| Max run-up | Largest trough-to-peak advance within the window. | $\max_s\big(C_s / \min_{u \le s} C_u - 1\big)$ | 63, 252 |
| Ulcer index | See volatility file. | | 14, 63 |
| Pain index | Mean drawdown over the window. | $\frac{1}{k}\sum_{i<k} \lvert DD_{t-i} \rvert$ | 63 |
| Calmar ratio | Annualized return over max drawdown. | $\dfrac{252\,\mu_{252}(r)}{\lvert MDD_{252} \rvert}$ | 252 |
| Sortino ratio | Excess return over downside deviation. | $\dfrac{(\mu_k(r) - r^f)\,252}{\sigma^-_k}$ | 252 |
| Martin ratio | Annualized return over the Ulcer index. | $252\,\mu_{252}(r)/UI_{252}$ | 252 |
| Recovery flag | Price back above the prior 63-day drawdown peak. | $\mathbb{1}[C_t \ge \max_{63}(C)_{t-1} \wedge DD_{t-1} < -0.05]$ | 63 |

## Serial dependence and memory

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Return autocorrelation at lags 1, 2, 5 | Linear predictability from past returns. | $\rho_j = \operatorname{corr}_k(r_t, r_{t-j})$ | 63, 252 |
| Absolute-return autocorrelation | Volatility clustering strength. | $\operatorname{corr}_k(\lvert r_t \rvert, \lvert r_{t-j} \rvert)$, $j = 1, 5$ | 63, 252 |
| Ljung-Box statistic | Joint test of the first $m$ autocorrelations. | $Q = n(n+2)\sum_{j=1}^{m}\dfrac{\hat\rho_j^2}{n - j}$ | $m = 5$, $n = 252$ |
| Variance ratio | Lo and MacKinlay (1988): variance of overlapping $q$-day returns over $q$ times the daily variance, with their small-sample correction; 1 under a random walk, above 1 trending, below 1 mean-reverting. | With $n$ daily returns and $\hat\mu$ the mean: $\hat\sigma^2_a = \frac{1}{n-1}\sum_k (r_k - \hat\mu)^2$; $\hat\sigma^2_c(q) = \frac{1}{m}\sum_{k=q}^{n}(r^{(q)}_k - q\hat\mu)^2$, $m = q(n - q + 1)(1 - q/n)$; $VR(q) = \hat\sigma^2_c(q)/\hat\sigma^2_a$ | $q = 5, 10, 21$; $n = 252$ |
| Variance-ratio z-statistic | Heteroskedasticity-robust test statistic for $VR(q) = 1$. | $z^*(q) = \dfrac{VR(q) - 1}{\sqrt{\hat\theta(q)}}$, $\hat\theta(q) = \sum_{j=1}^{q-1}\big[\tfrac{2(q-j)}{q}\big]^2\hat\delta_j$, $\hat\delta_j = \dfrac{\sum_t (r_t - \bar r)^2(r_{t-j} - \bar r)^2}{\big[\sum_t (r_t - \bar r)^2\big]^2}$ | same |
| Hurst exponent (R/S) | Long-memory measure computed on returns (never on price levels): 0.5 random walk, above 0.5 persistent, below 0.5 anti-persistent. Short windows give very noisy estimates; apply the Anis-Lloyd small-sample correction. | For block sizes $m$, split the window into $\lfloor n/m \rfloor$ blocks; in each, $Y_k = \sum_{i \le k}(r_i - \bar r)$, $R = \max_k Y_k - \min_k Y_k$, $S = \operatorname{std}(r)$; $(R/S)_m$ = block mean; $H$ = OLS slope of $\ln (R/S)_m$ on $\ln m$ | $m \in \{8, 16, 32, 64\}$; $n = 252, 504$ |
| Hurst exponent (DFA) | Peng et al. (1994) detrended fluctuation analysis: more robust to non-stationarity. Feed returns; the profile step integrates them once, so feeding log price would give $H + 1$. | Profile $y_j = \sum_{i \le j}(r_i - \bar r)$; split into boxes of size $m$, fit a linear trend in each, $F(m) = \sqrt{\frac{1}{N_m m}\sum (y - \hat y)^2}$; $H$ = slope of $\ln F(m)$ on $\ln m$ | $m$ from 10 to $n/4$; same windows |
| Generalized Hurst (Hurst on $\lvert r \rvert$) | Memory in volatility. | same on $\lvert r \rvert$ | 252 |
| Fractional differentiation | Lopez de Prado (2018, ch. 5): fixed-width fractional difference of log price, stationary yet memory-preserving. Choose $d$ on the training segment only and keep it fixed; re-selecting on the full history leaks. | $\tilde x_t = \sum_{k=0}^{l^*} w_k \ln C_{t-k}$, $w_0 = 1$, $w_k = -w_{k-1}\dfrac{d - k + 1}{k}$, $l^*$ the first $k$ with $\lvert w_k \rvert < 10^{-5}$; smallest $d$ on a grid with step 0.05 whose series passes the ADF test at 5% while $\operatorname{corr}(\tilde x, \ln C) > 0.9$; also $z_{21}(\tilde x)$ | $d \in [0.3, 0.6]$ typically |
| Augmented Dickey-Fuller statistic | Stationarity of log price over the window; very negative means mean-reverting. | $t$-stat of $\gamma$ in $\Delta x_t = \alpha + \gamma x_{t-1} + \sum_{j=1}^{p}\phi_j\Delta x_{t-j} + \varepsilon_t$ | $p = 1$; 252 |
| Half-life of mean reversion | From an AR(1) fit of log price (Ornstein-Uhlenbeck): days for a deviation to halve. | $\lambda$ = slope of $\Delta x_t$ on $x_{t-1}$; $HL = -\ln 2 / \ln(1 + \lambda)$ | 126 |
| Partial autocorrelation at lag 2 | Direct dependence on $r_{t-2}$ net of $r_{t-1}$. | $\phi_{22}$ from the Durbin-Levinson recursion | 252 |

## Complexity and entropy

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Sample entropy | Richman and Moorman (2000): negative log of the conditional probability that sequences matching for $m$ points (Chebyshev distance within $r$) also match for $m+1$, excluding self-matches; lower means more regular. Needs at least 126 points for enough matches. | $SampEn(m, r) = -\ln\dfrac{A}{B}$, $B$ = number of template pairs $i \ne j$ with $d(x^m_i, x^m_j) \le r$, $A$ = same for length $m + 1$; missing if $A = 0$ | $m = 2$, $r = 0.2\,\operatorname{std}$; $n = 126, 252$ |
| Approximate entropy | Pincus (1991): the self-matching predecessor of sample entropy. | $ApEn = \Phi^m - \Phi^{m+1}$, $\Phi^m = \dfrac{1}{N - m + 1}\sum_i \ln C^m_i(r)$ | same |
| Permutation entropy | Bandt and Pompe (2002): Shannon entropy of the distribution of ordinal patterns of length $m$; normalized by $\ln m!$. | $PE = -\sum_\pi p(\pi)\ln p(\pi)/\ln(m!)$ | $m = 3, 4$; $n = 126$ |
| Shannon entropy of binned returns | Entropy of the return histogram. | $-\sum_b p_b \ln p_b$ with 10 quantile bins fixed on the trailing year | 63 |
| Plug-in entropy rate | Lopez de Prado (2018, ch. 18): maximum-likelihood entropy of words of length $w$ in the discretized return sequence (binary signs, or deciles against the trailing year). | $\hat H_w = -\dfrac{1}{w}\sum_{y \in A^w}\hat p_w(y)\log_2\hat p_w(y)$, $\hat p_w$ the frequency of word $y$ among sliding windows | $w = 1, 2, 3$; $n = 126$ |
| Lempel-Ziv complexity | Number of distinct phrases in the Lempel-Ziv parsing of the sign sequence of returns, normalized to an entropy-rate estimate. | $\hat H_{LZ} = c(n)\log_2 n / n$ for the binary sequence $\mathbb{1}[r > 0]$, $c(n)$ the phrase count | 126, 252 |
| Redundancy | One minus the entropy rate relative to its maximum; near 0 is efficient, high values mean a predictable regime. | $R = 1 - \hat H / \log_2 \lvert A \rvert$ | binary or decile alphabet |
| Spectral entropy | Flatness of the power spectrum of returns; 1 for white noise. | $-\sum_f \hat p_f \ln \hat p_f / \ln N_f$, $\hat p_f$ the normalized periodogram | 126 |
| Kontoyiannis entropy rate | Kontoyiannis et al. (1998) match-length estimator of the entropy rate of the discretized return sequence. | $\hat H_{n,k} = \Big[\frac{1}{k}\sum_{i=n+1}^{n+k} \dfrac{\Lambda^n_i}{\log_2 n}\Big]^{-1}$, $\Lambda^n_i$ = 1 plus the length of the longest substring starting at $i$ that also appears in the preceding $n$ symbols | window 100 to 250 |
| Number of turning points | Local extrema count of the close in the window; a roughness measure. | $\sum_{i}\mathbb{1}[(C_i - C_{i-1})(C_{i+1} - C_i) < 0]$ | 21, 63 |
| Longest streak above mean | Longest run of closes above the window mean. | count | 63 |
| Mean absolute change and mean second difference | tsfresh-style roughness of the return path. | $\frac{1}{n}\sum\lvert \Delta r \rvert$; $\frac{1}{n}\sum\lvert \Delta^2 \ln C \rvert$ | 21 |
| Cid-CE complexity | tsfresh complexity estimate: length of the standardized return path. | $\sqrt{\sum_i (\tilde r_i - \tilde r_{i-1})^2}$, $\tilde r$ standardized over the window | 21, 63 |
| Benford first-digit deviation | Deviation of the first-digit distribution of daily dollar volume from Benford's law; a data-quality feature. | $\sum_d \lvert \hat p_d - \log_{10}(1 + 1/d) \rvert$ | 252 |

## Time-series shape statistics (tsfresh and catch22 style)

Generic time-series descriptors applied to the trailing window of $r_t$, $\lvert r_t \rvert$ or $\ln C_t$ (stated per row); $z_t$ denotes the window-standardized series. Windows 21, 63 and 252 unless given.

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| AR coefficients | OLS fit of an autoregression on returns; the first coefficients summarize short-memory structure. | $r_t = \phi_0 + \sum_{i=1}^{p}\phi_i r_{t-i} + \varepsilon_t$; features $\hat\phi_1, \dots, \hat\phi_p$ | $p = 5$; 252 |
| Third-order autocovariance (c3) | Nonlinear dependence invisible to the autocorrelation function. | $c3(\ell) = \dfrac{1}{n - 2\ell}\sum_i r_{i+2\ell}\,r_{i+\ell}\,r_i$, divided by $\sigma^3$ | $\ell = 1, 2, 3$ |
| Time-reversal asymmetry | Asymmetry between rises and falls in time; non-zero for irreversible dynamics such as slow climbs and fast drops. | $trev(\ell) = \dfrac{1}{n - 2\ell}\sum_i\big(r^2_{i+2\ell}r_{i+\ell} - r_{i+\ell}r^2_i\big)$; catch22 form $\mu\big((z_{t+1} - z_t)^3\big)$ | $\ell = 1, 2, 3$ |
| Zero and mean crossings | Number of sign changes of returns, or of log price through its window mean; high values mean chop. | $\frac{1}{n}\sum_i\mathbb{1}[\operatorname{sgn}(x_i) \ne \operatorname{sgn}(x_{i-1})]$ on $r$; on $\ln C - \mu_k(\ln C)$ | 21, 63 |
| Number of peaks of support $m$ | Local maxima of log price exceeding all $m$ neighbours on each side; a swing count. | $\sum_i\mathbb{1}[x_i > x_{i \pm j}\ \forall j \le m]$ on $\ln C$, and on $-\ln C$ for troughs, divided by $n$ | $m = 3, 5$; 63 |
| Spectral centroid and low-frequency share | Where the variance of returns sits in frequency: low-frequency dominance means persistence, a flat spectrum means noise. | Welch periodogram $S(f)$ of $r$ over the window; centroid = frequency where cumulative $S$ reaches half the total; low-frequency share $= \sum_{f \le f_{max}/5} S(f)/\sum_f S(f)$ | 63, 126 |
| Fourier coefficients and aggregates | tsfresh fft_aggregated: centroid, variance, skew and kurtosis of the amplitude spectrum, and the first few real and imaginary coefficients. | $A_f = \lvert FFT(r)_f \rvert$, $p_f = A_f/\sum A$; $\sum_f f p_f$, $\sum_f (f - c)^2 p_f$, and so on | 63 |
| Wavelet coefficients | Ricker (Mexican-hat) continuous wavelet transform of log price at several widths: local trend curvature at multiple horizons; also the count of scale-persistent peaks. | $CWT(w, t) = \sum_i x_i\,\psi_w(t - i)$, $\psi_w(u) = \dfrac{2}{\sqrt{3w}\pi^{1/4}}\big(1 - \tfrac{u^2}{w^2}\big)e^{-u^2/(2w^2)}$; last coefficient at each width | $w \in \{2, 5, 10, 20\}$ |
| Index mass quantile | Position in the window where half of the cumulative absolute return mass has accrued: below 0.5 means volatility was front-loaded and is calming, above 0.5 means it is recent. | $IMQ_q = \min\{i : \sum_{j \le i}\lvert r_j \rvert / \sum_j\lvert r_j \rvert \ge q\}/n$ | $q = 0.5$; 63 |
| Energy ratio of the last chunk | Share of the window's squared returns in the most recent tenth; a recent-versus-window variance ratio. | $\dfrac{\sum_{j \in \text{last } n/10} r_j^2}{\sum_j r_j^2}$ | 63, 252 |
| Aggregated autocorrelation | Summaries of the whole autocorrelation function: mean and variance of $\rho_1..\rho_{40}$, the first lag where the ACF of $\lvert r \rvert$ falls below $1/e$ (decorrelation time), and the first local minimum of the ACF. | $\mu(\rho_{1..40})$, $\operatorname{var}(\rho_{1..40})$; $\tau_{1/e} = \min\{\ell : \rho_\ell(\lvert r \rvert) < 1/e\}$; $\min\{\ell : \rho_{\ell-1} > \rho_\ell < \rho_{\ell+1}\}$ | 252 |
| Auto mutual information | Nonlinear lag dependence: mutual information between $z_t$ and $z_{t+2}$ from a 5-bin histogram, and the first minimum of the Gaussian-kernel AMI over lags 1 to 40. | $AMI(\tau) = \sum_{i,j} p_{ij}\ln\dfrac{p_{ij}}{p_i p_j}$; Gaussian form $-\tfrac{1}{2}\ln(1 - \rho_\tau^2)$ | 252 |
| Local forecast error | Predictability by a naive local mean: standard deviation of the residuals from forecasting $z_t$ by the mean of the previous three values, and the ratio of decorrelation times of differences and levels. | $e_t = z_t - \tfrac{1}{3}(z_{t-1} + z_{t-2} + z_{t-3})$; $\operatorname{std}(e)$; $\tau_{1/e}(\Delta z)/\tau_{1/e}(z)$ | 63, 252 |
| Symbolic motif entropy and transition irregularity | Coarse-grain returns into three symbols by terciles: the Shannon entropy of consecutive two-symbol words, and the trace of the covariance of the columns of the $3 \times 3$ transition matrix. | $hh = -\sum_{ab}p_{ab}\ln p_{ab}$; $T_{ij} = P(s_{t+\tau} = j \mid s_t = i)$, $\operatorname{tr}(\operatorname{cov}(T))$ | $\tau = 1$; 126, 252 |
| Outlier timing | Whether large positive or negative deviations occur early or late in the window: median time index of points beyond successive thresholds, relative to the window centre; positive means extremes cluster in the recent half. | for thresholds $\theta = 0, 0.01, \dots$ (in $\sigma$ units): $m_\theta = \operatorname{median}\{i : z_i \ge \theta\}/n - 0.5$; output median over $\theta$ with at least 2% of points; separately for $z_i \le -\theta$ | 126, 252 |
| Periodicity | First autocorrelation peak of spline-detrended log price above a small threshold: the dominant period in bars, 0 if none. | $\min\{\ell : \rho_{\ell-1} < \rho_\ell > \rho_{\ell+1},\ \rho_\ell > 0.01\}$ after detrending | 252 |
| Langevin drift polynomial | Fit the conditional mean of the next change as a cubic in the current level of the detrended log price; its root is the implied mean-reversion level and its slope the reversion speed. | bin $x = \ln C - SMA_{63}(\ln C)$ into 30 quantiles; fit $\mu(\Delta x \mid x) \approx \sum_{m=0}^{3} c_m x^m$; features $c_1, c_2, c_3$ and the largest real root | 252 |
| Matrix profile novelty | Nearest-neighbour z-normalized distance between the most recent subsequence and every earlier one in the window: how unusual the current pattern is; the window minimum is the strongest repeated motif. | $MP_i = \min_{\lvert i - j \rvert > m/2}\operatorname{dist}_z(x_{i..i+m-1}, x_{j..j+m-1})$ on $\ln C$; features $MP_{last}$, $\min MP$, $\max MP$ | $m = 10, 21$; 252 |
| Mean of the $n$ largest absolute returns | Tail magnitude independent of sign, in volatility units. | $\frac{1}{n}\sum$ of the $n$ largest $\lvert r \rvert$, divided by $\sigma_k$ | $n = 3, 5$; 63 |

## Structural breaks and explosiveness

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| CUSUM filter events | Lopez de Prado's symmetric CUSUM: flags when cumulative deviations of returns exceed a threshold; count and days since. | $S^+_t = \max(0, S^+_{t-1} + r_t - \mu)$, $S^-_t = \min(0, S^-_{t-1} + r_t - \mu)$; event when $S^+ > h$ or $S^- < -h$, then reset | $h = 2\sigma_{21}$ |
| Brown-Durbin-Evans CUSUM statistic | Recursive-residual CUSUM of a rolling market regression; tests parameter stability. | $W_t = \dfrac{1}{\hat\sigma_w}\sum_{s} w_s$ with recursive residuals $w_s$; feature $W_t/\sqrt{n}$ | 252 |
| Chu-Stinchcombe-White CUSUM on levels | Tests whether log price has departed from a reference level. | $S_{n,t} = \dfrac{\ln C_t - \ln C_n}{\hat\sigma_t\sqrt{t - n}}$, max over reference points $n$ | 63 |
| Supremum ADF (SADF) | Phillips, Shi and Yu (2015): maximum of ADF statistics over expanding windows ending at $t$ (the backward SADF sequence); high values flag explosive (bubble) behaviour. Apply to log price and to cumulative residuals against SPY. | $ADF_{t_0, t}$ = $t$-stat of $\beta$ in $\Delta y_s = \alpha + \beta y_{s-1} + \sum_{l=1}^{L}\gamma_l\Delta y_{s-l} + \varepsilon_s$ on $[t_0, t]$; $SADF_t = \sup_{t_0 \in [t - T, t - \tau_{min}]} ADF_{t_0, t}$ | $\tau_{min} = 63$, $T = 504$, $L = 1$ |
| Chow-type Dickey-Fuller (SDFC) | Homm and Breitung (2012): tests for a switch from a random walk to an explosive process at an unknown date. | $\Delta y_s = \delta\, y_{s-1}\mathbb{1}[s \ge \tau^*] + \varepsilon_s$; $DFC_{\tau^*} = \hat\delta/se(\hat\delta)$; $SDFC_t = \sup_{\tau^*} DFC_{\tau^*}$ with 10% trimming | 252, 504 |
| Sub- and super-martingale tests | Lopez de Prado (2018, ch. 17): trend $t$-statistics over expanding windows with a penalty for long windows. | $y_s = \alpha + \beta s + \varepsilon$ (or $\ln y$ on $\ln s$); $SMT_t = \sup_{t_0}\dfrac{\lvert \hat\beta_{t_0, t} \rvert}{se(\hat\beta)\,(t - t_0)^{\varphi}}$ | $\varphi \in \{0, 0.5, 1\}$ |
| Quandt-Andrews sup-F | Largest Chow F-statistic over candidate break dates in a rolling mean-return regression. | $\sup_{\tau} F(\tau)$ with 15% trimming | 252 |
| Rolling mean shift | Difference between the last 21 days' mean return and the prior 231 days', in standard errors. | $\dfrac{\mu_{21}(r) - \mu_{231}(r_{t-21..})}{\sqrt{\sigma^2_{21}/21 + \sigma^2_{231}/231}}$ | fixed |
| Volatility break statistic | Same for variance: ratio of 21-day to prior 231-day variance, log scale. | $\ln(\sigma^2_{21}/\sigma^2_{231,\text{prior}})$ | fixed |

## Market-state and conditioning features

Market-level variables that condition the asset's own signals; they are the same column for all 22 assets on a given day and are mostly used in interactions ($\text{feature} \times \text{state}$, the Gu-Kelly-Xiu construction). The Welch-Goyal predictors (dividend-price ratio, earnings-price ratio, term spread, default spread, net issuance, market variance) are listed in [Indexes and external data](../indexes_and_external_data.md).

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| Bear-market indicator | Daniel and Moskowitz (2016): the two-year market return is negative; momentum crashes happen in bear states with high market volatility. | $I^B_t = \mathbb{1}[\ln(C^M_t / C^M_{t-504}) < 0]$; interaction $I^B_t\,\sigma^2_{126}(r^M)_t$ | 504, 126 |
| Market state | Cooper, Gutierrez and Hameed (2004): momentum profits exist only after a positive three-year market return. | $\mathbb{1}[\ln(C^M_t / C^M_{t-756}) > 0]$ | 756 |
| Volatility-managed weight | Moreira and Muir (2017): scale exposure by inverse trailing variance; the weight itself is a feature and a sizing rule. | $w_t = \min\big(c / RV_{22}(t), w_{max}\big)$, $RV_{22} = \sum_{i<22}(r^M_{t-i} - \bar r)^2$, $c$ set so managed and unmanaged series have equal volatility; also $RV_{22}/\mu_{252}(RV_{22})$ | 22 |
| Momentum-strategy state | Trailing return and volatility of a winners-minus-losers portfolio built from the 22 assets by 12-1 momentum (top 5 minus bottom 5). | $WML^{(21)}_t$; $\operatorname{std}_{63}(WML)$ | 21, 63 |
| Aggregate tail risk | Kelly and Jiang (2014): Hill estimator pooled across the universe's daily returns each month; predicts market returns. | $\lambda_t = \dfrac{1}{K_t}\sum_{k=1}^{K_t}\ln\dfrac{R_{k,t}}{u_t}$ over the returns below the pooled 5th percentile $u_t$ | monthly, 22 assets × 21 days |
| Aggregate short interest | Rapach, Ringgenberg and Zhou (2016): detrended log equal-weighted short interest ratio across stocks; the strongest aggregate return predictor in their comparison (negative). | $SII_t$ = standardized residual of $\ln EWSI_t$ on a linear trend fit on the trailing 10 years | bi-monthly |
| Mean pairwise correlation and dispersion | Market-wide risk-on/off gauges (see above and the ETF file). | $\bar\rho_{63}$; $\operatorname{std}_i(r_{i,t})$ | 63 |
| Weekday interactions | Birru (2018): speculative (high IVOL, high beta, high MAX) assets underperform on Mondays and outperform on Fridays. | $\mathbb{1}[\text{Mon}]\times\operatorname{xrank}(IVOL)$; $\mathbb{1}[\text{Fri}]\times\operatorname{xrank}(IVOL)$ | |

## Regimes and cross-sectional transforms

| Feature | Definition | Formula | Parameters |
| --- | --- | --- | --- |
| HMM regime probabilities | Hamilton (1989): two- or three-state Gaussian hidden Markov model on (return, log range) fit by EM on a rolling window; use the filtered state probabilities at $t$, never the smoothed posteriors a library returns for the whole fitted window (they use future data). Sort states by fitted volatility so labels are stable across refits. | $\xi_{t \mid t}(j) \propto f(y_t \mid s_t = j)\sum_i A_{ij}\,\xi_{t-1 \mid t-1}(i)$, normalized to sum to 1; one-step-ahead $\xi_{t+1 \mid t} = A^{\top}\xi_{t \mid t}$ | 2 or 3 states; refit on 1000 days |
| Regime duration | Days since the most likely state last changed. | count | |
| Markov-switching variance ratio | Ratio of the high-state to low-state fitted volatilities. | $\hat\sigma_{high}/\hat\sigma_{low}$ | |
| PCA loadings | Loadings of the asset on the first three principal components of the 11-asset return panel. | eigenvectors of the $k$-day covariance matrix, signed so that the first component is positive on SPY | 63, 252 |
| PCA explained variance | Share of panel variance in the first component; a concentration-of-risk measure. | $\lambda_1 / \sum_j \lambda_j$ | 63 |
| Absorption ratio | Kritzman et al. (2011): variance explained by the first $n/5$ components. | $\sum_{j \le 2}\lambda_j / \sum_j\lambda_j$ | 252 |
| Cross-sectional z-scores | Each feature standardized across the 11 assets on day $t$. | $(x_{i,t} - \bar x_t)/\operatorname{std}_i(x_{\cdot,t})$ | daily |
| Cross-sectional ranks | Each feature ranked across the 11 assets (Kakushadze's `rank`). | $\operatorname{xrank}(x)_t$ | daily |
| Rolling rank of features | Each feature's percentile against its own trailing history. | $\operatorname{prank}_{252}(x)_t$ | 252 |
| Sector-neutralized features | Feature minus the mean of the feature for the asset's sector pair (stock and its ETF), Kakushadze's `indneutralize`. | $x_{i,t} - \bar x_{sector(i),t}$ | daily |
| Clustered feature importance groups | Not a feature: cluster the feature correlation matrix and keep one representative per cluster before modelling. | hierarchical clustering on $1 - \lvert \rho \rvert$ | |
