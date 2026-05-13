---
name: skms
description: |
  Use when the user asks for SKMS, the Stata skill, Stata empirical research code,
  .do/do-file work, or data/regression workflows in Stata. Triggers: skms, Stata,
  do file, CSMAR/国泰安, Wind/万得, reghdfe, outreg2, logout, winsor2, panel data/面板数据,
  fixed effects/固定效应, DID, event study/CAR, 数据清洗, 数据合并, 指标构造, 回归, 描述统计.
  Supports: raw import/cleaning, multi-source panel assembly, key-variable and
  financial-ratio construction, event-study CAR, fixed-effect/DID regressions,
  descriptive tables, figures, and reproducible research outputs.
  Not for: generic statistics explanation, non-Stata languages, generic programming,
  or generic Stata syntax unrelated to empirical research.
metadata:
  short-description: Empirical Stata workflows for research data and regressions
---

# SKMS - Stata Knowledge Management System

SKMS supports empirical accounting, finance, management, and auditing research workflows in Stata. Keep this file as the routing, SOP, and disclosure guide; load references and examples only when needed.

## Trigger And Route

Use SKMS for research-oriented Stata work:

| User need | Route |
|-----------|-------|
| CSMAR/Wind/raw vendor files, header cleanup, identifier/date cleaning | `raw_vendor_import` |
| append/merge/frame decisions, multi-source panel assembly | `multi_source_integration` |
| financial ratios, policy dummies, text indicators, key variables | `indicator_construction` |
| event-study/CAR construction | `indicator_construction / event_study_car` |
| `reghdfe`, fixed effects, DID, clustering, robustness | `regression_analysis` |
| table 1, descriptive stats, correlations, figures | `descriptive_reporting` |
| import -> integration -> indicators -> tables/regressions | `complete_workflow` |

Do not use SKMS for generic statistics explanation, non-Stata languages, general Stata syntax, dashboards, administrative spreadsheet cleanup, or purely conceptual econometrics.

## Research Stage And Artifact Policy

Classify the research stage before deciding what files to keep. Default to `working_iteration` for real research data construction and exploratory empirical analysis.

| Stage | Use when | Artifact policy |
|-------|----------|-----------------|
| `scratch_check` | quick command check, variable existence check, debug run, tiny regression smoke test | temporary `.do` and `.log`; report result; delete temp files and generic runtime logs unless user asks to keep them |
| `working_iteration` | key-variable construction, data assembly, preliminary tests, exploratory model versions, useful but not final research code | keep descriptive `.do` and `.log`; add short notes if useful; do not force a full `.md` |
| `milestone_release` | v1.0/v1.1, coauthor/RA handoff, paper table, reproducible archive, accepted analysis version | minimum reproducibility spine: `.do + .log + result summary .md`; add formal `.dta`, `.xlsx`, `.docx/.doc`, `.png` outputs as needed |

Stage can change during a task. If a scratch result becomes substantively useful, promote it to `working_iteration`. If an iteration becomes a stable version, promote it to `milestone_release`.

Use the local Stata batch command configured for the user's machine:

```bash
<STATA_BATCH_CMD> [file].do
```

## Required Declarations

Before presenting code, results, or final files, declare only what is relevant:

1. `Task family`: one route from the table above.
2. `Research stage`: `scratch_check`, `working_iteration`, or `milestone_release`.
3. `SOP`: the workflow choice and one-sentence rationale.
4. `References/examples`: which reference file or example family guided the work.
5. `Artifacts/logs`: what will be kept, where logs go, and what will be cleaned.

For identifier, date, or integration-heavy work, also declare the relevant policy briefly and load `references/data_patterns.md`. For formal milestone releases, include all material data, model, output, and artifact assumptions in the `.md`.

## Default SOP Choices

| Family | Default SOP |
|--------|-------------|
| `raw_vendor_import` | batch import -> header cleanup -> `Stkcd` normalization -> date conversion -> append same-schema files |
| `multi_source_integration` | frame workflow for 3+ distinct sources; merge for exactly 2 sources; append only for same-schema stacking |
| `indicator_construction` | verify panel keys -> construct variables -> validate -> save working dataset |
| `indicator_construction / event_study_car` | align event dates -> handle non-trading days -> calculate AR/CAR -> test/report |
| `regression_analysis` | check variables/panel -> run staged specs -> capture log/output -> record interpretable version |
| `descriptive_reporting` | sample check -> `tabstat`/`logout` -> group tests/correlations/figures as needed |
| `complete_workflow` | route each stage separately; avoid one monolithic snippet when data construction and analysis should be versioned |

## Progressive Disclosure

All paths below are relative to the skill root.

| Need | Read |
|------|------|
| import, CSMAR/Wind cleanup, `Stkcd`, dates, append/merge/frame | `references/data_patterns.md` |
| indicators, event study/CAR, regressions, descriptive output, tables, graphs | `references/analysis_patterns.md` |
| do-file roles, version iteration, hybrid data/regression code, log/output hygiene | `references/do_file_policy.md` |
| locating CSMAR, Wind, project, manual, or custom database paths | `references/paths.md` |
| schema gates, merge diagnostics, panel preflight, provenance, docx/xlsx companions | `references/advanced_layer.md` |

This public sharing version excludes project-specific `.do` examples. Use the references as the working pattern library.

## Stability Guardrails

- Normalize firm identifiers before joins, frames, panel setup, or regressions.
- Parse dates before extracting `year`, `month`, or event windows.
- Treat append, merge, and frame as distinct data-structure choices.
- Verify panel keys before lag operators, `xtset`, or `reghdfe`.
- Preserve diagnostics in kept logs for working iterations and milestone releases.
- Do not report numerical results unless they are verified from the current Stata log or exported output table.
- Put kept logs in the project log directory when one exists; do not leave `stata.log`, `stdin.log`, or other generic runtime logs as formal evidence.
- For chart/date-specific rules, load the relevant reference section instead of improvising.

## Dependencies

Install or check Stata user-written commands only when needed. Common commands include `reghdfe`, `ftools`, `outreg2`, `logout`, `winsor2`, and sometimes `ttable2`.

In an adapted skill, document the user's local Stata command, required Stata version, and package dependencies in the README or local setup notes.

## Version

SKMS v1.3.3 - 2026-05-05
