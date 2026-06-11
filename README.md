# Skill Reviewer

> 一个用于审查 Claude Skill 质量的 Skill,基于 Anthropic 官方最佳实践的 22 项检查清单。

## 概述

Skill Reviewer 是一个面向 Claude Skill 作者的审查工具,通过对 `SKILL.md` 及其配套文件进行系统化检查,帮助作者在发布前发现质量问题、规范化写法、明确分享层级。

本工具的检查规则提炼自 Anthropic 官方两篇核心文档,涵盖描述规范、正文结构、脚本质量、错误处理、验证机制、测试覆盖等 22 个维度。

## 参考来源

| 文档 | 说明 | 本地副本 |
|------|------|---------|
| [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | Anthropic 官方编写指南 | `docs/Skill authoring best practices.md` |
| [Introduction to Claude Skills](https://github.com/anthropics/anthropic-cookbook) | Anthropic Cookbook 入门篇 | `docs/Introduction to Claude Skills.md` |

此外,`docs/` 目录下提供了两份消化版文档(中英文各一),便于快速查阅核心要点。

## 评审维度

审查清单共 22 项,分为三类:

### 一、核心质量(第 1–10 项)

- 描述精准、同时包含「做什么」与「何时使用」
- SKILL.md 主体控制在 500 行以内
- 附加内容拆分到独立文件,主文件保持精简
- 不混入时效性信息(如 API 版本号、具体日期)
- 术语在描述、正文、脚本、报错中保持一致
- 示例具体、可运行,展示真实输入输出
- 文件引用层级不超过 2 层
- 体现三层渐进披露(Progressive Disclosure)结构
- 多步流程使用编号列表或决策树

### 二、代码与脚本(第 11–18 项)

- 脚本承担实际工作(解析、转换、校验),而非空壳转发
- 错误信息明确说明「失败原因」「修复方法」
- 常量与超时值附带理由注释,避免「魔法数字」
- 依赖项明确列出并在启动时校验
- 每个脚本包含用途、参数、返回值、示例
- 路径统一使用正斜杠(禁用 Windows 反斜杠)
- 关键操作(写文件、网络请求等)包含成功验证
- 质量敏感任务设有自评反馈环

### 三、测试与评估(第 19–22 项)

- 至少 3 个评测用例,覆盖典型 / 边界 / 错误路径
- 同时在 Haiku、Sonnet、Opus 上测试通过
- 使用真实场景输入,而非理想化 prompt
- 多人协作场景下完成至少一轮独立 review

## 安装

将本仓库克隆至 Claude Code 的 Skills 目录:

```bash
git clone <repo-url> ~/.claude/skills/skill-reviewer
```

或将目录拷贝到任意 Skill 加载路径下,Claude Code 启动时会自动识别。

## 使用方法

在 Claude Code 会话中,通过自然语言或斜杠命令触发:

```text
请用 skill-reviewer 检查 ~/skills/my-skill/ 目录
```

```text
/skill-reviewer
```

### 支持的输入模式

| 模式 | 输入形式 | 检查范围 |
|------|---------|---------|
| 目录模式 | Skill 根目录路径 | 全量审查,包含文件结构、嵌套深度、路径风格 |
| 文本模式 | 粘贴的 SKILL.md 内容 | 仅审查文本,结构类项目标记为 `SKIPPED` |

### 输出格式

审查报告分为三部分,末尾附自检确认:

1. **分级问题列表** —— 按严重程度划分为 Critical / Important / Minor,每条问题包含编号、定位、描述、修复建议
2. **22 项完整检查表** —— 每项给出 `PASS` / `FAIL` / `SKIPPED` / `N/A` 及简要说明
3. **分享层级建议** —— 依据下表给出明确结论
4. **自检声明** —— 报告末尾以 `Self-check passed.` 结尾,确认四点自检(具体性 / 完整性 / 抽样核查 / 层级一致性)已通过

| 分享层级 | 要求 |
|---------|------|
| 个人使用 | 第 1–3 项 PASS,无 Critical 失败 |
| 团队共享 | 核心质量(第 1–10 项)全 PASS,且 ≥ 3 个评测用例 |
| 公开分发 | 全部 22 项 PASS,多模型测试,获得团队反馈 |

## 目录结构

```
skill-reviewer/
├── README.md                                       # 本文件
├── SKILL.md                                        # 审查流程与 22 项检查清单
├── references/
│   └── writing-good-skills.md                      # 审查时使用的速查表
├── evals/                                          # 回归测试样例
│   ├── README.md                                   # 评测说明
│   ├── results.md                                  # 实际运行与多模型测试记录
│   ├── good-skill-example/                         # 规范写法样例
│   ├── medium-skill-example/                       # 含若干 Important 问题
│   └── bad-skill-example/                          # 含多个 Critical 问题
└── docs/
    ├── Skill authoring best practices.md           # 官方指南原文
    ├── Introduction to Claude Skills.md            # Cookbook 原文
    ├── how-to-write-good-claude-skills.md          # 英文消化版
    └── how-to-write-good-claude-skills-cn.md       # 中文消化版
```

每个 `evals/*-skill-example/` 下含一份 `SKILL.md`(被审对象)与一份 `expected-report.md`(期望输出),用于回归验证与多模型对比。实际运行结果、模型差异和独立 review 记录写入 [`evals/results.md`](evals/results.md)。详见 [`evals/README.md`](evals/README.md)。

## 适用场景

- **发布前自查** —— 在将 Skill 公开发布前进行最后一轮质量审查
- **团队协作** —— 统一 Skill 写作规范,降低 review 沟通成本
- **学习参考** —— 理解 Anthropic 官方推荐的 Skill 写法
- **存量改造** —— 对已有 Skill 进行规范化升级

## 反馈与贡献

如果在使用过程中发现检查项不准确、规则需要扩展或与最新 Anthropic 官方文档存在偏差,欢迎提交 Issue 或 Pull Request。
