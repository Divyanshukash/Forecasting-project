# Outlook magazine subscription forecasting

An exploratory analysis of opening active subscription balances and subscription expiries for five Outlook Group magazine titles. The project compares simple forecasting methods, demonstrates Excel trendlines, and presents the results in Power BI and a client-style slide deck.

**The data measures subscriptions, not sales or revenue.** The workbook's last observations are from 2022, so the 12-month forecasts in this repository cover 2023 as a historical forecasting exercise. No 2023 actuals are available here to verify those forecasts.

## Key findings

- The combined opening balance for the three monthly titles increased from **1,504,783 in January 2020** to **1,885,981 in December 2022**, a 25.3% endpoint increase. That is an annualized change of 8.05% over 35 months, not a steady growth rate.
- Using July–December 2022 as a six-month holdout, Holt–Winters had the lowest RMSE among the tested models for the combined monthly balance: **398,741 subscriptions RMSE** and **22.16% MAPE**. Seasonal naive had a lower MAPE, illustrating why the chosen accuracy metric matters.
- The raw month-wise index was highest in May (1.355) and lowest in February (0.749). Each month has only three annual observations; trend and unusual source values can affect these raw averages, so this is weak evidence of recurring seasonality.
- The combined January 2023 point forecast was approximately 1.51 million subscriptions. This is a forecast made from the end of 2022, not a current forecast or a verified outcome.

Sharp changes in the source, particularly in Outlook Business and at some year boundaries, remain **unverified**. They are retained in the analysis. No observed cost saving, revenue increase or causal business effect is claimed.

## Data and scope

The supplied Excel workbook contains 342 period records across five titles for 2020–2022:

| Title | Source frequency | Records |
| --- | --- | ---: |
| Outlook Business | Monthly | 36 |
| Outlook Traveller | Monthly | 36 |
| Outlook Money | Monthly | 36 |
| Outlook Hindi | Fortnightly issues | 78 |
| Outlook India | Weekly issues | 156 |

`Opening` is the active subscription balance at the start of a month or issue. `Expiry` counts subscriptions reaching expiry during that period; it does not establish whether the subscriptions were renewed or cancelled. The Hindi 2020 issue 25 opening value was confirmed as 146,222 and corrected in the cleaned data.

Issue dates were not provided for the weekly or fortnightly titles. Their records are retained at issue frequency, but they are not included in monthly aggregates or the 2023 monthly forecasts. The combined monthly series is therefore **not a complete group total**. A balance is a point-in-time stock; it should not be summed across months.

## Methods and validation

The notebook reshapes the workbook, calculates descriptive statistics and growth, explores monthly indices and additive seasonal decomposition, and performs ADF tests. It compares naive, seasonal naive, three-month moving average, simple exponential smoothing, Holt and Holt–Winters forecasts. Models are evaluated on the same six-month fixed-origin holdout, and the lowest RMSE is selected for each series. The forecast is then refitted on all 36 monthly observations.

Prediction intervals use model-based simulation for selected ETS models and residual resampling for selected moving-average models. They are conditional on their assumptions, have not demonstrated 95% empirical coverage, and do not fully capture model-selection uncertainty, source errors or future changes. SARIMA is omitted because the training sample is short and contains unexplained jumps. One short holdout is not sufficient evidence of future accuracy.

## Project files

| File | Purpose |
| --- | --- |
| [Source workbook](Data%20for%20sales%20forecasting%20and%20trend%20line%20analysis%20(1).xlsx) | Original five-sheet dataset. |
| [Python notebook](Part2_Subscription_Analysis.ipynb) | Reproducible cleaning, exploratory analysis, model comparison and forecasts. |
| [Analysis outputs](part2_results/) | Cleaned data, statistics, growth, model comparison, 2023 forecasts and a monthly chart. |
| [Excel trendlines](outputs/part3/Subscription_trendlines.xlsx) | Formula-based fitted curves, holdout comparison and Excel forecasting functions. |
| [Power BI report](Outlook_Subscriptions.pbix) | Three report pages with subscription measures, filters, trend, forecast, heatmap and growth comparison. |
| [Power BI guide](Power_BI_build_guide.md) | Step-by-step instructions for rebuilding and explaining the model. |
| [PDF dashboard export](Outlook_Subscriptions.pdf) | Static report views; the overview page was enlarged for readability. |
| [Dashboard page images](dashboard_images/) | PNG images rendered from the PDF export. |
| [Client-style deck](outputs/part5/Outlook_Subscription_Client_Deck.pptx) | Six editable PowerPoint slides. |
| [Consulting summary and resume bullets](outputs/Consulting_Summary_and_Resume.md) | Key results, recommendations, limitations and suggested bullets. |
| [Interview preparation](outputs/Interview_Preparation.md) | STAR explanations, technical questions and revision sheet. |

## Reproduce the analysis

Use Python 3.12 and run commands from the repository root:

```bash
python -m venv .venv
```

Activate the environment on Windows with `.venv\Scripts\activate`, or on macOS/Linux with `source .venv/bin/activate`. Then run:

```bash
python -m pip install -r requirements.txt
jupyter lab Part2_Subscription_Analysis.ipynb
```

Run the notebook cells in order. It reads the source workbook from the repository root and writes outputs to `part2_results/`, replacing files with the same names. Forecast simulation uses a fixed random seed, although package or platform differences can change numerical results slightly.

The `.pbix` was built from `part2_results/power_bi_monthly.csv`. It contains imported data, so the saved visuals can be viewed without refreshing. To refresh on another computer, open **Transform data → Data source settings** in Power BI Desktop and update the CSV path to the local checkout. The guide also provides a separate Power Query route from the original Excel workbook; that independently built model uses different table names from the saved report.

## Limitations

The three-year monthly history, single holdout and unverified source values constrain the analysis. Data on subscription acquisitions, renewals, cancellations, prices, copies sold, print returns and external drivers were not supplied. The forecast is not evidence of a print-run, financial or customer outcome. Longer verified histories, issue dates and repeated backtests would improve the basis for business decisions.
