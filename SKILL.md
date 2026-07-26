---
name: skill-reviewer
description: |
  Review Claude Skills against the official 22-item Best Practices checklist.
  Use when the user asks to review, audit, check, or evaluate a skill directory
  or SKILL.md, including "review my skill", "skill review", "audit skill",
  "检查 SKILL.md", "审查 skill", "is my skill ready", or /skill-reviewer.
  NOT for general code review, proofreading, JSON validation, or dependency
  security scanning.
---

# Skill Reviewer

Review Claude Skills against Anthropic's official Best Practices checklist
and the writing rules distilled in `How to write good Claude skills.md`.

## What to review

This skill supports two input modes:

1. **Skill directory path** — Read `SKILL.md`, scan `scripts/`, `references/`,
   `templates/` subdirectories; check file structure, naming, path depth,
   forward-slash paths.
2. **Pasted SKILL.md text** — Review the text-only content. Note that
   structural checks (#3 lines, #8 nesting depth, #15 file documentation,
   #16 forward-slash paths) will be limited or skipped.

## Review process

### Phase 1 — Gather

If given a directory path:
1. `ls -R <skill-dir>/` to understand the file structure.
2. Read `SKILL.md`.
3. Skim executable files in `scripts/` (check docs, error handling, voodoo constants).
4. Note file nesting depth, path style, and any auxiliary files.

If given pasted text:
1. Read the full text.
2. Note that structural checks will be marked `SKIPPED (no directory)`.

### Phase 1.5 — Calibration example (recommended for first-time use)

Before producing the user's report, run skill-reviewer on
`evals/medium-skill-example/` and compare the output with
`evals/medium-skill-example/expected-report.md`.

This is **recommended the first time you run this skill in a session**
because the LLM's first-pass tendency is to underweight item categories
(especially #17, #18, AP-15). The medium example exposes common
judgment boundaries before the user's report goes out. On subsequent
runs in the same session, this step may be skipped unless the user
changes the skill under review.

Do not include the medium example's output in the user's report unless
explicitly asked. Calibration is silent.

### Phase 2.0 — Mechanical pre-check (deterministic, runs before LLM)

Before LLM-driven semantic checks, run these deterministic greps on
SKILL.md and `scripts/`. Each yields a PASS/FAIL with concrete evidence.
This implements the operational recommendation in
[docs/perspectives/](docs/perspectives/) that items #3, #5, #16 are the
three best candidates for automation — mechanical checks save LLM tokens
and eliminate model variance on these purely structural properties.

| Item | Check | Command (POSIX) | Result |
|------|-------|-----------------|--------|
| #3 | SKILL.md body line count | `awk 'BEGIN{n=0} /^---$/{c++; next} c>=2{n++} END{print n}' SKILL.md` | `>500` ⇒ FAIL; `<250` ⇒ WARN (under-utilized body) — render as ⚠️ in Part B |
| #5 | Time-sensitive references | `grep -nE "(\bas of (20[0-9]{2}\|[A-Z][a-z]+ 20[0-9]{2})\|"v[0-9]+"\|v[0-9]+\.[0-9]+(\.[0-9]+)?\b)" SKILL.md \| grep -vE "^[0-9]+:\\| #?[0-9]+ \\|"` | non-empty ⇒ FAIL |
| #16 | Windows-style backslash paths | `grep -nF "\\" SKILL.md \| grep -vE "^[0-9]+:\\| #?[0-9]+ \\|"; [ -d scripts ] && grep -rnF "\\" scripts/ \| grep -vE "^[0-9]+:\\| #?[0-9]+ \\|"` | non-empty ⇒ FAIL |

Print the raw output of each command in the audit log so the user can
sanity-check. If checks pass, record as PASS without invoking LLM. If
checks fail, record as FAIL *with the grep evidence* — the LLM Phase 2
will still run for context. The mechanical verdict is locked EXCEPT for
matches on lines that document the rule itself (e.g., the row showing
the example of what to flag); the LLM adjudicates such self-references
as PASS-with-note.

Structural checks (path style, line count) belong in this phase.
Semantic checks (#1, #2, #6, #7, #8, #11-15, #17-18, #19-22) belong in
Phase 2 below.

### Phase 2 — Audit (22 items)

Go through every item below. For each, record: `PASS`, `FAIL`,
`WARN` (advisory, not a failure), `SKIPPED (reason)`, or `N/A`. For deeper context on *why* an item matters
and the additional features/anti-patterns not in this checklist, see
[references/writing-good-skills.md](references/writing-good-skills.md).

Note: items #3, #5, #16 are already verdicted by Phase 2.0 above and
do not need re-evaluation in this phase.

#### Core quality (items 1–10)

| # | Item | What to check |
|---|------|--------------|
| 1 | Description is specific and includes key terms | Does `description` name the exact scenario ("when user asks for .pdf") AND include domain-specific key terms (e.g., "markdown-to-PDF", "report generation")? A vague description like "handles documents" is FAIL. |
| 2 | Description includes both what and when | Two-part check: (a) what the skill produces, (b) when to use it. Missing either = FAIL. |
| 3 | SKILL.md body under 500 lines | Count lines of SKILL.md (excluding YAML frontmatter). >500 = FAIL. ~250-400 is ideal. |
| 4 | Additional details in separate files | Templates, configs, long references, or large code blocks should live outside SKILL.md. Everything crammed into SKILL.md = FAIL. |
| 5 | No time-sensitive information | Look for API versions ("v2", "2024"), dates ("as of June 2025"), temporary solutions. Any of these in the main body without an "old patterns" section = FAIL. |
| 6 | Consistent terminology throughout | Same concept uses the same word in description, body, scripts, and error messages. Mixing synonyms (e.g., "processor" in description, "engine" in body) = FAIL. |
| 7 | Concrete, runnable examples | Examples should show real input/output pairs, not abstract descriptions. "Create a chart" = FAIL. "Create a bar chart showing Q1 revenue with data labels" = PASS. |
| 8 | File references one level deep | References like `scripts/validate.py` are OK. References like `tools/helpers/submodules/checker.py` are NOT. Deep nesting (>2 levels from SKILL.md) = FAIL. |
| 9 | Progressive disclosure used appropriately | Tier 2 body compact, Tier 3 details deferred. Everything front-loaded = FAIL. The 3-tier model must be evident. |
| 10 | Workflows have clear steps | Multi-step procedures use numbered lists, sub-headings, or decision trees, not paragraphs. Steps that mix multiple conditions in one sentence = FAIL. |

#### Code and scripts (items 11–18)

| # | Item | What to check |
|---|------|--------------|
| 11 | Scripts solve problems rather than punt to Claude | Check if scripts do real work (parse, transform, validate) or just repackage a prompt for Claude. Scripts that are thin wrappers around Claude API calls = FAIL. |
| 12 | Error handling is explicit and helpful | Error messages should answer: (1) what failed, (2) why, (3) how to fix. "Error occurred" = FAIL. "Error: file not found at /x; check /parent exists and is writable" = PASS. |
| 13 | No "voodoo constants" (values justified) | Every magic number, timeout, or threshold has a rationale comment. Bare `timeout=30` = FAIL. `timeout=30  # max sec for render; longer means resource leak` = PASS. |
| 14 | Required packages listed and verified | SKILL.md should list dependencies (Python packages, system tools, API keys). Scripts should verify availability at startup with actionable install instructions on failure. |
| 15 | Scripts have clear documentation | Each script file should open with: purpose (1 line), parameters, return values / exit codes, 1-2 usage examples. This lives in the script itself, not in SKILL.md. |
| 16 | No Windows-style paths (all forward slashes) | Any backslash in a path (`scripts\tools\check.py`) = FAIL. All paths must use `/`. This is separate from the `${WIKIBASE_ROOT}` placeholder concern. |
| 17 | Validation/verification for critical operations | After file writes, network requests, or data transformations, the skill verifies success (check exit codes, file sizes, response status codes). No verification = FAIL. |
| 18 | Feedback loops for quality-critical tasks | Tasks where output quality needs human judgment (doc generation, code review) include a self-assessment step: Claude reviews its own output and flags improvements before delivering. |

#### Testing (items 19–22)

| # | Item | What to check |
|---|------|--------------|
| 19 | At least 3 evaluation cases | Check for test files or eval descriptions. 0 tests = FAIL. 1-2 tests = FAIL. 3+ with coverage of typical + edge + error paths = PASS. |
| 20 | Tested with Haiku, Sonnet, and Opus | Evidence of multi-model testing. Tested on only one model = FAIL. At minimum, one fast (Haiku) + one capable (Sonnet/Opus) = PASS. |
| 21 | Tested with real usage scenarios | Tests use realistic user prompts (messy, underspecified), real file formats, and real error data. Idealized prompts only = FAIL. |
| 22 | Team feedback incorporated (if applicable) | For multi-user skills: at least one independent review. Solo skills: N/A. |

### Phase 3 — Produce the report

Output a structured report in two parts.

#### Part A: Graded issues (Critical / Important / Minor)

Group failures by severity:

- **Critical** (must fix before sharing): description absent or useless, body >500 lines, path backslashes, no error handling, scripts punt to Claude.
- **Important** (should fix): inconsistent terminology, weak examples, missing prerequisites, no documentation on scripts, no verification steps.
- **Minor** (nice to fix): slightly long body (400-500 lines), off-by-a-few chars in frontmatter, ok-but-vague example.

For each issue, also tag the **impact layer** so the user knows where the
fix lands (which informs ROI; see the priority matrix in `references/`):

| Layer | Tag | Meaning | Typical items |
|-------|-----|---------|---------------|
| User-perception | `[user-perception]` | Description / terminology / examples — what users see and what triggers the skill. Fixes here recover the most callers. | #1, #2, #6, #7, AP-11, AP-12, AP-13, AP-20 |
| Engineering | `[engineering]` | Scripts / verification / errors — whether the skill works when called. | #3, #11, #12, #15, #17 |
| Collaboration | `[collaboration]` | Evals / multi-model / team feedback — gating to share or distribute. | #19, #20, #21, #22 |
| Hygiene | `[hygiene]` | Path style / nesting / progressive disclosure — clean but rarely high-impact. | #4, #8, #9, #10, #13, #14, #16, #18 |

(English tags are used to stay portable across locales; see
`references/writing-good-skills.md` for the canonical prose.)

A single Critical issue can carry multiple tags (e.g. an empty description
can be both `[user-perception]` and `[hygiene]`).

For each issue, cite the item #, the specific file:line, a clear description, and a concrete fix suggestion.

```markdown
### Critical

- **[#3] SKILL.md:87** — Body is 523 lines (limit: 500). The "Migration guide"
  section (lines 210-310) can be moved to `references/migration.md`.
```

#### Part A.5: Good anchors (what this skill already does right)

After listing failures, surface what the skill *gets right*. This
matters because:

- It anchors the user to a concrete target (they see what good looks
  like in their own skill, not just what is wrong).
- It is the inverse of the routing-signal insight: a good description
  is a positive anchor, not just an absence of failure.

Format:

```markdown
### Good anchors

- **Description** (items 1, 2) — Contains 4 specific trigger keywords
  (`docs`, `pdf`, `report`, `audit`) and explicit "what + when" structure.
- **Body length** (item 3) — 287 lines, well within the 250-400 ideal band.
- **Errors** (items 12, F-27) — All scripts use 3-element error pattern
  (what / why / how-to-fix); sample: `Error: file not found at /x;
  check /parent exists and is writable`.
- **Verification** (item 17) — Step 6 of workflow reads back the output
  file size and asserts it matches the expected row count.
```

Limit to 3-6 anchors. Pick the items that most distinguish this skill
from a generic prompt wrapper.

#### Part B: Full 22-item checklist

A table with every item, result, and brief note.

```markdown
| # | Category | Item | Result | Note |
|---|----------|------|--------|------|
| 1 | Core | Description specific + key terms | ✅ PASS | "Creates branded PDF reports from markdown — use when..." |
| 2 | Core | Description what + when | ❌ FAIL | Describes what but not when |
| 3 | Core | Body < 500 lines | ⚠️ WARN | 187 lines — under-utilized body; consider expanding |
| ... | ... | ... | ... | ... |
```

Verdict glyphs: `✅ PASS`, `❌ FAIL`, `⚠️ WARN`, `➖ SKIPPED (reason)`, `— N/A`. `WARN` is reserved for advisory findings that are not failures (e.g., a body shorter than the 250-line lower band) but deserve the reader's attention.

#### Failure clusters (fix-one-fix-many grouping)

Many FAILs share a single root cause. Before showing the summary, group
related FAILs so the user can fix several findings with one edit. This
implements the "fix-one-fix-many" pattern from
[docs/perspectives/](docs/perspectives/).

Format:

```markdown
### Failure clusters

- **Cluster A — Description has no routing signal** (affects #1, #2, AP-11, AP-12, AP-20)
  - Root cause: frontmatter description says "skill that helps" — no
    specific scenario, no trigger keywords.
  - Single fix: rewrite description to `<what> — use when <kw1>, <kw2>, <kw3>, <kw4>`
    per F-23 anchor pattern (see references).
  - Estimated recovery: 5 of 5 FAILs above resolved.
- **Cluster B — Scripts are prompt wrappers** (affects #11, AP-10, AP-14)
  - Root cause: `scripts/extract.py` body is `client.messages.create(prompt=...)`
    — "prompt 披了件脚本外衣".
  - Single fix: replace wrapper with real parse/transform/validate logic,
    or remove the script and inline the (small) logic into SKILL.md.
- **Cluster C** — none
```

Rules:

- A cluster needs at least 2 FAILs sharing one root cause. Singleton
  FAILs go in Part A only.
- Cite the item numbers resolved by each cluster so the user can verify.
- If no clusters exist, omit this section entirely.

End the report with a **summary**:
- Pass rate: X/22 (Y PASS, Z FAIL, U WARN, W SKIPPED)
- Gating recommendation:
  - **Clear for personal use** / **Ready for team share** / **Ready for public distribution** (per the tier table below), or
  - **Not ready** with specific blocking items

### Phase 4 — Self-check before delivery

Before showing the report to the user, run this five-point self-assessment.
A review skill that mis-judges items wastes the user's time more than no
review at all, so this step is non-optional.

1. **Specificity** — Every `FAIL` cites a concrete location: file path,
   line number, or a quoted snippet from the source. Vague verdicts like
   "description is unclear" are not acceptable. Either point to the exact
   phrase, or downgrade the verdict to a question for the user.

2. **Completeness** — All 22 items appear in Part B with a verdict. None
   silently dropped. `SKIPPED` and `N/A` are valid verdicts, but they
   must carry a one-line reason.

3. **Spot-check accuracy** — Pick 2 `FAIL` entries at random and re-verify
   the cited evidence really exists in the source. If you cannot find it,
   the `FAIL` is wrong; remove or revise it before delivering.

4. **Tier consistency** — The gating recommendation matches the actual
   results. Marking "Ready for team share" while #19 (evals) shows `FAIL`
   is inconsistent — fix one or the other.

5. **Cluster consistency** — FAILs grouped into a single cluster (Phase 3
   "Failure clusters" section) must share exactly one root cause. If two
   FAILs in a cluster have *different* root causes, split the cluster or
   remove the offending FAIL. The inverse is also true: if two FAILs
   share a root cause but appear in different clusters, merge them. This
   implements the meta-principle "审查者自己也要被审查" — cluster
   assignments are themselves a finding and must be self-checked.

End the report with one literal line: `Self-check passed.` If any of the
five checks triggered a fix, briefly note what changed (e.g.,
"Self-check: split cluster A on root-cause divergence; recounted summary").

## Complete example

Input:

```text
Please use skill-reviewer to check evals/medium-skill-example/
```

Expected behavior:

1. Read `evals/medium-skill-example/SKILL.md`.
2. Check directory structure and note that no executable script is present.
3. Produce Part A findings for the weak abstract examples, vague errors,
   undocumented constants, and missing test evidence.
4. Produce Part B with all 22 checklist rows.
5. Recommend `Clear for personal use`, then end with `Self-check passed.`

The expected report for this example lives at
`evals/medium-skill-example/expected-report.md`.

## Sharing-tier gating

| Tier | Requirement |
|------|------------|
| Personal use | Items 1-3 PASS, no critical failures |
| Team share | Core quality (1-10) all PASS, ≥3 evals (item 19) |
| Public distribution | All 22 PASS, multi-model tested (item 20), team feedback (item 22) |

### Community perspectives (advisory, used during audit context)

- [docs/perspectives/](docs/perspectives/) — Third-party reflections
  (ground-level engineering perspective). Reading order:
  - For priority framing on FAIL clusters → 2026-06-13-claw
  - For business-context examples (e.g. trigger-rate
    failure modes) → same

---

## Maintaining this skill

Regression cases live in `evals/`. After changing this checklist, report
format, or severity guidance:

1. Review `evals/good-skill-example/`,
   `evals/medium-skill-example/`, and `evals/bad-skill-example/`.
2. Compare each report against its `expected-report.md`.
3. Record the run in `evals/results.md`, including model, date, pass/fail,
   and any reviewer feedback.
4. Update expected reports if the rule change intentionally changes the output.

## References

### Working reference (used during audits)

- [references/writing-good-skills.md](references/writing-good-skills.md) —
  Distilled cross-comparison reference (267 lines, checklist format).
  Use during audits as a **lookup table**. Contains the "description as
  routing signal" insight, 5 additional features (F-23 to F-27), and 11
  additional anti-patterns (AP-10 to AP-20) that extend this checklist.

### Narrative docs (background reading, in docs/)

- [docs/how-to-write-good-claude-skills.md](docs/how-to-write-good-claude-skills.md) —
  Full English narrative (~490 lines) — the **source material** that
  `references/writing-good-skills.md` distills. Read this for background
  on skill design methodology, not during audits.

### Upstream sources (in docs/)

- [docs/Skill authoring best practices.md](docs/Skill%20authoring%20best%20practices.md) —
  Anthropic's official 1111-line guide. The 22-item checklist in this
  SKILL.md is distilled from this document.
- [docs/Introduction to Claude Skills.md](docs/Introduction%20to%20Claude%20Skills.md) —
  Anthropic Cookbook, 834 lines. The `[S]` (source) claims in
  `how-to-write-good-claude-skills.md` are quoted from this.
- [Anthropic official: Skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) —
  Same official guide, online URL. Use the local copy for offline work.
