---
title: "How to write good Claude skills"
source:
  - "Introduction to Claude Skills.md (Anthropic Cookbook)"
  - "Skill authoring best practices.md (Anthropic official checklist)"
created: 2026-06-11
updated: 2026-06-11
tags:
  - "skills"
  - "claude"
  - "best-practices"
description: "Synthesis of two Anthropic sources (Cookbook + official Best Practices checklist) into actionable writing rules and a share-ready review checklist. Uses [S]/[E]/[CK]/[F] tags throughout. Result of adversarial-review + debate across three subagents. English translation of the Chinese source-of-truth at how-to-write-good-claude-skills-cn.md"
---

# How to write good Claude skills

Writing rules distilled after three passes of summary over
`Introduction to Claude Skills.md` (Anthropic Cookbook, 836 lines).

## Reading guide

This document uses the following tags on every claim so readers can
**distinguish source text from author opinion at a glance**:

- `[S]` = Source has a direct statement or strong implication (with line number)
- `[E]` = Reasonable extension inferred from source spirit (author opinion)
- `[CK]` = Common-knowledge addition (not in source, but general engineering convention)
- `[F]` = Fabrication with no source basis (all removed)

**Section coloring:** Pass 1 is almost entirely `[S]`; Pass 2 mixes tags;
Pass 3 is almost entirely `[E]/[CK]`; Pass 4 is pure author synthesis,
**not a source summary**.

---

## Pass 1: Source skeleton (factual layer)

The document revolves around five things (all `[S]`):

1. **What Skills are** — `[S]` "a package of instructions, executable code,
   and resources" (line 182), one level above a single tool (line 190),
   an "expertise package" (line 182).
2. **Three-tier loading model** — `[S]` (line 209-211):
   - **Tier 1**: YAML frontmatter's `name` (≤64 chars) + `description`
     (≤1024 chars), always visible.
   - **Tier 2**: Full instructions (<5k tokens), loaded only when relevant.
   - **Tier 3**: Linked auxiliary files, loaded on demand.
3. **Two types + one key property** — `[S]`:
   - **Two types** (line 220-222 table): Anthropic-managed (xlsx/pptx/pdf/docx),
     Custom (user workflows).
   - **One key property**: Composable — multiple skills work together
     (line 203, 233). Note: the source lists "composability" under
     Benefits, not Types.
4. **Usage constraints** — `[S]` (line 241-253, 265-272): must use
   `client.beta.messages.create()`, must pair with `code_execution_2025-08-25`
   tool, `betas` array must include `skills-2025-10-02`.
5. **Token economy** — `[S]` (line 278-289):
   - Manual instructions: 5,000–10,000 tokens/request (line 280).
   - Skill (metadata only): nearly free (line 281).
   - Skill (fully loaded): ~5,000 tokens/trigger (line 282).
   - Case: explaining all of Excel needs ~8,000 tokens; with a skill,
     you only pay ~5,000 at call time (line 288-289).

---

## Pass 2: Source hard constraints + author inference

### Hard constraints (explicitly given in source)

- `[S]` `name` ≤ 64 chars, `description` ≤ 1024 chars (line 209).
- `[S]` Body instructions < 5,000 tokens (line 210).
- `[S]` Use `"latest"` to pull Anthropic-managed skills; custom skills
  use epoch timestamps (line 370-373).
- `[S]` Helper scripts are a core value of skills: tested, immediately
  usable code (line 198).
- `[S]` Skills are not just a prompt — they combine instructions, code,
  and resources (line 190).

### Inference (author extrapolation, consistent with source spirit)

> **Note:** Items tagged `[E]` below are reasonable inferences from the
> source spirit, **not hard rules from the source**.

- `[E]` `description` should be written like a "routing signal" — it
  determines when Claude decides to load the full content. Source line
  209 only says "Claude sees name + description", but since `description`
  is the only always-visible metadata, writing it as "trigger conditions"
  has a much larger impact on auto-load hit rate than writing it as
  "feature description".
  - `[CK]` Don't write too broad: counter-example "A useful skill for
    various tasks".
  - `[CK]` Don't write too narrow: counter-example "Only for Q3 sales
    reports".

- `[E]` The <5k-token body constraint is hard, but **where the budget
  goes** matters more:
  - `[E]` Assume Claude knows the basics (don't explain JSON/markdown).
  - `[E]` Give concrete decision rules ("if X, do Y"), not abstract advice.
  - `[E]` Include directly-reusable code snippets; just describing
    "library X can do Y" wastes tokens.

- `[E]` Helper scripts engineering conventions:
  - `[CK]` Frequently-reused logic/transforms conventionally live in
    `scripts/`.
  - `[CK]` Complex multi-step flows are wrapped as single-entry commands.
  - `[E]` Principle: if 5 lines of shell can do it, don't make Claude
    rewrite it every time (this is the author's reading of line 198's
    "tested, working code").

- `[E]` Engineering implications of composability: when writing a skill,
  assume it will run alongside other skills — don't write absolute paths
  in frontmatter, don't assume you're the only one in the environment
  (source line 233 only states "composability" as a property, it doesn't
  give concrete "how to achieve it" advice).

### Additional hard constraints (from the official Best Practices Checklist)

> **Note:** The constraints below come from Anthropic's official
> [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
> (56-line checklist), **not** the Cookbook (836-line Introduction)
> above. Both documents are complementary parts of the same knowledge
> system; all items are tagged `[S]`.

**Core quality:**

- `[S]` **SKILL.md body < 500 lines.** Independent of the <5k-token
  constraint from the Cookbook — the former is maintainability (long
  files are hard to review and modify), the latter is the API loading
  budget. ~250–400 lines is usually enough for a focused domain.
- `[S]` **File references no more than one level deep.** Reference
  `scripts/validate.py`, not `tools/helpers/submodules/checker.py`. Deep
  paths break during packaging and migration.
- `[S]` **No time-sensitive information.** API version numbers, specific
  dates, temporary workarounds. If a skill says "as of June 2025", the
  info is already or about to be outdated. For time-sensitive content:
  delete it, or put it in a dedicated `## Old patterns` section with
  validity period noted.
- `[S]` **Consistent terminology throughout.** The same concept uses
  the same word in description, SKILL.md body, script comments, and
  error messages. Don't call it "processor" in the description and
  "engine" in the script.
- `[S]` **Examples are concrete and runnable.** Not "Create a chart",
  but "Create a bar chart showing monthly revenue for Q1 2025 with data
  labels on each bar". Ideally examples can be copied directly as prompts.
- `[S]` **Workflows have clear steps.** Multi-step instructions cannot
  use consecutive paragraphs — use numbered lists, sub-headings, or
  decision trees. Each step does one thing. This is consistent with
  Cookbook spirit but the Checklist makes it an explicit hard requirement.

**Code and scripts:**

- `[S]` **No "voodoo constants" — every value has a reason.** Every
  number, string, or path that isn't self-explanatory must have a
  comment explaining why. Counter-example: `timeout=30`. Positive
  example: `timeout=30  # max seconds for PDF render; longer runs
  indicate resource leak`.
- `[S]` **Required dependencies listed in instructions and verified
  available.** SKILL.md should have a Prerequisites section listing all
  dependencies (Python packages, system tools, API keys), and verify at
  script entry with `try/except`, returning install guidance on failure.
- `[S]` **Scripts have clear documentation.** Each script starts with:
  one-line description, parameter list, return values/exit codes, one or
  two usage examples. This lives in the script file itself, not duplicated
  in SKILL.md.
- `[S]` **No Windows-style paths (all forward slashes).** Backslashes in
  `scripts\tools\check.py` are parsed as escape characters in Claude's
  Linux execution environment, causing immediate errors. Standardize on
  `scripts/check.py`.
- `[S]` **Critical operations have validation/confirmation steps.**
  Read-back verification after writes, sanity check after calculations,
  status code check after network requests. For irreversible operations
  (delete, overwrite), require confirmation before execution.
- `[S]` **Quality-critical tasks include feedback loops.** If the skill's
  output needs human judgment (document generation, code review), include
  a self-assessment step: Claude checks its own output and flags
  improvements before delivery.

**Testing and evaluation:**

- `[S]` **At least 3 evaluation cases.** Each covers: (1) most typical
  scenario, (2) edge case (empty input, oversized input), (3) error path
  (missing dependency, invalid config).
- `[S]` **Tested with Haiku, Sonnet, and Opus.** Different models behave
  differently on instruction following, output length, error modes.
  Minimum: at least one fast (Haiku) and one capable (Sonnet/Opus).
- `[S]` **Tested with real usage scenarios.** Test cases can't just be
  "idealized prompts". Use the phrases real users use, real file formats,
  real error data. If possible, let target users try it and observe where
  they get stuck.
- `[S]` **Incorporate team feedback (if applicable).** Multi-user skills
  should have at least one independent review — not a code review, but
  "does this person understand when the skill should be used and how".

**Note on testing standards:** Existing documentation (Pass 3 comparison
table, line 94 in source) says "at least one end-to-end test" (`[E]`),
which is the **personal use** minimum. The Checklist requires 3+ eval
cases + multi-model + real-scenario testing as the **pre-share**
recommended standard. The two don't contradict — they are different
standards for different scenarios. Recommended tiering:

- Personal-use skill: ≥1 end-to-end test.
- Team-shared skill: ≥3 eval cases, covering typical + edge + error paths.
- Public-distributed skill: 3+ eval cases + multi-model testing + team review.

---

## Pass 3: Author synthesis (engineering experience, not source)

> **Note:** Each cell in the table below uses tags to distinguish source.
> **All `[E]/[CK]` are the author's engineering extrapolation, not the
> Cookbook source.** The source's explicit hard constraints are only:
> name ≤64 chars, description ≤1024 chars, body instructions <5k tokens,
> use `"latest"` instead of fixed versions.

| Dimension | ✅ Good skill | ❌ Bad skill |
| --- | --- | --- |
| **YAML description** | `[E]` One sentence hits "do Y in scenario X" with clear trigger signals | `[E]` "A useful skill for various tasks" (vague) |
| **Instruction length** | `[S]` <5k tokens; `[E]` each section is decision rules or code | `[E]` Stuffing a whole domain tutorial in; or conversely <1k tokens without saying "when to use" |
| **Helper scripts** | `[S]` Includes tested, working helper code; `[CK]` conventionally in `scripts/` | `[E]` "Run a shell command to get X" — Claude re-assembles every time |
| **Path handling** | `[CK]` Use placeholders like `${WIKIBASE_ROOT}`, expand at deploy time | `[CK]` Hardcoded `/home/user1/project/...` — breaks on user change |
| **Error handling** | `[E]` Fail early, error message points to specific cause | `[E]` Swallow exception and return empty string, or throw "An error occurred" |
| **Composability** | `[E]` Don't read others' global state, don't lock exclusive files, don't depend on cwd | `[E]` Assume you're the only process, require exclusive env vars |
| **Testing** | `[E]` At least one end-to-end bash/python test | `[E]` No tests at all, "should work" |
| **Frontmatter metadata** | `[S]` Includes name + description; `[CK]` optional version field | `[E]` Has only description, no name; or name uses underscore with version suffix |
| **Examples** | `[S]` Source provides both "Simple (1-2 lines)" and "Detailed" sets (line 379, 390) | `[E]` Only one prompt, or no concrete examples |
| **Boundary conditions** | `[S]` Python 3.8+ is a prerequisite (line 21); `[E]` Explicitly state "assumes state X" | `[E]` Prerequisite unstated |

### Three places easiest to write wrong

1. **"A well-written prompt" ≠ skill** — `[S]` The former is a consumable
   (line 280 manual instructions 5k–10k), the latter is metadata-resident
   (line 281). `[E]` "The biggest temptation" is the author's emphasis:
   if you directly copy a good prompt as SKILL.md, you lose the cost
   advantage of progressive disclosure.

2. **YAML description written as "feature description" rather than
   "trigger conditions"** — `[E]` Source line 209 only says description
   is the metadata Claude sees, with no explicit "trigger conditions"
   language; this insight is the author's engineering extrapolation, but
   it has the largest impact on auto-load hit rate.
   - `[CK]` Counter-example: `"helps with PDF generation"` (too broad)
     vs `"creates branded PDF reports from markdown (use when user asks
     for .pdf, weekly-report, or formal-document)"` (has trigger signal).

3. **Helper scripts treated as "examples" placed in SKILL.md** — `[E]`
   The 5k token budget is real (line 210), but "don't inline" is the
   author's inference. Recommend SKILL.md only references script paths,
   **does not inline** full code.

---

## Author synthesis (not a source summary)

> **Important:** The formula below is the author's compression of
> Cookbook spirit + their own engineering experience, **not from the
> source**. The source only says description is the metadata Claude
> sees, instructions <5k tokens, helper scripts are tested-and-usable.

> **Good skill = accurate trigger signal (description) + tight decision
> rules (instructions) + reusable tools (scripts).**

If any one of these is written poorly, the skill degrades from "expert"
to "redundant prompt" — this inference comes from the token comparison
at line 280-282.

---

## Pass 5: Using the official Checklist as a review framework

> This section comes from Anthropic's official
> [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
> (56-line checklist). All items are marked `[S]` (source), author's
> extended readings marked `[E]`.

### How to use

This review table should be used **after** writing a skill, **before
sharing** — not item-by-item during writing. Process:

1. Draft SKILL.md following the rules from prior sections.
2. Before sharing, check against this table item by item.
3. For each item, record ✅ pass / ❌ fail / N/A not applicable. Fix
   fails before sharing.

### Core quality review table

| # | Item | Reading |
|---|------|---------|
| 1 | `[S]` Description contains specific key terms | Not "handles data processing" — "parses CSV/JSON/XML logs with schema inference (use when user asks about log analysis, data cleaning, or format conversion)". Key terms are intent-matching routing signals. |
| 2 | `[S]` Description states both "what" and "when" | Function scope + trigger conditions. Counter-example: "Creates PDF reports" (only what). Positive example: "Creates branded PDF reports from markdown — use when .pdf, weekly-report, invoice, formal-document". |
| 3 | `[S]` SKILL.md body < 500 lines | Independent of <5k token constraint. Former is maintainability, latter is API budget. ~250-400 lines usually enough for a focused domain. |
| 4 | `[S]` Additional details in separate helper files | Templates, configs, images, long-form references. SKILL.md keeps only "directing logic". |
| 5 | `[S]` No time-sensitive information (or in "old patterns" section) | Seeing "as of June 2025" means it's already or about to be outdated. Delete or move to `## Old patterns` with validity period. |
| 6 | `[S]` Consistent terminology throughout | description/SKILL.md/scripts/error messages use the same set of terms. |
| 7 | `[S]` Examples are concrete and runnable | Abstract: "Create a chart". Concrete: "Create a bar chart showing monthly revenue for Q1 2025 with data labels on each bar". Ideally examples can be copied directly as prompts. |
| 8 | `[S]` File references no more than one level deep | Reference `scripts/validate.py`, not `tools/helpers/submodules/checker.py`. Deep paths break during migration. |
| 9 | `[S]` Progressive disclosure used appropriately | Already covered (three-tier loading model, Pass 1). |
| 10 | `[S]` Workflows have clear steps | Numbered lists / sub-headings / decision trees. Each step does one thing. |

### Code and scripts review table

| # | Item | Reading |
|---|------|---------|
| 11 | `[S]` Scripts solve problems, don't punt to Claude | `[E]` If a script just calls the Claude API, that's a proxy, not a helper. Helper value is doing what Claude isn't good at: precise calculation, file I/O, format validation. |
| 12 | `[S]` Error handling explicit and helpful | `[E]` Not just "fail early". Error message should contain three elements: (1) what went wrong, (2) why, (3) how to fix. Positive example: `Error parsing config.yaml at line 12: expected 'key: value' but found '- item' (YAML list not valid here). Check indentation.` |
| 13 | `[S]` No "voodoo constants" — all values justified | `[E]` Every number/string/path must be explainable. Counter-example: `timeout=30`. Positive example: `timeout=30  # max seconds for PDF render; longer runs indicate resource leak`. |
| 14 | `[S]` Dependencies listed and verified available | `[E]` SKILL.md's Prerequisites section lists all dependencies. Script entry does `try/except` verification, returning install guidance on failure (not silent skip). |
| 15 | `[S]` Scripts have clear documentation | Script file itself writes: (1) one-line description, (2) parameters, (3) return values/exit codes, (4) 1-2 usage examples. Not in SKILL.md. |
| 16 | `[S]` No Windows paths — all forward slashes | `[CK]` Backslashes are escape characters in Claude's Linux environment. `scripts\tools\check.py` → `scripts/check.py`. This is **different** from the `${WIKIBASE_ROOT}` placeholder concern. |
| 17 | `[S]` Critical operations have validation/confirmation steps | Read-back verification after writes, sanity check after calculations, status code check after network requests. Irreversible operations (delete/overwrite) require confirmation before execution. |
| 18 | `[S]` Quality-critical tasks include feedback loops | If output needs human judgment (document generation, code review), include self-assessment step: Claude self-checks and points out improvements before delivery. |

### Testing review table

| # | Item | Reading |
|---|------|---------|
| 19 | `[S]` ≥3 evaluation cases | `[E]` Covers: (1) most typical scenario, (2) edge case, (3) error path. One case only proves "ran once". |
| 20 | `[S]` Tested with Haiku, Sonnet, and Opus | `[E]` Different models differ on instruction following, output length, error modes. At least one fast (Haiku) + one capable (Sonnet/Opus). |
| 21 | `[S]` Tested with real usage scenarios | Don't write "idealized prompts". Test with real user phrasings, real file formats, real error data. Have target users try and observe stuck points. |
| 22 | `[S]` Team feedback incorporated (if applicable) | At least one independent review — not code review, but "understands when the skill should be used and how". |

### Sharing-tier gating recommendations

| Sharing scope | Minimum requirement |
|----------|---------|
| Personal use | Sections 1-4 writing rules + ≥1 end-to-end test |
| Team shared | Above + core quality review table (10 items) + ≥3 eval cases |
| Public / distributed | Above + all 22 review items + multi-model testing + team review |

---

## Cheat sheet: what's "source", what's "author"

| Category | Content | Status |
| --- | --- | --- |
| Three-tier loading numbers | name ≤64, desc ≤1024, <5k tokens | `[S]` Cookbook |
| Skill definition and positioning | One level above a single tool | `[S]` Cookbook |
| Token economy comparison | 5k–10k vs metadata-only | `[S]` Cookbook |
| Versioning strategy | `"latest"` vs epoch | `[S]` Cookbook |
| Three Beta API identifiers | `code-execution-...`, `files-api-...`, `skills-...` | `[S]` Cookbook |
| SKILL.md < 500 lines | Line-count constraint (independent of token constraint) | `[S]` Checklist |
| File references one level deep | Path maintainability | `[S]` Checklist |
| No time-sensitive information | API versions/dates/workarounds to old patterns | `[S]` Checklist |
| Consistent terminology | Same terms throughout | `[S]` Checklist |
| Voodoo constants | All values must have reasons | `[S]` Checklist |
| Dependencies listed and verified | Prerequisites + startup self-check | `[S]` Checklist |
| No Windows paths | All forward slashes | `[S]` Checklist |
| Critical operation validation | Read-back verification after file writes | `[S]` Checklist |
| Feedback loops | Self-check for quality-critical tasks | `[S]` Checklist |
| ≥3 eval cases | Typical + edge + error | `[S]` Checklist |
| Multi-model testing | Haiku/Sonnet/Opus full coverage | `[S]` Checklist |
| Real-scenario testing | No idealized prompts | `[S]` Checklist |
| Team feedback | At least one independent review | `[S]` Checklist |
| Write description as "trigger conditions" | Engineering reading of routing signal | `[E]` author extrapolation |
| Where the instruction budget goes | "Decision rules" vs "background knowledge" | `[E]` author extrapolation |
| Helper scripts in `scripts/` | Directory layout convention | `[CK]` common knowledge |
| "Good skill = trigger signal + decision rules + tools" | Compressed to one sentence | Synthesis |

---

## Pass 6: Cross-validation with official Best Practices (review after writing)

> **Important:** This section is the v4-added "self-review" — feed the
> author's Pass 3 compressed conclusions **back** next to the official
> checklist to check consistency. All `[E]/[CK]` are author inferences,
> **not source-given hard constraints**.

### A vs B positioning differences (verified consensus)

| Dimension | A official Best Practices (1112 lines) | B this document (synthesis + 3-pass summary) |
|---|---|---|
| Document form | Specification-style checklist (22 items) | Narrative summary (5 passes + tag system) |
| Role of description | "what + when" (function + trigger scenarios) | **Routing signal** (determines correct selection from 100+ skills) |
| Starting method | Test with all models you'll deploy to | Build evals first, then docs (eval-driven) |
| Iteration loop | Claude A designs, Claude B uses, bidirectional loop | Same loop, emphasizing "observation over assumption" |
| Error handling granularity | "Solve, don't punt" | Three elements: what + why + how-to-fix (refinement) |
| Script engineering convention | In skill, using `scripts/` subdirectory | Same (`[CK]` common knowledge) |
| Composability | Not separately listed | Explicitly proposed: don't read global state, don't lock exclusive files, don't assume cwd |
| Time-sensitive information | Put in `## Old patterns` section | Same |

**One-sentence judgment:** A is "Anthropic's methodology for teaching
you to write skills", B is "a one-line speed-dial summary after
digesting Anthropic" (speed-dial = quick-access cheat sheet — meant to
be pulled up fast, not read cover-to-cover). The two are complementary
in the same knowledge system, **not contradictory**.

### 27 features: 22 from A + 5 unique to B

#### 22 A checklist items (already listed in Pass 5, not repeated here)

#### 5 features unique to B (v4 marked as F-23..F-27)

| # | Name | Tag | Value |
|---|---|---|---|
| F-23 | description is a routing signal | `[E]` key contribution | Affects auto-load hit rate, easiest to get wrong |
| F-24 | Use placeholders for paths (`${WIKIBASE_ROOT}`) | `[CK]` | Expand at deploy time, avoid hardcoding |
| F-25 | Composability: don't assume exclusive environment | `[E]` | Skills coexist with 100+ others, exclusive env var = guaranteed break |
| F-26 | Testing tier: personal/team/public three tiers | `[E]` | Different minimum requirements for different sharing scopes |
| F-27 | Error message three elements: what + why + how-to-fix | `[E]` refines A #12 | Error messages are actionable signals, not yes/no |

### 21 anti-patterns: 9 from A + 8 from B + 3 of my own observation

#### 9 A-explicit anti-patterns (listed in A document, not repeated)

#### 8 anti-patterns unique to B (v4 marked as AP-10..AP-17)

| # | Anti-pattern | Severity | Core principle violated |
|---|---|---|---|
| AP-10 | "Good prompt ≠ skill": directly copy a good prompt as SKILL.md | Critical | Loses progressive disclosure advantage |
| AP-11 | Description too broad: "helps with PDF generation" | Critical | All-scenario coverage = no-scenario coverage |
| AP-12 | Description too narrow: "Only for Q3 sales reports" | Critical | Trigger condition too strict = almost never fires |
| AP-13 | Stuffing whole domain tutorial into skill | Important | SKILL.md is a rules collection, not encyclopedia |
| AP-14 | Helper script inlined as "example" in SKILL.md | Important | Wastes 5k token budget |
| AP-15 | Swallow exception and return empty string (`except: return ""`) | Critical | Error messages should be actionable |
| AP-16 | Assume you're the only process (exclusive env var / global state) | Critical | Composability violation |
| AP-17 | No tests, "should work" | Varies by scenario | "Should work" is not verification standard |

#### My 3 observations (not explicitly in A or B but common)

| # | Anti-pattern | Severity | Notes |
|---|---|---|---|
| AP-18 | Mixed Markdown list hierarchies | Minor | Mixing `- [ ]` and `1.` confuses Claude's parsing |
| AP-19 | Frontmatter missing or empty description | Critical | Skill won't appear in available list (silent failure) |
| AP-20 | Example and main content language/scenario mismatch | Important | Trigger scenarios and examples don't align, routing signal breaks |

### One-sentence compression (restated from earlier for closure)

> **Good skill = trigger signal (description) + decision rules
> (instructions) + reusable tools (scripts).**

Conversely, **bad skills have only one thing in common**: the author
writes "what I want to say" as if it were "what Claude needs" — so
verbose, vague, full of background, missing decision rules, missing
reusable tools.

### Cheat sheet: A unique / B unique / shared

| Category | Content | Source |
|---|---|---|
| 22-item Checklist | progressive disclosure / one-level references / voodoo constants / multi-model testing ... | A unique (specification-level) |
| `[S]/[E]/[CK]/[F]` tag system | Distinguishes source/extrapolation/common-knowledge/fabrication | B unique (methodology) |
| "Good skill = trigger signal + decision rules + tools" | Compressed to one sentence | B synthesis (author) |
| F-23..F-27 | description is routing signal / placeholder paths / composability / testing tier / error three elements | B synthesis (author) |
| AP-10..AP-17 | 8 B-unique anti-patterns | B synthesis (author) |
| AP-18..AP-20 | 3 of my own observations | Synthesis (audit practice) |
| Three-tier loading model (Tier 1/2/3) | Progressive disclosure precise numbers | A + B shared |
| Eval-driven development | Build evals first, then docs | A + B shared |
| Claude A ↔ Claude B iteration loop | Observe-refine-observe again | A + B shared |
| "description is a routing signal" | Re-defines description | B key contribution (refines A spirit with sharper framing) |

---


## Pass 7: Addendum (revision v6, 2026-06-11)

> **Origin:** This section is the result of a section-by-section comparison
> with both source documents (`Introduction to Claude Skills.md` and
> `Skill authoring best practices.md`), identifying 10 gaps the prior
> six passes did not cover. Tagged using the existing `[S]/[E]/[CK]/[F]`
> scheme. **The Chinese version is the source of truth** — when in doubt,
> refer to `how-to-write-good-claude-skills-cn.md`.

### 7.1 Skills vs Tools vs Subagents hierarchy

> **[S]** `Introduction to Claude Skills.md` lines 188-196 explicitly state
> that Skills sit above individual tools, but below a full multi-agent
> system. They are conceptually independent of MCPs (Model Context
> Protocol).

**Abstraction levels (low to high):**

| Level | Concept | Core positioning | When to use |
|---|---|---|---|
| L0 | **Single Tool** | Single atomic capability (e.g. `bash`, `pdf_read`) | Smallest granularity; must be combined by the model on demand |
| L1 | **MCP Server** | Standard-protocol packaging of related tools | Reuse the same toolset across sessions / workflows |
| L2 | **Skill** (the subject of this document) | Instructions + tested code + resources, packaged | A reusable solution for a **specific workflow** |
| L3 | **Subagent** | Agent with its own context window and sub-task dispatch | Long chains where main context would be drowned |
| L4 | **Multi-agent System** | Multiple subagents in coordination (routing, state sharing) | Cross-domain complex problems |

**Key clarifications:**
- `[CK]` **Skill ≠ simple combination of tools** — a Skill bundles the
  full playbook of "when to activate / how to decide / how to verify".
  A tool is stateless and atomic.
- `[CK]` **Skill does not replace Subagent** — if a task requires long
  chains of reasoning and the main context is at risk of being
  swamped, escalate to L3 rather than stacking more skills.
- `[S]` When multiple skills work together (lines 198-200), they are
  still dispatched by a **single agent**; this is fundamentally
  different from L3-L4's "independent context".

> **One-line heuristic (author synthesis):** If a single sentence can
> answer "what / when to trigger / how to verify" — it's a skill. If a
> single sentence cannot, and the task is cross-domain — consider a
> subagent.

### 7.2 Skill Bundle directory structure

> **[S]** `Skill authoring best practices.md` lines 269-280 give a
> recommended directory structure.

**Minimal form (only `SKILL.md`):**

```
my-skill/
└── SKILL.md          # YAML frontmatter + body instructions
```

**Recommended form (as the skill grows):**

```
my-skill/
├── SKILL.md              # Body instructions (loaded on trigger, < 500 lines)
├── FORMS.md              # Form / template filling guide (loaded as needed)
├── reference.md          # API / field reference (loaded as needed)
├── examples.md           # Usage examples (loaded as needed)
└── scripts/
    ├── analyze.py        # Utility script (executed, not loaded)
    ├── validate.py       # Validation script
    └── fill_form.py      # Business script
```

**Directory conventions (by importance):**

| Convention | Source | Note |
|---|---|---|
| `[S]` `SKILL.md` is **recommended** to live at the root | line 271 | Clients read metadata from the root by convention; non-root placement may cause load failure |
| `[S]` Reference paths stay **one level deep** | line 371 | i.e. `reference.md`, not `docs/api/v2/reference.md` |
| `[E]` `scripts/` directory holds **only executables** | author synthesis | Don't place markdown or config files here (those go at the root) |
| `[CK]` Long references **split into separate files** rather than inlined | Pass 2 hard constraints | Maintain the 500-line SKILL.md cap |
| `[E]` Config files **centralized under** `config/` | author synthesis | Easier to reuse across skills and inject env vars |

**On "is it mandatory?":**
- The original document uses *figure* rather than *must* for "SKILL.md
  at the root"; treat it as a recommendation.
- `[E]` If the project already has a convention (e.g. all skills
  uniformly under `skills/<name>/` in a monorepo), follow that first.

### 7.3 Repair path for over-broad / over-narrow descriptions

> AP-11 / AP-12 list the symptoms; this section gives **the repair method**.

#### 7.3.1 Diagnosing the "broadness" of your current description

| Dimension | Over-broad symptom | Over-narrow symptom |
|---|---|---|
| Trigger frequency | Almost every task triggers it | Almost never triggers |
| User wording | Users rarely use the keywords | Users must say the exact keywords |
| Specificity | Abstract concept ("help with X") | Contains specific numbers / qualifiers ("Q3 only") |
| Peer skills | Many overlapping skills all match | No skill in the same domain matches |

#### 7.3.2 Repair steps (self-check style)

**Step 1 — Exhaust the keywords:** List 10 concrete phrasings real
users would use to trigger the skill.
- Bad: only "PDF task" — vague
- Good: "PDF form", "merge PDF", "extract table from PDF", "fill PDF field", etc.

**Step 2 — Compress to scenarios:** Distil the 10 phrasings into 3-5
core scenarios, each expressed as a specific verb phrase.

**Step 3 — Embed the trigger signals:** Write the scenario phrases
verbatim into the description (without ornamentation), forming
"loading triggered by these words".

**Step 4 — Explicit exclusions:** At the end of the description, list
scenarios where the skill **should not** be used, to prevent stealing
triggers from peer skills.

**Repair template:**

```yaml
# Bad (over-broad)
description: "Helps with PDF tasks"

# Bad (over-narrow)
description: "Only fills Acme Corp's Q3 2026 expense report PDF"

# After repair (right-sized)
description: |
  Extracts text/tables from PDFs, fills form fields, and merges
  multi-PDF documents. Use when user mentions: PDF, form, "fill in",
  "extract from PDF", "merge PDFs", or document extraction.
  NOT for: scanned image-only PDFs (use OCR skill), or password-
  protected PDFs (use decryption skill first).
```

### 7.4 Negative checklist (consolidated)

> AP-10..AP-20 are scattered across the document; this section
> consolidates them into a **single negative checklist** for fast
> review.

#### 7.4.1 Critical (must fix before release)

| # | Anti-pattern | One-line description | Repair direction |
|---|---|---|---|
| AP-19 | frontmatter missing description | Skill silently fails | Require non-empty description |
| AP-15 | Swallow exceptions, return empty string | Errors are untraceable | Throw exceptions with context |
| AP-16 | Assume exclusive environment | Conflicts with peer skills | Remove env-var / global-state dependencies |
| AP-10 | Copy a good prompt directly as SKILL.md | Loses progressive-disclosure benefit | Split into description / instructions / references |
| AP-11 | description too broad | Hits everything = hits nothing | Section 7.3, steps 1-4 |
| AP-12 | description too narrow | Hardly ever triggers | Section 7.3, steps 1-4 |

#### 7.4.2 Important (should fix before release)

| # | Anti-pattern | One-line description |
|---|---|---|
| AP-13 | Stuff the entire domain tutorial in | Split to `reference.md` |
| AP-14 | helper script inlined as an example | Switch to a path reference |
| AP-20 | Example and scenario out of alignment | Align description keywords with example titles |

#### 7.4.3 Context-dependent (depends on distribution scope)

| # | Anti-pattern | Personal use | Team / public |
|---|---|---|---|
| AP-17 | No tests | Tolerable | Must fix |
| AP-18 | Mixed Markdown list hierarchies | Tolerable | Should fix |

**How to use:** Print this list when reviewing. Go through item by
item (< 10 minutes total). Any Critical hit blocks release.

### 7.5 Why write Skills? (motivation narrative)

> **[S]** `Introduction to Claude Skills.md` lines 5-15, 186-201
> articulate the reason for Skills' existence.

**Why do we need Skills?**

**Motive 1 — Avoid repeating explanations (cost-driven)**
- `[S]` Original lines 276-289: every conversation that re-explains
  "how to do X" costs 5k–10k tokens. With a Skill, you only pay on the
  first trigger.
- `[CK]` Team context: every new hire no longer has to ask "how do we
  file expenses" — the Skill is already a "packaging of organizational
  knowledge".

**Motive 2 — Reduce the cognitive burden of prompt engineering (evolution-driven)**
- `[S]` Original lines 188-196: Skills are "one level above individual
  tools"; prototype with tools, stabilize as Skills.
- `[E]` Author observation: the marginal cost of writing one Skill
  (one-time 4-8 hours) is far below the cumulative cost of repeatedly
  writing prompts (1-2 hours/week × 50 weeks).

**Motive 3 — Codify "tested code" (quality-driven)**
- `[S]` Original line 198: helper scripts are the core value of Skills,
  "tested, working code".
- `[E]` Counter-evidence: making the model rewrite "how to parse CSV"
  every time has a notably higher error rate than reusing a tested
  script.

**Motive 4 — Composability (architecture-driven)**
- `[S]` Original lines 198-200: multiple Skills can work together in
  the same session.
- `[E]` UNIX-pipeline analogy: each Skill is a focused small tool;
  combining them yields infinite possibilities.

### 7.6 Working with filesystem, upload, and other Beta tools

> **[S]** `Introduction to Claude Skills.md` lines 241-272 detail how
> Skills compose with Beta tools.

**Cooperation matrix:**

| Beta tool | Cooperation with Skills | Typical use |
|---|---|---|
| `code_execution_2025-08-25` | Mandatory — executes code under `scripts/` | A Skill's `validate.py` is run by this tool |
| `files-api-...` | Upload / download file-type resources | Upload PDF / Excel for the Skill to process |
| `skills-2025-10-02` | Beta identifier that enables the Skill system | Must be in the `betas` array |
| `prompt-caching-...` | Cache loaded Skill contents | Significant cost reduction when calling the same Skill many times |

**Minimal call skeleton (Python):**

```python
response = client.beta.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=4096,
    betas=["skills-2025-10-02"],
    tools=[{"type": "code_execution_2025-08-25"}],
    container={"skills": [{"type": "anthropic", "skill_id": "pdf", "version": "latest"}]},
    messages=[{"role": "user", "content": "Extract tables from report.pdf"}]
)
```

**Mandatory constraints (any missing item will error):**
- `[S]` Must use `client.beta.messages.create()` (not the standard
  `messages.create`)
- `[S]` `betas` array must include `skills-2025-10-02`
- `[S]` `tools` must include `code_execution_2025-08-25`
- `[E]` `container.skills` should explicitly declare the Skills to load
  (loading everything by default burns tokens)

### 7.7 Loop iterations: Claude A designs ↔ Claude B uses

> **[S]** `Introduction to Claude Skills.md` lines 442-462 describe
> this loop.

**Full loop (5 steps):**

1. **Design (Claude A):** draft `SKILL.md` based on the workflow.
2. **Use (Claude B):** call the Skill in a real task.
3. **Observe:** record when B triggered, which tools it called, errors
   that came up, and user feedback.
4. **Revise (back to A):** adjust `SKILL.md` (description, instructions,
   scripts) based on observations.
5. **Use again (B again):** verify the revision was effective.

**Minimum collection checklist per iteration:**

| Item | Collection method | Purpose |
|---|---|---|
| Trigger frequency | Logs | Judge description broadness |
| Sections of SKILL.md actually cited after load | `claude_code` tools | Judge whether SKILL.md is bloated |
| Tool-call failure rate | Logs | Judge script robustness |
| User re-question rate | Conversation history | Judge clarity of instructions |
| Task completion rate | Human evaluation | Judge overall effectiveness |

**Suggested iteration cadence:**
- `[E]` First week: small revision every 2 real uses (tweak description
  / fix scripts)
- `[E]` First month: full 5-step loop once a week
- `[E]` After stabilization: review once a month, decide whether to
  split / merge Skills

### 7.8 Production deployment: versioning

> **[S]** `Introduction to Claude Skills.md` lines 368-376 detail
> versioning.

**Two versioning strategies:**

| Strategy | Applies to | Syntax | Risk |
|---|---|---|---|
| `"latest"` | Anthropic-managed Skills | `version: "latest"` | Auto-updates, may introduce breaking changes |
| **Epoch timestamp** | Custom Skills | `version: "1749667200"` (seconds) | Unreadable but fully controlled |
| Pinned version | Production environment | `version: "2025-11-15-v3"` | Must update manually; most stable |

**Production deployment checklist:**
- `[S]` **Pin versions** (don't use `latest`) — especially in production
- `[S]` **Preserve rollback path** — every version has a complete
  directory backup
- `[E]` **Canary release** — validate on 1% of traffic first, then
  full rollout
- `[E]` **Maintain a changelog** — every release includes change notes
  for audit
- `[CK]` **Monitor core metrics** — load success rate, tool-call failure
  rate, user re-question rate

**Anthropic-managed Skills update mechanism (lines 370-373):**
- `[S]` Anthropic updates managed Skills on its own; users get the
  latest with `latest`
- `[S]` Users can lock to historical versions via the pinned version
  number
- `[S]` Custom Skills **must** use epoch timestamps (no other choice)

### 7.9 Placeholders like `${WIKIBASE_ROOT}` — usage example

> F-24 (Pass 6) only mentioned the placeholder concept; this section
> gives **concrete usage**.

**Use case:** When a Skill is deployed on multiple machines and across
different users, hard-coded absolute paths cause:
- Teammate A's `/Users/alice/project` becomes B's `/Users/bob/project`
  → script fails
- CI's `/home/runner/work/...` doesn't exist on the local dev box

**Placeholder conventions:**

| Placeholder | Meaning | Example value |
|---|---|---|
| `${WIKIBASE_ROOT}` | Root of the project containing the Skill | `/Users/xiang/Work/sfx` |
| `${HOME}` | User's home directory | `/Users/xiang` |
| `${SKILL_DIR}` | Current Skill's directory | `${WIKIBASE_ROOT}/skills/pdf` |
| `${WORKSPACE}` | Runtime working directory | `/tmp/run-12345` |

**Usage in SKILL.md:**

```markdown
## Quick start

Place the PDF to be processed in `${WORKSPACE}/input/`, then run:

\`\`\`bash
python ${SKILL_DIR}/scripts/validate.py ${WORKSPACE}/input/report.pdf
\`\`\`

The script will:
1. Validate PDF integrity
2. Output metadata to `${WORKSPACE}/output/meta.json`
```

**Expansion at deploy time (pseudocode):**

```python
# deploy.py
content = open("SKILL.md").read()
content = content.replace("${WIKIBASE_ROOT}", os.environ["WIKIBASE_ROOT"])
content = content.replace("${SKILL_DIR}", os.environ["SKILL_DIR"])
content = content.replace("${WORKSPACE}", tempfile.mkdtemp())
open("SKILL.deployed.md", "w").write(content)
```

**With environment variables:**

```bash
# .env
WIKIBASE_ROOT=/Users/xiang/Work/sfx
SKILL_DIR=${WIKIBASE_ROOT}/skills/pdf
```

**Anti-example (hard-coded):**

```markdown
# ❌ Anti-example
python /Users/xiang/Work/sfx/skills/pdf/scripts/validate.py ~/Downloads/report.pdf
```

> **`[E]` One-line principle:** Every absolute path that appears in
> SKILL.md should be a placeholder; placeholders are injected from
> environment variables at deploy time.

### 7.10 Test pass/fail criteria

> The 22-item Checklist items 19-21 say "test", but don't define
> "what counts as passing". This section adds **criteria**.

#### 7.10.1 Pass criteria for each test type

| Test type | Quantity | Pass criterion | Measurement method |
|---|---|---|---|
| Typical scenario | ≥ 1 | Task 100% complete; user re-question count = 0 | Human eval + conversation logs |
| Edge case | ≥ 1 | Doesn't crash; returns explicit error message | Inject empty / oversized / malformed input |
| Error path | ≥ 1 | Error message contains all three elements (what / why / how-to-fix) | Grep output for three-element keywords |

#### 7.10.2 Quantitative thresholds for "passing"

- **[E] Personal use:**
  - ≥ 1 end-to-end test passes
  - The developer themselves is subjectively satisfied

- **[E] Team-shared:**
  - 3 tests all pass (typical + edge + error)
  - At least 1 team member (not the author) uses it independently and signs off

- **[E] Public distribution:**
  - Above + multi-model testing (Haiku / Sonnet / Opus each run once)
  - Real user sample ≥ 5 people
  - Re-question rate < 10%

#### 7.10.3 Failure handling flow

```
Test failure → classify (syntax / logic / docs)
            → syntax: fix immediately
            → logic: return to Pass 7.7 step 4, revise SKILL.md
            → docs: add examples / rewrite description
            → re-run all tests, only release after regression passes
```

#### 7.10.4 A minimal test case template

```python
# test_skill_e2e.py
import subprocess

def test_typical_scenario():
    """Typical: user uploads a PDF, skill extracts tables."""
    result = subprocess.run(
        ["python", "scripts/extract.py", "fixtures/sample.pdf"],
        capture_output=True, text=True
    )
    assert result.returncode == 0
    assert "table_1.csv" in result.stdout
    assert "table_2.csv" in result.stdout

def test_edge_empty_input():
    """Edge: empty PDF."""
    result = subprocess.run(
        ["python", "scripts/extract.py", "fixtures/empty.pdf"],
        capture_output=True, text=True
    )
    assert result.returncode != 0
    assert "Error" in result.stderr
    assert "empty" in result.stderr.lower()  # error msg contains "why"

def test_error_missing_dep():
    """Error: missing dependency (no pdftotext)."""
    env = os.environ.copy()
    env["PATH"] = "/nonexistent"  # simulate missing dep
    result = subprocess.run(
        ["python", "scripts/extract.py", "fixtures/sample.pdf"],
        capture_output=True, text=True, env=env
    )
    assert "pdftotext" in result.stderr  # error msg contains "how to fix"
    assert "brew install" in result.stderr or "apt install" in result.stderr
```

---

## Revision log

- **v1 (2026-06-11, early version)**: Three-pass summary, no distinction
  between source and author extrapolation. Adversarial reviewer pointed
  out 30+ unsupported claims; the comparison table was almost entirely
  fabricated.
- **v2 (this version)**: Introduced the `[S]/[E]/[CK]/[F]` tag system;
  removed all fabrication; explicitly placed "good skill = ..." in author
  synthesis, **not as source summary**.
- **v3 (2026-06-11)**: Absorbed Anthropic's official
  [Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices)
  checklist. Process: two independent subagents did gap analysis + merge
  proposal → third subagent arbitrated disagreements (5 disputed points
  all ruled on) → author wrote final version. Added: (1) "Additional
  hard constraints" at the end of Pass 2 (15 of the 22 Checklist items
  are new), (2) brand-new "Pass 5" review framework (three review
  tables + sharing-tier gating), (3) cheat sheet extended with both
  sources. All Checklist items marked `[S]`, readings marked
  `[E]/[CK]`.
- **v4 (2026-06-11)**: Added "Pass 6: Cross-validation with official
  Best Practices". Made A and B differences explicit: (1) 22 A
  checklist + 5 B-unique features (F-23..F-27) = 27 features, (2) 9 A
  anti-patterns + 8 B-unique anti-patterns (AP-10..AP-17) + 3 author
  observations (AP-18..AP-20) = 21 anti-patterns, (3) cheat sheet
  distinguishes A unique / B unique / shared. The 16 new features and
  11 new anti-patterns have been synced to
  `skills/skill-reviewer/references/writing-good-skills.md` as an
  audit comparison table.
- **v5 (2026-06-11, this file)**: English translation of the Chinese
  original. The Chinese source-of-truth lives at
  `how-to-write-good-claude-skills-cn.md` (co-located in
  `skills/skill-reviewer/docs/`) for native-Chinese readers. This
  English version is also in `skills/skill-reviewer/docs/` so the
  skill-reviewer can reference both the distilled checklist
  (`../references/writing-good-skills.md`) and the full narrative
  source without leaving the skill directory. Filenames follow
  kebab-case per the article's own "no spaces in filenames" principle.
- **v6 (2026-06-11, this file)**: After section-by-section comparison with
  both source documents, added "Pass 7: Addendum" with 10 sub-sections
  covering: (1) Skills vs Tools/Subagents hierarchy, (2) Bundle directory
  structure conventions, (3) description broadness diagnosis + 4-step
  repair, (4) consolidated negative checklist, (5) motivation narrative,
  (6) Beta tools cooperation matrix, (7) Claude A↔B 5-step iteration
  loop, (8) production deployment versioning, (9) `${WIKIBASE_ROOT}`
  usage example, (10) test pass/fail criteria. The Chinese version
  (`how-to-write-good-claude-skills-cn.md`) remains the source of truth.
  Also corrected the frontmatter path: the Chinese source-of-truth
  filename is `how-to-write-good-claude-skills-cn.md` (not the previously
  stated `../../How to write good Claude skills.md`).
