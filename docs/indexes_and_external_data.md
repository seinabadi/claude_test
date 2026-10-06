# Indexes and external data

External price indexes and data series to join to each asset's daily row, with ticker or series ID, frequency and what each tells the model. IDs are from memory (approximate); verify them before downloading.

## Equity indexes and sector, style and industry ETFs

For each series derive the same block: 1-, 5-, 21- and 63-day return, 21-day volatility, distance to the 50- and 200-day SMA, and the asset's return minus the series return. Yahoo Finance tickers shown.

| Series | Ticker | What it adds |
| --- | --- | --- |
| S&P 500 | ^GSPC, SPY, ES=F (futures) | market factor; futures give the overnight move |
| Nasdaq-100 | ^NDX, QQQ, NQ=F | growth and mega-cap tech factor (MSFT, AMZN, GOOGL) |
| Dow Jones Industrial | ^DJI, DIA, YM=F | large-cap value and industrials |
| Russell 2000 | ^RUT, IWM, RTY=F | small-cap risk appetite; IWM / SPY ratio |
| S&P 500 equal weight | RSP | breadth; RSP / SPY ratio flags mega-cap concentration |
| S&P 400 mid cap | MDY | mid-cap factor |
| Sector SPDRs | XLK, XLV, XLF, XLY, XLC, XLI, XLP, XLE, XLU, XLRE, XLB | own sector and the other 10; sector rotation ranks |
| Equal-weight sectors | RSPT, RSPH, RSPF, RSPD, RSPC, RSPN, RSPS, RSPG, RSPU, RSPR, RSPM | cap vs equal weight spread per sector |
| Risk-on ratios | XLY / XLP, XLK / XLU, IWM / SPY, SPHB / SPLV | cyclical vs defensive appetite |
| Growth vs value | IWF, IWD, VUG, VTV | style factor; IWF / IWD ratio |
| Momentum, quality, low vol | MTUM, QUAL, USMV, SPLV, SPHB | factor returns and factor momentum |
| Dividend | NOBL, VIG, SCHD | dividend factor (WMT, XOM, NEE, LIN) |
| Semiconductors | SOXX, SMH | AI and tech cycle (MSFT, GOOGL, AMZN) |
| Software and cloud | IGV, WCLD, SKYY | MSFT, GOOGL, AMZN peers |
| Banks | KBE, KRE, KBWB | JPM peers; regional-bank stress via KRE / SPY |
| Broker-dealers and insurance | IAI, KIE | financials sub-industries |
| Biotech and pharma | XBI, IBB, XPH, PPH | LLY peers |
| Health care equipment | IHI | health care sub-industry |
| Retail and consumer | XRT, RTH, PEJ | AMZN and WMT demand |
| Homebuilders | XHB, ITB | rate sensitivity; housing (PLD, NEE) |
| Transports and airlines | IYT, JETS | GE (jet engines), industrial demand, Dow theory |
| Aerospace and defense | ITA, PPA, XAR | GE peers |
| Oil and gas | XOP, OIH, IEO, AMLP | XOM peers; E&P and services |
| Clean energy | ICLN, TAN, QCLN | NEE renewables |
| Materials and metals | VAW, GDX, SLX, COPX | LIN peers; metals |
| REITs | VNQ, IYR, INDS | PLD peers; INDS is industrial REITs |
| Mega-cap group | MAGS, or the top-10 weight of SPY | concentration risk |
| Global | ACWI, EFA, EEM, VEU | world factor; US vs world spread |
| Peers and competitors | NVDA, AAPL, META, NVO, GS, BAC, COST, TGT, CVX, DUK, SO, APD, AMT, RTX, HON | direct peer returns and peer-group mean return |

## Volatility and risk-appetite indexes

| Series | Ticker or source | Features |
| --- | --- | --- |
| VIX | ^VIX (CBOE) | level; 1- and 5-day change; 252-day percentile; z-score; VIX / realized SPY vol(21) |
| VIX term structure | ^VIX9D, ^VIX, ^VIX3M, ^VIX6M; VX futures front and second month | VIX9D / VIX and VIX / VIX3M; contango or backwardation flag; roll yield |
| VIX of VIX | ^VVIX | vol-of-vol level and change |
| VIX1D | ^VIX1D | one-day implied vol; event-day pricing |
| Nasdaq, Russell, Dow vol | ^VXN, ^RVX, ^VXD | growth and small-cap fear; VXN − VIX spread |
| Oil vol | ^OVX | XOM and XLE regime |
| Gold vol | ^GVZ | commodity stress |
| Rates vol | MOVE index (ICE BofA) | bond-market fear; JPM, NEE, PLD sensitivity |
| FX vol | CVIX (Deutsche Bank), JPM G7 vol index | dollar regime |
| SKEW | ^SKEW (CBOE) | tail-risk pricing |
| Implied correlation | COR1M, COR3M (CBOE) | dispersion vs index risk |
| Put/call ratios | CBOE total, equity-only, index-only; ISEE | 1-day; 5- and 21-day means; z-scores |
| Credit risk appetite | HYG / LQD, HYG / IEF | spread-based risk-on gauge |
| Copper / gold | HG=F / GC=F | growth vs fear |
| Small / large | IWM / SPY | risk appetite |
| Crypto risk appetite | BTC-USD 21-day vol and return | tracks Nasdaq risk-on |
| Fear and Greed | CNN composite (daily, scraped) | 0 to 100 |
| Variance risk premium | VIX squared / 252 × 21 − realized variance(21) | priced vs delivered risk |

## Rates, curve and credit (FRED series IDs unless noted)

| Series | ID | Features |
| --- | --- | --- |
| Fed funds effective, target upper | DFF, DFEDTARU | level; days since the last change |
| Treasury yields | DGS1MO, DGS3MO, DGS6MO, DGS1, DGS2, DGS5, DGS10, DGS30 | level; 1-, 5- and 21-day change; 252-day z-score |
| Yahoo yield indexes | ^IRX, ^FVX, ^TNX, ^TYX | same-day values without the FRED lag |
| Curve slopes | T10Y2Y, T10Y3M; 30-year − 5-year | slope, slope change, inversion flag, days inverted |
| Real yields | DFII5, DFII10 | duration pressure on growth and utilities |
| Breakevens | T5YIE, T10YIE, T5YIFR | inflation expectations and change |
| Treasury futures | ZT=F, ZF=F, ZN=F, ZB=F | overnight rate moves before the open |
| Bond ETFs | SHY, IEF, TLT, TIP, AGG | tradable duration proxies; TLT / SPY ratio |
| Fed path pricing | fed funds futures ZQ=F, CME FedWatch; 2-year yield − fed funds | cuts or hikes priced for 3, 6 and 12 months |
| SOFR and money markets | SOFR, DTB3; SOFR − IORB | funding stress |
| Investment-grade OAS | BAMLC0A0CM, BAMLC0A4CBBB | credit conditions; change and percentile |
| High-yield OAS | BAMLH0A0HYM2, BAMLH0A3HYC (CCC) | risk appetite; HY − IG spread |
| Moody's spreads | BAA10Y, AAA10Y | long-history credit spread |
| Credit ETFs | LQD, HYG, JNK, BKLN | daily tradable proxies |
| Mortgage rate | MORTGAGE30US (weekly) | housing and REIT demand (PLD) |
| Bank lending | H.8 weekly loans and deposits; SLOOS (quarterly) | JPM loan growth and deposit flight |
| Global yields | Bund 10-year, JGB 10-year, Gilt 10-year | global rates and carry; US − Bund spread |
| Equity risk premium | S&P 500 forward earnings yield − DGS10 | equities vs bonds valuation |

## FX, commodities and crypto

| Series | Ticker | Relevant to |
| --- | --- | --- |
| Dollar index | DX-Y.NYB; DTWEXBGS (FRED broad) | all multinationals (MSFT, GOOGL, LIN, XOM, LLY); DXY 21-day z-score |
| Major pairs | EURUSD=X, JPY=X, GBPUSD=X, CNY=X, MXN=X, CAD=X | revenue exposure; USDJPY for carry unwinds |
| Risk FX | AUDJPY=X, USDCHF=X | risk-on and safe-haven flow |
| WTI and Brent | CL=F, BZ=F | XOM, XLE; AMZN and WMT logistics; inflation |
| Oil term structure | CL=F front minus second month | backwardation signals tightness |
| Natural gas | NG=F | XOM; NEE fuel cost; LIN input cost |
| Refined products | RB=F (gasoline), HO=F (heating oil); 3-2-1 crack spread | XOM refining margin; consumer wallet (WMT) |
| Gold and silver | GC=F, SI=F | real-rate and fear gauge |
| Copper | HG=F | global growth; LIN and GE industrial demand; copper / gold |
| Industrial metals | DBB (ETF); LME aluminium and nickel | materials demand |
| Steel and lumber | SLX (ETF), LBS=F | construction (PLD, LIN) |
| Agriculture | ZC=F, ZW=F, ZS=F, DBA | food inflation (WMT) |
| Broad commodities | DBC, GSG; BCOM index | inflation and growth regime |
| Freight | Baltic Dry Index, Freightos FBX, Drewry WCI | goods demand (PLD, AMZN, WMT, GE) |
| Carbon and power | EUA futures; PJM and ERCOT power prices | NEE margins |
| Uranium | URA (ETF) | NEE nuclear sentiment |
| Bitcoin and Ether | BTC-USD, ETH-USD | weekend return as a Monday feature; 21-day vol; correlation to QQQ |
| Crypto breadth | BTC dominance; stablecoin supply | risk appetite |

## Global markets that trade before the US open

Strict daily pipeline: use the prior session's close for Asia and Europe, or the move from the US close to 09:00 ET if the pipeline runs pre-open.

| Series | Ticker | Relevant to |
| --- | --- | --- |
| Nikkei 225, TOPIX | ^N225 | global risk; Japan carry (USDJPY) |
| Hang Seng, Shanghai, CSI 300 | ^HSI, 000001.SS | China demand (LIN, XOM, GE, WMT sourcing) |
| KOSPI, Taiwan | ^KS11, ^TWII | semis and AI cycle (MSFT, GOOGL, AMZN) |
| ASX 200 | ^AXJO | commodities (LIN, XOM) |
| Nifty 50 | ^NSEI | emerging-market risk |
| Euro Stoxx 50, STOXX 600 | ^STOXX50E, ^STOXX | the Europe open sets the US pre-market tone |
| DAX, CAC, FTSE 100 | ^GDAXI, ^FCHI, ^FTSE | industrials- and energy-heavy indexes |
| US index futures | ES=F, NQ=F, YM=F, RTY=F | return from the prior close to 09:00 ET |
| Global sector peers | ASML, SAP (MSFT); NVO (LLY); HSBC, BNP (JPM); Nestle, Unilever (WMT); Shell, BP, TotalEnergies (XOM); Siemens, Safran (GE); Iberdrola, Enel (NEE); Segro, Goodman (PLD); Air Liquide (LIN) | peer moves known before the US open |
| Sovereign yields | Bund 10-year, JGB 10-year, Gilt 10-year | global rate shocks |
| BoJ and ECB decisions | event calendar | policy surprises before the US open |
| Global ETFs during US hours | EFA, EEM, FXI, EWJ, EWG, VGK | US-hours proxies for the same regions |
| China monthly | Caixin PMI, credit impulse, trade data | demand for materials and industrials |

## Macro releases and surprise indexes

For each release: level, change, surprise = (actual − consensus) / std of past surprises, release-day flag, days since release. Forward-fill by release date, never by reference period.

| Release | FRED ID or source | Frequency |
| --- | --- | --- |
| CPI, core CPI | CPIAUCSL, CPILFESL | monthly |
| PCE, core PCE | PCEPI, PCEPILFE | monthly |
| PPI | PPIFIS, PPIACO | monthly |
| Nonfarm payrolls, unemployment, hourly earnings | PAYEMS, UNRATE, CES0500000003 | monthly, first Friday |
| Initial and continuing claims | ICSA, CCSA | weekly, Thursday |
| JOLTS openings and quits | JTSJOL, JTSQUR | monthly |
| GDP and GDPNow | A191RL1Q225SBEA; Atlanta Fed GDPNow | quarterly; GDPNow updates several times a month |
| ISM manufacturing and services PMI | ISM (scrape or vendor) | monthly |
| S&P Global PMIs, regional Fed surveys | Empire State, Philly Fed, Richmond, Dallas, Chicago PMI | monthly |
| Retail sales | RSAFS, RSXFS | monthly |
| Consumer confidence and sentiment | Conference Board; UMCSENT and 1-year inflation expectations | monthly |
| Industrial production, capacity utilization | INDPRO, TCU | monthly |
| Durable goods, factory orders | DGORDER, AMTMNO | monthly |
| Housing | HOUST, PERMIT, HSN1F, EXHOSLUSM495S, CSUSHPINSA | monthly |
| Trade balance, import prices | BOPGSTB, IR | monthly |
| Fed communications | FOMC statement, dot plot, minutes; speech count; hawkish-dovish text score | per event |
| Economic surprise index | Citi CESIUSD (Bloomberg), or build it from the surprises above | daily |
| Policy uncertainty | EPU daily index; Trade Policy Uncertainty; Geopolitical Risk index (GPR) | daily |
| Energy inventories | EIA crude, gasoline and distillate stocks (Wednesday); gas storage (Thursday); Baker Hughes rig count (Friday) | weekly |
| Weather | NOAA heating and cooling degree days; hurricane forecasts (NEE, XOM) | daily |
| Consumer high frequency | Redbook same-store sales (weekly); card-spend trackers; TSA checkpoint counts (daily, GE) | weekly, daily |
| Freight | Cass Freight index, ATA truck tonnage (PLD, GE, LIN) | monthly |
| Semiconductors | SIA monthly sales; hyperscaler capex from earnings (MSFT, GOOGL, AMZN) | monthly, quarterly |
| Health policy | FDA approval calendar, CMS rule dates, drug-pricing announcements (LLY) | per event |

## Breadth, positioning and sentiment

| Series | Source | Features |
| --- | --- | --- |
| Advance-decline | NYSE and Nasdaq A/D line and ratio | level; 10- and 21-day slope; divergence from SPY |
| Up and down volume | NYSE, Nasdaq | ratio; Arms index (TRIN) |
| New highs and lows | 52-week highs − lows, NYSE and Nasdaq | net; 10-day sum |
| McClellan | oscillator and summation index | level, sign, crossover |
| Percent above moving average | share of S&P 500 above the 50- and 200-day SMA (Barchart MMFI, MMTH) | level, change, extremes |
| Sector breadth | share of each sector's constituents above the 50-day SMA | own-sector breadth |
| Zweig breadth thrust | 10-day EMA of advances / (advances + declines) | thrust flag |
| CFTC Commitments of Traders | ES, NQ, VIX, 10-year, crude, gold, dollar; net non-commercial and asset-manager positions | weekly (Tuesday data, Friday release); 3-year z-score |
| Investor surveys | AAII bull minus bear; Investors Intelligence; NAAIM exposure | weekly; percentile |
| Fear and Greed | CNN composite | daily |
| Fund flows | ICI weekly equity and bond fund flows; SPY, QQQ and sector ETF daily flows; EPFR (paid) | weekly, daily |
| Margin debt | FINRA monthly | YoY change |
| Dealer positioning | gamma exposure (GEX), dark-pool index (DIX) from SqueezeMetrics or SpotGamma | daily |
| Short interest aggregate | exchange short interest (twice monthly); FINRA daily short volume | ratio, change |
| Insider activity aggregate | Form 4 buy / sell ratio across the market | weekly |
| Retail attention | Google Trends for market and ticker terms; Reddit and StockTwits mention counts and sentiment | daily |
| News sentiment | GDELT tone; FinBERT-scored headlines by asset and macro topic; RavenPack (paid) | daily |
| Buyback blackout | share of the S&P 500 in an earnings blackout | daily |
| Hedge fund beta | rolling beta of HFRX or a hedge-fund ETF proxy to SPY | 63-day |

## Financial conditions and liquidity

| Series | ID or source | Frequency and use |
| --- | --- | --- |
| Chicago Fed NFCI, ANFCI | NFCI, ANFCI (FRED) | weekly; level and 4-week change |
| St. Louis Fed FSI | STLFSI4 | weekly |
| Kansas City FSI | KCFSI | monthly |
| Own daily FCI | equal-weight z-scores of DXY, DGS10, HY OAS and −SPX | daily composite |
| Fed balance sheet | WALCL | weekly; 4- and 13-week change |
| Treasury General Account | WTREGEN (weekly); daily from the Treasury statement | drains or adds reserves |
| Reverse repo | RRPONTSYD | daily |
| Net liquidity | WALCL − TGA − RRP | daily or weekly; 21-day change |
| Bank reserves | WRESBAL | weekly |
| Money supply | M2SL | monthly; YoY growth |
| Funding stress | SOFR − IORB; SOFR 99th percentile − median; repo rates | daily |
| Fed path | cuts or hikes priced over 12 months from fed funds futures | daily |
| Term premium | ACM 10-year term premium (NY Fed) | daily |
| Bank credit | H.8 loans and leases, C&I loans, deposits | weekly (JPM) |
| Credit issuance | IG and HY issuance volume; IPO count | weekly, monthly |
| Cross-currency basis | EUR and JPY 3-month basis | daily; dollar funding stress |
| Global central banks | ECB and BoJ balance sheets; PBoC injections; global M2 | weekly, monthly |
| Bank stress | KRE / SPY ratio; JPM and major-bank CDS | daily |
| Recession gauges | NY Fed yield-curve recession probability; Sahm rule (SAHMREALTIME) | monthly |
| Equity valuation | S&P 500 forward P/E; CAPE; earnings yield − real 10-year | daily, monthly |

## Asset-specific drivers

| Asset | External series that most move it |
| --- | --- |
| MSFT | QQQ, XLK, IGV, SOXX and NVDA (AI capex cycle), 10-year real yield, DXY, hyperscaler capex guidance, cloud-growth prints from AMZN and GOOGL, antitrust and AI-regulation news |
| LLY | XLV, XBI, NVO (GLP-1 peer), FDA and CMS calendar, drug-pricing policy news, 10-year yield (long duration), DXY, weekly prescription trackers (IQVIA) |
| JPM | XLF, KBE and KRE, 2s10s slope, HY and IG OAS, fed funds path, MOVE, bank CDS, H.8 deposits and loans, SLOOS, VIX (trading revenue), Fed stress-test date (June), capital-rule news |
| AMZN | XLY, XRT, QQQ, consumer confidence, retail sales, card-spend trackers, cloud peers (MSFT, GOOGL), DXY, fuel and freight costs, Prime Day and holiday calendar, labor-cost news |
| GOOGL | XLC, QQQ, ad-market peers (META, TTD), DOJ antitrust rulings, AI search competition news, DXY, cloud capex, ad-spend surveys |
| GE | XLI, IYT, JETS, ITA, TSA checkpoint counts, jet-fuel price, Boeing and Airbus delivery data, Safran and RTX, ISM manufacturing, defense budget news |
| WMT | XLP, XRT, retail sales, consumer sentiment, food CPI, gasoline price, Redbook weekly, tariff and China-import news, USDCNY and USDMXN, labor-cost news, TGT and COST |
| XOM | XLE, XOP, OIH, WTI and Brent, crack spread, natural gas, OVX, EIA inventories, rig count, OPEC meeting dates, DXY, geopolitical risk index, CVX and Shell |
| NEE | XLU, 10- and 30-year yields, MOVE, natural gas, Florida weather and hurricane forecasts, renewables policy and tax-credit news, ICLN and TAN, Treasury auctions, DUK and SO |
| PLD | XLRE, VNQ, INDS, 10-year yield, mortgage and CMBS spreads, industrial vacancy (CBRE, quarterly), e-commerce growth (AMZN), Cass Freight, ISM, Segro and Goodman |
| LIN | XLB, industrial production, ISM, copper, natural gas and power prices (input cost), DXY (large non-US revenue), DAX, hydrogen and clean-energy policy news, APD and Air Liquide |
