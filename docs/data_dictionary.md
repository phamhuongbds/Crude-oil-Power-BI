# Data dictionary

This section provides an overview of the data dictionary, describing the datasets and variables used in the analysis. It also specifies the origin of each variable and how it was obtained or generated, distinguishing between downloaded data, values calculated in Python, manually entered research data, and measures created in Power BI. This documentation ensures transparency and provides a clear understanding of how the data was collected, processed, and prepared for analysis.

The **Origin** column uses four labels:

| Origin | Meaning |
|---|---|
| Downloaded | Taken as published from Yahoo Finance, FRED (US Federal Reserve data) or the ECB Data Portal |
| Calculated in Python | Derived in the notebook from downloaded values |
| Written by hand | Typed from research |
| Created in Power BI | Built inside Power BI with a formula (DAX) |

## fact_daily

One row per day on which WTI oil futures traded (2,721 rows in the files, 4 January 2016 to 28 October 2026). Prices are futures settlement prices, not spot prices. Rates are in percent per year, so 3.88 means 3.88%. Returns are fractions, so 0.05 means 5%.

| Column | Meaning | Unit or values | Origin | How it is calculated |
|---|---|---|---|---|
| date | The trading day | Date | Downloaded: Yahoo Finance | Taken as published; the index of the Yahoo download, kept as a date |
| trading_day_seq | Running count of trading days; 1 is 4 Jan 2016. Used to count "4 trading days after an event" | Whole number | Calculated in Python | df["trading_day_seq"] = range(1, len(df) + 1) after sorting by date |
| open | WTI price at the start of the day | USD per barrel | Downloaded: Yahoo (CL=F) | No calculation. Downloaded with yfinance for CL=F |
| high | Highest WTI price of the day | USD per barrel | Downloaded: Yahoo (CL=F) | No calculation. Downloaded with yfinance for CL=F |
| low | Lowest WTI price of the day | USD per barrel | Downloaded: Yahoo (CL=F) | No calculation. Downloaded with yfinance for CL=F |
| close | WTI price at the end of the day; the main price used everywhere. One value is negative: minus 37.63 on 20 Apr 2020 | USD per barrel | Downloaded: Yahoo (CL=F) | No calculation. Downloaded with yfinance for CL=F; the negative value is kept, not removed |
| volume | Number of contracts traded. Unreliable (7 days at zero, 6 repeating the previous day), so it is not used | Contracts | Downloaded: Yahoo (CL=F) | No calculation. Downloaded with yfinance for CL=F |
| is_negative_price | True if open, high, low or close was zero or below (2 days in April 2020) | True or False | Calculated in Python | (df[["open","high","low","close"]] \<= 0).any(axis=1) |
| is_zero_volume | True if volume is zero (7 days) | True or False | Calculated in Python | df["volume"] == 0 |
| is_repeated_volume | True if volume equals the previous day's, which is probably a Yahoo error (6 days) | True or False | Calculated in Python | df["volume"] == df["volume"].shift(1) |
| abs_change | close minus the previous close | USD per barrel | Calculated in Python | df["close"].diff() |
| pct_change | close divided by the previous close, minus 1. Blank when either price is zero or below | Fraction | Calculated in Python | df["close"] / df["close"].shift(1) - 1, then set to blank where either close is 0 or below |
| log_return | Natural log of close divided by the previous close. Blank when either price is zero or below. Used for volatility and correlation | Fraction | Calculated in Python | np.log(df["close"] / df["close"].shift(1)), then set to blank where either close is 0 or below |
| close_positive | close, but blank when it is zero or below. Used to convert to euros | USD per barrel | Calculated in Python | df["close"].where(df["close"] \> 0) |
| brent | Brent oil futures close (BZ=F), the European benchmark. If missing, the last value is carried forward for up to 3 days | USD per barrel | Downloaded: Yahoo; gap filling in Python | Downloaded with yfinance (BZ=F), joined to the WTI dates, then .ffill(limit=3) |
| eurusd | Dollars needed to buy one euro (EURUSD=X) | USD per EUR | Downloaded: Yahoo; gap filling in Python | Downloaded with yfinance (EURUSD=X), joined to the WTI dates, then .ffill(limit=3) |
| dxy | US dollar index against a basket of currencies (DX-Y.NYB); higher means a stronger dollar | Index points | Downloaded: Yahoo; gap filling in Python | Downloaded with yfinance (DX-Y.NYB), joined to the WTI dates, then .ffill(limit=3) |
| dax | German stock index DAX (^GDAXI) | Index points | Downloaded: Yahoo; gap filling in Python | Downloaded with yfinance (^GDAXI), joined to the WTI dates, then .ffill(limit=3) |
| xle | Price of the Energy Select Sector fund (US energy stocks), not adjusted for dividends | USD per share | Downloaded: Yahoo; gap filling in Python | Downloaded with yfinance (XLE, unadjusted close), joined to the WTI dates, then .ffill(limit=3) |
| brent_log_return, eurusd_log_return, dxy_log_return, dax_log_return, xle_log_return | Daily log return of each series above. Blank on days when the value was carried forward, so a holiday never shows as a 0% move | Fraction | Calculated in Python | np.log(s / s.shift(1)) for each series s, then set to blank on days when that value was carried forward |
| n_market_fields_filled | How many of the five market series (brent, eurusd, dxy, dax, xle) were carried forward that day | 0, 1 or 2 | Calculated in Python | Sum of the five carried-forward flags per day: filled.sum(axis=1) |
| fed_funds | US Federal Reserve policy rate (effective federal funds rate, FRED series DFF) | % per year | Downloaded: FRED | No calculation. FRED series DFF, joined on date |
| us10y | Yield on 10-year US government bonds (FRED series DGS10); carried forward on days with no figure | % per year | Downloaded: FRED | FRED series DGS10, joined on date, then .ffill() on days with no figure |
| us_cpi_yoy | US consumer price inflation compared with 12 months earlier. Monthly value, repeated on every day of the month, starting on the 1st | % | Downloaded: FRED (CPI index CPIAUCSL); percentage change calculated in Python | cpi.pct_change(12) \* 100 on the monthly CPIAUCSL index, dated the 1st of the month and repeated on every day of that month |
| us_recession | 1 during a US recession as dated by the NBER (FRED series USREC), otherwise 0. Only March and April 2020 are 1 | 0 or 1 | Downloaded: FRED | No calculation. FRED series USREC (monthly 0 or 1), repeated on every day of the month |
| ecb_deposit_rate | European Central Bank deposit facility rate; stays the same between ECB decisions | % per year | Downloaded: ECB Data Portal | No calculation. ECB deposit facility rate, joined on date, then .ffill() until the next decision |
| ea_hicp_yoy | Euro-area consumer price inflation compared with a year earlier (HICP). Monthly value repeated through the month | % | Downloaded: ECB Data Portal (two ECB series combined; the newer one is used wherever it has a value) | new.combine_first(old): the newer ECB series where it has a value, otherwise the older one; monthly value repeated through the month |

## dim_date

One row per calendar day, including weekends, from 1 Jan 2016. Nothing in it is downloaded; it exists for slicers and axes.

| Column | Meaning | Values | Origin | How it is calculated |
|---|---|---|---|---|
| date | The calendar day | Date | Calculated in Python | pd.date_range(first_date, last_date, freq="D") |
| year | Year | 2016 to 2026 | Calculated in Python | d["date"].dt.year |
| quarter | Quarter of the year | Q1 to Q4 | Calculated in Python | "Q" + d["date"].dt.quarter.astype(str) |
| month_num | Month number; use it to sort month_name | 1 to 12 | Calculated in Python | d["date"].dt.month |
| month_name | Month abbreviation | Jan to Dec | Calculated in Python | d["date"].dt.strftime("%b") |
| year_month | Year and month. The file holds text such as 2016-01, but Power BI reads it as the 1st of that month | Text, or a date in Power BI | Calculated in Python | d["date"].dt.strftime("%Y-%m") |
| iso_week | International (ISO) week number. Early January can belong to week 52 or 53 of the previous year | 1 to 53 | Calculated in Python | d["date"].dt.isocalendar().week |
| weekday_num | Day of the week | 1 is Monday, 7 is Sunday | Calculated in Python | d["date"].dt.weekday + 1 (pandas counts Monday as 0) |
| is_weekday | True from Monday to Friday | True or False | Calculated in Python | d["weekday_num"] \<= 5 |
| regime | The period the day belongs to, taken from the dates in dim_regime | The six names below | Calculated in Python, using the regime dates chosen by judgement | For each row of dim_regime: d.loc[d["date"].between(start_date, end_date), "regime"] = regime |

## dim_regime

Six periods defined to compare risk across different market conditions. The boundaries are set around known events.

| Column | Meaning | Values | Origin | How it is calculated |
|---|---|---|---|---|
| regime | Name of the period | Pre-COVID, COVID shock, Reopening recovery, Ukraine war spike, Post-spike normalisation, Hormuz crisis | Written by hand | No calculation. Typed in the REGIMES list at the top of the notebook |
| start_date | First day of the period | Pre-COVID starts 1 Jan 2016; COVID shock 20 Feb 2020; Reopening recovery 1 Jan 2021; Ukraine war spike 24 Feb 2022; Post-spike normalisation 1 Jan 2023; Hormuz crisis 28 Feb 2026 | Written by hand | No calculation. Typed in the REGIMES list; the dates mark the start of each period by judgement |
| end_date | Last day of the period. 31 Dec 2099 on Hormuz crisis means it is still ongoing | Date | Written by hand | No calculation. Typed in the REGIMES list; set to the day before the next period starts. The Hormuz crisis gets 2099-12-31 as an open end |

## dim_event

Forty events that may explain price moves. They are compiled from research. The last three columns were added in Python so that each event can be linked to the price rows.

| Column | Meaning | Values | Origin | How it is calculated |
|---|---|---|---|---|
| event_id | Unique code | E01 to E40 | Written by hand | No calculation. Typed in events_template.csv |
| event_date | Date of the event as reported; can fall on a weekend | Date | Written by hand | No calculation. Typed in events_template.csv; read with pd.to_datetime |
| date_precision | Whether the date is exact or only the month is known | day (37 events), month (3 events) | Written by hand | No calculation. Typed in events_template.csv (day when the exact date is known, month otherwise) |
| event_name | Short title | Text | Written by hand | No calculation. Typed in events_template.csv |
| category | Type of event | Supply & OPEC+ (10), Geopolitical (9), Market milestone (8), Monetary policy (5), Demand shock (4), Policy (3), Market structure (1) | Written by hand | No calculation. Typed in events_template.csv; one category per event |
| importance | Ranking of how big the event was (judgement) | 1 for major (14 events), 2 for other (26 events) | Written by hand | No calculation. Typed in events_template.csv; judgement (1 or 2) |
| description | What happened. Price moves quoted in it come from the WTI data | Text | Written by hand | No calculation. Typed in events_template.csv; price moves quoted there were read from the WTI file |
| source_name | Where the information comes from | Text | Written by hand | No calculation. Typed in events_template.csv |
| source_url | Link to the source (empty for 3 events) | Web address | Written by hand | No calculation. Typed in events_template.csv |
| verification | How far to trust the event | researched (a source was read; 37 events) or dataset (taken from the price data itself, with no outside source; 3 events) | Written by hand | No calculation. Typed in events_template.csv; one status per event |
| anchor_date | First trading day on or after event_date; a weekend event moves to Monday | Date | Calculated in Python | First trading day on or after event_date: trading_dates[trading_dates \>= event_date].min() |
| anchor_seq | The trading_day_seq of anchor_date; this links an event to the price rows | Whole number | Calculated in Python | Look up anchor_date in fact_daily and take its trading_day_seq (a merge on date) |
| is_day_precision | True when date_precision is day, meaning the event can be placed on a chart | True or False | Calculated in Python | ev["date_precision"] == "day" |

## Created inside Power BI

**Offset table.** A helper table with one column, Offset, holding the whole numbers from minus 5 to 20. It is the horizontal axis of the event path chart (trading days before and after an event). Origin: created in Power BI.

**What-if parameters.** The budget page uses three parameter tables, each with a slider: Annual fuel budget (EUR), Crude-linked share (0% to 100% in steps of 5 points, default 40%) and Oil shock % (-30% to +50% in steps of 5 points, default 0%). A small helper table, Shock Steps, feeds the budget chart. Origin: created in Power BI.

**Measures.** Formulas that Power BI recalculates for whatever filters are active. Origin of all of them: created in Power BI.

| Measure | What it answers | How it is calculated (DAX) |
|---|---|---|
| Last Close | The WTI price on the latest date in view | CALCULATE(MAX(fact_daily[close]), fact_daily[date] = MAX(fact_daily[date])): the close on the latest date left in the filter |
| Avg Close | Average WTI price in view | AVERAGE(fact_daily[close]) |
| Avg Close EUR | Average WTI price converted to euros | AVERAGEX(fact_daily, DIVIDE(fact_daily[close_positive], fact_daily[eurusd])): convert each day to euros first, then average |
| Avg EURUSD | Average exchange rate in view | AVERAGE(fact_daily[eurusd]) |
| Euro Gap % | How much cheaper or dearer a barrel is in euros than in dollars | DIVIDE([Avg Close EUR], [Avg Close]) - 1 |
| YTD Change % | Price change since the last close of the previous year | DIVIDE([Last Close], close on the last trading day of the previous year) - 1 |
| Annualised Vol | How much the price swings, scaled to a year; higher means riskier | STDEV.S(fact_daily[log_return]) \* SQRT(252): standard deviation of daily log returns, scaled to a year of 252 trading days |
| Vol 30D | The same swing measure, over the last 30 trading days | The same formula as Annualised Vol, restricted to the 30 trading days up to the date (a filter on trading_day_seq) |
| Drawdown | How far today's price is below the highest price so far in the selection | DIVIDE(close of the day, highest close up to that day in the selection) - 1; never below -100% |
| Max Drawdown | The deepest drawdown in the selection | MINX(VALUES(fact_daily[date]), [Drawdown]): the lowest Drawdown over the days in view |
| Event Window Return (5d) | Price change from the day before an event to 4 trading days after | DIVIDE(close at anchor_seq + 4, close at anchor_seq - 1) - 1, using the Offset table to move along trading_day_seq |
| Event Path Index | Price after an event, with the day before set to 100 | 100 \* DIVIDE(close at anchor_seq + Offset, close at anchor_seq - 1) |
| Event Marker, Event Names | Show events as markers and in tooltips on the price chart | Event Marker: the close on days that have an event, blank otherwise. Event Names: CONCATENATEX of the event names for the day, separated by a line break |
| Corr WTI vs ... | How closely daily oil returns moved with another series, from minus 1 to 1 | Pearson correlation written with SUMX over the days where both log returns exist: covariance divided by the product of the two standard deviations |
| Budget Impact EUR, Budget After Shock | What a chosen oil price change would do to a fuel budget | Budget Impact EUR = Annual fuel budget \* Crude-linked share \* Oil shock %. Budget After Shock = Annual fuel budget + Budget Impact EUR |
