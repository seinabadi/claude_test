# Calendar and event timing

Back to the [catalog index](../feature_catalog.md). Encode cyclical fields as sine and cosine pairs or as one-hot columns; flags are 0 or 1; countdowns are in trading days. Build every schedule from an exchange calendar (NYSE holidays and half days) plus published release calendars, and freeze each calendar as of the date it was known.

Cyclical encoding of a field $x$ with period $P$: $\sin(2\pi x / P)$ and $\cos(2\pi x / P)$.

| Feature | Definition | Formula or rule | Notes |
| --- | --- | --- | --- |
| Day of week | Weekday effect (historically weak Mondays, strong Fridays, now small). | one-hot of $\{Mon, \dots, Fri\}$ or $\sin, \cos$ with $P = 5$ | |
| Day of month, week of month | Position within the month. | $d \in \{1..31\}$ with $P = 31$; week of month $= \lceil d / 7 \rceil$ | |
| Month and quarter | Seasonal position in the year. | $m \in \{1..12\}$, $P = 12$; $q = \lceil m/3 \rceil$ | |
| Turn of month | Last trading day and first three trading days of a month, when pension and payroll flows arrive (Lakonishok and Smidt 1988). | $\mathbb{1}[\text{trading-day index in month} \in \{-1, 1, 2, 3\}]$ | one flag per position, or a single window flag |
| First and last day of week, month, quarter, year | Boundary flags. | eight 0/1 columns | |
| Trading days to and since month end | Countdowns that let the model learn flow timing beyond the fixed turn-of-month window. | $N^{m}_{to}(t)$, $N^{m}_{since}(t)$ | likewise for quarter end |
| Days in the trading week | 4 or 5 (holiday-shortened weeks trade differently). | count of sessions in the ISO week | |
| Holiday adjacency | Day before and day after an exchange holiday; half-day session flag (day after Thanksgiving, Christmas Eve, July 3). | three 0/1 flags | pre-holiday days have historically high returns and low volume |
| Options expiration | Third-Friday monthly expiry; days to it; quadruple witching (third Friday of Mar, Jun, Sep, Dec); Friday weekly expiry; the Monday after monthly expiry (pin release). | flags plus $N^{opx}_{to}(t)$ | matters for gamma-related features |
| Index rebalance | S&P quarterly rebalance effective at the close of the third Friday of Mar, Jun, Sep, Dec; Russell reconstitution (fourth Friday of June); MSCI semi-annual (end of May and November) and quarterly reviews. | flags; days to the next | huge closing-auction volume on these days |
| FOMC | Decision day (2 pm ET), day before, day after, days to the next meeting, minutes release day (3 weeks after), press-conference flag, blackout window (from the second Saturday before the meeting until the day after). | flags plus $N^{FOMC}_{to}(t)$ | pre-FOMC drift is a documented effect (Lucca and Moench 2015) |
| Macro release days | CPI, PPI, PCE, employment report (first Friday), retail sales, GDP advance estimate, ISM manufacturing (first business day) and services (third business day), JOLTS, weekly jobless claims (Thursday), consumer sentiment prelim and final, housing starts. | one flag per release; days to the next CPI and payrolls | the surprise itself is in the external-data file |
| Earnings season | Weeks 2 to 6 after quarter end; share of S&P 500 companies reporting that week. | $\mathbb{1}[\text{season}]$; fraction in $[0, 1]$ | |
| Own earnings date (stocks) | Days to the next confirmed report, days since the last, report-day flag, and whether the report is before the open or after the close (so the reaction day is $t$ or $t+1$). | $N^{E}_{to}(t)$, $N^{E}_{since}(t)$, flags | the pre-earnings window is also an options-IV feature |
| Peer earnings | A sector peer or a major customer reports within the next 2 days (for example NVDA for MSFT, TGT and COST for WMT). | flag and count | from the peer map in the external-data file |
| Ex-dividend | Ex-date flag, days to the ex-date, dividend yield on the day. | $\mathbb{1}[\text{ex-date}]$; $N^{div}_{to}(t)$; $D_t / C_t$ | price drops by roughly the dividend at the open |
| Treasury auctions | 2-, 3-, 5-, 7-, 10-, 20- and 30-year auction days. | one flag per tenor; a 10y-or-30y flag | rate-sensitive assets (NEE, PLD, JPM) |
| EIA and rig count | Wednesday crude inventories, Thursday natural-gas storage, Friday Baker Hughes rig count. | flags | XOM, NEE |
| Fed stress test and bank calendar | CCAR results (late June), bank-capital rule announcements. | flags | JPM |
| FDA and CMS calendar | PDUFA dates, advisory committee meetings, CMS rate announcements. | flags; days to the next PDUFA | LLY |
| Seasonal windows | January effect; Santa rally (last 5 and first 2 sessions); May to October; September; mid-December tax-loss window; summer low-volume window (July to August). | 0/1 flags | weak but cheap |
| Presidential cycle | Year 1 to 4 of the term, election day, midterm flag. | one-hot year; flags | |
| Daylight-saving transition weeks | Weeks containing the March and November clock changes (Kamstra, Kramer and Levi 2000). | flag | |
| Trading-day index | Position in the sample; lets a tree model carve regimes. Use with care, never extrapolate. | $t$ as an integer | drop if the model must generalise out of sample |
| Days since last large move | Trading days since the last $\lvert r \rvert > 2\sigma_{21}$ day. | $t - \max\{s \le t : \lvert r_s \rvert > 2\sigma_{21}(s)\}$ | also under volatility |
| Days since last gap | Trading days since the last $\lvert o \rvert > ATR_{14}/C$ gap. | analogous | |
| Scheduled-event density | Number of scheduled macro and own-company events in the next 5 trading days. | count | high density raises expected volatility |
| Pre-event volatility window | Flag for the 3 days before FOMC, CPI, payrolls or own earnings, when positioning is typically reduced. | $\mathbb{1}[N_{to} \le 3]$ for each event type | |
| Expiry-week flag | The week containing monthly options expiration. | $\mathbb{1}[0 \le N^{opx}_{to}(t) \le 4]$ | |
