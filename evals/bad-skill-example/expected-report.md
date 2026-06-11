# Expected Report — `evals/bad-skill-example/`

## Part A: Graded issues

### Critical

- **[#1, #2] SKILL.md (frontmatter)** — Description is "Helps with code.
  AI-powered coding assistant." 既没有场景,也没有「when to use」,
  Claude 完全无法判断何时该调用它。**修复建议**:重写 description,
  列出 3–5 个具体触发短语 + 至少一个反触发(NOT for X)。

- **[#11] SKILL.md (Scripts 段)** — 脚本本质上只是把用户输入原样转发给
  `client.messages.create`,没有任何解析、转换、校验。这是教科书式的
  "punt to Claude"。**修复建议**:要么删除脚本(让 skill 纯 prompt 形态),
  要么让脚本真做事(例如先抽取代码块、识别语言、注入上下文)。

- **[#12] SKILL.md (Errors 段)** — 「The script will fail if something
  is wrong. Check the error.」无信息量。**修复建议**:至少列 2 种典型
  失败(API 鉴权失败、网络超时)并给出修复步骤。

- **[#16] SKILL.md (Scripts / Submodules 段)** — 路径 `scripts\tools\helper.py`
  与 `scripts\tools\submodules\extras\bot.py` 使用反斜杠。这是 Windows
  风格,在跨平台环境里会直接出错。**修复建议**:全部改为 `scripts/...`。

### Important

- **[#5] SKILL.md (Updates 段)** — 含「as of October 2024」「As of June 2025,
  use API v3」「deprecated 2024-Q4」,大量时效性信息混在主体文档中。
  **修复建议**:把版本历史移到 `CHANGELOG.md`,主文档不出现日期。

- **[#6] SKILL.md (正文)** — "helper / bot / assistant / agent" 四种叫法
  在同一段里轮番出现,严重影响阅读。**修复建议**:统一为 "helper",
  全文替换。

- **[#7] SKILL.md (Examples 段)** — "Fix code / Write code / Explain code /
  Review code" 全部是单词级别的抽象描述,完全没有真实输入输出。

- **[#8] SKILL.md** — `scripts/tools/helper.py` 嵌套两层,
  `scripts/tools/submodules/extras/bot.py` 嵌套四层,均超过推荐的 ≤2 层。

- **[#9] SKILL.md** — 缺少分层结构,无 references/ 也无 docs/。

- **[#10] SKILL.md (How to use 段)** — "Just ask. The helper figures it out."
  不是一个工作流。

- **[#13] SKILL.md (脚本块)** — `max_tokens=1024`、`# timeout = 30`、
  `model="claude-3-5-sonnet-20241022"` 全部无解释,且 timeout 是注释,
  根本没生效。

- **[#14] SKILL.md** — 没有 Dependencies 段,`anthropic` 包未声明。

- **[#15] SKILL.md (脚本)** — 仅 `# helper.py` 一行注释,无 docstring、
  无参数说明、无返回码、无用法示例。

- **[#17, #18] SKILL.md** — 无任何对脚本输出的验证,也无质量自检环节
  (而代码助手恰恰是最需要自检的场景)。

- **[#19, #20, #21] SKILL.md** — 全无 evals/、无多模型测试痕迹、
  无真实场景测试样例。

---

## Part B: 22-item checklist

| # | Category | Item | Result | Note |
|---|----------|------|--------|------|
| 1 | Core | Description specific + key terms | ❌ FAIL | "Helps with code" 完全空洞 |
| 2 | Core | Description what + when | ❌ FAIL | 无「何时使用」 |
| 3 | Core | Body < 500 lines | ✅ PASS | ~55 行 |
| 4 | Core | Additional details in separate files | ✅ PASS | 体量小,目前无需拆 |
| 5 | Core | No time-sensitive information | ❌ FAIL | 含 "October 2024"、"June 2025"、"2024-Q4" |
| 6 | Core | Consistent terminology | ❌ FAIL | helper/bot/assistant/agent 混用 |
| 7 | Core | Concrete examples | ❌ FAIL | 全部抽象单词 |
| 8 | Core | File references one level deep | ❌ FAIL | 2 层 + 4 层嵌套 |
| 9 | Core | Progressive disclosure | ❌ FAIL | 无 Tier 3 文件 |
| 10 | Core | Workflows have clear steps | ❌ FAIL | 无工作流 |
| 11 | Code | Scripts solve problems | ❌ FAIL | 直接 punt to Claude |
| 12 | Code | Error handling helpful | ❌ FAIL | "Check the error" |
| 13 | Code | No voodoo constants | ❌ FAIL | max_tokens / timeout / model 均无解释 |
| 14 | Code | Required packages listed | ❌ FAIL | 未声明 `anthropic` |
| 15 | Code | Scripts have clear documentation | ❌ FAIL | 仅一行注释 |
| 16 | Code | No Windows-style paths | ❌ FAIL | 全部反斜杠 |
| 17 | Code | Validation for critical operations | ❌ FAIL | 无验证 |
| 18 | Code | Feedback loops for quality | ❌ FAIL | 代码助手需自检却没有 |
| 19 | Test | ≥ 3 evaluation cases | ❌ FAIL | 无评测 |
| 20 | Test | Tested with Haiku, Sonnet, Opus | ❌ FAIL | 无证据 |
| 21 | Test | Tested with real usage | ❌ FAIL | 无证据 |
| 22 | Test | Team feedback incorporated | ⚪ N/A | 单人 skill |

---

## Summary

- **Pass rate:** 2 PASS / 19 FAIL / 1 N/A(共 22 项)
- **Gating recommendation:** **Not ready**

阻塞项(必须先修):
- #1 + #2:描述完全无效
- #11:脚本毫无价值
- #12:错误信息空洞
- #16:反斜杠路径

> 这版 skill 不适合任何分享层级 —— 即便仅个人使用,也很可能因为描述
> 含糊导致 Claude 永远不会触发它。建议从重写 frontmatter description
> 开始,然后决定:要么去掉脚本走纯 prompt 路线,要么让脚本承担实质工作。

`Self-check passed.`
