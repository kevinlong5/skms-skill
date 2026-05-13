# Data Construction Patterns

## Table of Contents

1. [Default SOP Matrix by Workflow Type](#1-default-sop-matrix-by-workflow-type)
2. [Raw Import SOP (CSMAR/Wind)](#2-raw-import-sop-csmarwind)
3. [Frame Workflow](#3-frame-workflow)
4. [Data Loading](#4-data-loading)
5. [Date Processing](#5-date-processing)
6. [Grouping and Aggregation](#6-grouping-and-aggregation)
7. [Data Merging](#7-data-merging)

---

## 1. Default SOP Matrix by Workflow Type

| Task Family | Default Approach | Key Commands |
|-------------|------------------|--------------|
| **Raw vendor import** | Header cleanup, label extraction, Stkcd normalization, date conversion | `import excel`, `foreach` label loop, `destring`, `date()` |
| **Same-schema consolidation** | Append sequential files into one master file | `append using`, `duplicates drop` |
| **Multi-source integration** | Frame workflow preferred over merge chains | `frame create`, `frlink`, `frget` |
| **Panel alignment** | xtset after destringing identifiers | `destring Stkcd`, `xtset stkcd year` |
| **Indicator construction** | Event window logic, abnormal returns, group aggregation | `egen`, `bys`, rolling calculations |

### Decision Rules

**Use `append` when:**
- Multiple files have identical column structure
- Same data source split by time period or batch
- Need to consolidate before analysis

**Use `merge` when:**
- Two datasets with different variables, same keys
- One-time join (not part of multi-source integration)
- Small auxiliary data (< 1000 obs)

**Use `frame` when:**
- Three or more distinct data sources
- Need to preserve source integrity
- Repeated linking to different subsets
- Memory management matters

---

## 2. Raw Import SOP (CSMAR/Wind)

### Standard Import Template

```stata
***basic setting***
drop _all
unicode encoding set "GB18030"
set more off

***import single file***
cd "<DATA_ROOT>/CSMAR/[path]"
drop _all
import excel using Filename, firstrow

***extract labels from first row***
foreach x of varlist Var1 Var2 Var3 {
    local vlabel = `x'[1]
    label var `x' "`vlabel'"
}
drop in 1/2

***normalize identifiers***
rename Symbol Stkcd     // or: rename stkcd Stkcd
rename Scode Stkcd      // CNRDS variant
destring Stkcd, replace

***date handling***
gen date = date(DateVar, "YMD")
gen year = year(date)
gen month = month(date)
format date %tdCCYY-NN-DD

***final cleanup***
keep Stkcd year [other_vars]
duplicates drop
save processed_data, replace
```

### Batch Import Pattern

```stata
***process multiple files***
forvalues i = 1/6 {
    cd "<DATA_ROOT>/CSMAR/[path]/`i'"
    drop _all
    import excel using File`i', firstrow
    foreach x of varlist [vars] {
        local vlabel = `x'[1]
        label var `x' "`vlabel'"
    }
    drop in 1/2
    save temp`i', replace
}

***consolidate***
use temp1, clear
forvalues i = 2/6 {
    append using temp`i'
}
duplicates drop
save consolidated, replace
```

### Identifier Normalization Rules

| Source Field | Target | Command |
|--------------|--------|---------|
| `Symbol` | `Stkcd` | `rename Symbol Stkcd` |
| `stkcd` (lowercase) | `Stkcd` | `rename stkcd Stkcd` |
| `Scode` (CNRDS) | `Stkcd` | `rename Scode Stkcd` |
| Numeric string | Numeric | `destring Stkcd, replace` |
| 6-digit numeric | String | `tostring Stkcd, gen(Stkcd_str) format(%06.0f)` |

### Date Handling Defaults

| Task | Default Pattern |
|------|-----------------|
| CSMAR import | `gen date = date(Accper/Trddt, "YMD")` |
| Year extraction | `gen year = year(date)` after conversion |
| Fiscal year | `gen fyear = year` then adjust if needed |
| Panel alignment | Ensure `year` exists and is integer |

---

## 3. Frame Workflow

**Purpose**: Organize multiple data sources without memory pollution. Each frame is an independent dataset in memory.

### Create and Use Frames

```stata
// Create a frame for each data source
frame create BasicInfo
frame create Financial
frame create Returns

// Process data within each frame
frame BasicInfo {
    cd "<DATA_ROOT>/CSMAR/公司基本信息/上市公司基本信息年度表"
    use STKBasic_year, clear
    keep Stkcd year IndustryCode PROVINCE
    duplicates drop
}

frame Financial {
    cd "<DATA_ROOT>/CSMAR/财务报表/最新报表"
    use Acc, clear
    keep Stkcd year TA SALE DEBT NI
    destring TA SALE DEBT NI, replace
}

frame Returns {
    cd "<DATA_ROOT>/CSMAR/股票收益率/"
    use stk_month, clear
    gen year = substr(YM, 1, 4)
    destring year, replace
    bys Stkcd year: egen annual_ret = mean(rstk)
}
```

### Link Frames

```stata
// Switch to main frame (default)
frame change default

// Create links between frames
destring Stkcd, gen(stkcd)
frlink m:1 Stkcd year, frame(BasicInfo)
frlink m:1 Stkcd year, frame(Financial)
frlink m:1 stkcd year, frame(Returns)

// Get variables from linked frames
frget IndustryCode PROVINCE, from(BasicInfo)
frget TA SALE DEBT NI, from(Financial)
frget annual_ret, from(Returns)
```

### Frame Management

```stata
// List all frames
frame dir

// Change to a frame
frame change FrameName

// Drop a frame
frame drop FrameName

// Reset all frames
clear all
```

**Pattern note**: Derived from internal Stata pattern examples; this public sharing version omits project-specific .do files.

---

## 4. Data Loading

### Load .dta Files

```stata
cd "<DATA_ROOT>/path/to/data"
use filename, clear
```

### Import Excel Files (CSMAR Format)

```stata
cd "<DATA_ROOT>/path/to/data"
drop _all
import excel using filename.xlsx, firstrow

// Handle first row as labels (common in CSMAR exports)
foreach x of varlist Symbol ShortName IndustryCode {
    local vlabel = `x'[1]
    label var `x' "`vlabel'"
}
drop in 1/2
```

### Batch Import Multiple Files

```stata
// Method 1: fs command
fs *.xlsx
foreach file in `r(files)' {
    import excel using `file', firstrow
    // process file
    save `file'.dta, replace
}

// Method 2: forvalues loop (numbered directories)
forvalues i = 1/13 {
    cd "/path/to/data/`i'"
    fs *.xls
    foreach file in `r(files)' {
        import excel using `file', firstrow
        // process
    }
}
```

### Append Multiple Files

```stata
// Load first file
use file1, clear

// Append remaining files
append using file2 file3 file4

// Or append in loop
fs *.dta
foreach file in `r(files)' {
    append using `file'
}
duplicates drop
```

**Pattern note**: Derived from internal Stata pattern examples; this public sharing version omits project-specific .do files.

---

## 5. Date Processing

### Convert String to Date

```stata
// From "YMD" format string
gen date = date(datestring, "YMD")

// From "MDY" format
gen date = date(datestring, "MDY")

// Format for display
format date %tdCCYY-NN-DD
```

### Extract Date Components

```stata
gen year = year(date)
gen month = month(date)
gen day = day(date)
gen quarter = quarter(date)
gen ym = ym(year, month)  // Monthly date
format ym %tm
```

### Fiscal Year Adjustment

```stata
// Common in finance research: fiscal year ends in Dec
// Adjust if using calendar year for analysis

// Method 1: Shift year for months before cutoff
gen fyear = year
replace fyear = year - 1 if month < 10

// Method 2: Create fiscal year based on reporting date
gen fyear = year(date) - 1 if month(date) < 5
```

### Handle Monthly Data

#### Basic Monthly Date Creation

```stata
// Create year-month variable
gen ym = ym(year, month)
format ym %tm

// Sort by year-month
sort Stkcd ym

// Create lagged variables
xtset stkcd ym
gen var_lag = l.var
```

#### Chart-Ready Monthly Date Workflow

For plots with proper axis labels, create a formatted monthly date and derive tick positions from the data, not hard-coded integers.

```stata
*** Step 1: Create and format monthly date
gen mdate = ym(year, month)
format mdate %tm

*** Step 2: Sort for time-series operations
sort Stkcd mdate

*** Step 3: Optional panel declaration for lags/leads
xtset Stkcd mdate

*** Step 4: Validate date range (adjust bounds to your sample window)
summarize mdate, detail
// Example: assert r(min) >= ym(2016, 1) if r(N) > 0
// Example: assert r(max) <= ym(2023, 12) if r(N) > 0
// Or use generic sanity checks:
assert r(min) > ym(1900, 1) if r(N) > 0  // Reasonable lower bound
assert r(max) < ym(2100, 1) if r(N) > 0  // Reasonable upper bound

*** Step 5: Derive axis ticks from actual data (NEVER hard-code)
summarize mdate
local mdate_min = r(min)
local mdate_max = r(max)
local start_year = year(dofm(`mdate_min'))
local end_year = year(dofm(`mdate_max'))

// Create ticks at year boundaries using tm() literals derived from data
local ticks ""
forvalues y = `start_year'/`end_year' {
    local tick = tm(`y'm1)  // January of each year
    local ticks "`ticks' `tick'"
}
```

#### Monthly Date Diagnostics

```stata
*** Check for gaps in monthly sequence
bysort Stkcd (mdate): gen gap = mdate - mdate[_n-1]
tab gap if _n > 1  // Gap > 1 means missing months

*** Verify month extraction round-trips correctly
gen month_check = month(dofm(mdate))
assert month_check == month if !missing(mdate, month)
drop month_check

*** Count failed conversions
count if missing(mdate) & (!missing(year) | !missing(month))
di "Records with missing mdate but non-missing components: " r(N)
```

#### Graph-Ready Monthly Plot Template

```stata
*** Derive ticks from data, format with %tm
summarize mdate
local mdate_min = r(min)
local mdate_max = r(max)

*** Compute year-start ticks from actual data range
local start_year = year(dofm(`mdate_min'))
local end_year = year(dofm(`mdate_max'))
local tick_start = tm(`start_year'm1)
local tick_end = tm(`end_year'm1)

*** Plot with data-derived ticks
twoway (line roa mdate), ///
    xlabel(`tick_start'(12)`tick_end', format(%tmCCYY)) ///
    xtitle("Year") ///
    title("Monthly ROA 2016-2023") ///
    note("Axis labels derived from mdate variable, not hard-coded")

graph export "monthly_roa.png", width(1600) replace
confirm file "monthly_roa.png"
```

#### Synthetic Monthly Chart Smoke Test

```stata
* Acceptance smoke test: proves month sequence, chart execution, and file export
clear
set obs 72
local smoke_start = tm(2018m1)
local smoke_end = tm(2023m12)
local smoke_tick_end = tm(2023m1)

gen mdate = `smoke_start' + _n - 1
format mdate %tm
assert mdate[1] == `smoke_start'
assert mdate[_N] == `smoke_end'
assert mdate == `smoke_start' + _n - 1

gen monthly_roa = 0.03 + 0.001 * _n
gen trading_volume = 100 + 2 * _n

twoway ///
    (line monthly_roa mdate, yaxis(1)) ///
    (bar trading_volume mdate, yaxis(2)), ///
    xlabel(`smoke_start'(12)`smoke_tick_end', format(%tmCCYY)) ///
    ytitle("Monthly ROA", axis(1)) ///
    ytitle("Trading Volume", axis(2))

graph export "monthly_chart_smoke.png", width(1600) replace
confirm file "monthly_chart_smoke.png"
```

**Critical rule for monthly plots**: Build the monthly date variable with `ym(year, month)`, use `%tm` formatting for display, use `tm(year'month)` or `ym(year, month)` to derive chart ticks from actual data, never hard-code monthly integers like `xlabel(672(12)743)`, bind any secondary series with `yaxis(2)`, and verify `graph export` with `confirm file`.

### Date Arithmetic

```stata
// Days between dates
gen days_diff = date2 - date1

// Find nearest date after event
gen Dif = date - event_date
bys Stkcd year: egen MD = min(Dif) if Dif >= 0
```

**Pattern note**: Derived from internal Stata pattern examples; this public sharing version omits project-specific .do files.

---

## 6. Grouping and Aggregation

### Basic Aggregation with `egen`

```stata
// Sum
bys stkcd year: egen sum_var = sum(variable)

// Mean
bys stkcd year: egen mean_var = mean(variable)

// Count
bys stkcd year: egen count_var = count(variable)

// Standard deviation
bys stkcd year: egen sd_var = sd(variable)

// Max/Min
bys stkcd year: egen max_var = max(variable)
bys stkcd year: egen min_var = min(variable)

// Median
bys stkcd year: egen median_var = median(variable)
```

### Conditional Aggregation

```stata
// Sum with condition
bys stkcd year: egen sum_positive = sum(variable) if variable > 0

// Count unique values
bys stkcd year: egen n_unique = nvals(variable)

// Row statistics
egen row_mean = rowmean(var1 var2 var3)
egen row_sd = rowsd(var1 var2 var3)
```

### Handle Duplicates

```stata
// Method 1: Mark and drop duplicates
bys stkcd year: gen dup = cond(_N==1, 0, _n)
drop if dup > 1

// Method 2: Direct drop
duplicates drop stkcd year, force

// Method 3: Keep first/last occurrence
bys stkcd year: gen n = _n
keep if n == 1
```

### Create Group Identifiers

```stata
// Create group id
egen group_id = group(var1 var2 var3)

// Create sequential id within group
bys stkcd: gen obs_n = _n
bys stkcd: gen total_n = _N
```

**Pattern note**: Derived from internal Stata pattern examples; this public sharing version omits project-specific .do files.

---

## 7. Data Merging

### Merge Command

```stata
// 1:1 merge (one-to-one)
merge 1:1 stkcd year using otherdata

// m:1 merge (many-to-one)
merge m:1 stkcd year using otherdata

// 1:m merge (one-to-many)
merge 1:m stkcd year using otherdata

// Keep only matched observations
keep if _merge == 3
drop _merge

// Keep all from master (left join)
keep if _merge != 2
drop _merge
```

### frlink/frget (Frame-based Merge)

```stata
// Create link between frames
// 1:1 link
frlink 1:1 Stkcd year, frame(OtherFrame)

// m:1 link (many-to-one)
frlink m:1 Stkcd year, frame(OtherFrame)

// Get variables from linked frame
frget var1 var2 var3, from(OtherFrame)

// Conditional get
frget var1, from(OtherFrame) gen(new_varname)
```

### Merge with Date Ranges

```stata
// Merge with date condition
merge m:1 stkcd using otherdata
keep if date >= start_date & date <= end_date
drop _merge
```

### Multiple Merges

```stata
// Sequential merges
use maindata, clear
merge m:1 stkcd year using data1, keep(master match)
drop _merge
merge m:1 stkcd year using data2, keep(master match)
drop _merge
merge m:1 stkcd year using data3, keep(master match)
drop _merge
```

### Merge with Preserve/Restore

```stata
preserve
    use otherdata, clear
    keep stkcd year var1 var2
    save temp, replace
restore

merge m:1 stkcd year using temp
drop _merge
erase temp.dta
```

**Pattern note**: Derived from internal Stata pattern examples; this public sharing version omits project-specific .do files.

---

## Append vs Merge vs Frame: Operational Decision Guide

### Quick Decision Tree

```
Single source with multiple periods/batches?
    └── YES → Use append
        
Two datasets, different variables, same keys?
    └── YES → Use merge (one-time)
        
Three+ sources, repeated linking needed?
    └── YES → Use frame workflow
        
Large auxiliary data (>10k obs) linked multiple times?
    └── YES → Use frame with frlink
```

### Append Pattern (Same Schema)

```stata
// Scenario: 13 folders of daily returns, same columns
use stk1, clear
append using stk2 stk3 stk4 stk5 stk6 stk7 stk8 stk9 stk10 stk11 stk12 stk13
duplicates drop
save consolidated, replace
```

### Merge Pattern (Two Datasets)

```stata
// Scenario: Add financial data to stock returns
use returns, clear
merge m:1 Stkcd year using financial_data
tab _merge
keep if _merge == 3
drop _merge
```

### Frame Pattern (Multi-Source Integration)

```stata
// Scenario: Event study with returns, events, controls
frame create Events
frame Events {
    use inquiry_dates, clear
    keep Stkcd year date_issue
}

frame create Returns
frame Returns {
    use stk_day, clear
    rename stkcd Stkcd
    gen year = year(date)
}

frame change default
use main_sample, clear
frlink m:1 Stkcd year, frame(Events)
frget date_issue, from(Events)
```

---

## Best Practices

### 1. Always Use Frames for Multi-Source Data

```stata
// GOOD: Each source in its own frame
frame create Source1
frame Source1 { use data1, clear }
frame create Source2
frame Source2 { use data2, clear }

// BAD: Multiple merge chains
use data1, clear
merge m:1 id using data2
merge m:1 id using data3
merge m:1 id using data4
```

### 2. Check Merge Results

```stata
merge m:1 stkcd year using otherdata
tab _merge  // Always check merge result distribution
keep if _merge == 3
drop _merge
```

### 3. Handle Missing Values After Merge

```stata
frget var1, from(OtherFrame)
replace var1 = 0 if missing(var1)  // or appropriate value
```

### 4. Use Consistent Key Variables

```stata
// Ensure consistent types before merge
destring Stkcd, gen(stkcd)
// or
tostring stkcd, gen(Stkcd) format(%06.0f)
```

### 5. Normalize Stkcd Early

```stata
// Standard pattern: force Stkcd (capital S) as primary ID
rename symbol Stkcd      // lowercase variants
rename Symbol Stkcd      // Wind/CSMAR variants
rename Scode Stkcd       // CNRDS variant
destring Stkcd, replace  // ensure numeric if possible
```

### 6. Date Processing Checklist

```stata
// Step 1: Convert from string
gen date = date(OriginalDate, "YMD")

// Step 2: Extract components
gen year = year(date)
gen month = month(date)
gen quarter = quarter(date)

// Step 3: Format for display
format date %tdCCYY-NN-DD
format ym %tm

// Step 4: Check for missing conversions
count if missing(date) & !missing(OriginalDate)

// Step 5: Validate monthly dates if present
capture confirm var ym
if !_rc {
    assert !missing(ym) if !missing(year) & !missing(month)
    format ym %tm
}
```
