# Skill Reviewer Evaluation Results

This file records actual regression runs for `skill-reviewer`. Update it
whenever `SKILL.md`, `references/writing-good-skills.md`, or any
`expected-report.md` file changes.

## Acceptance criteria

- Run all three evaluation prompts from `evals/README.md`.
- Part B must match the corresponding `expected-report.md` exactly.
- Part A must include the same substantive findings; wording and severity may
  vary by one level where the issue is still clearly reported.
- The final sharing-tier recommendation must match the expected report.
- The report must end with `Self-check passed.`

## Current baseline

Status: scaffolded, pending full multi-model execution.

| Date | Model | Good case | Medium case | Bad case | Notes |
|------|-------|-----------|-------------|----------|-------|
| 2026-06-11 | Author review | PASS by inspection | PASS by inspection | PASS by inspection | Expected reports exist for all three cases; no model transcript captured. |

## Required public-release evidence

Before marking this skill ready for public distribution, capture at least:

| Evidence | Status | Required record |
|----------|--------|-----------------|
| Fast model run | TODO | Model name, date, three case results, mismatches |
| Capable model run | TODO | Model name, date, three case results, mismatches |
| Opus-class review | TODO | Model name, date, three case results, reviewer notes |
| Real usage scenario | DONE | See "Real usage scenarios" below |
| Independent review | TODO | Reviewer, date, accepted feedback or no-change note |

## Real usage scenarios

Idealized regression prompts (`请用 skill-reviewer 检查 evals/<case>/`) alone
do **not** satisfy checklist item #21. Record at least one messy,
underspecified, real-user-style run here.

### 2026-07-26 — self-review of skill-reviewer (item #21)

| Field | Value |
|-------|-------|
| Model | Cursor Grok 4.5 |
| Date | 2026-07-26 |
| Target | This repo root (`skill-reviewer/` itself) |
| User prompt (verbatim) | `用这个skill-reviewer 来review自己` |
| Why this counts as real usage | Messy / underspecified: no path, no "check against expected-report", no language preference spelled out; asks the skill to audit its own live directory, not a canned eval fixture. |

Report summary:

- Pass rate: 19/22 (15 PASS, 3 FAIL, 4 N/A)
- Tier: Ready for team share
- Failures at the time: #20 (multi-model), #21 (this evidence was still TODO), #22 (independent review)
- Ended with: `Self-check passed.`

Outcome for item #21: this run + the record below close the "idealized prompts only" gap. Re-audit after this write-up should mark #21 PASS.

Messy prompt variants that also qualify (use any one; paste the actual prompt + summary when run):

```text
帮我看看这个 skill 行不行
skill review 一下，感觉有点乱
is my skill ready? （指着仓库根目录，不给路径）
检查一下 SKILL.md 能不能分享给同事
```

## Run log template

Copy this section for each real run.

```markdown
### YYYY-MM-DD — <model or reviewer>

Prompts:

- `请用 skill-reviewer 检查 evals/good-skill-example/`
- `请用 skill-reviewer 检查 evals/medium-skill-example/`
- `请用 skill-reviewer 检查 evals/bad-skill-example/`

Results:

| Case | Result | Mismatches | Action |
|------|--------|------------|--------|
| good | PASS/FAIL | None / details | None / fix |
| medium | PASS/FAIL | None / details | None / fix |
| bad | PASS/FAIL | None / details | None / fix |

Conclusion:

- Regression status:
- Notes:
```
