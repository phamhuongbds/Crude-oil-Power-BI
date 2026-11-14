# Crude Oil Project: Methodology Report

This report explains how a Power BI report on ten years of crude oil prices was designed: which business questions it answers, which measures answer them and why, and why each data source was chosen. It is written for readers who want to judge the method and not only the charts.

## The problem and the reader

Oil is one of the largest and least predictable costs for a company that moves goods or runs machinery. The report is built for an analyst who advises such a company in Germany, for example a logistics firm or a manufacturer, and who must answer three practical questions: how risky is the oil price, what does it cost in euros, and what to expect when the next shock arrives?

The project is educational and descriptive. It does not forecast prices or recommend trades. Its value lies in showing how a messy, real-world dataset is turned into a clear answer to a business question, with every choice documented.

The data covers daily WTI crude futures from 04 January, 2016 to 28 October, 2026. The decade includes the 2020 collapse (including one negative price), the 2022 price spike after Russia's invasion of Ukraine, and a second spike in 2026 when shipping through the Strait of Hormuz was disrupted.

## Business questions

Starting from questions and not from the data keeps every page of the report tied to a decision. Six were chosen, each of which needs a different kind of evidence.

| # | Question | Why it matters to the company | Data used | Measures |
|---|---|---|---|---|
| 1 | How did oil's volatility change across market periods? | A fuel budget is only as reliable as the price is stable; volatility tells how wide the range of outcomes was. | WTI daily returns; six periods called regimes | Annualised Vol, Vol 30D |
| 2 | How deep were the falls, and how long did recovery take? | Shows the worst case someone buying at a peak would have faced, and how long the loss lasted. | WTI close | Drawdown, Max Drawdown |
| 3 | What did a barrel cost in euros compared with dollars? | The company pays in euros, so the exchange rate can soften or add to a price move. | WTI close, EUR/USD | Avg Close EUR, Avg EURUSD |
| 4 | How do rates, the dollar, inflation and other markets relate to oil, and did that change over time? | Gives the macro context for price moves and shows whether old relationships still held in a crisis. | Fed and ECB rates, 10-year US yield, US and euro inflation, DXY, DAX, XLE, Brent | Corr WTI vs ... (five measures) |
| 5 | How did the price react to major events, and did the reaction match the headline? | Links numbers to real causes and tests whether big news always moves prices the way people expect. | 40 dated events | Event Window Return (5d), Event Path Index, Event Marker |
| 6 | What would a change in oil mean for a fuel budget? | Turns price data into a decision the reader can try for themselves. | Annual budget and oil change as inputs | Budget Impact EUR, Budget After Shock |

The questions are ordered from the simplest (what happened to the price) to the most applied (what would it do to a fuel budget). Questions 1 to 3 describe the price on its own. Question 4 puts it in context, question 5 gives it causes, and question 6 makes it practical.

### Why these measures were chosen

For each question, the measure was chosen because it answers the question without misleading, and simpler alternatives that would have misled were rejected. The table gives the reason for each choice.

| Measure | Question | What it calculates | Why this one and not a simpler alternative |
|---|---|---|---|
| Annualised Vol | 1 | Standard deviation of daily log returns, multiplied by the square root of 252 trading days, for the selected period | Returns are comparable at any price level; a standard deviation of the price itself is not, because a \$2 move means something different at \$30 and at \$100. Annualising puts the figure on the scale used for other markets. Log returns can be computed safely around the negative price of 20 April 2020, where a percentage change is undefined. |
| Vol 30D | 1 | The same calculation over the last 30 trading days | One figure per period hides spikes inside it. A 30-day window is long enough for a stable estimate and short enough to show when a calm period turns into a crisis. |
| Drawdown | 2 | Today's close divided by the highest close so far, minus 1 | It measures the loss someone who bought at the high is carrying. A plain high-to-low range ignores the order of events (a high that came after the low is no loss), and a year-on-year change hides falls within the year. |
| Max Drawdown | 2 | The worst Drawdown within the selected period, capped at -100% | One number per period makes the six regimes comparable. The cap stops the negative price of 2020 from producing a loss above 100%. |
| Avg Close EUR and Avg EURUSD | 3 | Each day's close divided by the same day's EUR/USD rate, then averaged | Converting day by day keeps days when oil and the exchange rate moved together. Converting the average price with the average rate would hide that. An average is used instead of the last price because a company buys fuel throughout the year. |
| Euro Gap % | 3 | Average euro price divided by average dollar price, minus 1 | Shows in one number how much of the dollar price the exchange rate removes or adds. |
| Corr WTI vs Brent, DAX, XLE, EUR/USD, DXY | 4 | Pearson correlation between the daily log returns of WTI and each series, written out with SUMX | Correlating returns, not price levels, avoids the false links that two trending series show. Power BI's formula language (DAX) has no built-in correlation function that recalculates for each period and year, so the formula is written out. Days where either series has no return are left out. |
| Event Window Return (5d) | 5 | Close four trading days after the event divided by the close the day before it, minus 1 | Weekend and after-hours news reaches the market on the next trading day, so each event is anchored to its first trading day. A same-day change would miss delayed reactions, and a longer window would collect other news. |
| Event Path Index | 5 | Close on each day from 5 days before to 20 days after the event, divided by the close the day before, times 100 | Rebasing to 100 lets events at different price levels be compared, and it shows the shape of the reaction, not only the end point. |
| Event Marker and Event Names | 5 | The close on event days only, and the event names on those days | Puts the events on the price chart so a reader can link a move to its date and cause. |
| Budget Impact EUR and Budget After Shock | 6 | Annual budget times crude-linked share times oil price change; and the budget plus that impact | The simplest model a reader can check by hand. A measured pass-through would need German pump prices, which this project does not include, so the share is an explicit input the reader can change. |
| Last Close and YTD Change % | Overview | The latest close in the selection; and the latest close divided by the last close of the previous year, minus 1 | Give the reader the current level and the change since the start of the year before the six questions begin. |

## Data choices

Each source was added because a specific question needed it. Nothing was included just because it was available.

| Data | Source | Why it was chosen | Main limit |
|---|---|---|---|
| WTI crude oil, daily | Yahoo Finance, ticker CL=F | The core series; a US benchmark with a long, clean daily history. | A continuous futures contract, so prices can jump when the contract rolls. It is a futures price, not a spot price. |
| Brent crude, daily | Yahoo Finance, BZ=F | The European benchmark; the Brent-WTI spread shows how regional supply shocks differ. | Same futures limits as WTI. |
| EUR/USD | Yahoo Finance, EURUSD=X | Needed to price oil in euros (question 3). | None significant. |
| US dollar index (DXY) | Yahoo Finance, DX-Y.NYB | A broad measure of dollar strength, since oil is priced in dollars. | A few missing days. |
| DAX and the US energy fund XLE | Yahoo Finance, ^GDAXI, XLE | Shows how German stocks and energy stocks moved with oil. | XLE is not adjusted for dividends. Both have market-holiday gaps. |
| US federal funds rate, 10-year US yield, US consumer prices, US recession flag | FRED (Federal Reserve Bank of St. Louis) | The main interest-rate and inflation context for the dollar and for global energy demand. | Inflation is monthly and published weeks after the month ends. |
| ECB deposit rate and euro-area inflation | ECB Data Portal | The European side of the same story, which matters for a German company. | The inflation series was split in two and had to be joined. |
| 40 dated events | Public sources (EIA, Chase, news reports) | Gives each price move a real-world cause (question 5). No ready-made dataset exists, so it was built for this project. | A source link exists for 37 of the 40 events; the other 3 are taken from the price data itself. Dates, categories and importance are judgement. |

**Left out.** German pump prices from the EU Weekly Oil Bulletin would have shown how fast crude moves reach the consumer. They were excluded from this version because they need a manual download in a different format. They are the natural next step. The Adjusted_Close column (identical to Close on a futures series) and the trading volume (7 days at zero and 6 repeated values make it unreliable) were also dropped.

## Data preparation decisions

Real data needs judgement calls. These are the ones that change the results:

- **The negative price.** WTI closed at minus \$37.63 on 20 April 2020, when an expiring contract met full storage. A percentage change from or to a negative price has no meaning, so the returns for that day and the next are left blank, and the price column used for the euro conversion is blank for the same day. The days were kept in the data, not deleted, because they are real and part of the story. Drawdown is floored at minus 100%.

- **Log returns for volatility.** Volatility and correlations use the natural log of the day-to-day price ratio. Log returns can be added over several days and are comparable across large and small moves. Volatility is the standard deviation of daily log returns, multiplied by the square root of 252 (the usual number of trading days in a year) to express it per year.

- **Futures, not spot.** Yahoo's CL=F is the front-month futures contract. Some news sources quote spot prices, which can differ by several dollars. For example, one source puts Brent at \$113.96 at the end of September 2026, while the Brent futures close in this data is \$103.53. The report uses the futures series throughout.

- **One row per oil trading day.** The main table follows the days oil traded. Other markets close on different days (the DAX on German holidays, for instance). Where a value is missing, the last value is carried forward for at most three days, and the return that day is left blank, so a market holiday is never counted as a 0% move.

- **Inflation timing.** Monthly inflation is placed on the first day of the month it describes, before it was published. That suits describing history, but it would mislead any test of whether oil led or followed inflation, so the report does not make that claim.

- **The six regimes** are periods defined to compare risk: before COVID (to 19 February 2020), the COVID shock (to the end of 2020), the reopening recovery (2021 to 23 February 2022), the Ukraine war spike (24 February to the end of 2022), the post-spike normalisation (2023 to 27 February 2026) and the Hormuz crisis (from 28 February 2026). The boundaries are judgement, not data, so they are stored in a table anyone can edit.

- **Event windows.** For each event the report compares the close before the event with the close four trading days after it. Five sessions is long enough to capture the market's reaction, and short enough to limit other news. Events that fall on a weekend use the next trading day. Three events are known only by month, so they appear in the event list but not as markers on the price chart.

## Model and measures

**The model.** The report uses a star schema: one table of daily facts in the middle, linked to three small lookup tables.

| Table | One row per | Role |
|---|---|---|
| fact_daily | Oil trading day (2,721 rows) | WTI, Brent, EUR/USD, DXY, DAX, XLE, rates, inflation and the returns |
| dim_date | Calendar day | Year, quarter, month and the regime label for slicers |
| dim_event | Event (40 rows) | Event name, category, source and the trading day it maps to |
| dim_regime | Regime (6 rows) | Names and dates of the six periods, also used to sort them in time order |

The lookup tables filter the daily table in one direction only. This keeps the results predictable: a slicer on the year or the regime narrows the daily rows and never the reverse.

**The measures.** A measure is a formula that Power BI calculates. Each one answers one of the business questions.

| Measure | Logic in plain words | Question |
|---|---|---|
| Annualised Vol | Standard deviation of the daily log returns in the selection, times the square root of 252. Higher means a less stable price. | 1 |
| Vol 30D | The same calculation over the 30 trading days up to the last day selected. Shows how risk changes over time. | 1 |
| Drawdown | How far the latest price is below the highest price reached so far in the selection, floored at minus 100%. | 2 |
| Max Drawdown | The deepest drawdown in the selection. | 2 |
| Avg Close, Avg Close EUR, Avg EURUSD | The average dollar price, the average price after dividing each day's price by that day's EUR/USD rate, and the average exchange rate. | 3 |
| Euro Gap % | Average euro price divided by average dollar price, minus 1: how much of the dollar price the exchange rate removes or adds. | 3 |
| Corr WTI vs Brent, XLE, DAX, EURUSD, DXY | Pearson correlation between daily oil returns and the daily returns of the other series. A number between minus 1 and 1. Power BI has no built-in correlation function, so the formula is written out. | 4 |
| Event Window Return (5d) | Price change from the close before an event to the close four trading days after it. | 5 |
| Event Path Index | The price in the days around an event, with the day before set to 100. Lets events of very different size be compared. | 5 |
| Event Marker, Event Names | Show events as markers and in tooltips on the price chart. | 5 |
| Budget Impact EUR, Budget After Shock | Annual budget times the crude-linked share times the oil price change, and the budget after that change. All three inputs are sliders. | 6 |
| Last Close, YTD Change % | The latest price, and its change since the last close of the previous year. Context for the overview page. | Overview |

Each of these measures was checked against an independent calculation in Python; the checks are listed below.

**Checks.**

- The yearly average WTI prices in dollars and euros were recomputed from the daily file and matched the report.
- In every year, the euro gap equals 1 divided by the average EUR/USD, minus 1, to within half a point.
- Drawdown peaks, lows and recovery dates were recomputed from the daily closes (for example, the high of \$123.70 on 8 March 2022 and the low of \$55.27 on 16 December 2025).
- The event window returns and event paths for the events quoted in the report were recomputed from the daily closes.
- The budget figures were recomputed by hand: EUR 40,000 for every 10 points of oil price change at a 40% crude-linked share of a EUR 1,000,000 budget.

**Limits of the analysis.**

- **Description, not cause.** Correlations and event windows show what moved together or after each other. Other news arrives on the same days, so no event's effect can be separated out.

- **Futures prices.** The oil series is a continuous futures contract, which can jump when the contract changes and differs from spot prices quoted in the news.

- **Events.** The 40 events come from public sources, which are listed with a link for 37 of them. The other 3 are taken from the price data itself.

- **Regimes are a judgement.** Different boundary dates would give different numbers.

- **Inflation timing.** The monthly inflation values are dated to the start of the month they describe, before they were published.

- **The budget what-if rests on an assumption.** The share of a fuel bill that follows the crude price (40% by default) is a placeholder, not a measurement. Any real use needs the company's own figure.

- **No pump-price data yet.** The report shows crude and not the price at the pump, so it does not measure how quickly the consumer feels a change.

- **Source terms.** Yahoo Finance data is for personal use, so this is an educational project and not a data product.

- **Next steps.** Add German pump prices from the EU Weekly Oil Bulletin to measure how fast crude moves reach consumers, add a page on the Brent-WTI spread, and automate the refresh so the report updates whenever the notebook is run.

## Sources

- Market prices: Yahoo Finance (finance.yahoo.com), tickers CL=F, BZ=F, EURUSD=X, DX-Y.NYB, ^GDAXI, XLE.

- US interest rates, yields, consumer prices and recession dates: FRED, Federal Reserve Bank of St. Louis (fred.stlouisfed.org), series DFF, DGS10, CPIAUCSL and USREC.

- ECB deposit rate and euro-area inflation: ECB Data Portal (data.ecb.europa.eu).
