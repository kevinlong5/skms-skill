# SKMS Stata Skill

SKMS is a Stata skill for empirical accounting, finance, management, and auditing research workflows. It is designed as an agent-facing routing and SOP layer for:

- raw vendor data import and cleaning
- multi-source panel construction
- key-variable and financial-ratio construction
- event-study/CAR workflows
- fixed-effect, DID, and robustness regressions
- descriptive tables, figures, and reproducible outputs

## Sharing Version

This public repository is a clean sharing copy of the SKMS skill. Project-specific `.do` examples have been removed, and local paths have been replaced with placeholders such as `<DATA_ROOT>`.

This version is intended as a reusable template. Users should build their own `assets/` and example library from their own Stata `.do` files, then ask an agent to adapt the workflow references to their style.

## How To Build Your Own Stata Skill From This Repository

1. Clone or fork this repository.

2. Create an examples folder:

```text
assets/examples/
  README.md
  data_construction_*.do
  regression_*.do
  event_study_*.do
  descriptive_output_*.do
```

3. Select representative `.do` files from your own research workflow. A useful starting set is:

- raw database import and cleaning
- project dataset construction
- key-variable construction
- event-study or CAR analysis, if relevant
- regression/table output
- descriptive statistics or figure output

4. Clean the examples before adding them:

- remove sensitive data, credentials, and collaborator-sensitive paths
- replace local roots with placeholders such as `<DATA_ROOT>`, `<PROJECT_ROOT>`, and `<CUSTOM_DATA_ROOT>`
- replace machine-specific Stata commands or application paths with `<STATA_BATCH_CMD>`
- remove Stata version lines unless they are truly required for the user's project
- remove obsolete commented blocks that would confuse an agent
- keep enough code to reveal your Stata style and recurring patterns

5. Build `assets/examples/README.md` as an index. For each example, record:

- task family
- when an agent should read it
- key Stata patterns shown
- expected inputs and outputs
- any caveats about project-specific assumptions

6. Ask an agent to adapt the skill using your examples. A good prompt is:

```text
Read SKILL.md first, then read assets/examples/README.md.
Open only the smallest relevant .do examples.
Infer my Stata workflow style from these examples.
Update references/data_patterns.md, references/analysis_patterns.md,
references/paths.md, and references/do_file_policy.md as needed.
Keep SKILL.md compact as the trigger, SOP, and navigation layer.
Do not paste large code examples into SKILL.md.
```

7. Re-check the skill after adaptation:

```bash
ruby -ryaml -e 's=File.read("SKILL.md"); y=s.split(/^---\s*$/)[1]; YAML.safe_load(y); puts "yaml-ok"'
python3 -m json.tool evals/trigger-eval.json >/dev/null
```

Also run a local path and credential scan before making an adapted skill public.

8. Install the adapted skill into your agent environment as `skms` or another chosen skill name.

The main entry point is:

```text
SKILL.md
```

Detailed guidance is under:

```text
references/
```

## Usage

For Codex or Claude-style skill systems, place this repository's contents in the agent's skill directory as `skms`, then trigger it with requests involving Stata, `.do` files, empirical data construction, or regression workflows.

## Path Setup

Replace `<DATA_ROOT>` and other placeholders with your own local data roots before running generated Stata code.

Also define the local Stata batch command in your adapted README or setup notes. For example, map `<STATA_BATCH_CMD>` to the command that runs a `.do` file on your own machine.
