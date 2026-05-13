# Optional Advanced Layer (Non-Default Enhancements)

Use this layer only when the user explicitly wants extra hardening, auditability, or office-format companions. It does not change the default SOPs, required declarations, or the top-level `milestone_release` reproducibility spine plus formal-output contract.

| Optional addition | Applies to | Why it helps | Non-default boundary |
|-------------------|------------|--------------|----------------------|
| Schema/assert gates | Raw import, integration, indicator construction | Fails fast when required variables, date conversions, or key fields break | Add only when source quality is uncertain or high-stakes |
| Merge diagnostics | Multi-source integration | Makes unmatched keys and coverage loss visible before keeping/dropping observations | Keep out of the default SOP unless join risk is material |
| Panel preflight checks | Regression/analysis, panel indicator work | Catches duplicates, gaps, and missing panel keys before `xtset`/`reghdfe` | Use as a hardening pass, not a required baseline |
| Event-study helper scaffold | Indicator construction subfamily: event study / CAR | Standardizes non-trading-day tolerance, estimation windows, and CAR windows across repeated runs | Keep nested under indicator construction; do not promote event study to a top-level family |
| Provenance/run summary | All families | Leaves an auditable record of source paths, sample counts, and generated artifacts in the log/markdown handoff | Optional traceability layer only |
| Result-export helpers (`docx` / `xlsx`) | Regression/analysis, descriptive/reporting | Produces shareable office-format companions after the evidence spine is stable | Never replace `.do + .log + _summary.md` as the reproducibility spine |
| Log-verified reporting | All families with reported numbers | Blocks hallucinated sample sizes, coefficients, p-values, or table claims | Required for numeric claims; optional hardening adds richer evidence links |

---

## 1. Schema/assert gates

Use when vendor headers drift, multiple raw files arrive from different systems, or the user wants a fail-fast import/integration script.

```stata
* Optional fail-fast gate before transformation
foreach v in Stkcd RawDate {
    cap confirm variable `v'
    if _rc {
        di as err "Missing required variable: `v'"
        error 111
    }
}

count if missing(Stkcd)
assert r(N) == 0

count if missing(date) & !missing(RawDate)
assert r(N) == 0
```

---

## 2. Merge diagnostics

Use when linking high-value control sets, combining heterogeneous sources, or defending sample attrition in the final `.md`.

```stata
* Optional diagnostic pass before filtering merged data
merge m:1 Stkcd year using controls_data
tab _merge
count if _merge == 1
count if _merge == 2

* Example policy: block unexpected using-only rows
assert _merge != 2

keep if _merge == 3
drop _merge
```

For frame workflows, use the same idea after `frget`: count missing pulled variables and record unmatched coverage in the log before continuing.

---

## 3. Panel preflight checks

Use when the workflow will call `xtset`, `reghdfe`, lag operators, or within-firm comparisons.

```stata
* Optional panel hardening pass
count if missing(stkcd) | missing(year)
assert r(N) == 0

isid stkcd year, sort
xtset stkcd year
xtdescribe
xtsum Y X C1 C2
```

---

## 4. Event-study helper scaffold

Use when repeated CAR runs need consistent windows, estimation periods, or non-trading-day handling.

```stata
* Optional helper scaffold for repeated CAR designs
local trading_tolerance 10
local est_start -130
local est_end -10
local car_lo -1
local car_hi 1

gen Dif = date - event_date
bys Stkcd year: egen MD = min(Dif) if Dif >= 0
replace event = 1 if Dif == MD & missing(event) & MD <= `trading_tolerance'
```

If a project needs multiple windows, define them as locals/macros once and reuse them across CAR calculations instead of hand-editing each block.

---

## 5. Provenance capture and run summary

Use when the user needs an auditable trail for replication, memo writing, or review by a coauthor/advisor.

```stata
* Optional provenance block
local source_returns "<DATA_ROOT>/CSMAR/股票收益率/"
local source_controls "<DATA_ROOT>/CSMAR/财务报表/最新报表"
quietly count
local final_n = r(N)

di as txt "RUN_SUMMARY returns=`source_returns' controls=`source_controls' N=`final_n'"
di as txt "RUN_SUMMARY artifacts=analysis.do analysis.log analysis.md"
```

Preferred pattern: keep the formal deliverable unchanged, but surface the same provenance lines in the `.md` metadata or execution section.

---

## 6. Log-verified reporting

Use when reporting sample sizes, descriptive statistics, correlations, coefficients, p-values, CARs, table entries, or any other numeric empirical result.

Rule: no current log or exported output table, no numerical claim. If the run failed or the log is missing, report that the code is prepared or that the result still needs verification.

For milestones, the result summary should point to:

```text
Do-file: [basename].do
Log: [basename].log
Formal outputs: [basename]_tables.xlsx, [basename]_regs.docx, [basename]_fig*.png
Evidence note: numeric claims are taken from the current log or exported output tables.
```

If a run produces generic `stata.log` or `stdin.log`, do not treat it as a stable artifact. Delete it after a scratch check, or move/rename it into the log directory with a meaningful basename if it is needed as evidence.

---

## 7. Optional result-export helpers for `docx` / `xlsx`

Use when the user wants a shareable Word or Excel companion for tables after the core SKMS package is already defined.

Regression/analysis helper:

- Keep `outreg2` as the Stata-side table builder.
- Treat `.doc` or converted `.docx` tables as formal presentation outputs, not as replacements for the reproducibility spine.
- If a polished Word-native handoff is requested, generate the `.do + .log + _summary.md` spine first, then create a companion `.docx` from the stable regression output.

Descriptive/reporting helper:

- Keep `logout ... , excel` as the optional table export path.
- Treat `.xlsx` files as review or formatting helpers for descriptive tables, balance tests, and correlations.
- If spreadsheet formatting is requested, build it as a formal output after the evidence spine is complete, and keep any formulas/error checks inside that optional spreadsheet workflow.

Naming guidance:

```text
[basename].do
[basename].log
[basename]_summary.md
[basename]_regs.docx
[basename]_desc.xlsx
```

These helpers are downstream formal outputs. They improve delivery usability, while SKMS still keeps one executable file, one log, and one concise result summary as the milestone evidence spine.
