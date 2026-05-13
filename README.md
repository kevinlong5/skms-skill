# SKMS Stata Skill

Language: [中文](#中文) | [English](#english)

---

## 中文

SKMS 是一个面向 Stata 实证研究工作流的 agent skill 模板，适用于会计、金融、管理、审计等领域的经验研究。它不是完整项目模板，而是帮助 agent 在 Stata 任务中进行触发、路由、SOP 选择和渐进式资料读取。

主要覆盖：

- 原始数据库导入与清洗
- 多源面板数据构建
- 关键变量与财务指标构造
- 事件研究与 CAR
- 固定效应、DID 与稳健性回归
- 描述统计、表格、图形与可复现输出

### 分享版说明

这个仓库是 SKMS 的公开分享模板。项目特定的 `.do` 示例已经移除，本地路径也已经替换为 `<DATA_ROOT>`、`<PROJECT_ROOT>`、`<CUSTOM_DATA_ROOT>`、`<STATA_BATCH_CMD>` 等占位符。

这个版本的目标不是直接复制某个人的 Stata 风格，而是帮助使用者基于自己的 `.do` 文件构建自己的 Stata skill。

### 如何基于本仓库构建自己的 Stata Skill

1. 克隆或 fork 本仓库。

2. 创建示例目录：

```text
assets/examples/
  README.md
  data_construction_*.do
  regression_*.do
  event_study_*.do
  descriptive_output_*.do
```

3. 从自己的研究工作流中挑选有代表性的 `.do` 文件。建议至少包括：

- 原始数据库导入与清洗
- 项目数据集构建
- 关键变量构造
- 事件研究或 CAR 分析，如适用
- 回归与表格输出
- 描述统计或图形输出

4. 放入示例前先清理这些 `.do` 文件：

- 删除敏感数据、凭据和合作者敏感路径
- 将本地路径替换为 `<DATA_ROOT>`、`<PROJECT_ROOT>`、`<CUSTOM_DATA_ROOT>`
- 将本机 Stata 命令或应用路径替换为 `<STATA_BATCH_CMD>`
- 删除不必要的 Stata 版本行，除非项目复现确实需要
- 删除会误导 agent 的过期注释块
- 保留足够代码，让 agent 能识别你的 Stata 风格和重复模式

5. 编写 `assets/examples/README.md` 索引。每个示例建议记录：

- task family
- agent 什么时候应该读取它
- 展示了哪些关键 Stata 模式
- 输入与输出
- 项目特定假设或注意事项

6. 让 agent 根据你的示例改造 skill。推荐提示词：

```text
Read SKILL.md first, then read assets/examples/README.md.
Open only the smallest relevant .do examples.
Infer my Stata workflow style from these examples.
Update references/data_patterns.md, references/analysis_patterns.md,
references/paths.md, and references/do_file_policy.md as needed.
Keep SKILL.md compact as the trigger, SOP, and navigation layer.
Do not paste large code examples into SKILL.md.
```

7. 改造后做基本检查：

```bash
ruby -ryaml -e 's=File.read("SKILL.md"); y=s.split(/^---\s*$/)[1]; YAML.safe_load(y); puts "yaml-ok"'
python3 -m json.tool evals/trigger-eval.json >/dev/null
```

公开改造后的 skill 前，还应扫描本地路径和凭据残留。

8. 将改造后的 skill 安装到自己的 agent 环境中，命名为 `skms` 或其他名称。

### 入口文件

```text
SKILL.md
```

详细规则位于：

```text
references/
```

### 路径与运行命令

运行生成的 Stata 代码前，请将占位符替换为自己的本地配置：

- `<DATA_ROOT>`：共享数据库根目录
- `<PROJECT_ROOT>`：项目根目录
- `<CUSTOM_DATA_ROOT>`：手工或自建数据库根目录
- `<STATA_BATCH_CMD>`：本机运行 Stata `.do` 文件的批处理命令

---

## English

SKMS is an agent skill template for empirical research workflows in Stata, especially in accounting, finance, management, and auditing research. It is not a full research-project scaffold. Its purpose is to help an agent trigger the right workflow, route the task, select an SOP, and load detailed references progressively.

It covers:

- raw database import and cleaning
- multi-source panel construction
- key-variable and financial-ratio construction
- event-study and CAR workflows
- fixed-effect, DID, and robustness regressions
- descriptive tables, figures, and reproducible outputs

### Sharing Version

This repository is a public sharing template for SKMS. Project-specific `.do` examples have been removed, and local paths have been replaced with placeholders such as `<DATA_ROOT>`, `<PROJECT_ROOT>`, `<CUSTOM_DATA_ROOT>`, and `<STATA_BATCH_CMD>`.

This version is not meant to copy one person's Stata style. It is meant to help users build their own Stata skill from their own `.do` files.

### How To Build Your Own Stata Skill From This Repository

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
- regression and table output
- descriptive statistics or figure output

4. Clean the examples before adding them:

- remove sensitive data, credentials, and collaborator-sensitive paths
- replace local roots with `<DATA_ROOT>`, `<PROJECT_ROOT>`, and `<CUSTOM_DATA_ROOT>`
- replace machine-specific Stata commands or application paths with `<STATA_BATCH_CMD>`
- remove Stata version lines unless they are required for reproducibility
- remove obsolete commented blocks that would confuse an agent
- keep enough code to reveal your Stata style and recurring patterns

5. Build `assets/examples/README.md` as an index. For each example, record:

- task family
- when an agent should read it
- key Stata patterns shown
- expected inputs and outputs
- caveats about project-specific assumptions

6. Ask an agent to adapt the skill using your examples. A useful prompt is:

```text
Read SKILL.md first, then read assets/examples/README.md.
Open only the smallest relevant .do examples.
Infer my Stata workflow style from these examples.
Update references/data_patterns.md, references/analysis_patterns.md,
references/paths.md, and references/do_file_policy.md as needed.
Keep SKILL.md compact as the trigger, SOP, and navigation layer.
Do not paste large code examples into SKILL.md.
```

7. Re-check the adapted skill:

```bash
ruby -ryaml -e 's=File.read("SKILL.md"); y=s.split(/^---\s*$/)[1]; YAML.safe_load(y); puts "yaml-ok"'
python3 -m json.tool evals/trigger-eval.json >/dev/null
```

Before making an adapted skill public, also scan for local paths and credential remnants.

8. Install the adapted skill into your agent environment as `skms` or another chosen skill name.

### Entry Point

```text
SKILL.md
```

Detailed rules are under:

```text
references/
```

### Paths And Runtime Command

Before running generated Stata code, replace placeholders with your own local configuration:

- `<DATA_ROOT>`: root directory for shared databases
- `<PROJECT_ROOT>`: project root directory
- `<CUSTOM_DATA_ROOT>`: root directory for manual or custom datasets
- `<STATA_BATCH_CMD>`: local batch command for running a Stata `.do` file
