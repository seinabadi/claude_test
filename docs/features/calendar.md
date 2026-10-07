# Calendar and event timing

Back to the [catalog index](../feature_catalog.md). Encode cyclical fields as sine and cosine pairs or as one-hot columns; flags are 0 or 1; countdowns are in trading days. Build every schedule from an exchange calendar (NYSE holidays and half days) plus published release calendars, and freeze each calendar as of the date it was known.

Cyclical encoding of a field $x$ with period $P$: $\sin(2\pi x / P)$ and $\cos(2\pi x / P)$.

| Feature | Definition | Formula or rule | Notes | Evidence |
| --- | --- | --- | --- | --- |
| Day of week | Weekday effect (historically weak Mondays, strong Fridays, now small). | one-hot of $\{Mon, \dots, Fri\}$ or $\sin, \cos$ with $P = 5$ | | A: French (1980); Birru (2018); effect weak since the 1990s |
| Day of month, week of month | Position within the month. | $d \in \{1..31\}$ with $P = 31$; week of month $= \lceil d / 7 \rceil$ | | A: Ariel (1987) |
| Month and quarter | Seasonal position in the year. | $m \in \{1..12\}$, $P = 12$; $q = \lceil m/3 \rceil$ | | A: Keim (1983); Bouman and Jacobsen (2002) |
| Turn of month | Last trading day and first three trading days of a month, when pension and payroll flows arrive (Lakonishok and Smidt 1988). | $\mathbb{1}[\text{trading-day index in month} \in \{-1, 1, 2, 3\}]$ | one flag per position, or a single window flag | A: Lakonishok and Smidt (1988); McConnell and Xu (2008) |
| First and last day of week, month, quarter, year | Boundary flags. | eight 0/1 columns | | D: derived |
| Trading days to and since month end | Countdowns that let the model learn flow timing beyond the fixed turn-of-month window. | $N^{m}_{to}(t)$, $N^{m}_{since}(t)$ | likewise for quarter end | D: derived from the turn-of-month literature |
| Days in the trading week | 4 or 5 (holiday-shortened weeks trade differently). | count of sessions in the ISO week | | D: derived |
| Holiday adjacency | Day before and day after an exchange holiday; half-day session flag (day after Thanksgiving, Christmas Eve, July 3). | three 0/1 flags | pre-holiday days have historically high returns and low volume | A: Ariel (1990); Lakonishok and Smidt (1988) |
| Options expiration | Third-Friday monthly expiry; days to it; quadruple witching (third Friday of Mar, Jun, Sep, Dec); Friday weekly expiry; the Monday after monthly expiry (pin release). | flags plus $N^{opx}_{to}(t)$ | matters for gamma-related features | A: Stoll and Whaley (1987); Ni, Pearson and Poteshman (2005) on pinning |
| Index rebalance | S&P quarterly rebalance effective at the close of the third Friday of Mar, Jun, Sep, Dec; Russell reconstitution (fourth Friday of June); MSCI semi-annual (end of May and November) and quarterly reviews. | flags; days to the next | huge closing-auction volume on these days | A: Harris and Gurel (1986); Chen, Noronha and Singal (2004) |
| FOMC | Decision day (2 pm ET), day before, day after, days to the next meeting, minutes release day (3 weeks after), press-conference flag, blackout window (from the second Saturday before the meeting until the day after). | flags plus $N^{FOMC}_{to}(t)$ | pre-FOMC drift is a documented effect (Lucca and Moench 2015) | A: Lucca and Moench (2015); Savor and Wilson (2013) |
| Macro release days | CPI, PPI, PCE, employment report (first Friday), retail sales, GDP advance estimate, ISM manufacturing (first business day) and services (third business day), JOLTS, weekly jobless claims (Thursday), consumer sentiment prelim and final, housing starts. | one flag per release; days to the next CPI and payrolls | the surprise itself is in the external-data file | A: Savor and Wilson (2013) |
| Earnings season | Weeks 2 to 6 after quarter end; share of S&P 500 companies reporting that week. | $\mathbb{1}[\text{season}]$; fraction in $[0, 1]$ | | D: derived; cf. Frazzini and Lamont (2007) |
| Own earnings date (stocks) | Days to the next confirmed report, days since the last, report-day flag, and whether the report is before the open or after the close (so the reaction day is $t$ or $t+1$). | $N^{E}_{to}(t)$, $N^{E}_{since}(t)$, flags | the pre-earnings window is also an options-IV feature | A: Frazzini and Lamont (2007); Barber, De George, Lehavy and Trueman (2013) |
| Peer earnings | A sector peer or a major customer reports within the next 2 days (for example NVDA for MSFT, TGT and COST for WMT). | flag and count | from the peer map in the external-data file | A: Ramnath (2002) intra-industry information transfer |
| Ex-dividend | Ex-date flag, days to the ex-date, dividend yield on the day. | $\mathbb{1}[\text{ex-date}]$; $N^{div}_{to}(t)$; $D_t / C_t$ | price drops by roughly the dividend at the open | A: Elton and Gruber (1970); Hartzmark and Solomon (2013) |
| Treasury auctions | 2-, 3-, 5-, 7-, 10-, 20- and 30-year auction days. | one flag per tenor; a 10y-or-30y flag | rate-sensitive assets (NEE, PLD, JPM) | B: Lou, Yan and Zhang (2013) for Treasuries; untested for equities |
| EIA and rig count | Wednesday crude inventories, Thursday natural-gas storage, Friday Baker Hughes rig count. | flags | XOM, NEE | D: untested |
| Fed stress test and bank calendar | CCAR results (late June), bank-capital rule announcements. | flags | JPM | D: untested |
| FDA and CMS calendar | PDUFA dates, advisory committee meetings, CMS rate announcements. | flags; days to the next PDUFA | LLY | D: event flags; untested as features |
| Seasonal windows | January effect; Santa rally (last 5 and first 2 sessions); May to October; September; mid-December tax-loss window; summer low-volume window (July to August). | 0/1 flags | weak but cheap | A: Bouman and Jacobsen (2002); Keim (1983); Santa rally weak |
| Presidential cycle | Year 1 to 4 of the term, election day, midterm flag. | one-hot year; flags | | A: Santa-Clara and Valkanov (2003) |
| Daylight-saving transition weeks | Weeks containing the March and November clock changes (Kamstra, Kramer and Levi 2000). | flag | | A: Kamstra, Kramer and Levi (2000); disputed by Pinegar (2002) |
| Trading-day index | Position in the sample; lets a tree model carve regimes. Use with care, never extrapolate. | $t$ as an integer | drop if the model must generalise out of sample | D: modelling device |
| Days since last large move | Trading days since the last $\lvert r \rvert > 2\sigma_{21}$ day. | $t - \max\{s \le t : \lvert r_s \rvert > 2\sigma_{21}(s)\}$ | also under volatility | D: derived |
| Days since last gap | Trading days since the last $\lvert o \rvert > ATR_{14}/C$ gap. | analogous | | D: derived |
| Scheduled-event density | Number of scheduled macro and own-company events in the next 5 trading days. | count | high density raises expected volatility | D: derived |
| Pre-event volatility window | Flag for the 3 days before FOMC, CPI, payrolls or own earnings, when positioning is typically reduced. | $\mathbb{1}[N_{to} \le 3]$ for each event type | | D: derived; cf. Lucca and Moench (2015) |
| Expiry-week flag | The week containing monthly options expiration. | $\mathbb{1}[0 \le N^{opx}_{to}(t) \le 4]$ | | D: derived; cf. Stoll and Whaley (1987) |
