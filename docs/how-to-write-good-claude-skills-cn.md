---
title: "How to write good Claude skills"
source:
  - "Introduction to Claude Skills.md (Anthropic Cookbook)"
  - "Skill authoring best practices.md (Anthropic official checklist)"
created: 2026-06-11
updated: 2026-06-11
tags:
  - "skills"
  - "claude"
  - "best-practices"
description: "Synthesis of two Anthropic sources (Cookbook + official Best Practices checklist) into actionable writing rules and a share-ready review checklist. Uses [S]/[E]/[CK]/[F] tags throughout. Result of adversarial-review + debate across three subagents. 中文版为 source of truth，英文译本位于 docs/how-to-write-good-claude-skills.md。"
---

# How to write good Claude skills

本准则系作者对 `Introduction to Claude Skills.md`（Anthropic 官方 Cookbook, 全 836 行）反复精读三遍后，所作的体系化提炼与诠释。

## 阅读说明

文中每一条论断均带标签，**便于读者即时辨识其来源属性**：

- `[S]`（Source）= 来自原典的直接陈述或强暗示（附行号）
- `[E]`（Extension）= 作者基于原典精神的合理延伸
- `[CK]`（Common Knowledge）= 通识性补充（虽未见于原典，但属工程领域普遍共识）
- `[F]`（Fabrication）= 缺乏原典依据的凭空捏造（v1 版本曾出现，v2 已悉数删除）

**分节色彩**：第一遍几乎全是 `[S]`；第二遍混合 `[S]` 与 `[E]`；第三遍几乎全是 `[E]/[CK]`；第四遍为纯作者综合，**并非原典摘要**。

---

## 第一遍：原典骨架（事实层）

原典全文围绕五个核心命题展开（**以下均属 `[S]` 类论断**）：

1. **Skills 的本质** —— `[S]` 原文将其界定为 "a package of instructions, executable code, and resources"（line 182），并强调其定位高于单一 tool（line 190），本质上是一份"领域专长的封装"（"expertise package"，line 182）。
2. **三级加载模型** —— `[S]`（line 209-211）：
   - **Tier 1**：YAML frontmatter 的 name（≤64 字符）与 description（≤1024 字符），常驻可见。
   - **Tier 2**：完整指令（< 5k tokens），仅在判定相关时方加载。
   - **Tier 3**：链接的辅助文件，按需再加载。
3. **两类类型 + 一项关键性质** —— `[S]`：
   - **两类类型**（line 220-222 表格）：Anthropic 托管型（xlsx/pptx/pdf/docx）与用户自定义型（用户工作流）。
   - **一项关键性质**：Composable —— 多个 skill 可协同工作（line 203, 233）。须留意：原典将"组合"列于 Benefits 而非 Types 之下。
4. **使用约束** —— `[S]`（line 241-253, 265-272）：必须调用 `client.beta.messages.create()`，必须搭配 `code_execution_2025-08-25` 工具，`betas` 数组须含 `skills-2025-10-02`。
5. **Token 经济性** —— `[S]`（line 278-289）：
   - 手动指令 5,000–10,000 tokens/请求（line 280）。
   - Skill（仅 metadata）：几乎免费（line 281）。
   - Skill（完整加载）：~5,000 tokens/触发（line 282）。
   - 实例佐证：完整解释 Excel 全功能需 ~8,000 tokens；改用 skill 后，仅在调用时支付 ~5,000 tokens（line 288-289）。

---

## 第二遍：原文硬约束 + 作者推断

### 硬约束（原典明确给出的）

- `[S]` name ≤ 64 字符，description ≤ 1024 字符（line 209）。
- `[S]` 主体指令 < 5,000 tokens（line 210）。
- `[S]` 用 `"latest"` 拉取 Anthropic 托管型 skill；自定义型 skill 须以 epoch 时间戳版本化（line 370-373）。
- `[S]` helper scripts 是 skill 核心价值之一：已测试、可直接使用的代码（line 198）。
- `[S]` Skills 远不止一个 prompt——它有机融合指令、代码与资源（line 190）。

### 推断（作者外推，原典未明示但精神一致）

> **注**：以下条目以 `[E]` 标记者，系从原典精神中提炼的合理推断，**并非原典明文的硬性规则**。

- `[E]` description 应被视作"路由信号"——它实质上决定了 Claude 何时加载完整内容。原典 line 209 仅言"Claude sees name + description"，但既然 description 是 Claude 唯一能常驻感知的元数据，将其写成"触发条件"远胜于写成"功能描述"，对自动加载命中率影响最大。
  - `[CK]` 不宜过宽：反例如 "A useful skill for various tasks"。
  - `[CK]` 不宜过窄：反例如 "Only for Q3 sales reports"。

- `[E]` < 5k tokens 虽是硬约束，但更关键的是**预算的投向**：
  - `[E]` 默认 Claude 已掌握基础知识（不必赘述 JSON/markdown）。
  - `[E]` 给出可执行判断规则（"if X, do Y"），而非抽象建议。
  - `[E]` 嵌入可直接复用的 code snippet；仅描述"用 XX 库可达成"实属 token 浪费。

- `[E]` helper scripts 的工程惯例：
  - `[CK]` 高频复用的判断/转换逻辑，依惯例置于 `scripts/` 之下。
  - `[CK]` 复杂多步流程宜封装为单一入口命令。
  - `[E]` 原则：5 行 shell 可毕之事，不应让 Claude 每轮重写（此条系作者对 line 198 "tested, working code" 的释读）。

- `[E]` 可组合性的工程含义：编写 skill 时应预设其将与其它 skill 共存——frontmatter 不写绝对路径，不预设环境内独此一家（原典 line 233 仅指明 "composability" 为性质，并未给出"如何实现"的具体建议）。

### 补充硬约束（源自官方 Best Practices Checklist）

> **注**：以下约束出自 Anthropic 官方 [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)（56 行 checklist），**并非**上述 Cookbook（836 行 Introduction）的内容。两份文档同属一个知识体系，互为补充，均标记为 `[S]`。

**核心质量：**

- `[S]` **SKILL.md body < 500 行。** 与 Cookbook 中 < 5k tokens 的约束彼此独立——前者关乎可维护性（长文件难审难改），后者关乎 API 加载预算。~250–400 行通常已足以覆盖一个聚焦领域。
- `[S]` **文件引用不超过一层深度。** 引用 `scripts/validate.py` 即可，切勿引用 `tools/helpers/submodules/checker.py`。深层路径在打包与迁移时极易断裂。
- `[S]` **不包含时效性信息。** API 版本号、特定日期、临时方案皆属此列。一旦出现 "as of June 2025" 之类措辞，相关信息已或即将过时。对时效性内容：或删之，或专设 `## Old patterns` 小节并标注有效期。
- `[S]` **全文术语一致。** 同一概念在 description、SKILL.md body、脚本注释、错误信息中须使用同一套名词。切莫在 description 中称 "processor"，于脚本中又称 "engine"。
- `[S]` **示例须具体、可运行。** 绝非 "Create a chart"，而是 "Create a bar chart showing monthly revenue for Q1 2025 with data labels on each bar"。理想状态下，示例应可直接复制为 prompt。
- `[S]` **工作流须有明确步骤。** 多步指令不可用连续段落堆砌——须采用编号列表、子标题或决策树。每一步只承载一个动作。此点与 Cookbook 精神一脉相承，Checklist 进一步将其升格为硬性要求。

**代码与脚本：**

- `[S]` **无"巫术常量"——每个值皆有来由。** 凡非自解释之数字、字符串、路径，皆须附注释说明其取值理由。反例：`timeout=30`；正例：`timeout=30  # max seconds for PDF render; longer runs indicate resource leak`。
- `[S]` **所需依赖须在指令中列出并经确认可用。** SKILL.md 应设有 Prerequisites 小节，枚举全部依赖（Python 包、系统工具、API key），并在脚本入口以 `try/except` 验证；遇阻时返回安装指引，而非静默跳过。
- `[S]` **脚本须有清晰文档。** 每个脚本文件自身应包含：一句话说明、参数列表、返回值/退出码、一至两个使用示例。文档就栖于脚本之内，不必在 SKILL.md 中重复。
- `[S]` **无 Windows 风格路径（全用正斜杠）。** `scripts\tools\check.py` 中的反斜杠，在 Claude 的 Linux 执行环境中将被解析为转义字符，立即报错。统一使用 `scripts/check.py`。
- `[S]` **关键操作须有验证/确认步骤。** 写入后回读校验、计算后作 sanity check、网络请求后检查状态码。对于不可逆操作（删除、覆盖），须于执行前显式确认。
- `[S]` **质量关键任务须含反馈循环。** 若 skill 输出需借人工判断（文档生成、代码审查），应内嵌自我评估步骤：Claude 生成后自行检查，指出改进点，再行交付。

**测试与评估：**

- `[S]` **至少 3 个评估用例。** 分别覆盖：(1) 最典型场景，(2) 边界情形（空输入、超大输入），(3) 错误路径（依赖缺失、配置无效）。
- `[S]` **经 Haiku、Sonnet、Opus 三档测试。** 不同模型在指令遵从、输出长度、错误模式上行为各异。最低要求：至少覆盖一档 fast（Haiku）与一档 capable（Sonnet/Opus）。
- `[S]` **以真实使用场景测试。** 测试用例不可止于"理想化 prompt"。应采用真实用户的措辞、真实的文件格式、真实的错误数据。如有可能，让目标用户亲试并观察其卡点。
- `[S]` **纳入团队反馈（如适用）。** 多人共用之 skill 须经至少一位独立人士审查——非代码审查，而是"此人是否理解 skill 应于何时启用、如何调用"。

**关于测试标准的说明：** 既有的"第三遍对照表"（line 94）所提"至少一个端到端测试"（`[E]`），仅构成**个人使用**之最低门槛。Checklist 要求的 3+ 评估用例 + 多模型 + 真实场景测试，则属**分享前**的推荐标准。二者并不抵牾——它们分别对应不同场景的不同要求。建议分级如下：

- 个人使用：≥ 1 个端到端测试。
- 团队内共享：≥ 3 个评估用例，覆盖典型 + 边界 + 错误路径。
- 公开/分发：3+ 评估用例 + 多模型测试 + 团队审查。

---

## 第三遍：作者综合（工程经验，非原典）

> **注**：下表每格皆以标签辨明来源。**所有 `[E]/[CK]` 条目均为作者之工程外推，并非 Cookbook 原典**。原典明确给出的硬约束仅有：name ≤ 64 字符、description ≤ 1024 字符、完整指令 < 5k tokens、调用 `"latest"` 而非固定版本。

| 维度 | ✅ 好 skill | ❌ 坏 skill |
| --- | --- | --- |
| **YAML description** | `[E]` 一句话点出"于 X 场景下做 Y"，含明确触发信号 | `[E]` "A useful skill for various tasks"（空泛） |
| **指令长度** | `[S]` < 5k tokens；`[E]` 每段皆为判断规则或代码 | `[E]` 将整个领域教程塞入；或反向 < 1k tokens 而未明言"何时用" |
| **辅助脚本** | `[S]` 含已测试可用的 helper 代码；`[CK]` 依惯例置于 `scripts/` 下 | `[E]` "运行 shell 命令获取 X"——Claude 每轮重新拼接 |
| **路径处理** | `[CK]` 采用 `${WIKIBASE_ROOT}` 一类占位符，于部署时再行展开 | `[CK]` 硬编码 `/home/user1/project/...`——换一个用户即崩 |
| **错误处理** | `[E]` 快速 fail，错误信息直指具体原因 | `[E]` 吞咽异常并返回空字符串，或抛出 "An error occurred" |
| **可组合性** | `[E]` 不读取他人全局状态，不独占文件，不依赖 cwd | `[E]` 预设自身为唯一进程，要求独占 env var |
| **测试** | `[E]` 至少一项端到端 bash/python 测试 | `[E]` 无任何测试，仅以"应该能跑"作结 |
| **frontmatter 元数据** | `[S]` 必含 name + description；`[CK]` 可选 version 字段 | `[E]` 仅有 description 而无 name；或 name 以下划线拼接版本号 |
| **示例** | `[S]` 原典同时提供 "Simple (1-2 lines)" 与 "Detailed" 两组示例（line 379, 390） | `[E]` 仅给一条 prompt，或根本无具体示例 |
| **边界条件** | `[S]` Python 3.8+ 为先决条件（line 21）；`[E]` 显式声明"假设 X 状态" | `[E]` 先决条件未予声明 |

### 三处最易写坏的地方

1. **"在 prompt 中写得很漂亮的指令" ≠ skill** —— `[S]` 前者系消耗品（line 280 手动指令 5k–10k），后者系 metadata-级常驻（line 281）。`[E]` 所谓"最大的诱惑"，系作者特意强调：若将一份上佳 prompt 直接复制为 SKILL.md，便会丧失渐进披露所附带的成本优势。

2. **YAML description 写成"功能描述"而非"触发条件"** —— `[E]` 原典 line 209 仅述及 description 是 Claude 所见之元数据，并无"触发条件"的明文措辞；此条经验系作者之工程外推，却对自动加载命中率影响最深。
   - `[CK]` 反例对照：`"helps with PDF generation"`（过宽） vs `"creates branded PDF reports from markdown (use when user asks for .pdf, weekly-report, or formal-document)"`（含触发信号）。

3. **辅助脚本被当作"示例"嵌入 SKILL.md** —— `[E]` 5k tokens 的预算确凿无疑（line 210），但"不宜内联"则系作者推断。建议 SKILL.md 中仅引用脚本路径，**不内联**完整代码。

---

## 作者综合（并非原典摘要）

> **重要**：以下公式系作者将 Cookbook 精神与自身工程经验凝炼而成的一句话，**非原典所明示**。原典仅言 description 是 Claude 所见之元数据、指令 < 5k tokens、helper scripts 已测试可用。

> **好 skill = 精准的触发信号（description）+ 紧凑的判断规则（指令）+ 可重用的工具（脚本）。**

三者中任何一个写砸了，skill 就从"专家"退化成"冗余的 prompt"——这条推论来自 line 280-282 的 token 对比。
## 第五遍：将官方 Checklist 转化为审查框架

> 本节内容取材自 Anthropic 官方 [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)（56 行 checklist）。所有条目以 `[S]` 标记者系原典，以 `[E]` 标记者系作者之外推解读。

### 使用方式

本审查表的使用时机，应是 skill **写毕之后**、**分享之前**——而非在写作过程中逐条对照。流程如下：

1. 依前四节所立准则，起草 SKILL.md。
2. 行将分享前，对照本表逐条核查。
3. 对每一条记录 ✅ 通过 / ❌ 未通过 / N/A 不适用。未通过者，务先修复而后分享。

### 核心质量审查表

| # | 条目 | 解读 |
|---|------|------|
| 1 | `[S]` Description 须含具体关键术语 | 不可 "handles data processing"——须 "parses CSV/JSON/XML logs with schema inference (use when user asks about log analysis, data cleaning, or format conversion)"。关键术语即意图匹配之路由信号。 |
| 2 | `[S]` Description 须同时阐明"做什么"与"何时用" | 功能范围与触发条件兼陈。反例："Creates PDF reports"（仅言"做什么"）。正例："Creates branded PDF reports from markdown — use when .pdf, weekly-report, invoice, formal-document"。 |
| 3 | `[S]` SKILL.md body < 500 行 | 与 < 5k tokens 系彼此独立之二约束。前者关乎可维护性，后者关乎 API 预算。~250–400 行通常已足以承载一个聚焦领域。 |
| 4 | `[S]` 细节另置于独立辅助文件 | 模板、配置、图片、长文本参考皆属此列。SKILL.md 仅保留"指挥逻辑"。 |
| 5 | `[S]` 不含时效性信息（或仅置于 "old patterns" 分区） | 一旦出现 "as of June 2025" 之类措辞，相关信息已或将过时。删之，或专设 `## Old patterns` 并标注有效期。 |
| 6 | `[S]` 全文术语一致 | description / SKILL.md / 脚本 / 错误信息，须共用同一套名词。 |
| 7 | `[S]` 示例须具体、可运行 | 抽象："Create a chart"。具体："Create a bar chart showing monthly revenue for Q1 2025 with data labels on each bar"。理想情形下，示例可直接复制为 prompt。 |
| 8 | `[S]` 文件引用不超过一层深度 | 引用 `scripts/validate.py` 即可，切莫引用 `tools/helpers/submodules/checker.py`。深层路径于迁移时极易断裂。 |
| 9 | `[S]` 渐进披露之运用须恰到好处 | 已在第一遍"三级加载模型"中铺陈。 |
| 10 | `[S]` 工作流须有清晰步骤 | 采编号列表 / 子标题 / 决策树等结构化形式。每一步仅承载一个动作。 |

### 代码与脚本审查表

| # | 条目 | 解读 |
|---|------|------|
| 11 | `[S]` 脚本须真正解决问题，不可将问题推回 Claude | `[E]` 若脚本仅止于调用 Claude API，则不称 helper script，乃 proxy 而已。Helper 真正的价值，恰在于承担 Claude 所不擅之事：精确计算、文件 I/O、格式校验。 |
| 12 | `[S]` 错误处理须显式且有效 | `[E]` 远不止"快速 fail"。错误信息应囊括三要素：(1) 何处出错，(2) 为何出错，(3) 如何修复。正例：`Error parsing config.yaml at line 12: expected 'key: value' but found '- item' (YAML list not valid here). Check indentation.` |
| 13 | `[S]` 无"巫术常量"——每一取值皆须有据 | `[E]` 任何数字 / 字符串 / 路径皆须可被解释。反例：`timeout=30`。正例：`timeout=30  # max seconds for PDF render; longer runs indicate resource leak`。 |
| 14 | `[S]` 依赖须列出且经确认可用 | `[E]` SKILL.md 中设 Prerequisites 小节以枚举全部依赖。脚本入口以 `try/except` 验证，遇阻返回安装指引（而非静默跳过）。 |
| 15 | `[S]` 脚本须附清晰文档 | 文档栖于脚本文件自身，须含：(1) 一句话说明，(2) 参数列表，(3) 返回值/退出码，(4) 1–2 个使用示例。SKILL.md 中毋须复述。 |
| 16 | `[S]` 摒弃 Windows 风格路径——一律正斜杠 | `[CK]` 反斜杠在 Claude 的 Linux 环境中即为转义字符。`scripts\tools\check.py`→`scripts/check.py`。此点与 `${WIKIBASE_ROOT}` 占位符系**不同**之可移植性关切。 |
| 17 | `[S]` 关键操作须有验证/确认步骤 | 写入后回读校验、计算后作 sanity check、网络请求后检查状态码。不可逆操作（删除、覆盖）于执行前显式请求确认。 |
| 18 | `[S]` 质量关键任务须内嵌反馈循环 | 若输出须借人工判断（文档生成、代码审查），应设自我评估步骤：Claude 生成后自检并指出改进点，方行交付。 |

### 测试审查表

| # | 条目 | 解读 |
|---|------|------|
| 19 | `[S]` ≥ 3 个评估用例 | `[E]` 须分别覆盖：(1) 最典型场景，(2) 边界情形，(3) 错误路径。单一用例仅能证明"曾跑通一次"。 |
| 20 | `[S]` 经 Haiku、Sonnet、Opus 三档测试 | `[E]` 各模型于指令遵从、输出长度、错误模式上行为各异。至少须覆盖一档 fast（Haiku）与一档 capable（Sonnet/Opus）。 |
| 21 | `[S]` 须以真实使用场景测试 | 不可止于"理想化 prompt"。应采用真实用户之措辞、真实之文件格式、真实之错误数据。让目标用户亲试并观察其卡点。 |
| 22 | `[S]` 须纳入团队反馈（如适用） | 至少须有一位独立人士进行审查——非代码层面的审查，而是"此人是否理解 skill 应于何时启用、如何调用"。 |

### 审批门禁建议

| 分享范围 | 最低要求 |
|----------|----------|
| 个人使用 | 第一至四节之写作准则 + ≥ 1 个端到端测试 |
| 团队内共享 | 上项 + 核心质量审查表（10 项全数通过）+ ≥ 3 个评估用例 |
| 公开 / 分发 | 上项 + 全 22 项审查表全数通过 + 多模型测试 + 团队审查 |


## 速查：哪些是"原典"，哪些是"作者"

| 类别 | 内容 | 状态 |
| --- | --- | --- |
| 三级加载模型的数字 | name ≤64, desc ≤1024, <5k tokens | `[S]` Cookbook |
| Skill 的定义与定位 | 比单 tool 高一档 | `[S]` Cookbook |
| Token 经济对比 | 5k–10k vs metadata-only | `[S]` Cookbook |
| 版本策略 | `"latest"` vs epoch | `[S]` Cookbook |
| 三类 Beta API 标识符 | `code-execution-...`, `files-api-...`, `skills-...` | `[S]` Cookbook |
| SKILL.md < 500 行 | 行数约束（独立于 token 约束） | `[S]` Checklist |
| 文件引用一层深 | 路径可维护性 | `[S]` Checklist |
| 无时效性信息 | API 版本/日期/临时方案放到 old patterns | `[S]` Checklist |
| 术语一致 | 全文用同一套名词 | `[S]` Checklist |
| 巫术常量 | 所有值必须有理由 | `[S]` Checklist |
| 依赖列出且验证 | Prerequisites + 启动时自检 | `[S]` Checklist |
| 无 Windows 路径 | 全正斜杠 | `[S]` Checklist |
| 关键操作验证 | 文件写入后回读校验 | `[S]` Checklist |
| 反馈循环 | 质量关键任务自检 | `[S]` Checklist |
| ≥3 评估用例 | 典型+边界+错误 | `[S]` Checklist |
| 多模型测试 | Haiku/Sonnet/Opus 全覆盖 | `[S]` Checklist |
| 真实场景测试 | 不写理想化 prompt | `[S]` Checklist |
| 团队反馈 | 至少一人独立审查 | `[S]` Checklist |
| 把 description 写成"触发条件" | 路由信号的工程化解读 | `[E]` 作者外推 |
| 指令花在哪 | "判断规则" vs "背景知识" | `[E]` 作者外推 |


---

## 第六遍：与官方 Best Practices 的交叉验证（写毕之后回看）

> **重要**：本节系 v4 补充的"自我审查"环节——将作者第三遍的速记结论 **回灌** 至官方 checklist 之侧，以观其一致性。所有 `[E]/[CK]` 条目皆系作者之推断，**并非原典所明文给出的硬约束**。

### A vs B 的定位差异（已验证的共识）

| 维度 | A 官方 Best Practices（1112 行） | B 本文档（综合 + 三遍总结） |
|---|---|---|
| 文档形态 | 规范式 checklist（22 项） | 叙事式总结（5 遍 + 标签系统） |
| description 的角色 | "what + when"（功能 + 触发场景） | **路由信号**（于上百个 skill 中决定被正确选中） |
| 起点方法 | Test with all models you'll deploy to | 先建 evals，再写文档（eval-driven） |
| 迭代循环 | Claude A 设计与 Claude B 使用之间的双向循环 | 同一循环，更强调"基于观察，而非假设" |
| 错误处理粒度 | "solve, don't punt" | 三要素：何错 + 为何 + 如何修（细化） |
| 脚本工程惯例 | 置于 skill 内、以 `scripts/` 为子目录 | 同上（`[CK]` 通识） |
| 可组合性 | 未单列 | 显式提出：不读全局状态、不锁独占文件、不假设 cwd |
| 时效性信息 | 归入 `## Old patterns` 分区 | 同上 |

**一句话判断**：A 是"Anthropic 教你写 skill 的方法论"，B 是"消化过 Anthropic 之后凝炼为一句话的速记"。二者于同一知识体系下互为补充，**并不矛盾**。

### 27 条特征：22 项 A + 5 项 B 独有

#### 22 项 A checklist（已于第五遍详列，此处不赘）

#### B 独有的 5 项特征（v4 标记为 F-23..F-27）

| # | 名称 | 标签 | 价值 |
|---|---|---|---|
| F-23 | description 是路由信号 | `[E]` 关键贡献 | 影响自动加载命中率，最易写错之一 |
| F-24 | 路径以占位符表达（`${WIKIBASE_ROOT}`） | `[CK]` | 部署时再行展开，避免硬编码 |
| F-25 | 可组合性：不假设独占环境 | `[E]` | skill 与上百同类并存，独占 env var 必致崩溃 |
| F-26 | 测试分级：个人 / 团队 / 公开 三档 | `[E]` | 不同分享范围对应不同最低要求 |
| F-27 | 错误信息三要素：何错 + 为何 + 如何修 | `[E]` 细化 A #12 | 错误信息乃可执行信号，非 yes/no |

### 21 个反模式：9 项 A + 8 项 B + 3 项作者之观察

#### 9 项 A 明确 anti-patterns（已于 A 文档中列示，此处不赘）

#### B 独有的 8 项反模式（v4 标记为 AP-10..AP-17）

| # | 反模式 | 严重度 | 违反之核心原则 |
|---|---|---|---|
| AP-10 | "好 prompt ≠ skill"：将好 prompt 直接复制为 SKILL.md | Critical | 丢失 progressive disclosure 优势 |
| AP-11 | description 过宽："helps with PDF generation" | Critical | 全场景覆盖 = 全场景不覆盖 |
| AP-12 | description 过窄："Only for Q3 sales reports" | Critical | 命中条件过严 = 几乎不触发 |
| AP-13 | 将整个领域教程塞入 skill | Important | SKILL.md 是规则之集，非百科全书 |
| AP-14 | 将 helper script 视为"示例"内联进 SKILL.md | Important | 浪费 5k tokens 预算 |
| AP-15 | 吞咽异常并返回空字符串（`except: return ""`） | Critical | 错误信息须可 actionable |
| AP-16 | 预设自身为唯一进程（独占 env var / 全局状态） | Critical | 可组合性之违反 |
| AP-17 | 无任何测试，仅以"应该能跑"作结 | 视场景 | "should work" 非验证标准 |

#### 作者之 3 项观察（A 与 B 皆未明言，然于实践中常见）

| # | 反模式 | 严重度 | 说明 |
|---|---|---|---|
| AP-18 | 混用 Markdown 列表层级 | Minor | `- [ ]` 与 `1.` 混用，致 Claude 解析困惑 |
| AP-19 | frontmatter 缺失 description 或 description 为空 | Critical | skill 不会现身于可用列表（静默失败） |
| AP-20 | 示例与正文之语言/场景不一致 | Important | 触发场景与示例失对齐，路由信号失效 |

### 一句话压缩（前文已述，此处复述以闭环）

> **好 skill = 触发信号（description）+ 判断规则（指令）+ 可重用的工具（脚本）。**

反向观之，**坏 skill 的共同点仅有一桩**：作者将"我想说什么"误作"Claude 需要什么"来写——故冗长、模糊、堆砌背景、缺判断规则、缺可重用工件。

### 速查：A 独有 / B 独有 / 共有

| 类别 | 内容 | 来源 |
|---|---|---|
| 22 项 Checklist | progressive disclosure / 一层引用 / 巫术常量 / 多模型测试 … | A 独有（规范级） |
| `[S]/[E]/[CK]/[F]` 标签系统 | 区分原典 / 外推 / 通识 / 捏造 | B 独有（方法论） |
| "好 skill = 触发信号 + 判断规则 + 工具" | 凝为一句话 | B 综合（作者） |
| F-23..F-27 | description 是路由信号 / 占位符路径 / 可组合性 / 测试分级 / 错误三要素 | B 综合（作者） |
| AP-10..AP-17 | 8 项 B 独有反模式 | B 综合（作者） |
| AP-18..AP-20 | 作者之 3 项观察 | 综合（审计实践） |
| 三级加载模型（Tier 1/2/3） | 渐进披露之精确数字 | A + B 共有 |
| Eval-driven development | 先建 evals，再写文档 | A + B 共有 |
| Claude A ↔ Claude B 迭代循环 | 观察—修正—再观察 | A + B 共有 |
| "description 是路由信号" | 对 description 的再定义 | B 关键贡献（基于 A 精神而更精确） |



## 第七遍：补遗（修订 v6，2026-06-11）

> **缘起**：本节系基于与两份原典（《Introduction to Claude Skills.md》、《Skill authoring best practices.md》）的逐节比对，归纳前六遍未及覆盖之 10 项要点，依优先级补入。标签体系沿用 `[S]/[E]/[CK]/[F]`。

### 7.1 Skills 与工具、子代理的层级关系

> **[S]** 原典 `Introduction to Claude Skills.md` line 188-196 明确指出，Skills 高于单 tool、却低于完整 multi-agent system；其与 MCPs（Model Context Protocol）相对独立。

**抽象层次（由低至高）：**

| 层级 | 概念 | 核心定位 | 何时启用 |
|---|---|---|---|
| L0 | **单 Tool** | 单一原子能力（如 `bash`、`pdf_read`）| 颗粒度最小，须由模型按需组合 |
| L1 | **MCP Server** | 一组相关 tool 的标准协议封装 | 跨会话 / 跨工作流复用同一批 tool |
| L2 | **Skill**（本文主角） | 指令 + 已测试代码 + 资源之封装 | 解决某一**特定工作流**的可复用方案 |
| L3 | **Subagent** | 拥有独立上下文窗口与子任务调度的代理 | 长链任务，须隔离上下文 |
| L4 | **Multi-agent System** | 多 subagent 协同（含路由、状态共享）| 跨域复杂问题 |

**关键澄清：**
- `[CK]` **Skill ≠ Tool 的简单组合**——Skill 附带了"何时启用 / 如何决策 / 验证结果"的整套指令，tool 本身只是无状态的原子能力。
- `[CK]` **Skill 不替代 Subagent**——若任务须长链推理且主上下文易被淹没，应提升至 L3 而非堆叠更多 skill。
- `[S]` 多个 skill 协同工作时（line 198-200），仍由**单一 agent** 调度；这与 L3-L4 的"独立上下文"截然不同。

> **判别口诀（作者综合）：** 一句话能答清"做什么 / 何时触发 / 如何验证"——是 skill；一句话答不清、需多步跨域——考虑 subagent。

### 7.2 Skill Bundle 目录结构

> **[S]** 原典 `Skill authoring best practices.md` line 269-280 给出推荐目录结构。

**最简形式（仅含 SKILL.md）：**

```
my-skill/
└── SKILL.md          # 含 YAML frontmatter + 主体指令
```

**推荐形式（随规模演进）：**

```
my-skill/
├── SKILL.md              # 主体指令（触发时加载，< 500 行）
├── FORMS.md              # 表单 / 模板填写指南（按需加载）
├── reference.md          # API / 字段参考（按需加载）
├── examples.md           # 使用示例集（按需加载）
└── scripts/
    ├── analyze.py        # 工具脚本（执行，非加载）
    ├── validate.py       # 校验脚本
    └── fill_form.py      # 业务脚本
```

**目录约定（依重要程度排列）：**

| 约定 | 出处 | 备注 |
|---|---|---|
| `[S]` `SKILL.md` **建议**置于根目录 | line 271 | 客户端按惯例从根目录读取元数据；非根目录可能触发加载失败 |
| `[S]` 引用文件路径保持**一层深度** | line 371 | 即 `reference.md` 而非 `docs/api/v2/reference.md` |
| `[E]` `scripts/` 目录**只放可执行文件** | 作者综合 | 不放 markdown 或配置文件（这些放根目录） |
| `[CK]` 长篇参考**分文件存放**而非内联于 SKILL.md | 第二遍硬约束 | 维持 SKILL.md 500 行上限 |
| `[E]` 配置文件**集中放** `config/` | 作者综合 | 便于跨 skill 复用与环境变量注入 |

**关于"是否强制"：**
- 原典对"根目录放 `SKILL.md`"用的是 *figure* 而非 *must* 措辞；按"建议层级"理解。
- `[E]` 若项目已有约定（如 monorepo 中所有 skill 统一放 `skills/<name>/`），优先遵循项目约定。

### 7.3 description 过宽 / 过窄的修复路径

> 原典与本档 AP-11 / AP-12 列举了过宽与过窄的反模式，此处补**修复方法**。

#### 7.3.1 判定当前 description 的"宽窄度"

| 维度 | 过宽症状 | 过窄症状 |
|---|---|---|
| 触发频次 | 几乎所有任务都触发 | 几乎从不触发 |
| 用户措辞 | 用户很少用其中关键词 | 用户必须精确说出关键词 |
| 描述具体度 | 抽象概念（"help with X"） | 含特定数字 / 限定（"Q3 only"） |
| 同质 skill 数 | 大量重叠的 skill 都被命中 | 同领域无 skill 被命中 |

#### 7.3.2 修复步骤（自检式）

**Step 1 — 关键词穷举：** 列出 10 个真实用户会用的触发该 skill 的具体表述。
- 反例：仅有"PDF 任务"这种宽泛表述
- 正例：含 "PDF 表单"、"合并 PDF"、"从 PDF 提取表格"、"填写 PDF 字段"等具体说法

**Step 2 — 场景压缩：** 将 10 个表述归纳为 3-5 个核心场景，每个场景用一个具体动词短语表达。

**Step 3 — 触发信号嵌入：** 把场景短语原样写入 description（不加修饰），形成"看到这些词就加载"。

**Step 4 — 反例排除：** 在 description 末尾显式声明**不**适用的场景，避免与同类 skill 误抢触发。

**修复模板：**

```yaml
# 反例（过宽）
description: "Helps with PDF tasks"

# 反例（过窄）
description: "Only fills Acme Corp's Q3 2026 expense report PDF"

# 修复后（适度）
description: |
  Extracts text/tables from PDFs, fills form fields, and merges
  multi-PDF documents. Use when user mentions: PDF, form, "fill in",
  "extract from PDF", "merge PDFs", or document extraction.
  NOT for: scanned image-only PDFs (use OCR skill), or password-
  protected PDFs (use decryption skill first).
```

### 7.4 负面清单汇总（集中版）

> 前文 AP-10..AP-20 散落各处，此处汇总为**单一负面清单**，便于审查时一眼对照。

#### 7.4.1 致命错误（Critical，发布前必须修复）

| # | 反模式 | 一句话描述 | 修复方向 |
|---|---|---|---|
| AP-19 | frontmatter 缺 description | skill 静默失效 | 强制要求非空 description |
| AP-15 | 吞异常返回空字符串 | 错误无法追溯 | 改为抛出含上下文的异常 |
| AP-16 | 假设独占环境 | 与其他 skill 冲突 | 移除 env var / 全局状态依赖 |
| AP-10 | 直接复制好 prompt 当 SKILL.md | 失去渐进披露优势 | 拆分 description / 指令 / 引用 |
| AP-11 | description 过宽 | 命中过多 = 不命中 | 7.3 节步骤 1-4 |
| AP-12 | description 过窄 | 命中过少 | 7.3 节步骤 1-4 |

#### 7.4.2 重要问题（Important，发布前应修复）

| # | 反模式 | 一句话描述 |
|---|---|---|
| AP-13 | 把整个领域教程塞入 | 拆分到 `reference.md` |
| AP-14 | helper script 内联为示例 | 改为引用路径 |
| AP-20 | 示例与场景不一致 | 对齐 description 关键词与示例标题 |

#### 7.4.3 视场景而定（视分发范围决定）

| # | 反模式 | 个人用 | 团队 / 公开 |
|---|---|---|---|
| AP-17 | 无测试 | 容忍 | 必须修复 |
| AP-18 | 列表层级混用 | 容忍 | 应修复 |

**使用方式**：审查时打印此清单，逐条对照（10 分钟内可完成）；任何 Critical 命中即阻塞发布。

### 7.5 写作动机叙事

> **[S]** 原典 `Introduction to Claude Skills.md` line 5-15、line 186-201 阐述 Skills 的存在理由。

**为什么需要 Skills？**

**动机一：减少重复解释（成本驱动）**
- `[S]` 原典 line 276-289：每次对话中重复解释"如何做某事"消耗 5k–10k tokens。Skill 化后，仅在首次触发时加载。
- `[CK]` 团队场景：每个新人入组时不必再问"我们怎么填报销单"——skill 已是"组织知识的封装"。

**动机二：降低 prompt engineering 的认知负担（演进驱动）**
- `[S]` 原典 line 188-196：Skills 是"比单 tool 高一档"的封装，原型用 tool、稳态用 skill。
- `[E]` 作者观察：写一个 skill 的边际成本（一次性 4-8 小时）远低于反复写 prompt 的累计成本（每周 1-2 小时 × 50 周）。

**动机三：沉淀"已测试的代码"（质量驱动）**
- `[S]` 原典 line 198：helper scripts 是 skill 的核心价值，"tested, working code"。
- `[E]` 反证：让模型每次重写"如何解析 CSV"——错误率明显高于复用已测试脚本。

**动机四：可组合性（架构驱动）**
- `[S]` 原典 line 198-200：多个 skill 可在同一会话协同工作。
- `[E]` 类比 UNIX 工具链：每个 skill 是一个专注的小工具，组合出无限可能。

### 7.6 与文件系统、Upload 等 Beta 工具的组合

> **[S]** 原典 `Introduction to Claude Skills.md` line 241-272 详述 Skills 与 Beta 工具的配合。

**协同矩阵：**

| Beta 工具 | 与 Skills 的协同 | 典型用法 |
|---|---|---|
| `code_execution_2025-08-25` | 强制项——执行 `scripts/` 下的代码 | skill 定义的 `validate.py` 由本工具执行 |
| `files-api-...` | 上传 / 下载文件型资源 | 把 PDF / Excel 上传供 skill 处理 |
| `skills-2025-10-02` | 启用 skill 系统的 Beta 标识 | 须列入 `betas` 数组 |
| `prompt-caching-...` | 缓存已加载的 skill 内容 | 多次调用同一 skill 时显著降本 |

**最小调用骨架（Python）：**

```python
response = client.beta.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=4096,
    betas=["skills-2025-10-02"],
    tools=[{"type": "code_execution_2025-08-25"}],
    container={"skills": [{"type": "anthropic", "skill_id": "pdf", "version": "latest"}]},
    messages=[{"role": "user", "content": "Extract tables from report.pdf"}]
)
```

**关键约束（任一缺失即报错）：**
- `[S]` 必须用 `client.beta.messages.create()`（非标准 `messages.create`）
- `[S]` `betas` 数组须含 `skills-2025-10-02`
- `[S]` `tools` 须含 `code_execution_2025-08-25`
- `[E]` `container.skills` 显式声明加载的 skill 列表（默认全部加载会耗 token）

### 7.7 Loop iterations：Claude A 设计 ↔ Claude B 使用的循环

> **[S]** 原典 `Introduction to Claude Skills.md` line 442-462 描述该循环。

**完整循环（5 步）：**

1. **设计（Claude A）**：根据工作流草拟 SKILL.md
2. **使用（Claude B）**：在实际任务中调用该 skill
3. **观察**：记录 B 触发的时机、调用的工具、产生的错误、用户反馈
4. **修正（回到 A）**：根据观察调整 SKILL.md（描述、指令、脚本）
5. **再使用（B 再次）**：验证修正有效

**每轮迭代的最小收集清单：**

| 项 | 收集方法 | 用途 |
|---|---|---|
| 触发频次 | 日志 | 判定 description 宽窄度 |
| 加载后被引用的章节 | `claude_code` 工具 | 判定 SKILL.md 是否冗余 |
| 工具调用失败率 | 日志 | 判定 scripts 健壮性 |
| 用户二次追问率 | 对话记录 | 判定指令是否清晰 |
| 任务完成度 | 人工评估 | 判定整体有效性 |

**迭代节奏建议：**
- `[E]` 第一周：每 2 个真实使用做一次小修正（微调 description / 修脚本）
- `[E]` 第一个月：每周一次完整 5 步循环
- `[E]` 稳定后：每月一次回顾，判断是否需要拆分 / 合并 skill

### 7.8 生产部署：版本管理

> **[S]** 原典 `Introduction to Claude Skills.md` line 368-376 详述 versioning。

**两类版本策略：**

| 策略 | 适用 | 写法 | 风险 |
|---|---|---|---|
| `"latest"` | Anthropic 托管 skill | `version: "latest"` | 自动更新，可能引入 breaking change |
| **Epoch 时间戳** | 自定义 skill | `version: "1749667200"`（秒） | 不可读，但完全可控 |
| 固定版本号 | 生产环境 | `version: "2025-11-15-v3"` | 须手动更新；最稳 |

**生产部署 checklist：**
- `[S]` **锁定版本**（不用 `latest`）——尤其在生产环境
- `[S]` **保留回滚路径**——每个版本都有完整目录备份
- `[E]` **灰度发布**——先在 1% 流量验证，再全量
- `[E]` **记录 changelog**——每次发布附变更说明，便于审计
- `[CK]` **监控核心指标**——加载成功率、工具调用失败率、用户二次追问率

**Anthropic 托管 skill 的更新机制（line 370-373）：**
- `[S]` Anthropic 自主更新托管 skill；用户用 `latest` 自动获得最新版
- `[S]` 用户可通过固定版本号锁定到历史版本
- `[S]` 自定义 skill **必须**用 epoch 时间戳（无其它选择）

### 7.9 占位符 `${WIKIBASE_ROOT}` 的使用示例

> 原 F-24（Pass 6）仅提及占位符概念，此处补**具体用法**。

**使用场景：** skill 部署在多台机器、不同用户的机器上时，硬编码绝对路径会导致：
- 团队成员 A 的 `/Users/alice/project` 在 B 处为 `/Users/bob/project` → 脚本失败
- CI 环境的 `/home/runner/work/...` 在本地开发环境不存在

**占位符约定：**

| 占位符 | 含义 | 示例值 |
|---|---|---|
| `${WIKIBASE_ROOT}` | skill 所在项目的根目录 | `/Users/xiang/Work/sfx` |
| `${HOME}` | 用户主目录 | `/Users/xiang` |
| `${SKILL_DIR}` | 当前 skill 的目录 | `${WIKIBASE_ROOT}/skills/pdf` |
| `${WORKSPACE}` | 运行时工作目录 | `/tmp/run-12345` |

**SKILL.md 中的使用方式：**

```markdown
## 快速开始

将待处理 PDF 放入 `${WORKSPACE}/input/` 后运行：

\`\`\`bash
python ${SKILL_DIR}/scripts/validate.py ${WORKSPACE}/input/report.pdf
\`\`\`

脚本会：
1. 校验 PDF 完整性
2. 输出元数据至 `${WORKSPACE}/output/meta.json`
```

**部署时展开（伪代码）：**

```python
# 部署脚本 deploy.py
content = open("SKILL.md").read()
content = content.replace("${WIKIBASE_ROOT}", os.environ["WIKIBASE_ROOT"])
content = content.replace("${SKILL_DIR}", os.environ["SKILL_DIR"])
content = content.replace("${WORKSPACE}", tempfile.mkdtemp())
open("SKILL.deployed.md", "w").write(content)
```

**配合环境变量：**

```bash
# .env
WIKIBASE_ROOT=/Users/xiang/Work/sfx
SKILL_DIR=${WIKIBASE_ROOT}/skills/pdf
```

**反例（硬编码）：**

```markdown
# ❌ 反例
python /Users/xiang/Work/sfx/skills/pdf/scripts/validate.py ~/Downloads/report.pdf
```

> **`[E]` 一句话原则：** 任何在 SKILL.md 出现的绝对路径，都应改为占位符；占位符在部署时由环境变量注入。

### 7.10 测试成功标准定义

> 原 22 项 Checklist 第 19-21 条提了"要测"，但未定义"怎样算通过"。此处补**判定标准**。

#### 7.10.1 三类测试的判定标准

| 测试类型 | 数量 | 判定标准 | 度量方法 |
|---|---|---|---|
| 典型场景 | ≥ 1 | 任务 100% 完成；用户零追问 | 人工评估 + 对话日志 |
| 边界情况 | ≥ 1 | 不崩溃；返回明确错误信息 | 注入空 / 超大 / 畸形输入 |
| 错误路径 | ≥ 1 | 错误信息含三要素（何错 / 为何 / 如何修） | grep 输出含三要素关键词 |

#### 7.10.2 "通过"的量化门槛

- **[E] 个人使用：**
  - ≥ 1 个端到端测试通过
  - 开发者本人主观满意

- **[E] 团队共享：**
  - 3 个测试全过（典型 + 边界 + 错误）
  - 至少 1 位团队成员（非作者）独立使用并签署

- **[E] 公开分发：**
  - 上述 + 多模型测试（Haiku / Sonnet / Opus 各跑一遍）
  - 真实用户样本 ≥ 5 人
  - 二次追问率 < 10%

#### 7.10.3 失败的处理流程

```
测试失败 → 分类（语法 / 逻辑 / 文档）
        → 语法：立即修复
        → 逻辑：回到 Pass 7.7 循环第 4 步修正 SKILL.md
        → 文档：补充示例 / 改写 description
        → 重新跑全部测试，回归通过后才发布
```

#### 7.10.4 一个最小测试用例模板

```python
# test_skill_e2e.py
import subprocess

def test_typical_scenario():
    """典型场景：用户上传 PDF，skill 提取表格。"""
    result = subprocess.run(
        ["python", "scripts/extract.py", "fixtures/sample.pdf"],
        capture_output=True, text=True
    )
    assert result.returncode == 0
    assert "table_1.csv" in result.stdout
    assert "table_2.csv" in result.stdout

def test_edge_empty_input():
    """边界：空 PDF。"""
    result = subprocess.run(
        ["python", "scripts/extract.py", "fixtures/empty.pdf"],
        capture_output=True, text=True
    )
    assert result.returncode != 0
    assert "Error" in result.stderr
    assert "empty" in result.stderr.lower()  # 错误信息含"为什么"

def test_error_missing_dep():
    """错误：缺依赖（无 pdftotext）。"""
    env = os.environ.copy()
    env["PATH"] = "/nonexistent"  # 模拟无依赖
    result = subprocess.run(
        ["python", "scripts/extract.py", "fixtures/sample.pdf"],
        capture_output=True, text=True, env=env
    )
    assert "pdftotext" in result.stderr  # 错误信息含"如何修"
    assert "brew install" in result.stderr or "apt install" in result.stderr
```

---

## 修订记录

- **v1 (2026-06-11 早期版)**: 三遍总结，未区分原典与作者外推。Adversarial reviewer 指出 30+ 处 unsupported claim，对照表几近全为捏造。
- **v2 (本版)**: 引入 `[S]/[E]/[CK]/[F]` 标签系统；悉数删除 fabrication；明确将"好 skill = …"归入作者综合，**不当作原典摘要**。
- **v3 (2026-06-11)**: 汲取 Anthropic 官方 [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) checklist 之要义。流程：两位独立 subagent 分别承担 gap analysis 与 merge 提案 → 第三位 subagent 仲裁分歧（5 处争议点悉数裁决）→ 作者最终定稿。新增：(1) 第二遍末尾的"补充硬约束"（22 项 Checklist 中 15 项为新内容），(2) 全新"第五遍"审查框架（三张审查表 + 审批门禁建议），(3) 速查表扩为新旧两源。所有 Checklist 条目标记为 `[S]`，解读标记为 `[E]`/`[CK]`。
- **v4 (2026-06-11)**: 增设"第六遍：与官方 Best Practices 的交叉验证"。将 A 与 B 之差异显式化：(1) 22 项 A checklist + 5 项 B 独有特征（F-23..F-27）= 27 条特征，(2) 9 项 A 反模式 + 8 项 B 独有反模式（AP-10..AP-17）+ 3 项作者观察（AP-18..AP-20）= 21 条 anti-patterns，(3) 速查表细分为 A 独有 / B 独有 / 共有三类。新增之 16 条特征与 11 条反模式已同步至 `skills/skill-reviewer/references/writing-good-skills.md`，以供审计对照之用。
- **v5 (2026-06-11, 本文件)**: 全文学术化润色。统一术语（"原典 / 外推 / 通识"），行文趋于严谨，保留带温度的版本叙事。
- **v6 (2026-06-11, 本版)**: 经与两份原典逐节比对，新增"第七遍：补遗"10 节，覆盖：(1) Skills 与 Tools/Subagents 层级关系，(2) Bundle 目录结构约定，(3) description 宽窄度判定与修复步骤，(4) 负面清单汇总，(5) 写作动机叙事，(6) Beta 工具协同矩阵，(7) Claude A↔B 五步迭代循环，(8) 生产部署版本管理，(9) 占位符具体用法示例，(10) 测试通过判定标准。
