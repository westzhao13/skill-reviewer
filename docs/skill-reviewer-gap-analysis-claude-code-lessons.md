# skill-reviewer vs. Claude Code Lessons — Gap Analysis

Cross-check of `skill-reviewer` against
[Lessons from Building Claude Code - How We Use Skills.md](Lessons%20from%20Building%20Claude%20Code%20-%20How%20We%20Use%20Skills.md)
(Thariq / Anthropic, 2026-03-18).

This document serves two purposes:

1. **Self-audit** — Does `skill-reviewer` itself follow the article?
2. **Reviewer gaps** — What should `skill-reviewer` add to its audit process so
   it can judge other skills against this newer operational guidance?

---

## Executive summary

| Lens | Verdict |
|------|---------|
| skill-reviewer **as a skill** | Strong on description routing, progressive disclosure, evals, self-check. Weak on scripts/, Gotchas, hooks, usage measurement (**P0 OPEN**). |
| skill-reviewer **as a reviewer** | Covers official 22-item best practices. Phase 2.1 + F-28–F-32 / AP-21–AP-22 **DONE** for advisory Claude Code Lessons checks; dogfood and public-evidence items remain OPEN. |

**Priority fixes for the reviewer:** add advisory checks for Gotchas, skill-type clarity, scripts-when-needed, hooks/setup/memory where applicable; dogfood `scripts/mechanical-check.sh`.

---

## Part A — skill-reviewer as a skill (self-audit)

Article section | Article expectation | skill-reviewer today | Gap
---|---|---|---
**Skill is a folder** | scripts, assets, data, not just markdown | `references/`, `docs/`, `evals/` — no `scripts/`, `assets/` | Missing `scripts/` despite Phase 2.0 prescribing shell checks |
**Type 6: Code Quality & Review** | Deterministic scripts; hooks or GitHub Action | LLM + inline grep/awk in SKILL.md | Article's gold standard for this type not met |
**Don't state the obvious** | Push Claude out of default thinking | Focuses on routing, anti-patterns, audit mechanics | PASS — checklist table is somewhat redundant with official doc |
**Gotchas section** | Highest-signal content; iterate over time | No `## Gotchas` in SKILL.md | FAIL — gotchas live only implicitly in `references/` |
**Progressive disclosure** | Point to files; templates in `assets/` | `references/`, `docs/` split | PASS for refs; no `templates/` or `assets/` |
**Avoid railroading** | Give info + flexibility | Rigid Phase 1–4, fixed report schema | PARTIAL — justified for consistency, but article would flag as heavy |
**Setup / config.json** | Ask user once; store in config | Not needed for core use | N/A for now |
**Description for the model** | Routing signal, not summary | Excellent triggers + NOT-for exclusions | PASS — aligns with F-23 |
**Memory** | Append logs; use `${CLAUDE_PLUGIN_DATA}` | Manual `evals/results.md` in skill dir | PARTIAL — no stable-data guidance; upgrade may wipe logs |
**Store scripts** | Scripts for composition, not re-boilerplate | Documents commands, no script files | FAIL — "cobbler's children have no shoes" |
**On-demand hooks** | Session hooks when skill loads | None in frontmatter | FAIL — no `/skill-reviewer`-scoped guardrails |
**Distribution / curation** | Marketplace curation before release | evals + gating tiers + expected-report | PASS |
**Measuring skills** | PreToolUse hook logs usage | No telemetry | FAIL |

### Self-audit tier (article lens)

- **Personal use:** PASS
- **Team share:** PASS (evals, description, structure)
- **Anthropic-style "excellent Type-6 skill":** NOT YET — needs `scripts/`, Gotchas, optional hooks

---

## Part B — skill-reviewer as a reviewer (what the checklist misses)

The 22-item checklist predates or does not encode several **operational** lessons
from the article. Below: article point → current coverage → recommended reviewer action.

### B1. Skill type taxonomy (9 categories)

**Article:** Best skills fit one of nine types (Library, Verification, Data,
Automation, Scaffolding, Code Quality, CI/CD, Runbooks, Infra Ops). Confusing
skills straddle several.

**22-item coverage:** None. No prompt to classify the skill under review.

**Reviewer should add (advisory, not FAIL-by-default):**

- Ask: *Which single type does this skill primarily serve?*
- Flag if description + body + scripts suggest **multiple unrelated types**
  (e.g. "standup + deploy + PDF" in one skill).
- Suggest split or rename when type is unclear.

**Severity:** Important when sharing; Minor for personal skills.

---

### B2. Gotchas section

**Article:** "The highest-signal content in any skill is the Gotchas section."

**22-item coverage:** Partially via #7 (examples) and AP-13 (encyclopedia), but
**no explicit Gotchas check**.

**Reviewer should add:**

- Look for `## Gotchas`, `## Footguns`, `## Common mistakes`, or equivalent.
- For Library / Verification / Scaffolding types: **FAIL or Important** if no
  gotchas after the skill has non-trivial domain rules.
- Report: "Add Gotchas built from real Claude failure modes."

**Maps to:** new **AP-21** in `references/writing-good-skills.md`.

---

### B3. Don't state the obvious (knowledge skills)

**Article:** Knowledge skills should only add what Claude doesn't already know.

**22-item coverage:** **AP-13** (encyclopedia-style) — close but not identical.

**Reviewer should add:**

- For type Library / Reference: sample 2–3 paragraphs — do they teach
  org-specific facts or re-explain generic CS concepts?
- Cite lines that could be deleted without losing org-specific value.

**Maps to:** extend AP-13 audit guidance.

---

### B4. Scripts folder when skill needs determinism

**Article:** Verification and Code Quality skills should include scripts;
compose helpers instead of regenerating boilerplate each run.

**22-item coverage:** #11–#15 when `scripts/` exists; **N/A when absent** —
reviewer does not ask *should* this skill have scripts?

**Reviewer should add:**

- For types **Verification**, **Code Quality**, **Data**, **Infra Ops**:
  if no `scripts/` and body contains repeated shell one-liners or "run this
  grep", recommend extracting to `scripts/`.
- For **skill-reviewer itself:** dogfood `scripts/mechanical-check.sh`.

**Maps to:** new **F-28** (type-appropriate scripts expectation).

---

### B5. Setup and config.json

**Article:** Skills that need user context should use `config.json` + ask on
first run; use AskUserQuestion for structured choices.

**22-item coverage:** None.

**Reviewer should add (when skill touches external systems):**

- Check for setup docs or `config.json` / `config.example.json`.
- If skill posts to Slack, deploys, or uses credentials: **Important** if no
  setup path documented.

**Maps to:** new **F-29**.

---

### B6. Memory and persistent data

**Article:** Append logs / JSON; store durable data in `${CLAUDE_PLUGIN_DATA}`,
not only inside the skill folder (upgrade may delete).

**22-item coverage:** None.

**Reviewer should add (when skill mentions history, logs, prior runs):**

- If skill writes `*.log` or state under its own directory → warn about upgrade
  data loss; suggest `${CLAUDE_PLUGIN_DATA}` or external path.
- If skill is stateless → N/A.

**Maps to:** new **F-30**.

---

### B7. On-demand hooks (Claude Code)

**Article:** Skills can register session hooks (e.g. `/careful`, `/freeze`).

**22-item coverage:** None.

**Reviewer should add:**

- For **Verification**, **Code Quality**, **Infra Ops**: advisory note if hooks
  would help (auto-run on save, block destructive ops) but are absent.
- Do not FAIL skills that are purely instructional with no automation need.

**Maps to:** new **F-31**.

---

### B8. Avoid railroading

**Article:** Over-specific step lists reduce reuse across situations.

**22-item coverage:** #10 requires clear steps — can conflict with this article.

**Reviewer should add:**

- Balance #10 PASS with railroading check: are steps **conditional** ("if
  directory mode…", "if user only pasted text…") vs one rigid path only?
- Flag skills with no input variants and 20+ mandatory micro-steps.

**Maps to:** new **AP-22**.

---

### B9. Composing skills

**Article:** Reference other skills by name; model invokes if installed.

**22-item coverage:** F-25 (composability / don't break others) — not
"depends on skill X".

**Reviewer should add:**

- If skill says "use the pdf skill" or similar: verify named skill is cited
  clearly; note dependency for marketplace installs.

**Maps to:** extend F-25.

---

### B10. Measuring skills (usage telemetry)

**Article:** PreToolUse hook to log skill usage; find undertriggering.

**22-item coverage:** None.

**Reviewer should add (team/public tier only):**

- For skills intended for org-wide use: ask if owner tracks trigger rate.
- Optional: point to article gist for logging hook.

**Maps to:** new **F-32** (collaboration layer).

---

## Part C — Recommended improvements (prioritized)

Status legend: **DONE** = landed in repo; **OPEN** = still outstanding.

### P0 — Dogfood (fix skill-reviewer itself)

| # | Action | Status | Resolves |
|---|--------|--------|----------|
| 1 | Add `scripts/mechanical-check.sh` (items #3, #5, #16) | OPEN | Article "Store Scripts"; self #11 gap |
| 2 | Add `## Gotchas` to SKILL.md (or `references/gotchas.md`) | OPEN | Article highest-signal section |
| 3 | Document `${CLAUDE_PLUGIN_DATA}` for `results.md` append-only logs | OPEN | Article Memory |

### P1 — Extend reviewer (audit others)

| # | Action | Status | Resolves |
|---|--------|--------|----------|
| 4 | Add **Phase 2.1 — Claude Code Lessons (advisory)** in SKILL.md | DONE | B1–B10 |
| 5 | Add F-28–F-32 and AP-21–AP-22 to `references/writing-good-skills.md` | DONE | Lookup during audits |
| 6 | Report new findings under Part A with `[claude-code-lessons]` tag | DONE | Traceability; also in Phase 3 impact-layer table |
| 6b | Calibrate Part C on good / medium / bad `expected-report.md` | DONE | Regression for Phase 2.1 |

### P2 — Distribution evidence

| # | Action | Status | Resolves |
|---|--------|--------|----------|
| 7 | Complete `evals/results.md` #20, #22 | OPEN | Existing checklist |
| 8 | Optional PreToolUse usage hook for skill-reviewer | OPEN | Article Measuring Skills |

---

## Part D — Quick mapping table (article → 22-item → gap)

| Article tip | Closest 22-item / extension | Gap for reviewer |
|-------------|----------------------------|------------------|
| 9 skill types | F-28 | Covered (Phase 2.1) |
| Gotchas | AP-21 | Covered (Phase 2.1) |
| Folder not file | #4, #8, #9 | OK when present; no dedicated `assets/` check |
| Don't state obvious | AP-13 (extended) | Covered |
| Railroading | AP-22 vs #10 | Covered (advisory) |
| config.json setup | F-29 | Covered (Phase 2.1) |
| Description for model | #1, #2, F-23 | Covered |
| Memory / PLUGIN_DATA | F-30 | Covered (Phase 2.1) |
| Scripts | #11–#15 + F-28 type-appropriate scripts | Covered |
| Hooks | F-31 | Covered (advisory) |
| Curation / evals | #19–#22 | Covered |
| Compose skills | F-25 (named deps) | Covered |
| Usage metrics | F-32 | Covered (advisory) |

---

## References

- Local article copy:
  [Lessons from Building Claude Code - How We Use Skills.md](Lessons%20from%20Building%20Claude%20Code%20-%20How%20We%20Use%20Skills.md)
- Audit extensions (F-28+, AP-21+):
  [references/writing-good-skills.md](../references/writing-good-skills.md)
- Core checklist: [SKILL.md](../SKILL.md)
