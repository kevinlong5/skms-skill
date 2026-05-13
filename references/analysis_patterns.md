# Analysis and Output Patterns

## Table of Contents

1. [Default SOP Matrix by Workflow Type](#1-default-sop-matrix-by-workflow-type)
2. [Panel Data Setup SOP](#2-panel-data-setup-sop)
3. [Indicator Construction](#3-indicator-construction)
   - [Event Study (CAR Analysis)](#event-study-car-analysis)
4. [Descriptive Statistics](#4-descriptive-statistics)
5. [Regression Analysis SOP](#5-regression-analysis-sop)
6. [Output Formatting](#6-output-formatting)
7. [SKMS Output Templates](#7-skms-output-templates)

---

## 1. Default SOP Matrix by Workflow Type

| Task Family | Default Approach | Key Commands |
|-------------|------------------|--------------|
| **Panel setup** | xtset after identifier normalization | `destring`, `xtset stkcd year`, `xtdescribe` |
| **Variable processing** | Winsorize at 1%/99% before analysis | `winsor2`, `log`, `gen` dummies |
| **Indicator construction** | egen/bys aggregation, event windows | `egen`, `bys`, `sum/arithmetic` |
| **Event study (subfamily)** | Market-adjusted model, [-1,+1] window | `ar = rstk - rmkt`, rolling sum |
| **Regression analysis** | reghdfe with industry/year FE, clustered SE | `reghdfe`, `absorb()`, `vce(cluster)` |
| **Descriptive/statistical reporting** | tabstat/logout, ttable2, outreg2 | `logout`, `tabstat`, `outreg2` |

### Standard Workflow Order

```
1. Panel setup (xtset after destringing Stkcd)
2. Variable processing (winsor2, logs, interactions)
3. Indicator construction (event dummies, financial ratios)
4. Regression analysis (reghdfe sequence with outreg2)
5. Descriptive statistics (logout/tabstat/ttable2)
6. Output formatting (tables and figures)
```

**Note on ordering**: The regression analysis SOP internally orders steps as: variable processing → panel verification → model sequence. This is consistent with the broader workflow because regression analysis itself comes after indicator construction in the overall pipeline.

---

## 2. Panel Data Setup SOP

### Standard Panel Setup

```stata
***Step 1: Normalize identifiers***
destring Stkcd, gen(stkcd)    // Create numeric ID if needed
// or: rename stkcd Stkcd      // Ensure capital S for consistency

***Step 2: Declare panel structure***
xtset stkcd year

***Step 3: Verify panel***
xtdescribe                     // Check panel structure
xtsum var1 var2 var3          // Panel summary statistics
```

### Panel Setup Checklist

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `destring Stkcd, gen(stkcd)` | Create numeric firm ID |
| 2 | `xtset stkcd year` | Declare panel structure |
| 3 | `xtdescribe` | Verify balance and gaps |
| 4 | `xtsum [vars]` | Check within/between variation |

### Common Panel Issues

```stata
***Handle gaps in panel***
bys stkcd: gen gap = year[_n] - year[_n-1]
tab gap  // Check for irregular spacing

***Handle duplicates***
bys stkcd year: gen dup = _N > 1
tab dup  // Check for duplicate firm-years
duplicates drop stkcd year, force  // Remove if appropriate
```

---

## 3. Indicator Construction

### Financial Ratio Construction

```stata
***after xtset and loading Acc data (CSMAR variable codes)***
* CSMAR codes: A001000000 = Total Assets, B002000000 = Net Income
*                     A002000000 = Total Debt, B001100000 = Sales
rename A001000000 TA
rename B002000000 NI
rename A002000000 DEBT
rename B001100000 SALE

***standard ratios***
gen Lev = DEBT / TA               // Leverage
gen ROA = NI / TA                 // Return on Assets
gen Growth = SALE / l.SALE        // Sales growth
gen Loss = NI < 0                 // Loss dummy
gen logTA = log(TA)               // Log assets
gen logMV = log(MV)               // Log market value (from Financial frame)
```

### Policy/Event Dummies

```stata
***time-based policy dummy***
gen Post = year >= 2020           // Post-treatment period
gen Treat = (Plate == "主板")     // Treatment group
gen DID = Post * Treat            // Interaction

***event-based dummy***
gen Event = (date == event_date)  // Event day
gen Window = inrange(day_to_event, -10, 10)  // Event window
```

### Event Study (CAR Analysis)

**Purpose**: Calculate cumulative abnormal returns around event dates. This is a subfamily of indicator construction.

#### Step 1: Event Date Identification

```stata
// Load stock return data
cd "<DATA_ROOT>/CSMAR/股票收益率/"
use stk_day, clear
rename stkcd Stkcd
gen year = year(date)
gen month = month(date)

// Link to event data (e.g., announcement dates)
frlink m:1 Stkcd year, frame(Events)
frget event_date, from(Events)
drop if Events == .  // Keep only firms with events
```

#### Step 2: Handle Non-Trading Days

```stata
// Mark event day (exact match)
gen event = 1 if date == event_date

// Find next trading day if event_date is non-trading day
gen Dif = date - event_date
bys Stkcd year: egen MD = min(Dif) if Dif >= 0
replace event = 1 if Dif == MD & event == . & Dif != . & MD <= 10
replace event = 0 if missing(event)
```

#### Step 3: Calculate Abnormal Returns

```stata
// Market-adjusted model (default)
gen ar = rstk - rmkt

// Or use market model (requires estimation window)
sort Stkcd date
bys Stkcd: asreg rstk rmkt, win(date -130 -10)
gen ar = rstk - _b_rmkt * rmkt - _b_cons
```

#### Step 4: Calculate CAR

```stata
// Short window [-1, +1]
sort Stkcd date
gen car_3d = ar[_n-1] + ar + ar[_n+1] if event == 1

// Medium window [-2, +2]
gen car_5d = ar[_n-2] + ar[_n-1] + ar + ar[_n+1] + ar[_n+2] if event == 1

// Long window (e.g., [-30, -10] for pre-event)
preserve
    keep if Dif >= -30 & Dif <= -10
    bys Stkcd year: egen car_pre_30_10 = sum(ar)
    keep Stkcd year car_pre_30_10
    duplicates drop
    save temp_car, replace
restore
merge m:1 Stkcd year using temp_car
drop _merge
```

#### Step 5: Event Window Definition

```stata
// Create day-to-event variable
gen day_to_event = .
forvalue x = 1/10 {
    bys Stkcd year: replace day_to_event = `x' if event[_n-`x'] == 1
    bys Stkcd year: replace day_to_event = -`x' if event[_n+`x'] == 1
}
replace day_to_event = 0 if event == 1

// Keep event window
keep if day_to_event >= -10 & day_to_event <= 10
```

#### Step 6: Statistical Tests

```stata
// Keep only event day observations
keep if event == 1

// Output statistics
logout, save(CAR_results) excel replace: ///
    tabstat car_3d car_5d, s(mean median sd N) by(group)

// t-test against zero
ttest car_3d == 0

// Group comparison
ttable2 car_3d, by(treatment_group)
```

**Pattern note**: Derived from internal Stata pattern examples; this public sharing version omits project-specific .do files.

---

## 4. Descriptive Statistics

### Basic Descriptive Statistics

```stata
// Full sample
tabstat var1 var2 var3 var4 var5, s(mean sd min p50 max N)

// By group
tabstat var1 var2 var3, s(mean sd) by(group)

// Multiple statistics
tabstat var1 var2, s(mean median sd skew kurt) format(%9.3f)
```

### Export to Excel

```stata
// Single table
logout, save(descriptive) excel replace: ///
    tabstat var1 var2 var3, s(mean sd min max) by(group)

// Frequency table
logout, save(frequency) excel replace: ///
    tab var1 var2

// Cross-tabulation
logout, save(crosstab) excel replace: ///
    tabulate var1 var2, row col
```

### Group Comparisons

```stata
// t-test
ttest var1, by(group)

// t-test with multiple variables (requires ttable2)
ttable2 var1 var2 var3 var4, by(group)

// Export t-test results
logout, save(ttest_results) excel replace: ///
    ttable2 var1 var2 var3, by(group)
```

### Correlation Matrix

```stata
// Correlation
correlate var1 var2 var3 var4

// Export correlation
logout, save(correlation) excel replace: ///
    correlate var1 var2 var3 var4

// With significance (using pwcorr)
logout, save(correlation_sig) excel replace: ///
    pwcorr var1 var2 var3 var4, sig
```

### Visualization

```stata
// Bar chart
graph bar (mean) var1 var2 var3, over(group) ///
    blabel(bar, format(%9.2f) size(small)) ///
    legend(label(1 "Var1") label(2 "Var2") label(3 "Var3")) ///
    title("Descriptive Statistics by Group") ///
    ytitle("Mean Value")

graph export "descriptive_bar.png", width(2000) replace

// Histogram
histogram var1, frequency by(group) ///
    title("Distribution of Var1")

graph export "histogram.png", width(2000) replace

// Scatter plot
scatter var2 var1, msize(small) ///
    title("Var2 vs Var1") ///
    xtitle("Var1") ytitle("Var2")

graph export "scatter.png", width(2000) replace
```

#### Monthly Time-Series Template

```stata
* Assumes `mdate` is an existing monthly Stata date created with ym(year, month)
format mdate %tm
sort mdate

summarize mdate
local start_year = year(dofm(r(min)))
local end_year = year(dofm(r(max)))
local tick_start = ym(`start_year', 1)
local tick_end = ym(`end_year', 1)
local axis_labsize "small"

twoway ///
    (line roa mdate, lcolor(navy) lwidth(medthick) yaxis(1)), ///
    xlabel(`tick_start'(12)`tick_end', format(%tmCCYY) labsize(`axis_labsize')) ///
    xtitle("Month", size(`axis_labsize')) ///
    ytitle("ROA", axis(1) size(`axis_labsize')) ///
    ylabel(, axis(1) labsize(`axis_labsize')) ///
    title("Monthly ROA Trend")

graph export "monthly_roa_trend.png", width(2000) replace
confirm file "monthly_roa_trend.png"
```

#### Monthly Dual-Axis Bar + Line Template

```stata
* Assumes `mdate` is an existing monthly Stata date created with ym(year, month)
format mdate %tm
sort mdate

summarize mdate
local start_year = year(dofm(r(min)))
local end_year = year(dofm(r(max)))
local tick_start = ym(`start_year', 1)
local tick_end = ym(`end_year', 1)
local axis_labsize "small"

twoway ///
    (line monthly_ret mdate, lcolor(navy) lwidth(medthick) yaxis(1)) ///
    (bar trading_volume mdate, barwidth(0.8) color(gs12) yaxis(2)), ///
    xlabel(`tick_start'(12)`tick_end', format(%tmCCYY) labsize(`axis_labsize')) ///
    xtitle("Month", size(`axis_labsize')) ///
    ytitle("Monthly Return", axis(1) size(`axis_labsize')) ///
    ytitle("Trading Volume", axis(2) size(`axis_labsize')) ///
    ylabel(, axis(1) labsize(`axis_labsize')) ///
    ylabel(, axis(2) labsize(`axis_labsize')) ///
    legend(order(1 "Monthly Return (left axis)" 2 "Trading Volume (right axis)") size(`axis_labsize')) ///
    title("Monthly Return and Trading Volume")

graph export "monthly_return_volume_dual_axis.png", width(2000) replace
confirm file "monthly_return_volume_dual_axis.png"
```

#### Monthly Chart Smoke-Test Acceptance

```stata
* Use this before trusting a new monthly plot or dual-axis overlay
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

gen monthly_ret = 0.01 * sin(_n / 6)
gen trading_volume = 100 + 2 * _n

twoway ///
    (line monthly_ret mdate, lcolor(navy) yaxis(1)) ///
    (bar trading_volume mdate, color(gs12) yaxis(2)), ///
    xlabel(`smoke_start'(12)`smoke_tick_end', format(%tmCCYY)) ///
    ytitle("Monthly Return", axis(1)) ///
    ytitle("Trading Volume", axis(2))

graph export "monthly_chart_smoke.png", width(1600) replace
confirm file "monthly_chart_smoke.png"
```

Acceptance means the month sequence is proven with explicit `%tm` anchors, the secondary series is bound with `yaxis(2)`, and the exported file is checked with `confirm file` rather than visual inspection alone.

**Pattern note**: Derived from internal Stata pattern examples; this public sharing version omits project-specific .do files.

---

## 5. Regression Analysis SOP

### Standard Regression Workflow

```
Step 1: Variable processing (winsor2, logs, interactions)
Step 2: Panel setup verification (xtset, diagnostics)
Step 3: Baseline model
Step 4: Add controls
Step 5: Subsample analysis
Step 6: Export with outreg2
```

**Alignment with broader workflow**: Within the regression family, variable processing (winsorization, transformations) comes first, then panel structure is verified before estimation. This ordering ensures clean data enters the regression sequence while maintaining the overall pipeline flow.

### Variable Processing SOP

```stata
***Step 1: Winsorize at 1% and 99%***
// Method 1: winsor2 for multiple variables (default)
winsor2 Y X C1 C2 C3, cuts(1 99) replace

// Method 2: Individual winsorization
winsor var1, gen(temp) p(0.01)
replace var1 = temp
drop temp

***Step 2: Log transformations***
gen logTA = log(TA)
gen logMV = log(MV)
gen logSALE = log(SALE)

***Step 3: Interaction terms***
gen X_Z = X * Z
gen X_high = X * high_dummy

***Step 4: Dummy variables***
gen Loss = NI < 0
gen SOE = (ownership == "state")
gen Post = year >= policy_year
```

### Panel Setup (before regression)

```stata
***Ensure numeric firm ID***
destring Stkcd, gen(stkcd_num)
xtset stkcd_num year

***Verify panel***
xtdescribe
xtsum Y X C1 C2  // Check within/between variation
```

### Regression Sequence

```stata
***Model 1: Baseline***
reghdfe Y X, absorb(Ind fyear) vce(cluster stkcd)
outreg2 using "regression.doc", replace stats(coef se) ///
    tdec(3) bdec(3) e(r2_a N) ctitle(Baseline)

***Model 2: Add controls***
reghdfe Y X C1 C2 C3, absorb(Ind fyear) vce(cluster stkcd)
outreg2 using "regression.doc", append stats(coef se) ///
    tdec(3) bdec(3) e(r2_a N) ctitle(With Controls)

***Model 3: Full specification***
reghdfe Y X C1 C2 C3 C4 C5, absorb(Ind fyear) vce(cluster stkcd)
outreg2 using "regression.doc", append stats(coef se) ///
    tdec(3) bdec(3) e(r2_a N) ctitle(Full Model)
```

### Fixed Effects Specification Guide

| Specification | absorb() | Use When |
|---------------|----------|----------|
| Industry + Year | `absorb(Ind fyear)` | Standard panel |
| Firm + Year | `absorb(stkcd fyear)` | Within-firm variation |
| Industry*Year | `absorb(Ind#fyear)` | Industry-specific trends |
| Two-way | `absorb(Ind fyear)` with `vce(cluster stkcd)` | Standard robustness |

### Standard Error Clustering

```stata
***Default: Cluster by firm***
reghdfe Y X C1 C2, absorb(Ind fyear) vce(cluster stkcd)

***Two-way clustering***
reghdfe Y X C1 C2, absorb(Ind fyear) vce(cluster stkcd Ind)

***Robust SE***
reghdfe Y X C1 C2, absorb(Ind fyear) vce(robust)
```

### Alternative Regression Commands

```stata
***OLS (for comparison only)***
reg Y X C1 C2 C3

***OLS with robust SE***
reg Y X C1 C2 C3, robust

***OLS with clustered SE***
reg Y X C1 C2 C3, vce(cluster stkcd)

***Fixed effects (xtreg)***
xtreg Y X C1 C2, fe cluster(stkcd)
```

**Pattern note**: Derived from internal Stata pattern examples; this public sharing version omits project-specific .do files.

---

## 6. Output Formatting

### outreg2 Options

```stata
// Basic usage
outreg2 using "filename.doc", replace

// With statistics
outreg2 using "filename.doc", replace stats(coef se) tdec(3) bdec(3)

// With R-squared and N
outreg2 using "filename.doc", replace e(r2_a N)

// Multiple columns
outreg2 using "filename.doc", replace ctitle(Model1)
outreg2 using "filename.doc", append ctitle(Model2)

// Label variables
outreg2 using "filename.doc", replace label

// Keep specific variables
outreg2 using "filename.doc", replace keep(X C1 C2)

// Add notes
outreg2 using "filename.doc", replace addnote("Standard errors clustered by firm" "*** p<0.01, ** p<0.05, * p<0.1")
```

### logout Options

```stata
// Export tabstat
logout, save(filename) excel replace: tabstat var1 var2, s(mean sd)

// Export tab
logout, save(filename) excel replace: tab var1 var2

// Export correlation
logout, save(filename) excel replace: correlate var1 var2 var3

// Export regression
logout, save(filename) excel replace: reg Y X C1 C2
```

### graph export Options

```stata
// PNG format
graph export "figure.png", width(2000) replace

// PDF format
graph export "figure.pdf", replace

// SVG format
graph export "figure.svg", replace

// EMF format (for Word)
graph export "figure.emf", replace
```

**Pattern note**: Derived from internal Stata pattern examples; this public sharing version omits project-specific .do files.

---

## 7. SKMS Research Stages and Outputs

### Scratch Check

Use `scratch_check` for quick command checks, debugging, variable-existence checks, and small smoke-test regressions. Write a temporary `.do`, run it with direct Stata batch execution, read the generated `.log`, report the result in conversation, and delete the temporary `.do` and `.log` unless the user asks to keep them.

```bash
<STATA_BATCH_CMD> _skms_temp_test.do
```

Scratch checks are conversation-first. If the result becomes substantively useful, promote the code to a working iteration with a descriptive filename.

Do not leave generic runtime logs such as `stata.log` or `stdin.log` as formal evidence. Delete them after the check, or rename/move them only if they are the sole evidence for a kept run.

### Working Iteration

Use `working_iteration` for key-variable construction, data assembly, preliminary tests, exploratory regressions, and useful but still-evolving model versions.

Recommended kept files:

```
[descriptive_name].do     # Research code for the current iteration
[descriptive_name].log    # Execution transcript and diagnostics
[descriptive_name].md     # Optional short notes if interpretation or handoff value exists
```

Working iterations should be easy to rerun and inspect, but they do not require a full formal explanation file. Use descriptive names such as `construct_key_variable_v03.do`, `merge_panel_controls_v02.do`, or `baseline_tests_v04.do`.

### Milestone Release Package

Use `milestone_release` for v1.0/v1.1, coauthor or RA handoff, paper-table output, and reproducible archives. Formal SKMS releases use a shared-basename reproducibility spine:

```
[analysis].do     # Executable code
[analysis].log    # Execution transcript
[analysis]_summary.md     # Result index, evidence sources, and decisions
```

This is the evidence contract. A complete milestone should also include formal outputs when the analysis produces datasets, tables, or figures: `.dta`, `.xlsx`, `.docx`, `.doc`, or `.png`.

### Package Contents by File

| File | Role | Key Contents |
|------|------|--------------|
| `.do` | Executable | Complete, commented Stata code; all steps from data load to final output |
| `.log` | Verification | Command echo and Stata output; proof of execution; diagnostic info |
| `_summary.md` | Result index | Research question, data sources, execution command, output files, evidence sources, decision notes |
| `.xlsx` / `.docx` / `.doc` / `.png` | Formal output | Tables, figures, or paper-facing artifacts |

### Template: Result Summary

```markdown
# [Analysis Title]

## Objective
[Brief research question]

## Data Sources
- Primary: [path/description]
- Period: [start] to [end]
- Sample: N observations

## Source Artifacts
- Do-file: [path]
- Log: [path]
- Output tables/figures: [path(s)]
- Verification note: numerical claims below are taken from the current log or exported output tables.

## Methodology

### Variable Definitions
| Variable | Definition | Construction |
|----------|------------|--------------|
| Y | [outcome] | [how calculated] |
| X | [predictor] | [how calculated] |

### Model/Approach
[Description of regression model, event study window, or descriptive approach]

## Results

### [Table/Statistic Title]
| | Value |
|---|---|
| [Metric] | [value] |

[Interpretation of key findings]

## Execution

```bash
<STATA_BATCH_CMD> [analysis].do
```

Generated: [analysis].do, [analysis].log, [analysis]_summary.md, [formal outputs]
```

### Template: .do File

```stata
* ============================================================
* [Analysis Title]
* Generated: [Date]
* Stata: add a project-specific version line only if required
* Output log: [log path]
* ============================================================

clear all
set more off
set varabbrev off
capture log close
log using "[log_dir]/[analysis].log", replace text

*-------------------------------------------------------------
* Section 1: Setup
*-------------------------------------------------------------

// Load data
cd "[path]"
use data, clear

// Panel setup
destring Stkcd, gen(stkcd)
xtset stkcd year

*-------------------------------------------------------------
* Section 2: Analysis
*-------------------------------------------------------------

// [Task-specific code: regression, event study, or descriptive]

*-------------------------------------------------------------
* Section 3: Output
*-------------------------------------------------------------

// [Output commands: outreg2, logout, graph export]

log close
```

### Task Family Variations

In milestone release, the reproducibility spine is fixed; what varies is the internal workflow and the formal output companions.

**Regression/Analysis Workflow**
```
.do:  winsor2 → xtset → reghdfe → outreg2
.log: Coefficients, standard errors, R-squared, F-stats
summary.md:  Model comparison table, coefficient interpretation, evidence links
```

**Event Study Workflow**
```
.do:  Event date alignment → abnormal returns → CAR calculation → t-tests
.log: CAR means by window, test statistics
summary.md:  Event window description, CAR table, significance notes, evidence links
```

**Descriptive/Reporting Workflow**
```
.do:  tabstat sequence → logout → ttable2 → graphs
.log: Summary statistics, test results
summary.md:  Sample characteristics table, group comparisons, visual summaries, evidence links
```

**Indicator Construction Workflow**
```
.do:  Frame setup → frlink/frget → ratio calculations
.log: Variable creation steps, merge diagnostics
summary.md:  Variable definitions, data coverage, validation checks, evidence links
```

### Companion Artifacts

Inside working iterations or milestone releases, you may produce companion files:

```
[basename]_panel.dta     # saved analysis panel or constructed key-variable dataset
[basename]_regs.doc      # outreg2 regression table
[basename]_desc.xlsx     # logout/descriptive output
[basename]_fig.png       # exported figure
```

In `working_iteration`, companions can be the main practical output. In `milestone_release`, they are formal outputs linked to the `.do + .log + _summary.md` reproducibility spine, not replacements for it.

### Execution

```bash
# Standard batch execution
<STATA_BATCH_CMD> analysis.do

# Verify package
ls analysis.*
# → analysis.do, analysis.log, analysis_summary.md, formal outputs
```
