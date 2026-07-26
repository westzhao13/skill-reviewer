---
name: writing-good-skills
description: >
  Cross-comparison reference for skill-reviewer audits. Distills writing rules
  and anti-patterns that go beyond the 22-item Best Practices checklist. Use
  as deeper context during Phase 2 audits (when an item needs rationale), and
  as the "how to think" layer when reporting findings. Read when the user asks
  why a description is bad, why a script is bad, or how to make a skill more
  discoverable. NOT a substitute for the 22-item checklist in SKILL.md — it
  extends it with features F-23–F-32 and anti-patterns AP-10–AP-22: the
  original set from "How to write good Claude skills.md", observed-in-practice
  patterns, plus Claude Code Lessons (F-28–F-32, AP-21–AP-22).
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

**Concrete anchor examples** (use as references when the user asks
"show me what good looks like"):

| Pattern | Example | Why it works |
|---------|---------|--------------|
| Specific + 4 keywords | `"Creates branded PDF reports from markdown — use when .pdf, weekly-report, invoice, formal-document"` | All four anchors present; what + when explicit |
| Verb-led + domain | `"Parses CSV/JSON/XML logs with schema inference — use when log analysis, data cleaning, format conversion"` | Concrete verbs; domain is named not gestured at |
| Anti-anchor (too broad) | `"Helps with documents"` | Should be flagged in audit; covers everything, serves nothing |
| Anti-anchor (no `use when`) | `"Creates PDF reports"` | No trigger condition; Claude cannot tell when to load |

The first row is the recommended template whenever the user asks for
a starting point.

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

## Features NOT in the 22-item checklist (F-23–F-32)

F-23–F-27 are from "How to write good Claude skills.md". F-28–F-32 are from
[Lessons from Building Claude Code - How We Use Skills](../docs/Lessons%20from%20Building%20Claude%20Code%20-%20How%20We%20Use%20Skills.md)
(advisory in Phase 2.1 unless the user asks for full / Claude Code lessons mode).

### F-23. Description as routing signal (extends #1, #2)

See "The revolutionary insight" above. Critical severity — impacts
discoverability. When auditing #1 or #2 FAILs, ask: would this description
route Claude to this skill from a 100-skill pool?

### F-24. Placeholder paths, not hardcoded paths

- ✅ Good: `${SKILL_ROOT}/wiki/INDEX.md`
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

Also check **named skill dependencies** (Claude Code Lessons): if the body
says "use the pdf skill" / "invoke X", cite the dependency by skill `name`
so marketplace installs know what else is required. Native dependency
graphs are not built into skills yet — naming is the contract.

**Severity:** Critical when sole-process assumptions break others;
Important when named dependencies are implied but not stated.

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

### F-28. Skill-type fit (from Claude Code Lessons, 2026-03)

Anthropic's operational taxonomy: Library, Verification, Data, Automation,
Scaffolding, Code Quality, CI/CD, Runbooks, Infra Ops. Best skills fit **one**
type; confusing skills straddle several unrelated jobs.

**Type-appropriate scripts:** For Verification, Code Quality, Data, and Infra
Ops, if the body repeats shell one-liners / greps and there is no `scripts/`
folder, recommend extracting helpers (extends #11 — ask whether scripts
*should* exist, not only whether existing scripts are good).

**Audit signal:** Description promises standup + deploy + PDF in one skill;
body mixes unrelated workflows with no clear primary purpose.

**Severity:** Important for team/public; Minor for personal.

**When auditing:** Ask which single type applies; suggest split if unclear.

### F-29. Setup path for external integrations (from Claude Code Lessons)

Skills that post to Slack, deploy, call external HTTP APIs, or use credentials
should document first-run setup — often `config.json` / `config.example.json`
plus AskUserQuestion for structured choices. Environment variables
(`TIMEOUT=…`, API keys) also count as setup: they must be listed with defaults
and meaning, not only bare names.

**Audit signal:** Skill calls external APIs with no setup section and no
config template (or lists env vars with no rationale / install path).

**Severity:** Important when external side effects exist; N/A for pure docs.

### F-30. Durable memory location (from Claude Code Lessons)

Skills that keep history (logs, prior runs) should not assume data in the skill
folder survives upgrades. Prefer `${CLAUDE_PLUGIN_DATA}` or an explicit external
path documented in SKILL.md.

**Audit signal:** `standups.log` or state JSON written only under the skill
directory with no upgrade warning.

**Severity:** Important when skill is stateful; N/A when stateless.

### F-31. On-demand hooks where automation helps (from Claude Code Lessons)

**Verification**, Code Quality, and Infra Ops skills may register session hooks
(e.g. block destructive Bash when skill loads, or re-run checks on save).
Advisory only — not every skill needs hooks.

**Audit signal:** Verification / Code Quality / Infra Ops skill with
manual-only workflow where hooks would obviously help (PR gate, save-time
lint). Pure instructional Library skills: usually N/A.

**Severity:** Minor (suggestion); never Critical by default.

### F-32. Usage measurement for org-wide skills (from Claude Code Lessons)

Team/public skills benefit from trigger logging (e.g. PreToolUse hook) to
detect undertriggering vs description expectations.

**Audit signal:** Marketplace-bound skill with no mention of how owners track
usage.

**Severity:** Minor for team; Important for public distribution at scale.

## Anti-patterns NOT in the 22-item checklist (AP-10–AP-22)

### From synthesis (AP-10–AP-17)

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

#### AP-13. Encyclopedia-style skill / stating the obvious

Stuffing Wikipedia-level content into SKILL.md, squeezing the 5k token
budget. Claude Code Lessons: knowledge skills should push Claude *out* of
default opinions — org-specific facts, footguns, taste — not re-teach
generic CS.

**Audit signal:** SKILL.md explains background concepts Claude already
knows (what JSON is, what a REST API is). For Library / Reference types,
sample 2–3 paragraphs: keep only lines that would be lost if deleted
(org-specific). Cite deletable lines in the report.

**Severity:** Important (violates "Concise is key" / "Don't state the
obvious").

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

### Observed in practice (AP-18–AP-20) + Claude Code Lessons (AP-21–AP-22)

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

#### AP-21. No Gotchas section (from Claude Code Lessons)

Article: "The highest-signal content in any skill is the Gotchas section."

**Audit signal:** Library, Verification, or Scaffolding skill with domain
rules but no `## Gotchas` / footguns / common-mistakes section. Example:
a markdown link-checker (Verification) that never lists HEAD-timeout,
redirect loops, or relative-path pitfalls.

**Severity:** Important for types that accumulate edge cases (especially
Verification / Library); Minor for trivial one-shot skills.

#### AP-22. Over-rigid workflow (railroading)

Article: give Claude information but flexibility to adapt. Conflicts with
#10 when taken to extremes.

**Audit signal:** Single input path only; 20+ mandatory micro-steps with no
conditionals; no "if pasted text / if directory" variants.

**Severity:** Important when skill is meant for varied real prompts; Minor
when consistency is the point (e.g. regulated audit output).

## Severity calibration quick reference

| Pattern | Severity |
|---|---|
| F-23 (description as routing signal) | Critical |
| F-24 (placeholder paths) | Minor |
| F-25 (composability) | Critical when violated |
| F-26 (testing tier) | Important (gating) |
| F-27 (error 3-element) | Important |
| F-28 (skill-type fit) | Important / Minor by tier |
| F-29 (setup path) | Important when external |
| F-30 (durable memory) | Important when stateful |
| F-31 (on-demand hooks) | Minor (advisory) |
| F-32 (usage measurement) | Minor–Important by tier |
| AP-10 to AP-17 | Critical to Important |
| AP-18 | Minor |
| AP-19 | Critical |
| AP-20 | Important |
| AP-21 (no Gotchas) | Important / Minor |
| AP-22 (railroading) | Important / Minor |

## How to use this reference during an audit

1. **Phase 2 of SKILL.md** runs the 22-item checklist. For each FAIL,
   consult this file to understand *why* and *how to fix*.
2. **Phase 2.1 (advisory)** — By default, F-28–F-32 and AP-21–AP-22 do
   **not** change the 22/22 pass rate. Report them under Part A with the
   `[claude-code-lessons]` tag when relevant. Use full / Claude Code
   lessons mode only when the user asks.
3. **Phase 3 report** — when citing a finding, use feature and
   anti-pattern numbers alongside the 22-item numbers. Format:
   `[#3 + AP-13]` means "fails the 500-line check AND shows
   encyclopedia-style content". Cite F-23–F-32 and AP-10–AP-22 as needed.
4. **Gating recommendation** — F-26 changes the recommendation based on
   sharing scope. Don't apply public-grade standards to a personal skill.
5. **User follow-up** — when the user asks "is my skill discoverable?" or
   "why isn't this working?", the routing-signal insight (top section) is
   the most common answer.

---

## Priority matrix (fix in this order for maximum ROI)

When many FAILs accumulate, fix them in this layered order. Each
layer's fixes are higher leverage than the next.

| Priority | Layer | Item numbers | Why here |
|----------|-------|--------------|----------|
| L1 (highest) | User-perception | #1, #2, #6, #7, AP-11, AP-12, AP-13, AP-20, F-28 | Description, terminology, and type-fit are loaded into every session. Improving them multiplies across all triggers. Empirical cluster: ~80% of skill audit FAILs originate here. |
| L2 | Engineering | #3, #11, #12, #15, #17, AP-21, F-29 | Scripts, errors, Gotchas, and setup determine whether the skill works when called. Required for personal use beyond toy projects. |
| L3 | Collaboration | #19, #20, #21, #22, F-32 | Evals, multi-model coverage, team feedback, usage measurement. Required for team share and public distribution per the sharing-tier gating. |
| L4 (lowest) | Hygiene | #4, #8, #9, #10, #13, #14, #16, #18, F-24, F-30, F-31, AP-22 | Path style, nesting, progressive disclosure, hooks, memory location, railroading. Each individually minor; total improvement matters once L1-L3 are stable. |

**Heuristic:** if the audit is delivering ≥15 FAILs, at least 8 of them
typically share a root cause inside L1. Fix L1 first, re-run, and watch
the FAIL count drop sharply. This is the "fix-one-fix-many" pattern.

---

## Source attribution

This reference distills insights from:
- **Anthropic official [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)** — the 22-item source
- **[Lessons from Building Claude Code - How We Use Skills](../docs/Lessons%20from%20Building%20Claude%20Code%20-%20How%20We%20Use%20Skills.md)** — operational taxonomy, Gotchas, hooks, memory (F-28–F-32, AP-21–AP-22)
- **`How to write good Claude skills.md`** — the community synthesis with
  `[S]/[E]/[CK]` label system
- **Adversarial review pass** — 3 patterns (AP-18, AP-19, AP-20) observed
  in practice, not present in either source

All 22 checklist items in SKILL.md are from Anthropic's official doc
(marked `[S]`). Features F-23–F-27 and anti-patterns AP-10–AP-17 are marked
`[E]` (author extension) in the source synthesis. AP-18–AP-20 are audit-
practice observations. F-28–F-32 and AP-21–AP-22 are from Claude Code
Lessons (Phase 2.1 advisory).
