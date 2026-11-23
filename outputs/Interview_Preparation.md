# Interview preparation: subscription forecasting

Use the first person only for work you actually performed. The current project is a reconstruction using the supplied dataset and assisted tooling. Never imply the forecast was deployed or that it saved money unless you have separate evidence.

## 60-second STAR answer

**Situation:** “I had a magazine dataset with subscription openings and expiries and wanted to understand the trend and forecast future balances.”

**Task:** “The aim was to build a defensible forecasting process and a clear business-facing report.”

**Action:** “I first checked definitions and found that three titles were monthly while two were issue-based without calendar dates. I analysed the monthly titles, compared six methods using the last six months as a holdout, and built Excel and Power BI views. I kept unusual source values but flagged them as unverified.”

**Result:** “The combined balance of the three monthly titles rose 25.3% between January 2020 and December 2022. Holt–Winters had the lowest test RMSE of 398,741 subscriptions, with 22.16% MAPE. The wide uncertainty means I would use the forecast as a scenario and validate the data before making print commitments.”

## Two-minute STAR answer

**Situation:** “In a magazine subscription project, the available workbook contained opening subscription counts and expiries for five titles. I wanted to reconstruct the analysis in a way I could explain to a business stakeholder.”

**Task:** “My objective was to identify historical patterns, compare suitable forecasting methods and present an honest estimate with its limitations. I needed to avoid calling an opening subscription balance sales.”

**Action:** “I inspected each sheet before modelling. Three titles had 36 monthly observations, but the other two were weekly or fortnightly, and their issue dates were not available. I kept those at their original frequency rather than forcing them into monthly totals. I confirmed an ambiguous value with the data owner and marked sharp changes as unverified. I calculated descriptive statistics, growth rates, a monthly index and stationarity tests. I then held out July to December 2022 and compared naive, seasonal-naive, moving-average, simple exponential smoothing, Holt and Holt–Winters forecasts. I selected on RMSE and examined MAPE as a second measure. I also used Excel trendlines and created Power BI views of balances, title comparisons and a forecast.”

**Result:** “The combined opening balance for the three monthly titles increased from about 1.50 million to 1.89 million over the observed endpoints, a 25.3% increase. The combined Holt–Winters model had the lowest holdout RMSE, about 399,000 subscriptions, and 22.16% MAPE. I would not claim an operational saving from that. The business recommendation is to investigate the source jumps, collect dated subscription movements and compare new forecasts with simple benchmarks before using them for decisions.”

## 20 likely questions and model answers

1. **What did you forecast?** Opening active subscription balances and expiries. For the combined monthly forecast I used opening balances for Outlook Business, Traveller and Money. Those balances are not revenue or new sales.
2. **What is the difference between a stock and a flow?** An opening balance is a stock at a point in time. Expiries are a count over a period. Summing the same opening balance across months would repeatedly count subscriptions.
3. **Why not include all five titles in the monthly total?** Hindi and India have issue-level observations but no issue dates. Without dates and an agreed balance convention, I cannot reliably map their openings to calendar months.
4. **What cleaning did you perform?** I reshaped the year blocks, converted comma-formatted text to numbers, checked period keys, and applied the confirmed Hindi value of 146,222. I retained other unusual values as unverified.
5. **What is the overall trend?** The three-title combined opening balance was 1,504,783 in January 2020 and 1,885,981 in December 2022, a 25.3% endpoint increase. The series fluctuated sharply in between.
6. **How did you calculate CAGR?** I used `(ending balance / starting balance)^(12 / 35) - 1`, because the endpoints are 35 months apart. The result is 8.05% annualized, not a claim of steady monthly growth.
7. **How did you handle seasonality?** I examined raw month-wise indices and a 12-month additive decomposition. Only three years were available, so I treated recurring seasonality as tentative and tested seasonal forecasts rather than assuming them correct.
8. **What does the May index of 1.355 mean?** May's average observed combined balance was 35.5% above the overall monthly mean. The raw index can reflect trend and outliers as well as seasonality, so it does not prove a seasonal cause.
9. **What is stationarity and why does it matter?** A stationary process has stable statistical properties over time. Many time-series methods rely on assumptions about stability; non-stationarity can make an apparent relationship unreliable. The ADF test is one diagnostic, not a complete definition or proof.
10. **What did the ADF test show?** For the combined opening series the p-value was approximately 0.111, so I did not reject the unit-root null at 5%. With 36 observations, the test has limited power; failure to reject does not prove non-stationarity.
11. **Why use a holdout?** A model can fit historical data well and forecast poorly. Keeping July–December 2022 out of fitting lets us compare six-month fixed-origin predictions with values the model had not seen.
12. **Why Holt–Winters for the combined series?** It had the lowest RMSE, approximately 398,741 subscriptions, among the methods compared in that six-month holdout. That is a test-specific result, not proof that it will always perform best.
13. **Why not simply choose the model with the lowest MAPE?** MAPE and RMSE penalize errors differently. Seasonal naive had lower combined MAPE, 16.06%, while Holt–Winters had lower RMSE. I chose RMSE to emphasize large absolute misses, but would discuss the business cost before finalizing the criterion.
14. **What does RMSE mean?** Take each error, square it, average those squares and take the square root. The result is in the original unit and is more sensitive to large errors than an average absolute error.
15. **What does MAPE mean?** It is the average absolute forecast error divided by the corresponding actual, expressed as a percentage. It is hard to use when actuals are zero or close to zero.
16. **What is R²?** It describes the proportion of historical outcome variation explained by a fitted relationship under the relevant definition. The combined linear trendline's R² was about 0.0099. A high in-sample R² would not guarantee a good extrapolation.
17. **Why not use linear regression alone?** The line is interpretable but explained very little of this volatile history. I used it as a benchmark and compared holdout performance rather than assuming a straight line captures the pattern.
18. **Why not SARIMA or machine learning?** The monthly training sample had only 30 observations, unexplained jumps and about two and a half annual cycles. More complex models could overfit and would need additional validation; simplicity is a data-driven choice here.
19. **How reliable is the forecast?** The combined selected model's MAPE was 22.16% in one six-month window. Intervals are wide and depend on model assumptions. I would not treat the nominal 95% interval as a guarantee, especially with unverified inputs.
20. **How would you present this to a client, and what would you do next?** Lead with the change in balances and the uncertainty, show the holdout error and explain the limited scope. Recommend verifying jumps, obtaining issue dates and subscription movements, and testing several historical windows before operational use.

## Eight Power BI questions

1. **Power Query versus DAX?** Power Query imports and transforms data during refresh. DAX defines model calculations, including measures that respond to report filters.
2. **Calculated column versus measure?** A calculated column is evaluated for each row at refresh and stored. A measure is evaluated in the current filter context when used in a visual. An opening-balance KPI should use a measure so the selected date matters.
3. **Why a Date table?** A contiguous date dimension gives the model a consistent calendar for filtering, grouping and classic time-intelligence calculations. It relates one date to many monthly title observations.
4. **How does SAMEPERIODLASTYEAR work?** It returns the corresponding dates shifted one year back in the current date context. `CALCULATE` evaluates a measure over those dates. Check the date context and mark a suitable Date table for classic time intelligence.
5. **How does the built-in forecast work, and what are its limits?** Power BI's line-chart forecast uses exponential smoothing. A continuous axis, a single series and appropriate settings matter. The output depends on the history and settings and does not by itself demonstrate holdout accuracy.
6. **What is a star schema?** A central fact table contains observations and numeric measures, while related dimension tables describe attributes such as date and title. Dimension filters flow to the fact table. The saved report uses a Date dimension and the title field in the fact table; the rebuild guide also describes a separate magazine dimension.
7. **How do slicers filter visuals?** A slicer creates filter context. The model relationship and visual interactions determine which data a visual receives. I tested the title slicer: selecting Business changed its YoY figure to 280.80%.
8. **How would you share the dashboard?** For a static review, export to PDF with the selected filter state. For interactive sharing, publish through an appropriately licensed Power BI service workspace and grant the client access according to the organization's permissions. Confirm confidentiality and access requirements first.

## Five difficult follow-ups

1. **Your slide says the balance rose 25.3%, yet your model forecast ends below December 2022. Is that contradictory?** No. The 25.3% compares two historical endpoints. The forecast is a separate projection of future month-start balances based on the selected model; it is uncertain and need not continue the endpoint increase.
2. **Why is Business's December 2022 YoY 280.8%? Did the business really grow that quickly?** It is the arithmetic change from the December 2021 recorded opening to December 2022. The low 2021 value and the jump are unverified; I would not assert a business cause or treat it as confirmed performance.
3. **Why does your forecasted combined total differ from the sum of title forecasts?** I fitted the aggregate series independently from individual title series. Independent forecasts are not automatically coherent. A reconciliation method could enforce consistency if the business needs both levels to add up.
4. **Why might the 95% interval be too narrow even if simulated correctly?** The simulation assumes the selected model and its fitted error process are appropriate. It omits some uncertainty from model selection, reporting errors, changing behaviour and external shocks, and has not demonstrated empirical 95% coverage.
5. **What exactly did Power BI's 12-period forecast predict?** The line chart extrapolates the combined monthly opening subscription balance from the data ending December 2022. Twelve forecast points cover 2023. It is not a prediction of revenue, new sales or actual results observed in 2023.

## One-page revision sheet

### Key numbers

| Item | Value and scope |
|---|---|
| Source | Five titles, 2020–2022; 342 period records |
| Monthly forecast scope | Business, Traveller and Money; 36 observations each |
| Combined opening balance | 1,504,783 in Jan 2020; 1,885,981 in Dec 2022 |
| Endpoint change / annualized | +25.3% / +8.05% over 35 months |
| Combined historical high / low | 3,068,328 in May 2022 / 1,031,442 in Dec 2021 |
| Combined raw month indices | May 1.355; Feb 0.749; three observations per month |
| Holdout / winner | Jul–Dec 2022; Holt–Winters by RMSE |
| Combined holdout errors | RMSE 398,741 subscriptions; MAPE 22.16% |
| Forecast horizon | Jan–Dec 2023, immediately after the source data |
| Main limits | Source jumps unverified; 36 months; no issue dates for two titles; no sales/revenue |

### Formulas

- **MoM / YoY:** `(current − prior comparable period) / prior comparable period`.
- **Annualized endpoint change:** `(end / start)^(12 / elapsed months) − 1`.
- **RMSE:** `sqrt(mean((actual − forecast)^2))`.
- **MAPE:** `mean(abs((actual − forecast) / actual)) × 100`, where actual is nonzero.
- **Excel linear forecast:** `=FORECAST.LINEAR(new_x, known_y, known_x)`.
- **DAX examples:** `Expiries = SUM(Monthly[Expiry])`; `YTD Expiries = TOTALYTD([Expiries], 'Date'[Date])`; prior-year evaluation uses `CALCULATE` and `SAMEPERIODLASTYEAR`. See the build guide for the full semi-additive balance and growth measures.

The word “one-page” refers to this compact revision section; print scaling may vary by application.
