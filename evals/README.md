# Skill Reviewer — Evaluation Cases

本目录包含三个**故意构造**的样例 skill,用于回归测试 skill-reviewer
本身。每个样例代表一种典型的写作水平,对应 skill-reviewer 输出报告
里的一种分享层级。

## 用途

1. **回归测试** —— 任何对 `SKILL.md` 检查清单或评分逻辑的改动,都应该
   先在三个样例上跑一遍,确认与 `expected-report.md` 仍然对得上。
2. **教学样本** —— 用户第一次使用 skill-reviewer 时,可以让它先审一遍
   `evals/medium-skill-example/`,直接看到一份真实输出长什么样。
3. **多模型对比** —— 把同一样例分别交给 Haiku / Sonnet / Opus 审查,
   对比报告差异,定位哪些检查项对模型能力敏感。

## 目录结构

```
evals/
├── README.md                                # 本文件
├── results.md                               # 实际回归运行与多模型测试记录
├── good-skill-example/
│   ├── SKILL.md                             # 规范写法的样例
│   └── expected-report.md                   # 期望的审查报告
├── medium-skill-example/
│   ├── SKILL.md                             # 含若干 Important 问题
│   └── expected-report.md
└── bad-skill-example/
    ├── SKILL.md                             # 含多个 Critical 问题
    └── expected-report.md
```

## 三个样例的设计意图

| 样例 | 主题 | 故意设计的问题 | 预期分享层级 |
|------|------|---------------|-------------|
| good | CSV 去重 | 几乎无问题 | Ready for team share |
| medium | Markdown 链接检查 | 术语混用、示例抽象、错误处理含糊 | Clear for personal use |
| bad | 通用「代码助手」 | 描述空洞、术语混乱、时效信息、反斜杠路径、脚本只是包装 prompt | Not ready |

## 如何运行评测

在 Claude Code 会话里依次执行:

```
请用 skill-reviewer 检查 evals/good-skill-example/
请用 skill-reviewer 检查 evals/medium-skill-example/
请用 skill-reviewer 检查 evals/bad-skill-example/
```

然后将实际输出与各目录下的 `expected-report.md` 做差异对比。**Part B
表格的逐项判定应当完全一致**;Part A 的措辞、Critical/Important 分类
允许有合理出入,只要要害问题没遗漏即可。

每次真实运行后,把模型、日期、三组样例结果、差异和修复动作记录到
`results.md`。如果只是人工检查或本地草稿,必须明确标记为 inspection
或 pending,不要写成已经完成的多模型测试。

## 关于期望报告的精确度

- `expected-report.md` 中的 Part B 是 **强约束**:逐项 PASS / FAIL / N/A
  必须匹配。
- Part A 是 **弱约束**:列出的问题必须全部出现,但文字表述、措辞、
  严重性分类可以有 ±1 档的浮动。
- 最终的「分享层级」必须匹配。

## 维护说明

修改 `SKILL.md` 的检查清单后,**必须**回头更新三份 `expected-report.md`,
否则评测必然失败。建议在 PR 描述里勾选「已更新 evals」清单项。

如果目标是公开分发,`results.md` 还应包含至少一个快速模型、一个能力
模型、一个 Opus 级模型的运行记录,以及一次独立 review 或团队反馈记录。
