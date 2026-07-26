---
title: 一个 Skill 凭什么审查另一个 Skill？我拆了 westzhao13/skill-reviewer 的全部 22 项检查
author: Claw 的硬核拆解
date: 2026-06-13
tags: [Claude, Agent Skills, 工程化, 实战拆解]
---

# 一个 Skill 凭什么审查另一个 Skill？我拆了 westzhao13/skill-reviewer 的全部 22 项检查

## 引子：当 Skill 多到需要"质检员"

Claude Skills 体系的核心思路是"渐进披露"（Progressive Disclosure）——Skills 按需加载，只把用得上的内容塞进上下文。

然而几乎所有写过两三个 Skill 的人都会撞上同一面墙：

> "这个 Skill 写得行不行？能不能发出去？"

Anthropic 官方给了两篇文档——`Skill authoring best practices`（1111 行）和 `Introduction to Claude Skills`（834 行）——但真要拿这两份文档当 checklist 一项项去对，体感跟"读完 ISO 9001 再去审工厂"差不多：知道标准是啥，但不知道从哪下手。

`westzhao13/skill-reviewer` 这个仓库做的事情，就是把官方那两篇文档**压成 22 项可执行的检查项**，再加自检机制、严重度分级、分享层级 gating。换句话说，它是一个**用 Skill 形态写成的 Skill 质检员**。

我花了两个小时把它的 `SKILL.md`、`references/writing-good-skills.md`（267 行速查表）、三份 `evals/*-skill-example/expected-report.md` 全过了一遍。下面是这份审计工具的全部干货。

---

## 一、整体长什么样

仓库核心结构非常简洁：

```
skill-reviewer/
├── SKILL.md                                    # 22 项检查清单 + 4 阶段流程
├── references/writing-good-skills.md           # 267 行速查表
├── evals/
│   ├── good-skill-example/                     # 规范写法
│   ├── medium-skill-example/                   # 含若干 Important 问题
│   └── bad-skill-example/                      # 含多个 Critical 问题
└── docs/                                       # 官方原文 + 中英消化版
```

跑起来的方式只有一种：扔给 Claude Code 一句"请用 skill-reviewer 检查 ~/skills/my-skill/"，剩下的它自己干。返回的报告分两部分：

- **Part A** — 按 Critical / Important / Minor 分级的问题清单，每条带定位 + 描述 + 修复建议
- **Part B** — 22 项完整表格，每项 `PASS / FAIL / SKIPPED / N/A`

末尾必须以 `Self-check passed.` 结尾。这个"自检声明"是整个 skill 最值钱的设计之一，后面单说。

---

## 二、22 项检查清单（带实战解读）

我把三类 22 项全部展开，并加上**我自己对每一项的实战注解**——哪些是真坑、哪些是过度设计。

### A. 核心质量（#1 – #10）

| # | 项目 | 一句话解读 |
|---|------|----------|
| 1 | Description 要具体，含关键词 | "handles documents" 是 FAIL；"Creates branded PDF reports from markdown — use when .pdf, weekly-report" 才是 PASS |
| 2 | Description 要"做什么 + 何时用" | 缺一项直接 FAIL；触发短语要给 3–5 个 |
| 3 | SKILL.md 正文 ≤ 500 行 | 250–400 行最理想；超过 500 行的要拆 |
| 4 | 附加内容拆出去 | 模板、配置、长引用放独立文件，主体保持精简 |
| 5 | 不混时效信息 | API 版本号、日期、"as of June 2025"——出现一个就 FAIL |
| 6 | 术语全文一致 | 同一概念在 description / body / scripts / errors 里叫同一个名字 |
| 7 | 示例要可跑、真实 | "Create a chart" 是 FAIL；"Create a bar chart of Q1 revenue with data labels" 是 PASS |
| 8 | 文件引用 ≤ 2 层 | `scripts/validate.py` OK；`tools/helpers/submodules/checker.py` FAIL |
| 9 | 体现三层渐进披露 | 主体精简、详情 defer；一股脑堆在 SKILL.md 是 FAIL |
| 10 | 多步流程用编号列表 | 段落叙述 FAIL；编号、子标题、决策树才 PASS |

> **实战注解**：第 1、2、6、7 项是 80% 的 Skill 翻车的真正原因。一个能用的 description 长这样：`Creates branded PDF reports from markdown — use when .pdf, weekly-report, invoice, formal-document`。包含**做什么 + 触发关键词 + 触发场景**。缺一个都掉档。

### B. 代码与脚本（#11 – #18）

| # | 项目 | 一句话解读 |
|---|------|----------|
| 11 | 脚本真做事 | 解析、转换、校验；空壳转发 `client.messages.create` 是 FAIL |
| 12 | 错误信息三要素 | 失败原因 + 修复方法；"Error occurred" 是 FAIL |
| 13 | 别用魔法数字 | `timeout=30` 裸值是 FAIL；带 ` # max sec for render; longer means resource leak` 注释才是 PASS |
| 14 | 依赖要列、要校验 | SKILL.md 写清依赖；脚本启动时检查，失败时给可执行的安装指令 |
| 15 | 脚本要写好文档 | 用途、参数、返回码、示例——在脚本文件里写，不在 SKILL.md 写 |
| 16 | 路径全用正斜杠 | `scripts\tools\check.py` 一律 FAIL；必须 `scripts/tools/check.py` |
| 17 | 关键操作要验证 | 写文件后查 size / 行数；网络请求后查 status code |
| 18 | 质量敏感任务要自评环 | 文档生成、代码审查类任务——Claude 必须先 review 自己的输出再交付 |

> **实战注解**：第 11、12、15、17 项是工程化 Skill 的命门。第 11 项尤其重要——很多 Skill 写了个 `scripts/extract.py` 但里面其实就是 `client.messages.create(prompt=...)`，那是 prompt 披了件脚本外衣。第 17 项容易忘，写了文件不验证，下游出 bug 还以为是上游问题。

### C. 测试与评估（#19 – #22）

| # | 项目 | 一句话解读 |
|---|------|----------|
| 19 | 至少 3 个评测用例 | 0/1/2 个都是 FAIL；要覆盖典型 + 边界 + 错误路径 |
| 20 | 多模型测试 | Haiku + Sonnet/Opus 至少各跑一遍；只在一款上跑是 FAIL |
| 21 | 用真实场景 | prompt 要拟真（脏数据、欠规格）；理想化 prompt 是 FAIL |
| 22 | 多人协作要有独立 review | 至少一轮；单人 Skill 标 N/A |

> **实战注解**：19–22 这四项是"分享层级 gating"的硬指标。一个 Skill 自我感觉良好没用，得有 evals 目录、有多模型测试记录、最好有团队反馈。

---

## 三、最值钱的两块设计：F-23 与自检

`references/writing-good-skills.md`（267 行速查表）里有 5 个**官方没明说**的特征和 11 个**实战常见**的反模式。挑两个最值钱的讲。

### F-23：description 是路由信号，不只是"描述"

> **一个 Skill 在 100+ 候选池里能不能被加载，全看 description 写得好不好。**

官方说 description 要"what + when"，但 F-23 更进一步：

| 失败模式 | 例子 | 为什么失败 |
|---------|------|----------|
| 太宽 | "helps with documents" | 啥都触发，啥也帮不上 |
| 太窄 | "only for Q3 sales reports" | 几乎从不触发 |
| 缺触发 | "Creates PDF reports" | Claude 不知道啥时候该加载 |
| 体例不一致 | description 提了 `.pdf`，body 里没 PDF 例子 | 路由信号和内容脱钩 |

> 这条 insight 把"写 description"从"补充说明"升级成了**路由设计**。我做电商客服 Skill 的时候最深的体会就是：description 写得烂，调用率直接腰斩。

### 自检机制：末尾必须 `Self-check passed.`

> 整个 skill 最让我服气的设计是**末尾的四点自检**。

Phase 4 强制 Claude 在交付前做：

1. **具体性** — 每个 FAIL 必须带 `file:line` 或引用原文片段；模糊的"description 不清"不算
2. **完整性** — Part B 必须出现全部 22 项；漏一项都不行
3. **抽样核查** — 随机抽 2 个 FAIL 回去对源文件，找不到证据就删掉或改判
4. **层级一致性** — gating 建议必须跟结果对得上；标"Ready for team share"但 #19 是 FAIL 是矛盾

末尾必须以一行 `Self-check passed.` 结尾。

> **为什么这个值钱？** 因为审查 Skill 最容易翻车的地方是**审查者自己掉链子**——给一堆没有定位的 FAIL、漏掉某项、对错位置。强制自检 + 固定结尾，相当于把"审查者复盘"做成 Skill 的硬约束，不是可选项。

---

## 四、实战对比：好、中、坏三个样例的真实报告

光看规则不直观，我直接搬运仓库里三份 `expected-report.md` 的核心结论给你看。

### 4.1 Good 样例：CSV 去重（满分）

> **Pass rate：17 PASS / 0 FAIL / 5 N/A-or-SKIPPED**
> **Gating：Ready for team share**

这个 Skill 故意写得"几乎无问题"。值得抄的几个点：

- Description 里直接列触发短语：`"dedup my CSV"`、关键词 `"case-insensitive"`、`"match-key"`
- 主体只 85 行，细节全在 `references/csv-edge-cases.md`
- Step 6 显式 verify 输出文件行数（命中 #17）
- Evals 用了"exact-match, case-insensitive, subset-column"三个 fixture（命中 #19）

### 4.2 Medium 样例：Markdown 链接检查（若干 Important）

> **Pass rate：中**
> **Gating：Clear for personal use**（团队共享被 #19 评测不足卡住）

故意制造的"中级问题"：
- 术语混用：description 写"link check"，body 写"URL validator"
- 示例抽象："Find broken links"——没有具体输入输出
- 错误信息只写"link check failed"，没说怎么修

### 4.3 Bad 样例：通用"代码助手"（教科书反面教材）

> **Pass rate：2 PASS / 19 FAIL / 1 N/A**
> **Gating：Not ready**（连个人使用都不过）

`Helps with code. AI-powered coding assistant.` —— 整个仓库的反面教材。Critical 问题包括：

- **#1, #2 描述完全无效**：无场景、无触发，Claude 完全不知道何时该用
- **#11 脚本空壳**：只是 `client.messages.create(prompt=user_input)`，教科书式 "punt to Claude"
- **#12 错误信息空洞**："The script will fail if something is wrong. Check the error."
- **#16 反斜杠路径**：`scripts\tools\helper.py` 在 macOS/Linux 直接炸

外加一堆 Important：

- 大量 "October 2024"、"June 2025" 时效信息
- helper / bot / assistant / agent 四种叫法混用
- 示例全是 "Fix code / Write code / Explain code" 单词级抽象
- 路径嵌套 4 层
- 魔法数字：`max_tokens=1024`、`# timeout = 30`（注释掉的，更离谱）
- 无 evals、无多模型测试、无真实场景

> **这个 bad 样例本身就是最好的教学材料**——你随便抽两个项，都能在自己的 Skill 里找到同款问题。

---

## 五、分享层级 Gating：把"我觉着 OK"换成"系统判定"

22 项检查的最大实战价值，是把"这个 Skill 能不能分享"从主观判断换成 gating 逻辑：

| 分享层级 | 硬性要求 |
|---------|---------|
| **个人使用** | #1-3 PASS，无 Critical 失败 |
| **团队共享** | 核心质量 #1-10 全 PASS + ≥ 3 个评测用例（#19） |
| **公开分发** | 22 项全 PASS + 多模型测试（#20）+ 团队反馈（#22） |

> **这解决了一个真问题**：很多 Skill 写完直接发出去，结果在别人机器上跑不起来；或者给别人用时被问"你测过没？"答不上来。gating 让你在发出去之前就知道"还差什么"。

---

## 六、把它接到自己的 Skill 体系里

我自己的做法是把它**当 lint 用**——每写完一个 Skill，跑一遍 `skill-reviewer`，Part B 里凡是 FAIL 的项就过一遍修复。

但有两个**加值动作**推荐你做：

1. **把它当 Skill 模板**——22 项本身就是"Self 写 Skill 的质量标准"。下次写新 Skill，**先用这 22 项当 checklist 反向校准**，比写完再回头审要省事得多。
2. **把它接到 CI/发布前检查**——理论上你可以把 Part B 的 22 项固化成自动化脚本（脚本读 SKILL.md + 跑评测 + 跑多模型 + 输出表格）。这个仓库目前还是纯 LLM 驱动，自己做一层工程化包装不难。

> **反过来说**：`skill-reviewer` 自己也还不是"自动化审计"——它仍然依赖 Claude 的判断力。所以多模型测试那一项（#20）才那么重要：换个模型跑跑，看 Part B 的判定还一不一样。

---

## 七、五个容易被忽略的细节

1. **`SKIPPED` 也是合法结果**——文本模式（只粘贴 SKILL.md 内容）下，结构性检查（#3 行数、#8 嵌套、#15 脚本文档、#16 路径）标 `SKIPPED (no directory)`，这是合规的。
2. **"#16 全用正斜杠" ≠ "#F-24 用占位符"**——前者管 `/` vs `\` 字符，后者管 `${SKILL_ROOT}` vs 硬编码绝对路径，是两件事。
3. **AP-15 静默吞异常是 Critical**——`except: pass` 看似无害，实际让 Claude 拿到空数据后瞎调，直接归 Critical。
4. **AP-17 "应该能用"不算验证**——个人使用至少 1 个 e2e 测试，团队 3 个，公开 3+ 多模型。零测试在任何层级都 FAIL。
5. **单人 Skill 的 #22 标 N/A**——team feedback 那一项对单人不适用，但不能不标，必须显式 N/A。

---

## 八、给作者的一点建议

`skill-reviewer` 本身**质量极高**，但还有几个工程化空间：

- `evals/results.md` 目前还停留在"待多模型测试"状态（scaffolded, pending full multi-model execution）。公开分发前需要补 Haiku / Sonnet / Opus 三档模型跑过的实证记录。
- 22 项都是"语义判断"，没看到自动化脚本。如果未来要做 CI 集成，建议把 #5 时效信息、#16 反斜杠、#3 行数这些**机械可检的**先脚本化，剩下 18 项继续走 LLM。
- expected-report.md 是**强约束**（Part B 必须逐项匹配），Part A 是弱约束（措辞可 ±1 档）。这个分层很好，但要在 PR 模板里勾选"已更新 evals"防止漏改。

---

## 写在最后

22 项检查、4 阶段流程、4 点自检、3 档 gating——这不是 22 个孤立规则，是一套**完整的 Skill 质量方法论**。

它的核心 insight 其实是两个：

> **description 是路由信号，不是描述。** 写不好，Skill 永远不会被加载。
> **审查 Skill 的人自己也要被审查。** 自检不是可选项。

下次你写完一个 Skill，扔进去跑一遍；如果 Part B 出现一堆 FAIL，别急着改 Skill——先想想是描述没写对、还是结构没分层、还是缺评测。改完再跑一遍，直到 gating 升到你要的层级再发出去。

这是目前 Claude Skills 生态里**最值得抄的工程实践**。

---

*附：三个评测样例的 expected-report.md 是这篇文章最值钱的素材，建议直接读原文（`evals/good-skill-example/expected-report.md` / `medium-` / `bad-`），比我这里的摘要更具体。*
