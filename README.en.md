# Skill Reviewer

[中文](README.md) | [English](README.en.md)

> A skill for reviewing the quality of Claude Skills, based on a 22-item checklist derived from Anthropic's official best practices.

## Overview

Skill Reviewer is a review tool aimed at Claude Skill authors. By systematically inspecting a `SKILL.md` and its companion files, it helps authors surface quality issues, normalize conventions, and clarify the appropriate sharing tier before publishing.

The inspection rules are distilled from two core Anthropic documents, covering 22 dimensions including description conventions, body structure, script quality, error handling, validation mechanisms, and test coverage.

## References

| Document | Description | Local Copy |
|----------|-------------|------------|
| [Skill Authoring Best Practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) | Anthropic's official authoring guide | `docs/Skill authoring best practices.md` |
| [Introduction to Claude Skills](https://github.com/anthropics/anthropic-cookbook) | Anthropic Cookbook primer | `docs/Introduction to Claude Skills.md` |

In addition, the `docs/` directory contains two digested documents (one in English, one in Chinese) for quickly reviewing the key points.

## Review Dimensions

The checklist contains 22 items, grouped into three categories:

### 1. Core Quality (Items 1–10)

- Description is precise and includes both "what it does" and "when to use it"
- `SKILL.md` body is kept under 500 lines
- Supplementary content is split into separate files, keeping the main file lean
- No time-sensitive information is included (e.g., API version numbers, specific dates)
- Terminology is consistent across description, body, scripts, and error messages
- Examples are concrete and runnable, showing real inputs and outputs
- File references stay within 2 levels of nesting
- Reflects the three-layer Progressive Disclosure structure
- Multi-step processes use numbered lists or decision trees

### 2. Code & Scripts (Items 11–18)

- Scripts do real work (parsing, transforming, validating), not empty pass-through shells
- Error messages clearly explain "why it failed" and "how to fix it"
- Constants and timeout values include rationale comments, avoiding "magic numbers"
- Dependencies are explicitly listed and verified at startup
- Every script includes purpose, arguments, return values, and examples
- Paths use forward slashes consistently (no Windows backslashes)
- Critical operations (file writes, network requests, etc.) include success verification
- Quality-sensitive tasks include a self-evaluation feedback loop

### 3. Testing & Evaluation (Items 19–22)

- At least 3 evaluation cases covering typical / boundary / error paths
- Tests pass on Haiku, Sonnet, and Opus
- Use real-world inputs rather than idealized prompts
- In multi-person collaboration, complete at least one independent review pass

## Installation

Clone this repository into Claude Code's skills directory:

```bash
git clone <repo-url> ~/.claude/skills/skill-reviewer
```

Alternatively, copy the directory into any skill load path; Claude Code will detect it automatically at startup.

## Usage

In a Claude Code session, trigger via natural language or slash command:

```text
Please use skill-reviewer to inspect the ~/skills/my-skill/ directory
```

```text
/skill-reviewer
```

### Supported Input Modes

| Mode | Input | Inspection Scope |
|------|-------|------------------|
| Directory mode | Path to a skill root | Full review, including file structure, nesting depth, and path style |
| Text mode | Pasted `SKILL.md` content | Text-only review; structural items are marked as `SKIPPED` |

### Output Format

The review report is split into three sections, followed by a self-check confirmation:

1. **Tiered issue list** — Issues are categorized by severity as Critical / Important / Minor; each entry includes an ID, location, description, and remediation suggestion
2. **Complete 22-item checklist** — Each item shows `PASS` / `FAIL` / `SKIPPED` / `N/A` with a brief note
3. **Sharing tier recommendation** — A clear conclusion based on the table below
4. **Self-check statement** — Report ends with `Self-check passed.`, confirming the four self-check points (specificity / completeness / spot-check / tier consistency) have passed

| Sharing Tier | Requirement |
|--------------|-------------|
| Personal use | Items 1–3 PASS, no Critical failures |
| Team sharing | Core quality (Items 1–10) all PASS, with ≥ 3 evaluation cases |
| Public distribution | All 22 items PASS, multi-model testing, team feedback obtained |

## Directory Structure

```
skill-reviewer/
├── README.md                                       # Chinese version (default)
├── README.en.md                                    # English version of this file
├── SKILL.md                                        # Review process and 22-item checklist
├── references/
│   └── writing-good-skills.md                      # Quick reference used during review
├── evals/                                          # Regression test fixtures
│   ├── README.md                                   # Evaluation guide
│   ├── results.md                                  # Real runs and multi-model test records
│   ├── good-skill-example/                         # Well-formed example
│   ├── medium-skill-example/                       # Contains some Important issues
│   └── bad-skill-example/                          # Contains multiple Critical issues
└── docs/
    ├── Skill authoring best practices.md           # Official guide source
    ├── Introduction to Claude Skills.md            # Cookbook source
    ├── how-to-write-good-claude-skills.md          # English digest
    └── how-to-write-good-claude-skills-cn.md       # Chinese digest
```

Each `evals/*-skill-example/` contains a `SKILL.md` (the subject under review) and an `expected-report.md` (the expected output), used for regression validation and multi-model comparison. Actual run results, model differences, and independent review records are written to [`evals/results.md`](evals/results.md). See [`evals/README.md`](evals/README.md) for details.

## Use Cases

- **Pre-publish self-check** — A final quality review before publicly publishing a Skill
- **Team collaboration** — Unify Skill authoring conventions and reduce review communication overhead
- **Learning reference** — Understand Anthropic's recommended Skill authoring style
- **Legacy upgrade** — Normalize and upgrade existing Skills to the current standard

## Feedback & Contributions

If you find inspection items inaccurate, rules that need extension, or deviations from the latest Anthropic documentation, please open an Issue or submit a Pull Request.
