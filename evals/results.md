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
| Real usage scenario | TODO | User-like prompt, target skill, actual report summary |
| Independent review | TODO | Reviewer, date, accepted feedback or no-change note |

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
