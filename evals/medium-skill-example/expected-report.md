# Expected Report — `evals/medium-skill-example/`

## Part A: Graded issues

### Critical
*(none)*

### Important

- **[#6] SKILL.md** — 术语在文中反复切换:`Markdown Link Checker`(标题)、
  `markdown link validator`(Overview)、`The validator`(When to use)、
  `The checker`(Usage)。同一对象至少出现三种叫法。
  **修复建议**:统一为 "checker",在所有文本中替换 "validator"。

- **[#7] SKILL.md (Examples 段)** — 三条示例「Check a README / Check a
  docs folder / Find dead links in a project」全部抽象,没有真实输入输出。
  **修复建议**:替换为具体场景,例如「输入 `docs/` 含 47 个 .md 文件,
  检测出 3 个 404 外链 + 5 个失效相对路径,输出到 `report.txt`」。

- **[#12] SKILL.md (Errors 段)** — 「If something fails, the script
  prints an error. Check the error message and retry.」没有说明失败模式、
  原因、修复步骤,等于没写。
  **修复建议**:列出 2–3 个典型失败(超时、404、文件不存在),分别给出
  「what / cause / fix」三段式信息。

- **[#19] SKILL.md** — 全文未提及任何评测用例,也无 `evals/` 目录。
  **修复建议**:至少准备 3 份样例(全部正常 / 含死链 / 含超时),分别
  存为 fixture。

### Minor

- **[#13] SKILL.md (Configuration 段)** — `TIMEOUT=10` 与 `MAX_PARALLEL=8`
  两个常量直接给值,没有解释为什么是这个量级。
  **修复建议**:补一句注释,例如「10s 覆盖大多数外网响应又能避免 CI 卡死」。

- **[#17] SKILL.md** — 工作流缺少对自身输出的验证步骤,例如「确认报告
  行数 = 失败链接数」。

- **[#1] SKILL.md (frontmatter)** — 描述中缺少「broken link / dead link /
  404」等用户最常用的触发词,可能导致 skill 无法被路由到。**建议**补充
  这些关键词以提高发现率。

---

## Part B: 22-item checklist

| # | Category | Item | Result | Note |
|---|----------|------|--------|------|
| 1 | Core | Description specific + key terms | ⚠️ PASS (弱) | 含场景但缺 "broken/dead/404" 关键词 |
| 2 | Core | Description what + when | ✅ PASS | 简短但齐全 |
| 3 | Core | Body < 500 lines | ✅ PASS | ~60 行 |
| 4 | Core | Additional details in separate files | ✅ PASS | 正文本身已精简 |
| 5 | Core | No time-sensitive information | ✅ PASS | 无日期 / 版本号 |
| 6 | Core | Consistent terminology | ❌ FAIL | checker / validator 混用 |
| 7 | Core | Concrete examples | ❌ FAIL | 示例完全抽象 |
| 8 | Core | File references one level deep | ✅ PASS | `scripts/check.py` |
| 9 | Core | Progressive disclosure | ✅ PASS | 体量小,无需拆分 |
| 10 | Core | Workflows have clear steps | ✅ PASS | Usage 段编号清晰 |
| 11 | Code | Scripts solve problems | ✅ PASS | 解析 + 解析路径 + HTTP 请求 |
| 12 | Code | Error handling helpful | ❌ FAIL | 「something fails... check the error」无信息量 |
| 13 | Code | No voodoo constants | ❌ FAIL | TIMEOUT=10 / MAX_PARALLEL=8 无依据 |
| 14 | Code | Required packages listed | ✅ PASS | Python 3 + requests + markdown-it-py |
| 15 | Code | Scripts have clear documentation | ⚪ SKIPPED | 脚本文件未提供,无法核查 |
| 16 | Code | No Windows-style paths | ✅ PASS | 全部正斜杠 |
| 17 | Code | Validation for critical operations | ❌ FAIL | 工作流无自我验证步骤 |
| 18 | Code | Feedback loops for quality | ⚪ N/A | 机械任务,无质量判断环节 |
| 19 | Test | ≥ 3 evaluation cases | ❌ FAIL | 无 evals/,无样例 |
| 20 | Test | Tested with Haiku, Sonnet, Opus | ❌ FAIL | 无多模型测试证据 |
| 21 | Test | Tested with real usage | ❌ FAIL | 无真实场景测试痕迹 |
| 22 | Test | Team feedback incorporated | ⚪ N/A | 单人 skill |

---

## Summary

- **Pass rate:** 11 PASS / 8 FAIL / 3 N-or-SKIPPED(共 22 项)
- **Gating recommendation:** **Clear for personal use**

依据:第 1–3 项 PASS,无 Critical 失败 → 个人使用层级达标。但术语混乱
(#6)、示例抽象(#7)、错误信息空洞(#12)在「核心质量(1–10)」里占
两项 FAIL,**不达到** Team Share。要升到 Team Share,优先处理上述四项
Important 问题 + 加 ≥3 个评测样例(#19)。

`Self-check passed.`
