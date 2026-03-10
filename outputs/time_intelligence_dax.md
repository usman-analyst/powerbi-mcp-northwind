# Generated DAX — Time Intelligence Measures
**Dataset:** Northwind Traders (Kaggle — jeetahirwar)
**Date range:** 2013-07-04 to 2015-05-06
**Requires:** DimDate calendar table connected to orders[orderDate]

---

## Month / Quarter / Year to Date

```dax
-- Revenue MTD
-- Revenue from the 1st of the current month to the current date.
Revenue MTD =
TOTALMTD( [Total Revenue], DimDate[Date] )
```

```dax
-- Revenue QTD
-- Revenue from the 1st of the current quarter to the current date.
Revenue QTD =
TOTALQTD( [Total Revenue], DimDate[Date] )
```

```dax
-- Revenue YTD
-- Revenue from 1st January of the current year to the current date.
Revenue YTD =
TOTALYTD( [Total Revenue], DimDate[Date] )
```

---

## Prior Period Comparisons

```dax
-- Revenue Previous Month
-- Total revenue for the equivalent period one month prior.
-- Uses DATEADD to shift the date context back by one month.
Revenue Previous Month =
CALCULATE(
    [Total Revenue],
    DATEADD( DimDate[Date], -1, MONTH )
)
```

```dax
-- Revenue Previous Quarter
-- Total revenue for the equivalent period one quarter prior.
Revenue Previous Quarter =
CALCULATE(
    [Total Revenue],
    DATEADD( DimDate[Date], -1, QUARTER )
)
```

```dax
-- Revenue Previous Year
-- Total revenue for the same period in the prior year.
-- Uses SAMEPERIODLASTYEAR for cleaner YoY comparisons.
Revenue Previous Year =
CALCULATE(
    [Total Revenue],
    SAMEPERIODLASTYEAR( DimDate[Date] )
)
```

---

## Variance Measures

```dax
-- Revenue vs Previous Month
-- Absolute difference in revenue vs the prior month.
-- Positive = growth, Negative = decline.
Revenue vs Previous Month =
[Total Revenue] - [Revenue Previous Month]
```

```dax
-- Revenue vs Previous Year
-- Absolute revenue difference vs same period last year.
Revenue vs Previous Year =
[Total Revenue] - [Revenue Previous Year]
```

```dax
-- MoM Growth %
-- Month-over-month revenue growth as a percentage.
-- Returns BLANK (not zero) when prior month has no data,
-- to avoid misleading 100% growth at the start of the dataset.
MoM Growth % =
DIVIDE(
    [Revenue vs Previous Month],
    [Revenue Previous Month],
    BLANK()
)
```

```dax
-- YoY Growth %
-- Year-over-year revenue growth as a percentage.
-- Returns BLANK when prior year has no data.
YoY Growth % =
DIVIDE(
    [Revenue vs Previous Year],
    [Revenue Previous Year],
    BLANK()
)
```

---

## Rolling Windows

```dax
-- Rolling 3 Month Revenue
-- Total revenue for the last 3 months up to the current date.
-- Useful for smoothing seasonal spikes in the Northwind data.
Rolling 3 Month Revenue =
CALCULATE(
    [Total Revenue],
    DATESINPERIOD(
        DimDate[Date],
        LASTDATE( DimDate[Date] ),
        -3,
        MONTH
    )
)
```

```dax
-- Rolling 12 Month Revenue
-- Total revenue for the last 12 months (rolling annual view).
Rolling 12 Month Revenue =
CALCULATE(
    [Total Revenue],
    DATESINPERIOD(
        DimDate[Date],
        LASTDATE( DimDate[Date] ),
        -12,
        MONTH
    )
)
```

```dax
-- 3 Month Moving Average
-- Average monthly revenue over a rolling 3-month window.
-- Divide rolling total by 3 to get the monthly average.
3 Month Moving Average =
DIVIDE(
    [Rolling 3 Month Revenue],
    3,
    BLANK()
)
```

---

## Orders Time Intelligence

```dax
-- Orders MTD
Orders MTD =
TOTALMTD( [Total Orders], DimDate[Date] )
```

```dax
-- Orders YTD
Orders YTD =
TOTALYTD( [Total Orders], DimDate[Date] )
```

```dax
-- Orders Previous Year
Orders Previous Year =
CALCULATE(
    [Total Orders],
    SAMEPERIODLASTYEAR( DimDate[Date] )
)
```

```dax
-- Orders YoY Growth %
Orders YoY Growth % =
DIVIDE(
    [Total Orders] - [Orders Previous Year],
    [Orders Previous Year],
    BLANK()
)
```
