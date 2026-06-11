---
name: writing-good-skills
description: >
  Cross-comparison reference for skill-reviewer audits. Distills writing rules
  and anti-patterns that go beyond the 22-item Best Practices checklist. Use
  as deeper context during Phase 2 audits (when an item needs rationale), and
  as the "how to think" layer when reporting findings. Read when the user asks
  why a description is bad, why a script is bad, or how to make a skill more
  discoverable. NOT a substitute for the 22-item checklist in SKILL.md — it
  extends it with the 5 additional features and 11 additional anti-patterns
  from "How to write good Claude skills.md" plus 3 observed-in-practice patterns.
---

# Writing Good Claude Skills — Cross-Comparison Reference

Companion to the 22-item checklist in `../SKILL.md`. SKILL.md answers
**what to check**; this file answers **how to think** about the check.

## The one-line formula

**好 skill = 触发信号(description) + 判断规则(指令) + 工具(脚本)**

Any one written poorly → skill degrades from "expert" to "redundant prompt".

## The revolutionary insight: description is a routing signal

Anthropic's official docs say description should include "what + when".
This synthesis goes further: **description is the routing signal** —
it's the only metadata Claude sees pre-loaded alongside `name`. Of 100+
available skills, the one whose description's keywords and scenarios
match the user's request gets loaded.

**Four failure modes to flag in audits:**

| Failure | Example | Why it fails |
|---|---|---|
| Too broad | "helps with documents" | Triggered for everything → useful for nothing |
| Too narrow | "only for Q3 sales reports" | Almost never triggered |
| Feature description (no triggers) | "Creates PDF reports" | Claude can't tell when to load |
| Body/scenario mismatch | Description mentions `.pdf`, body has no PDF example | Routing signal decoupled from content |

**Good baseline:** `"Creates branded PDF reports from markdown — use when
.pdf, weekly-report, invoice, formal-document"` — what + keywords +
triggering scenarios in one sentence.

## 4-step process for writing a good skill

Use this when the user is **writing** a skill (not just reviewing):

1. **Run the bare task** with Claude A — observe where it gets stuck or
   asks for context Claude should have known
2. **Extract 5–10 conditional rules** from the gaps — `if X, then Y` style,
   not narrative paragraphs
3. **Write 3 eval cases FIRST** — typical + edge + error paths, *before*
   writing the docs
4. **Optimize description last** — once rules and scripts are stable, write
   the description that routes Claude to the skill

Step 3 is the most-skipped and most-important: without evals, the
description and rules are guesses.

## 5 features NOT in the 22-item checklist

(B's unique contributions from "How to write good Claude skills.md".)

### F-23. Description as routing signal (extends #1, #2)

See "The revolutionary insight" above. Critical severity — impacts
discoverability. When auditing #1 or #2 FAILs, ask: would this description
route Claude to this skill from a 100-skill pool?

### F-24. Placeholder paths, not hardcoded paths

- ✅ Good: `${WIKIBASE_ROOT}/wiki/INDEX.md`
- ❌ Bad: `/home/user1/project/wiki/INDEX.md`

**Why:** Portability across deployments. Distinguish from item #16 (path
separator): #16 is about `/` vs `\`, F-24 is about literal-path vs
placeholder.

**Severity:** Minor (deploy-dependent).

### F-25. Composability — don't assume sole-process

The skill runs alongside 100+ others. Violations:
- Reading global state on startup
- Locking exclusive files
- Hardcoded cwd assumptions
- Claiming exclusive env vars
- Requiring specific shell history / shell state

**Severity:** Critical when violated (breaks other skills' assumptions).

### F-26. Tiered testing requirements (extends #19)

| Sharing scope | Minimum test requirement |
|---|---|
| Personal use | ≥1 end-to-end test |
| Team share | ≥3 eval cases (typical + edge + error) |
| Public distribution | 3+ evals + multi-model (item #20) + team feedback (#22) |

**Why:** Cost/benefit differs at each tier. Don't require public-grade
testing for personal skills.

**Severity:** Important (gating recommendation depends on tier).

### F-27. Error messages follow the 3-element pattern (extends #12)

Good error messages answer:
1. **What** failed
2. **Why** it failed
3. **How** to fix

| Bad | Good |
|---|---|
| `Error occurred` | `Error parsing config.yaml at line 12: expected 'key: value' but found '- item' (YAML list not valid here). Check indentation.` |
| `An unexpected error has occurred.` | `Cannot connect to database at localhost:5432. Is postgres running? Try: sudo systemctl start postgresql` |

**Severity:** Important (errors are how Claude self-corrects).

## 11 anti-patterns NOT in the 22-item checklist

### B's 8 unique anti-patterns

#### AP-10. "Good prompt ≠ good skill"

Copying a well-crafted prompt directly as SKILL.md loses the progressive
disclosure cost advantage. The skill should be routing metadata + lean
rules, not a 5000-token prompt.

**Audit signal:** SKILL.md reads like a single long prompt with no
table-of-contents structure.

**Severity:** Critical (defeats the skill system's purpose).

#### AP-11. Description too broad

"helps with PDF generation" — triggered for everything, useful for nothing.

**Severity:** Critical (impacts discoverability + noise).

#### AP-12. Description too narrow

"Only for Q3 sales reports" — almost never triggered.

**Severity:** Critical (kills discoverability).

#### AP-13. Encyclopedia-style skill

Stuffing Wikipedia-level content into SKILL.md, squeezing the 5k token
budget. **Audit signal:** SKILL.md explains background concepts Claude
already knows (what JSON is, what a REST API is).

**Severity:** Important (violates "Concise is key" principle).

#### AP-14. Helper script inlined as "example"

Spending the 5k token budget on full script code in SKILL.md.
**Audit signal:** Long code blocks (>30 lines) in SKILL.md body that
could be `scripts/foo.py` with a one-line reference.

**Severity:** Important (wastes the token budget).

#### AP-15. Silent exception swallowing

`except: return ""` or `except Exception: pass` — Claude gets empty data
and debugs into confusion.

**Audit signal:** Empty `except:` blocks, `try/except` with no re-raise
or no informative error.

**Severity:** Critical (violates the "explicit error handling" principle;
directly harms Claude's ability to recover).

#### AP-16. Sole-process assumption

Hardcoding env vars, exclusive files, or cwd expectations. Breaks when
other skills run in the same environment.

**Audit signal:** References like `os.environ["MY_SKILL_STATE"]` without
namespacing, `os.chdir("/abs/path")` calls, exclusive file locks.

**Severity:** Critical (composability violation).

#### AP-17. No tests, "should work" reasoning

"Should be fine" is not a verification standard. The skill needs at
least 1 end-to-end test for personal use, 3+ for team/public.

**Audit signal:** No `evals/`, no `test_*.sh`, no examples of the skill
actually running successfully.

**Severity:** Critical for team/public; Important for personal.

### My 3 additional observations (not in A or B)

#### AP-18. Mixed Markdown list styles

Mixing `- [ ]`, `1.`, `*`, `-` inconsistently across sections. Claude's
list-parsing can get confused about nesting and counts.

**Audit signal:** Inconsistent bullet character or numbering within a
single file.

**Severity:** Minor.

#### AP-19. Missing or empty frontmatter description

Skills without a `description` field (or with an empty one) don't appear
in the available skills list. Silent failure.

**Audit signal:** `description: ""` or frontmatter missing the
`description:` line entirely.

**Severity:** Critical (skill is invisible to Claude).

#### AP-20. Example/main language or scenario mismatch

Body in Chinese, examples in English, triggering scenarios don't align
with description's keywords.

**Audit signal:** Description mentions trigger X; SKILL.md has no
example of X.

**Severity:** Important (decouples routing signal from content).

## Severity calibration quick reference

| Pattern | Severity |
|---|---|
| F-23 (description as routing signal) | Critical |
| F-24 (placeholder paths) | Minor |
| F-25 (composability) | Critical when violated |
| F-26 (testing tier) | Important (gating) |
| F-27 (error 3-element) | Important |
| AP-10 to AP-17 | Critical to Important |
| AP-18 | Minor |
| AP-19 | Critical |
| AP-20 | Important |

## How to use this reference during an audit

1. **Phase 2 of SKILL.md** runs the 22-item checklist. For each FAIL,
   consult this file to understand *why* and *how to fix*.
2. **Phase 3 report** — when citing a finding, use the additional features
   (F-23 to F-27) and anti-patterns (AP-10 to AP-20) numbers alongside the
   22-item numbers. Format: `[#3 + AP-13]` means "fails the 500-line check
   AND shows encyclopedia-style content".
3. **Gating recommendation** — F-26 changes the recommendation based on
   sharing scope. Don't apply public-grade standards to a personal skill.
4. **User follow-up** — when the user asks "is my skill discoverable?" or
   "why isn't this working?", the routing-signal insight (top section) is
   the most common answer.

## Source attribution

This reference distills insights from:
- **Anthropic official [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)** — the 22-item source
- **`How to write good Claude skills.md`** — the community synthesis with
  `[S]/[E]/[CK]` label system
- **Adversarial review pass** — 3 patterns (AP-18, AP-19, AP-20) observed
  in practice, not present in either source

All 22 checklist items in SKILL.md are from Anthropic's official doc
(marked `[S]`). The 5 features (F-23 to F-27) and 8 anti-patterns (AP-10
to AP-17) are marked `[E]` (author extension) in the source synthesis.
AP-18 to AP-20 are new observations from audit practice.
