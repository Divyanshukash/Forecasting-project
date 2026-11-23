# Power BI dashboard: build guide

This guide builds a three-page report from the three monthly sheets in the original workbook. It measures active subscription balances and expiries, not sales. Outlook Hindi and Outlook India are not included in monthly charts because issue dates are not available. Retain unusual values until the source owner verifies them.

## 1. Import and clean with Power Query

1. Open Power BI Desktop and choose **Blank report**. Select **Home > Transform data > Transform data** to open Power Query.
2. Select **Home > New Source > Blank Query**. In Query Settings, rename it `Monthly`. Select **Home > Advanced Editor**, replace the existing code with the M script below and choose **Done**.
3. If asked, grant access only to the local Excel file. Replace the example path on the `File.Contents` line with the location of the source workbook on your computer. This guide describes a fresh model with `Monthly` and `Magazine` tables; the supplied `.pbix` instead imports `part2_results/power_bi_monthly.csv` as `power_bi_monthly` and uses the title column in that table.

```powerquery
let
    Source = Excel.Workbook(File.Contents("C:\path\to\Data for sales forecasting and trend line analysis (1).xlsx"), null, true),
    TitleSheets = {
        [Sheet="Outlook Business.", Title="Outlook Business"],
        [Sheet="Outlook Traveller", Title="Outlook Traveller"],
        [Sheet="Outlook Money", Title="Outlook Money"]
    },
    Years = {[Year=2020, Offset=0], [Year=2021, Offset=4], [Year=2022, Offset=8]},
    ReadBlock = (sheet as text, title as text, year as number, offset as number) as table =>
        let
            Raw = Source{[Item=sheet, Kind="Sheet"]}[Data],
            ColumnNames = Table.ColumnNames(Raw),
            ThreeColumns = Table.SelectColumns(Raw, List.Range(ColumnNames, offset, 3)),
            Renamed = Table.RenameColumns(ThreeColumns, List.Zip({List.Range(ColumnNames, offset, 3), {"Month", "Opening", "Expiry"}})),
            DataRows = Table.Skip(Renamed, 2),
            Nonblank = Table.SelectRows(DataRows, each [Month] <> null),
            Typed = Table.TransformColumns(Nonblank, {
                {"Month", each Int64.From(_), Int64.Type},
                {"Opening", each Number.FromText(Text.Replace(Text.From(_), ",", ""), "en-US"), Int64.Type},
                {"Expiry", each Int64.From(_), Int64.Type}
            }),
            WithDate = Table.AddColumn(Typed, "Date", each #date(year, [Month], 1), type date),
            WithTitle = Table.AddColumn(WithDate, "Title", each title, type text),
            Output = Table.SelectColumns(WithTitle, {"Date", "Title", "Opening", "Expiry"})
        in
            Output,
    Blocks = List.Combine(List.Transform(TitleSheets, (t) => List.Transform(Years, (y) => ReadBlock(t[Sheet], t[Title], y[Year], y[Offset])))),
    Combined = Table.Combine(Blocks),
    Sorted = Table.Sort(Combined, {{"Date", Order.Ascending}, {"Title", Order.Ascending}})
in
    Sorted
```

**What each step does:** `Source` opens the workbook. `TitleSheets` identifies the three monthly sheets; `Years` identifies their three side-by-side blocks. `ReadBlock` selects one three-column block, labels its columns, removes the two header rows, checks for blank periods, converts numbers—including comma-separated text—and makes a first-of-month date. `Blocks` creates all nine title/year combinations; `Table.Combine` appends them. Because the source uses year blocks rather than title columns in a single table, appending the blocks is the appropriate reshape here; a conventional Unpivot operation is not needed.

4. Check that the preview has `Date`, `Title`, `Opening`, `Expiry` and **108 rows**: three titles × 36 months. All dates should be the first of the month. Check there are no errors or blanks in those columns. `Opening` and `Expiry` should be Whole number; `Date` should be Date. In Power Query, **View > Column quality** can help identify errors.
5. Select **Home > Close & Apply**. In Data view, confirm the earliest month is January 2020 and latest is December 2022.

## 2. Date and magazine dimensions

1. Select **Modeling > New table** and enter:

```DAX
Date =
ADDCOLUMNS (
    CALENDAR ( DATE ( 2020, 1, 1 ), DATE ( 2022, 12, 31 ) ),
    "Year", YEAR ( [Date] ),
    "Month No", MONTH ( [Date] ),
    "Month", FORMAT ( [Date], "MMM" ),
    "Quarter", "Q" & FORMAT ( [Date], "Q" ),
    "Month Start", DATE ( YEAR ( [Date] ), MONTH ( [Date] ), 1 )
)
```

`CALENDAR` creates one row per day for complete years. `ADDCOLUMNS` adds labels and a month-start key. A continuous date table makes time comparisons consistent even though the facts are monthly. If the data is extended, extend the date table to include complete years.

2. Select the `Date` table, then **Table tools > Mark as date table**, and choose `Date[Date]`. Select `Date[Month]`, then **Column tools > Sort by column > Month No**.
3. Create another table with **Modeling > New table**:

```DAX
Magazine = DISTINCT ( Monthly[Title] )
```

4. Open **Model view > Manage relationships > New**. Create active, single-direction relationships `Date[Date]` (one) → `Monthly[Date]` (many), and `Magazine[Title]` (one) → `Monthly[Title]` (many). Filters should flow from the dimensions to `Monthly`.

This is a small star schema: the fact table holds numeric observations and the dimensions describe dates and titles. The date table must have unique, contiguous, nonblank dates for classic time intelligence.

## 3. Measures

Create each measure using **Modeling > New measure**. Use the measure names exactly as written. Set percentage measures to **Percentage** with one or two decimal places. Format counts with thousands separators and no decimals. Measures respond to report filters; raw subscription balances must not be summed across different months.

### Opening balance as of the latest selected observation

```DAX
Opening Balance =
VAR LatestObservation = MAX ( Monthly[Date] )
RETURN
    IF (
        NOT ISBLANK ( LatestObservation ),
        CALCULATE ( SUM ( Monthly[Opening] ), 'Date'[Date] = LatestObservation )
    )
```

`MAX` finds the latest actual month in the current date/title filter context. `CALCULATE` restricts the sum to that month. `SUM` then adds the titles at that one date. The `IF` leaves an empty context blank. With all dates selected, this is the December 2022 opening balance; it is **not** the sum of all 36 balances.

### Month-on-month change

```DAX
MoM % =
VAR CurrentDate = MAX ( Monthly[Date] )
VAR CurrentBalance = [Opening Balance]
VAR PreviousBalance =
    CALCULATE (
        [Opening Balance],
        REMOVEFILTERS ( 'Date' ),
        'Date'[Date] = EDATE ( CurrentDate, -1 )
    )
RETURN DIVIDE ( CurrentBalance - PreviousBalance, PreviousBalance )
```

`EDATE` finds the prior month. `REMOVEFILTERS` allows the calculation to reach it even across a year boundary. `DIVIDE` returns blank when the denominator is zero or absent.

### Year-on-year change using SAMEPERIODLASTYEAR

```DAX
YoY % =
VAR CurrentDate = MAX ( Monthly[Date] )
VAR CurrentBalance = [Opening Balance]
VAR PreviousBalance =
    CALCULATE (
        [Opening Balance],
        REMOVEFILTERS ( 'Date' ),
        SAMEPERIODLASTYEAR (
            DATESBETWEEN ( 'Date'[Date], CurrentDate, EOMONTH ( CurrentDate, 0 ) )
        )
    )
RETURN DIVIDE ( CurrentBalance - PreviousBalance, PreviousBalance )
```

`DATESBETWEEN` describes the entire current month; `SAMEPERIODLASTYEAR` shifts those dates back one year. `CALCULATE` evaluates the balance in that prior period. On a card this compares the latest selected month with the same month a year earlier. January–December 2020 has no prior-year observations, so its YoY values are blank.

### Three-month moving average

```DAX
Opening Balance 3M Average =
VAR CurrentDate = MAX ( Monthly[Date] )
VAR Months =
    CALCULATETABLE (
        VALUES ( 'Date'[Month Start] ),
        REMOVEFILTERS ( 'Date' ),
        DATESBETWEEN ( 'Date'[Date], EDATE ( CurrentDate, -2 ), EOMONTH ( CurrentDate, 0 ) )
    )
VAR AvailableMonths =
    FILTER (
        Months,
        VAR ThisMonth = 'Date'[Month Start]
        RETURN NOT ISBLANK (
            CALCULATE ( [Opening Balance], REMOVEFILTERS ( 'Date' ), 'Date'[Date] = ThisMonth )
        )
    )
RETURN
    IF (
        COUNTROWS ( AvailableMonths ) = 3,
        AVERAGEX (
            AvailableMonths,
            VAR ThisMonth = 'Date'[Month Start]
            RETURN CALCULATE ( [Opening Balance], REMOVEFILTERS ( 'Date' ), 'Date'[Date] = ThisMonth )
        )
    )
```

`DATESBETWEEN` identifies the current month and the two preceding months. `VALUES` gives each month once; `FILTER` removes months without an observation. `AVERAGEX` computes the average of three monthly balances, rather than summing a stock. The measure is blank until three months are available.

### Expiries and year-to-date expiries

```DAX
Expiries = SUM ( Monthly[Expiry] )
```

```DAX
YTD Expiries = TOTALYTD ( [Expiries], 'Date'[Date] )
```

Expiries are counts over periods, so they may be summed across months. `TOTALYTD` accumulates them from the start of the calendar year. Do not calculate a YTD sum of opening balances; a stock is measured at a point in time.

### Annualized change in opening balance

```DAX
Balance CAGR % =
VAR StartDt = CALCULATE ( MIN ( Monthly[Date] ), ALLSELECTED ( 'Date' ) )
VAR EndDt = CALCULATE ( MAX ( Monthly[Date] ), ALLSELECTED ( 'Date' ) )
VAR FirstBalance = CALCULATE ( [Opening Balance], REMOVEFILTERS ( 'Date' ), 'Date'[Date] = StartDt )
VAR LastBalance = CALCULATE ( [Opening Balance], REMOVEFILTERS ( 'Date' ), 'Date'[Date] = EndDt )
VAR Months = DATEDIFF ( StartDt, EndDt, MONTH )
RETURN IF ( Months > 0 && FirstBalance > 0 && LastBalance > 0, POWER ( DIVIDE ( LastBalance, FirstBalance ), 12 / Months ) - 1 )
```

`ALLSELECTED` keeps the date slicer selection while finding its first and last available months. The two `CALCULATE` expressions retrieve endpoint balances. `DATEDIFF` measures elapsed months; raising the ratio to `12 / Months` annualizes it. The result is blank if an endpoint is missing, nonpositive, or there is no elapsed time. This describes endpoints, not smooth monthly growth.

### Share and best title

```DAX
Share of Selected Titles % =
DIVIDE (
    [Opening Balance],
    CALCULATE ( [Opening Balance], ALLSELECTED ( Magazine[Title] ) )
)
```

The numerator is the current title's balance. `ALLSELECTED` removes the chart row's title filter while retaining the user's title slicer selection. The denominator is therefore the balance across selected titles at the same date.

```DAX
Best Title =
VAR Ranked =
    TOPN ( 1, ALLSELECTED ( Magazine[Title] ), [Opening Balance], DESC, Magazine[Title], ASC )
RETURN CONCATENATEX ( Ranked, Magazine[Title], ", " )
```

`TOPN` selects the title with the largest latest selected opening balance; title name breaks ties alphabetically. `CONCATENATEX` returns a label for the card. This is largest balance, not highest sales or fastest growth.

```DAX
Growth Direction =
IF ( ISBLANK ( [YoY %] ), "No prior year", IF ( [YoY %] > 0, "Growing", IF ( [YoY %] < 0, "Declining", "Flat" ) ) )
```

This classifies the change in the latest selected balance versus the same month of the previous year, not a claim about the long-term trend.

## 4. Build Page 1: Executive overview

1. Double-click the first page tab and rename it `Executive overview`.
2. Add four **Card** visuals. Drag one measure into each: `[Opening Balance]`, `[YoY %]`, `[Balance CAGR %]`, `[Best Title]`. Give each a precise title, e.g. `Opening subscriptions · latest selected month`.
3. Add a **Line chart**. Put `Date[Date]` on X-axis, choose the date column rather than Date hierarchy, put `[Opening Balance]` on Y-axis, and put `Magazine[Title]` in Legend. Set X-axis to Continuous and format it to show months.
4. Add a **Clustered bar chart**: `Magazine[Title]` on Y-axis, `[Opening Balance]` on X-axis. This compares titles at the latest month in the current filter context. Sort descending.
5. Add two **Slicer** visuals: `Date[Year]` and `Magazine[Title]`. Clear filters to see all monthly titles. Keep the monthly-title scope in a subtitle.
6. Optionally add a table with title, `[Opening Balance]`, `[Share of Selected Titles %]` and `[YTD Expiries]`.

## 5. Build Page 2: Trend & forecast

1. Add a page with the **+** tab and rename it `Trend & forecast`.
2. Add a **Line chart**, using `Date[Date]` (not the hierarchy) on X-axis and `[Opening Balance]` on Y-axis. Under **Format visual > X-axis**, set Type to **Continuous**. Leave Legend empty; forecasting requires a single series. Add a magazine slicer if you want to select one title, but do not split the series by legend.
3. Select the chart, open **Analytics** (magnifying-glass icon), expand **Forecast**, and select **Add** or turn it on. Set forecast length to **12 months**, confidence interval to **95%**, and initially choose seasonality **12** for a monthly annual cycle. Apply. A setting of Auto is an alternative to compare, not proof that a stable annual pattern exists.
4. With the chart selected, look for **Trend line** in Analytics and add it if your version makes it available for this visual. If it is unavailable with the forecast, duplicate the line chart, remove the forecast from the duplicate, and add its trend line separately. Do not infer that a missing menu means a model was fitted.
5. Add a text box: `Forecast begins after Dec 2022. Source history is short and contains unverified jumps.`

Power BI's built-in forecast uses exponential smoothing methods. Seasonality sets a cycle length; a 95% interval is model-based uncertainty, not a guarantee. The Excel and Python implementations and their model choices can differ from Power BI's. Check the history, units, date selection and holdout accuracy before comparing values. The built-in forecast is an on-visual estimate; it does not itself establish holdout accuracy.

## 6. Build Page 3: Seasonality & growth

1. Add a page named `Seasonality & growth`.
2. Insert a **Matrix**. Put `Date[Month]` in Rows, `Date[Year]` in Columns, and `[Opening Balance]` in Values. Turn on **Cell elements > Background color > fx** and choose a color scale. Check the month labels are sorted Jan–Dec. Each cell is a balance for that month, not a sum across months.
3. Add a **Clustered bar chart** with `Magazine[Title]` and `[YoY %]`; it compares the latest selected month to the same month a year earlier. Add a table containing `Magazine[Title]`, `[Opening Balance]`, `[YoY %]` and `[Growth Direction]`.
4. Add year and title slicers. Select 2022 to compare December 2022 against December 2021 on the title-level cards and bars; month-level matrix cells still show each individual month's balance. Sync slicers across pages only if that behaviour is desired.

## 7. Design and checks

Use a small palette, white or light background, consistent typography, clear units and the same title colours on every page. Write insight titles only after checking the visual; do not claim a festival peak as fact. Include the scope and last observation date on each page.

With all dates and titles selected, check:

- Latest combined opening balance: **1,885,981** in December 2022.
- December 2022 combined YoY: approximately **82.85%** versus December 2021. The result is driven by an unverified Business change.
- Full-period annualized endpoint change: approximately **8.05%**, over 35 months.
- December 2022 title balances: Business **978,523**, Traveller **286,258**, Money **621,200**. Shares should sum to 100% when all three are selected.
- January 2020 YoY is blank; the three-month average is blank until three monthly observations exist.
- Test a single year, a single title, and a month in a matrix. Confirm totals show the latest balance in context, not a sum of monthly balances.

## 8. Save, export and capture

1. Select **File > Save As**, choose the project folder and save a `.pbix` file. Reopen it and confirm that the four cards, slicers, model relationships and forecast still work.
2. Select **File > Export > Export to PDF**. Review all three PDF pages for clipped visuals, unreadable titles and the correct filter state.
3. For portfolio screenshots, open each page at a readable zoom and use Windows Snipping Tool, saving one image per page. Review the image and remove any private information before publishing. A screenshot is a static view; it does not demonstrate slicer interaction.

## Limits and interview explanation

Only three years of monthly data exist, seasonal estimates are fragile, and there are unverified jumps. This report does not measure revenue, new sales, copies sold or renewals, and it does not include a complete group total. The forecast begins in 2023 because the source ends in 2022. An interview-safe description is: “I built a subscription-balance dashboard with a date dimension and measures that avoid summing balances across months. I compared year-on-year balances, expiries and a forecast, and flagged the short history and unverified jumps.”
