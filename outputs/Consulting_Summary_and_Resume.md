# Outlook magazine subscription analysis

## Consulting summary

**The combined opening subscription balance for the three monthly titles increased from 1,504,783 in January 2020 to 1,885,981 in December 2022, but large unverified fluctuations make forward estimates uncertain.** The endpoint increase was 25.3%, equivalent to an 8.05% annualized rate over the 35 months between the observations. These are active subscription balances at the start of a month, not new sales or revenue.

Outlook Business accounted for most of the combined series' visible variability. The combined balance peaked at 3,068,328 in May 2022 and reached a low of 1,031,442 in December 2021. The Business data and several year-boundary changes remain unverified, so the analysis does not attribute changes to a particular campaign, market shift or operational event.

**The forecast should be treated as a scenario.** Among the models tested on July–December 2022, a Holt–Winters model had the lowest combined-series RMSE, at 398,741 subscriptions, with MAPE of 22.16%. It was selected by RMSE, which penalizes large errors more heavily. A seasonal-naive model had a lower MAPE of 16.06%, demonstrating that model choice depends on the error measure and business decision. One six-month test period is insufficient to establish future reliability.

Refitting Holt–Winters to the full history produced forecasts for January–December **2023**, the year immediately after the supplied data. The combined January 2023 point forecast was approximately 1,512,208 and the December forecast was approximately 1,406,929. The simulated December interval was approximately 195,164–2,614,227. These are historical projections from the end of 2022, not forecasts for the coming year or verified 2023 results. The approximate 95% interval is conditional on the fitted model and does not fully reflect source errors, model selection or structural change.

**Seasonality is indicative, not established.** The raw May index was 1.355 and the February index was 0.749, relative to the overall monthly mean. Each calendar-month average contains only three observations, and the raw index also reflects trend and unusual values. It does not establish a festival effect or a stable recurring peak.

### Method

The original workbook contains five titles. Three supply 36 monthly observations each. Outlook Hindi and Outlook India have fortnightly and weekly issue observations respectively; issue dates are absent, so they were excluded from calendar-month aggregates. The user confirmed that the ambiguous Hindi 2020 issue 25 value was 146,222. Other unusual values were retained and flagged as unverified.

For each series, the analysis calculated descriptive statistics and examined growth and variation. For the monthly series it calculated raw monthly indices, used additive seasonal decomposition and ran ADF tests. It compared naive, seasonal-naive, three-month moving average, simple exponential smoothing, Holt and Holt–Winters forecasts using a six-month chronological holdout. RMSE was the selection criterion and MAPE a secondary comparison. An additive seasonal model was fitted cautiously; SARIMA was omitted because the training sample contained only 30 months and unexplained jumps.

### Recommendations

1. **Validate the unusual values and reporting definitions** with the data owner before using the forecast for a print or budget commitment.
2. **Build a dated subscription movement dataset** with openings, acquisitions, renewals, expiries and cancellations. Add issue-level copies sold, returns, price and campaign information for sales and print-run planning.
3. **Use a range of scenarios and repeat backtesting** as new observations arrive. Compare against simple benchmarks over multiple historical windows and track actual forecast errors.

### Limitations

There are only three annual cycles for monthly titles, one short holdout and no supplied external drivers. The forecast intervals are model-dependent. Expiries are not necessarily cancellations, and no evidence establishes revenue, new sales, profit or waste reduction. The combined monthly series excludes two titles and is not a complete group total. No operational or financial impact has been measured.

## Resume bullets

Use a bullet only if it accurately reflects work you personally completed or can explain and reproduce. If you worked with AI or another person, describe your contribution truthfully. These bullets describe the rebuilt analysis; do not attribute it to the original internship unless that is accurate.

- Analysed 342 subscription-period records across five magazine titles, identified mixed reporting frequencies and documented an ambiguous source value before cleaning.
- Compared six forecasting approaches on a six-month holdout for three monthly titles and their combined balance; the selected combined Holt–Winters model recorded 398,741 subscriptions RMSE and 22.16% MAPE.
- Built an Excel trendline workbook and a three-page Power BI subscription report, with a 12-period forecast, a 95% model-based interval and year/title filters.
- Produced a six-slide client-style presentation explaining the 25.3% change in the three-title combined opening balance, forecast limitations and data-quality recommendations.

**Skills, if you can demonstrate them:** Power BI (Power Query, DAX), Excel (trendlines, forecasting functions), Python (pandas, statsmodels, matplotlib), PowerPoint.
