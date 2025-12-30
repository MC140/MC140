- 👋 Hi, I’m Manohar Chekka
- 👀 I’m Data Engineer
- 📫 Reach me at : manoharch0698@gmail.com


<!---
MC140/MC140 is a ✨ special ✨ repository because its `README.md` (this file) appears on your GitHub profile.
You can click the Preview link to take a look at your changes.
--->

DateTable = 
VAR FiscalStartMonth = 7 -- Edit this to change your Fiscal Year start (e.g., 7 for July)
VAR StartDate = MIN('Sales'[OrderDate]) -- Or use DATE(2020, 1, 1) for a fixed start
VAR EndDate = MAX('Sales'[OrderDate])   -- Or use DATE(2025, 12, 31)

RETURN
ADDCOLUMNS (
    CALENDAR ( StartDate, EndDate ),
    "Year", YEAR ( [Date] ),
    "Quarter", "Q" & FORMAT ( [Date], "Q" ),
    "Quarter Number", QUARTER ( [Date] ),
    "Month Name", FORMAT ( [Date], "MMMM" ),
    "Month Short", FORMAT ( [Date], "MMM" ),
    "Month Number", MONTH ( [Date] ),
    "Week Number", WEEKNUM ( [Date] ),
    "Day of Week", FORMAT ( [Date], "dddd" ),
    "Day of Week Short", FORMAT ( [Date], "ddd" ),
    "Day of Week Number", WEEKDAY ( [Date], 2 ), -- 1 = Monday
    "Year-Month", FORMAT ( [Date], "YYYY-MM" ),
    "Month-Year Sort", YEAR ( [Date] ) * 100 + MONTH ( [Date] ),

    -- FISCAL COLUMNS
    "Fiscal Year", 
        VAR FY = IF ( MONTH ( [Date] ) >= FiscalStartMonth, YEAR ( [Date] ) + 1, YEAR ( [Date] ) )
        RETURN "FY" & RIGHT(FY, 2),
    
    "Fiscal Quarter", 
        VAR FQ = CEILING ( MOD ( MONTH ( [Date] ) - FiscalStartMonth + 12, 12 ) + 1, 3 ) / 3
        RETURN "FQ" & FQ,
    
    "Fiscal Month Number", 
        MOD ( MONTH ( [Date] ) - FiscalStartMonth + 12, 12 ) + 1,

    -- PERIOD FLAGS (Ultimate for filtering)
    "Is Current Day", IF ( [Date] = TODAY(), "Today", "Other" ),
    "Is Past", IF ( [Date] < TODAY(), 1, 0 ),
    "Is Working Day", IF ( WEEKDAY([Date], 2) < 6, 1, 0 )
)



Calendar =
VAR FiscalStartMonth = 4   -- 1=Jan, 4=Apr, 11=Nov, etc. Change as needed.
VAR StartDate =
    DATE ( YEAR ( MINX ( ALL ( 'Fact' ), 'Fact'[Date] ) ), 1, 1 )
VAR EndDate =
    DATE ( YEAR ( MAXX ( ALL ( 'Fact' ), 'Fact'[Date] ) ), 12, 31 )
VAR TodayDate = TODAY ()
RETURN
ADDCOLUMNS (
    CALENDAR ( StartDate, EndDate ),

    /* ===== Core date parts ===== */
    "DateKey", VALUE ( FORMAT ( [Date], "yyyymmdd" ) ),
    "Year", YEAR ( [Date] ),
    "YearStart", DATE ( YEAR ( [Date] ), 1, 1 ),
    "YearEnd", DATE ( YEAR ( [Date] ), 12, 31 ),

    "QuarterNo", QUARTER ( [Date] ),
    "Quarter", "Q" & QUARTER ( [Date] ),
    "YearQuarter", YEAR ( [Date] ) & " Q" & QUARTER ( [Date] ),
    "QuarterStart", DATE ( YEAR ( [Date] ), ( QUARTER ( [Date] ) - 1 ) * 3 + 1, 1 ),
    "QuarterEnd", EOMONTH ( DATE ( YEAR ( [Date] ), ( QUARTER ( [Date] ) - 1 ) * 3 + 1, 1 ), 2 ),

    "MonthNo", MONTH ( [Date] ),
    "MonthName", FORMAT ( [Date], "MMMM" ),
    "MonthShort", FORMAT ( [Date], "MMM" ),
    "YearMonth", FORMAT ( [Date], "YYYY-MMM" ),
    "YearMonthKey", YEAR ( [Date] ) * 100 + MONTH ( [Date] ),
    "MonthStart", DATE ( YEAR ( [Date] ), MONTH ( [Date] ), 1 ),
    "MonthEnd", EOMONTH ( [Date], 0 ),

    /* Week (ISO-style with Monday start) */
    "WeekStart", [Date] - WEEKDAY ( [Date], 2 ) + 1,              -- Monday
    "WeekEnd",   [Date] - WEEKDAY ( [Date], 2 ) + 7,
    "ISOWeekNo", WEEKNUM ( [Date], 21 ),                           -- ISO week number
    "ISOWeekYear",
        VAR d = [Date]
        VAR thursday = d - WEEKDAY ( d, 2 ) + 4
        RETURN YEAR ( thursday ),
    "ISOYearWeek",
        VAR d = [Date]
        VAR thursday = d - WEEKDAY ( d, 2 ) + 4
        RETURN YEAR ( thursday ) & "-W" & FORMAT ( WEEKNUM ( d, 21 ), "00" ),

    /* Day */
    "DayOfMonth", DAY ( [Date] ),
    "DayOfWeekNo", WEEKDAY ( [Date], 2 ),                          -- 1=Mon..7=Sun
    "DayName", FORMAT ( [Date], "dddd" ),
    "DayShort", FORMAT ( [Date], "ddd" ),
    "IsWeekend", IF ( WEEKDAY ( [Date], 2 ) >= 6, TRUE (), FALSE () ),

    /* ===== Relative / usability flags ===== */
    "IsToday", [Date] = TodayDate,
    "IsCurrentYear", YEAR ( [Date] ) = YEAR ( TodayDate ),
    "IsCurrentMonth",
        YEAR ( [Date] ) = YEAR ( TodayDate ) && MONTH ( [Date] ) = MONTH ( TodayDate ),
    "IsCurrentWeek",
        VAR ws = [Date] - WEEKDAY ( [Date], 2 ) + 1
        VAR tws = TodayDate - WEEKDAY ( TodayDate, 2 ) + 1
        RETURN ws = tws,
    "DayOffset", DATEDIFF ( [Date], TodayDate, DAY ),
    "MonthOffset",
        ( YEAR ( [Date] ) * 12 + MONTH ( [Date] ) ) - ( YEAR ( TodayDate ) * 12 + MONTH ( TodayDate ) ),
    "YearOffset", YEAR ( [Date] ) - YEAR ( TodayDate ),

    /* ===== Fiscal (configurable start month) ===== */
    "FiscalYear",
        VAR y = YEAR ( [Date] )
        VAR m = MONTH ( [Date] )
        RETURN IF ( m >= FiscalStartMonth, y + 1, y ),             -- fiscal year label (ending year)
    "FiscalYearLabel",
        "FY" &
        RIGHT (
            FORMAT (
                VAR y = YEAR ( [Date] )
                VAR m = MONTH ( [Date] )
                RETURN IF ( m >= FiscalStartMonth, y + 1, y ),
                "0000"
            ),
            2
        ),

    "FiscalMonthNo",
        VAR m = MONTH ( [Date] )
        RETURN MOD ( m - FiscalStartMonth + 12, 12 ) + 1,          -- 1..12 inside fiscal year

    "FiscalMonth",
        VAR fm =
            MOD ( MONTH ( [Date] ) - FiscalStartMonth + 12, 12 ) + 1
        RETURN "FM" & FORMAT ( fm, "00" ),

    "FiscalYearMonthKey",
        VAR fy =
            IF ( MONTH ( [Date] ) >= FiscalStartMonth, YEAR ( [Date] ) + 1, YEAR ( [Date] ) )
        VAR fm =
            MOD ( MONTH ( [Date] ) - FiscalStartMonth + 12, 12 ) + 1
        RETURN fy * 100 + fm,

    "FiscalQuarterNo",
        VAR fm =
            MOD ( MONTH ( [Date] ) - FiscalStartMonth + 12, 12 ) + 1
        RETURN INT ( ( fm - 1 ) / 3 ) + 1,

    "FiscalQuarter",
        VAR fm =
            MOD ( MONTH ( [Date] ) - FiscalStartMonth + 12, 12 ) + 1
        VAR fq = INT ( ( fm - 1 ) / 3 ) + 1
        RETURN "FQ" & fq,

    "FiscalYearQuarter",
        VAR fy =
            IF ( MONTH ( [Date] ) >= FiscalStartMonth, YEAR ( [Date] ) + 1, YEAR ( [Date] ) )
        VAR fm =
            MOD ( MONTH ( [Date] ) - FiscalStartMonth + 12, 12 ) + 1
        VAR fq = INT ( ( fm - 1 ) / 3 ) + 1
        RETURN "FY" & RIGHT ( FORMAT ( fy, "0000" ), 2 ) & " FQ" & fq,

    "FiscalYearStart",
        VAR fy =
            IF ( MONTH ( [Date] ) >= FiscalStartMonth, YEAR ( [Date] ) + 1, YEAR ( [Date] ) )
        RETURN DATE ( fy - 1, FiscalStartMonth, 1 ),

    "FiscalYearEnd",
        VAR fy =
            IF ( MONTH ( [Date] ) >= FiscalStartMonth, YEAR ( [Date] ) + 1, YEAR ( [Date] ) )
        RETURN EOMONTH ( DATE ( fy, FiscalStartMonth, 1 ), -1 ),

    /* Fiscal Week (Monday-based, aligned to fiscal year start) */
    "FiscalWeekStart",
        VAR fys =
            VAR fy =
                IF ( MONTH ( [Date] ) >= FiscalStartMonth, YEAR ( [Date] ) + 1, YEAR ( [Date] ) )
            RETURN DATE ( fy - 1, FiscalStartMonth, 1 )
        VAR firstWeekStart = fys - WEEKDAY ( fys, 2 ) + 1
        VAR thisWeekStart = [Date] - WEEKDAY ( [Date], 2 ) + 1
        RETURN thisWeekStart,

    "FiscalWeekNo",
        VAR fys =
            VAR fy =
                IF ( MONTH ( [Date] ) >= FiscalStartMonth, YEAR ( [Date] ) + 1, YEAR ( [Date] ) )
            RETURN DATE ( fy - 1, FiscalStartMonth, 1 )
        VAR firstWeekStart = fys - WEEKDAY ( fys, 2 ) + 1
        VAR thisWeekStart = [Date] - WEEKDAY ( [Date], 2 ) + 1
        RETURN INT ( DIVIDE ( DATEDIFF ( firstWeekStart, thisWeekStart, DAY ), 7 ) ) + 1
)