# Do-File, Log, and Artifact Policy

Use this reference when a task involves do-file placement, version iteration, hybrid data/regression code, log cleanup, or milestone outputs.

## 1. Do-file roles are defaults, not hard folders

| Role | Typical location | Purpose | Typical output |
|------|------------------|---------|----------------|
| `raw_data` | the relevant `<DATA_ROOT>/...` source directory | clean downloaded/vendor files and convert them to reusable `.dta` | reusable base `.dta`, import log |
| `data_construct` | project analysis/code directory, or an established project code directory | read shared DATA files, build key variables, merge/frame sources, create the project panel/sample | project `.dta`, construction log |
| `regression` | project analysis/code directory, or an established project code directory | run descriptive statistics, regressions, tables, and figures from a project dataset | result tables, figures, regression log |
| `hybrid_iteration` | existing project working code location | temporary overlap where exploratory regressions also add variables, merges, or data fixes | working `.do + .log`, optional note |

Real research often moves between these roles. Do not force a split during exploration. Instead, declare the current primary role and whether temporary construction logic should later be promoted into a `data_construct` file.

## 2. Placement policy

- Follow the user's existing project layout unless asked to standardize it.
- In structured research projects, default to:
  - code: `analysis/code/`
  - project data/intermediate data: `analysis/data/`
  - current results: `analysis/output/result/`
  - figures: `analysis/output/figures/`
  - logs and diagnostics: `analysis/output/logs/`
- Large raw or shared vendor data normally stay under `<DATA_ROOT>`; do not mirror them inside the project workspace unless needed.
- Raw-data do files may stay beside the data they clean when that is the established local convention.

## 3. Version iteration

- Set a project-specific Stata version line only when reproducibility requires it.
- Do not inherit another user's Stata version. Use the version required by the local project and installed Stata environment.
- Date-stamped and `v02/v03` filenames are acceptable for working research iterations.
- For non-trivial changes, add a short header note with: purpose, data sources, output targets, and the change from the previous version.
- Do not bulk rename or reorganize old archive files unless the user asks for project cleanup.

## 4. Log policy

Logs are evidence targets for agents, not presentation outputs.

- Numerical claims require evidence from the current `.log` or exported output table.
- Use meaningful log names such as `[basename].log` or `[basename]_diagnostics_YYYYMMDD.log`.
- Do not keep generic `stata.log`, `stdin.log`, or unnamed runtime logs as formal records. Delete them after a scratch check, or move/rename them if they are the only evidence for a kept run.
- For `scratch_check`, create a temporary log, read it, report the result, then delete it unless asked to keep it.
- For `working_iteration`, keep the descriptive `.do + .log`; short notes are optional.
- For `milestone_release`, keep the log as part of the reproducibility spine and point the result summary to the formal outputs.

## 5. Milestone output contract

A milestone is a reproducible evidence chain plus formal research outputs.

Minimum reproducibility spine:

```text
[basename].do
[basename].log
[basename]_summary.md
```

Formal outputs as needed:

```text
[basename]_panel.dta
[basename]_tables.xlsx
[basename]_regs.doc or [basename]_regs.docx
[basename]_fig*.png
```

The `.md` file should be a concise result index: objective, data sources, command used, output files, numeric evidence sources, and decision notes. It does not need to become a long report when the main presentation outputs are Word/Excel tables.

## 6. Multi-agent handoff

When Codex and Claude may both touch the project:

- Record the main current do-file, output directory, and log directory in the project index when one exists.
- Do not leave results only in conversation; write or update the relevant result summary or project note for useful working iterations and milestones.
- On handoff, state which artifacts are current and which generic logs or temp files were cleaned.
