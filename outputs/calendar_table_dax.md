# Generated DAX — Calendar / Date Dimension Table
**Dataset:** Northwind Traders | **Date range:** 2013-07-04 to 2015-05-06
**Fiscal year:** July start (FY2014 = Jul 2013 – Jun 2014, matching this dataset)

In Power BI Desktop: **Modeling tab → New Table** → paste code below.
Then: **Table Tools → Mark as Date Table → select Date column → OK**

---

## Full Calendar Table DAX

```dax
DimDate =
-- ─────────────────────────────────────────────────────────
-- Northwind Traders date range: 2013-07-04 to 2015-05-06
-- Extended to full years for clean YTD/YoY calculations
-- ─────────────────────────────────────────────────────────
VAR _startDate = DATE( 2013, 1, 1 )
VAR _endDate   = DATE( 2015, 12, 31 )

RETURN
ADDCOLUMNS(
    CALENDAR( _startDate, _endDate ),

    -- ─────────────────────────────────────────────────────
    -- Core date parts
    -- ─────────────────────────────────────────────────────
    "Year",
        YEAR( [Date] ),

    "Quarter",
        "Q" & QUARTER( [Date] ),

    "QuarterNumber",
        QUARTER( [Date] ),

    "MonthNumber",
        MONTH( [Date] ),

    "MonthName",
        FORMAT( [Date], "MMMM" ),

    "MonthShort",
        FORMAT( [Date], "MMM" ),

    "MonthYearLabel",
        FORMAT( [Date], "MMM YYYY" ),

    "WeekNumber",
        WEEKNUM( [Date], 2 ),

    "DayOfMonth",
        DAY( [Date] ),

    "DayOfWeekNumber",
        WEEKDAY( [Date], 2 ),

    "DayName",
        FORMAT( [Date], "dddd" ),

    "DayShort",
        FORMAT( [Date], "ddd" ),

    -- ─────────────────────────────────────────────────────
    -- Flags
    -- ─────────────────────────────────────────────────────
    "IsWeekend",
        IF( WEEKDAY( [Date], 2 ) >= 6, TRUE, FALSE ),

    "IsWorkday",
        IF( WEEKDAY( [Date], 2 ) < 6, TRUE, FALSE ),

    -- ─────────────────────────────────────────────────────
    -- Sort columns
    -- In Power BI: select MonthName → Column Tools → Sort By Column → MonthNumber
    -- Apply same for MonthYearLabel (sort by MonthSort), Quarter (sort by QuarterNumber)
    -- ─────────────────────────────────────────────────────
    "MonthSort",
        YEAR( [Date] ) * 100 + MONTH( [Date] ),

    "QuarterSort",
        YEAR( [Date] ) * 10 + QUARTER( [Date] ),

    "WeekSort",
        YEAR( [Date] ) * 100 + WEEKNUM( [Date], 2 ),

    -- ─────────────────────────────────────────────────────
    -- Fiscal Year — July start
    -- FY2014 = July 2013 to June 2014
    -- Matches the Northwind dataset which starts July 2013
    -- ─────────────────────────────────────────────────────
    "FiscalYear",
        IF(
            MONTH( [Date] ) >= 7,
            YEAR( [Date] ) + 1,
            YEAR( [Date] )
        ),

    "FiscalYearLabel",
        "FY" & IF(
            MONTH( [Date] ) >= 7,
            YEAR( [Date] ) + 1,
            YEAR( [Date] )
        ),

    "FiscalQuarter",
        "FQ" & SWITCH(
            TRUE(),
            MONTH( [Date] ) IN { 7, 8, 9 },    1,
            MONTH( [Date] ) IN { 10, 11, 12 }, 2,
            MONTH( [Date] ) IN { 1, 2, 3 },    3,
            MONTH( [Date] ) IN { 4, 5, 6 },    4
        ),

    "FiscalMonthNumber",
        MOD( MONTH( [Date] ) - 7 + 12, 12 ) + 1,

    -- ─────────────────────────────────────────────────────
    -- Numeric keys (useful for relationships or sorting)
    -- ─────────────────────────────────────────────────────
    "YearMonthKey",
        YEAR( [Date] ) * 100 + MONTH( [Date] ),

    "DateKey",
        YEAR( [Date] ) * 10000
            + MONTH( [Date] ) * 100
            + DAY( [Date] )
)
```

---

## After Creating the Table — Checklist

**Mark as Date Table**
- Select `DimDate` in Fields pane
- Table Tools → Mark as Date Table → select `Date` → OK

**Set Sort By Column** (so months appear in correct order in visuals)

| Column | Sort By |
|--------|---------|
| MonthName | MonthNumber |
| MonthShort | MonthNumber |
| MonthYearLabel | MonthSort |
| DayName | DayOfWeekNumber |
| Quarter | QuarterNumber |
| FiscalQuarter | FiscalMonthNumber |

**Create Relationship**
- Model view → drag `DimDate[Date]` → `orders[orderDate]`
- Cardinality: One (DimDate) to Many (orders)
- Filter direction: Single (DimDate → orders)

---

## Minimal Version

```dax
DimDate =
ADDCOLUMNS(
    CALENDAR( DATE(2013,1,1), DATE(2015,12,31) ),
    "Year",        YEAR( [Date] ),
    "MonthNumber", MONTH( [Date] ),
    "MonthName",   FORMAT( [Date], "MMMM" ),
    "Quarter",     "Q" & QUARTER( [Date] ),
    "IsWeekend",   IF( WEEKDAY([Date], 2) >= 6, TRUE, FALSE )
)
```
