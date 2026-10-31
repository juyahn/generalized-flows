# Workflow: Implementation (Phase 3, 3.5, 3.6)

> **When to read:** Implementing from a dev plan, debugging issues.

---

## Phase 3: Execution (Implementation Mode)

### Pre-Implementation Gate

Before writing any code, **verify a dev plan exists** (unless 📄 QUICK depth with no existing doc):

```
Check: Does 02_dev_plans/[JIRA-ID]_*.md exist?
├── YES → Read it and use as implementation guide
├── NO + QUICK depth → OK, proceed with verbal plan
└── NO + STANDARD/COMPREHENSIVE → STOP. Go back to Phase 2 (workflow-01-planning.md).
    Do NOT use EnterPlanMode as a shortcut. Write the dev plan file first.
```

Start a **new session** with the completed spec and dev plan:
- Reference both spec and dev plan files directly
- Follow the dev plan as the implementation guide
- Read the project's learnings file for implementation gotchas: `03_knowledges/learnings-[project].md`
- Flag any ambiguities found during implementation

### Pre-Implementation: Worktree & Branch Setup

**Default to dedicated worktree per ticket** (📋 STANDARD / 📚 COMPREHENSIVE / ⚔️ ADVERSARIAL). Skip only for 📄 QUICK.

```
Depth check:
├── 📄 QUICK → Branch verify only (no worktree)
└── 📋+ → Create or enter dedicated worktree
```

**📋 STANDARD and above:**

Check if worktree already exists for this ticket. If not, create one:

```bash
git worktree add ../[repo]-[JIRA-ID] -b [JIRA-ID]   # new branch
# or, if branch already exists:
git worktree add ../[repo]-[JIRA-ID] [JIRA-ID]
```

Then `cd` into it and run all subsequent commands from there. The git worktree viewer surfaces it; cleanup is handled by routines, so don't remove manually mid-flow.

**📄 QUICK only:**

Branch verify in place — no worktree:

```bash
git branch --show-current   # expect [JIRA-ID]
```

If not on the correct branch, checkout or create: `git checkout -b [JIRA-ID]` or `git checkout [JIRA-ID]`.

**Branch naming:** in my own repos, `[JIRA-ID]` only (e.g., `PROJ-123`), no description suffix. **In team repos this does not apply**: a repo may use its own pattern such as `fix/<customer>-<slug>`. Check `.claude/team.json` before naming a branch.

### Implementation Workflow

**Run implementation to completion first, then review, then verify with user.** Don't stop after each step for review/verify — that fragments the flow and burns context. Commit at task or feature boundaries, not arbitrarily.

```
┌────────────────────────────────────────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────────┐
│  IMPLEMENT all dev plan steps                  │────▶│   REVIEW     │────▶│   FIX        │────▶│  USER REVIEW     │
│  (commit per task/feature unit as you go)      │     │  (Quality)   │     │  (findings)  │     │  (HTML + verify) │
└────────────────────────────────────────────────┘     └──────────────┘     └──────────────┘     └──────────────────┘
                                                                                   │                       │
                                                                                   └──── if user finds issues ───┐
                                                                                                                 ▼
                                                                                                          back to FIX
```

| Phase | Actions | Gate |
|-------|---------|------|
| **Implement** | Work through all dev plan steps end-to-end. Commit at each task/feature boundary (not per micro-step). Update **Reality Log** as you go; if >3 major deviations, pause and revise the dev plan. | All planned steps done, code compiles |
| **Review** | Spawn `code-reviewer` once over the full set of changes (see below) | Verdict received |
| **Fix** | Address review findings; commit fixes as `fix:` or `refactor:` | Findings resolved or accepted |
| **User Review** | Generate HTML code flow (unless 📄 QUICK / trivial), run code with user, debug issues together, commit any fixes | User confirms it works |

### Review (Claude's Responsibility)

After all dev plan steps are implemented and committed, spawn `code-reviewer` once over the cumulative changes:

```
Task tool → code-reviewer agent
Input: dev plan path + branch (or list of changed files)
Output: PASS / NEEDS FIXES (per-finding severity + confidence)
```

**What it checks:**
1. **Spec Compliance** - Does the code match the dev plan?
2. **Code Quality** - Simplicity, DRY, correctness, lint

**Auto-trigger:** After the last implementation commit, automatically spawn `code-reviewer` before handing off to the user. Address findings, commit fixes, then move to user review.

### User Review (User's Responsibility)

User runs the code and confirms it works. Claude assists with debugging anything that surfaces.

**Code Flow Review HTML:** Before user runs it, generate an interactive HTML walkthrough so the user can trace the execution flow:

1. Run `/review-html-generator` (or say "generate review html" / "code flow review")
2. Steps ordered by **call hierarchy** (L0 entry → L1 callees → L2 deeper), not file-line order
3. Opens in browser with collapsible code snippets for the user to review
4. **Output:** `00_CLAUDE/05_reviews/[JIRA-ID]-code-flow-review.html`
5. **When to skip:** 📄 QUICK depth or trivial changes (< 3 files, no new logic)

If the user finds issues, fix → commit (`fix:` or `refactor:`) → re-verify. Loop until the user confirms.

### Commit Message Format

Use [Conventional Commits](https://www.conventionalcommits.org/) format:

```
<type>: <short description>
```

**Types:** `feat`, `fix`, `refactor`, `docs`, `test`, `chore`

**Examples:**
```
feat: add WebSocket reconnection with exponential backoff
fix: handle null response from API
refactor: extract validation logic to helper
```

### Rules
- **Grep before you write.** Before adding a helper, util, type, or pattern, search for an existing one — re-implementing what already lives a few files over is the most common slop. A pre-write reflex; cheaper than letting `code-reviewer` catch the dup after the fact.
- **Run the full dev plan to completion before stopping for review.** Do not pause after each step waiting for user input — implement, commit, move on.
- **Commit at task or feature boundaries**, not per micro-step. A single logical change = one commit. If a "step" in the dev plan is a one-line tweak, fold it into a related commit; don't fragment.
- **DO NOT skip steps** — follow the dev plan order, even if you batch their commits.
- **MUST FOLLOW** commit format, **DO NOT** use commit messages starting with `[JIRA-ID]`
- If changes span multiple repos, commit each repo separately
- Review-stage and user-review fixes ship as their own commits (`fix:` / `refactor:`), not amended into earlier ones.

### Pre-PR Review

After all tasks are committed, before creating PR. The main thread fires reviewers in parallel, collects structured verdicts, and aggregates via disagreement triage.

#### Parallel Fan-Out

> **Auto-trigger:** CC fires these in parallel (single message, multiple tool calls). Do not wait for user to request.

| Reviewer | Depth trigger | How to invoke |
|----------|---------------|---------------|
| **`pr-reviewer`** (Claude Opus) | Always | `Task` tool, subagent_type=`pr-reviewer`. Input: branch name + dev plan path. |
| **Codex adversarial** | Required: ⚔️ ADVERSARIAL, OR large PR (5+ files), OR security-sensitive change. Optional: any other 📋 or 📚 PR, when an independent read is worth one call | `! 00_CODEX/03_scripts/codex-run.sh -r review "..."`, or `-r plan` when the PR moves a permission boundary, a consistency guarantee, or a concurrency invariant. Save output to `00_CODEX/02_reviews/prs/[JIRA-ID]-codex-review.md`. See `codex-review` skill Mode 2 for the prompt. |

All reviewers emit the **structured verdict schema**:

```
verdict: SHIP | HOLD | BLOCK
confidence: high | medium | low
top_risk: <one sentence>
<findings grouped by severity: BLOCK / HIGH / MEDIUM / LOW>
<evidence coverage: Checked / Not checked>
```

#### Aggregation (Disagreement Triage)

After all reviewers return, CC computes the aggregate. **Do NOT use a percentage threshold** — with 2-3 reviewers and binary verdicts, percentages collapse to unanimity. Use these rules instead:

| Condition | Action |
|-----------|--------|
| Any reviewer returns **BLOCK** | **HALT.** Fix the BLOCK findings. Re-run the reviewers that blocked. Do not create PR. |
| All reviewers return **SHIP** | **Proceed to PR creation.** Include `top_risk` lines in PR description under "Known risks". |
| Mix of **HOLD** and **SHIP** | **Present the split to the user.** Show each reviewer's verdict + top_risk + key HIGH findings. User decides ship vs hold. |
| All **HOLD** | **Fix HIGH findings** (or downgrade with user acknowledgment). Re-run. |
| Unclear / reviewer failed to emit schema | **Re-run that reviewer.** Do not synthesize from malformed output. |

Write the aggregated result to `00_CODEX/02_reviews/prs/[JIRA-ID]-combined-review.md`:

```markdown
## Combined PR Review: [JIRA-ID]

### Aggregate
- **Final verdict:** SHIP | HOLD | BLOCK | USER_DECIDE
- **Top risks (deduped):**
  1. <from reviewer X>
  2. <from reviewer Y>

### Per-reviewer verdicts
| Reviewer | Verdict | Confidence | Top risk |
|----------|---------|------------|----------|
| pr-reviewer | SHIP/HOLD/BLOCK | high/med/low | ... |
| codex | SHIP/HOLD/BLOCK | high/med/low | ... |

### Deduplicated findings
**BLOCK** — <finding, flagged by {pr-reviewer, codex}>
**HIGH** — ...

### Disagreements
- <Point where reviewers differ>: pr-reviewer said X; codex said Y. Resolution: <fix / user decision / accepted risk>.

### Evidence coverage gaps
- Neither reviewer checked: <e.g., live CVE lookup, cross-repo runtime> — note if relevant.
```

#### Rules

- **Evidence coverage matters more than reviewer count.** If both reviewers say "SHIP" but neither checked cross-repo integration and the change touches a cross-repo boundary, that's a coverage gap — surface it, don't ignore it.
- **No auto-averaging of disagreements.** If pr-reviewer says SHIP and Codex says HOLD, do not pick one silently. Present both to the user with the specific finding that caused the split.
- **Re-run reviewers after fixing BLOCK/HIGH issues.** A fresh verdict, not a patched one.
- **Token note:** `pr-reviewer` is Opus and Codex CLI is billed separately. Both fire on every ⚔️ PR review by design — this is where the "spend tokens for independent scrutiny" principle applies.

### Parallel Review (Agent Teams)

For **large PRs** or **high-risk changes**, use agent teams to review from multiple perspectives simultaneously.

**When to Use:**
- Changes spanning 5+ files
- Security-sensitive code (auth, payments, data access)
- Performance-critical paths
- Architectural changes

**How it works:** Claude is the lead — spawns the reviewers in parallel and synthesizes their findings into one prioritized list.

**Standard Reviewers:**

| Reviewer | Focus | What It Looks For |
|----------|-------|-------------------|
| **Spec Compliance** | Dev plan adherence | Does implementation match spec? Missing requirements? |
| **Security** | OWASP, auth, data | Injection, XSS, auth bypass, data exposure |
| **Performance** | Efficiency, scaling | N+1 queries, memory leaks, blocking calls |
| **Test Coverage** | Test quality | Edge cases covered? Assertions meaningful? |

**Trigger Phrases:**
- "Parallel review this"
- "Review from multiple angles"
- "Deep review before PR"

**Output:** Each reviewer reports issues with severity (critical/warning/info). Lead synthesizes into a single review summary with prioritized action items.

---

### Session End: Update Active Context

> **Auto-trigger:** At the end of every session that touched code in a project with an active-context file, update that file before stopping. Cheap, high leverage for resuming cold next session.

For each project with an active-context file at `00_CLAUDE/10_projects/[project]-active.md` (today: none — create one when a repo becomes a recurring focus), overwrite these fields:

- **Current focus** — 1-3 lines on what's being worked on
- **Last commit** — branch + short SHA + conventional-commit message (from `git log -1 --oneline` on the working branch)
- **Next step** — the literal next action to take when resuming (not a goal)
- **Blockers** — anything preventing the next step; "None" if unblocked
- **Related docs** — paths to current spec / dev plan / debugging doc being worked on
- **Last updated** — absolute date (YYYY-MM-DD)

Overwrite, do not append. The file is state, not history. If no session activity touched the project, do not update.

---

## Phase 3.5: Debugging (When Issues Arise)

> **Not for customer bot behavior.** If the thing behaving wrong is a deployed agent (a customer ticket, a playbook, a prompt, a KB answer), this phase is the wrong shape: it has no SOP gate, no repo-vs-deployed drift check, no layer classification, and it assumes a deterministic repro. Go to `workflow-04-delivery.md` (D-track) instead.

When bugs or issues are encountered during or after implementation:

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│  1. UNDERSTAND  │────▶│  2. REPRODUCE   │────▶│  3. INVESTIGATE │────▶│  4. FIX/VERIFY  │────▶│  5. PREVENT     │
│  (Symptoms)     │     │  (Confirm)      │     │  (Root Cause)   │     │  (Solution)     │     │  (Learn)        │
└─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘     └─────────────────┘
```

| Phase | Actions | Output |
|-------|---------|--------|
| **Understand** | Gather error messages, logs, stack traces, context | Clear problem statement |
| **Reproduce** | Identify steps to trigger the bug | Reproduction steps |
| **Investigate** | Search codebase, trace execution, check related code | Root cause identified |
| **Fix/Verify** | Propose fix, implement, test | Working code |
| **Prevent** | Identify what could prevent this class of bug in the future | Updates to guidelines/templates |

**Root cause = the shared chokepoint, not the reported path.** Before editing a function, grep every caller of it. One guard in the shared function is a smaller diff than a guard per caller — and patching only the path the ticket names leaves every sibling caller still broken. Fix it once, where all callers route through.

### Quick Decision: Doc or No Doc?

```
Bug Found
    │
    ├─── Can fix in < 5 min AND root cause is obvious?
    │       └── Yes → Fix, verify, commit (no doc needed)
    │
    ├─── Root cause unclear after 5 min?
    │       └── Yes → Spawn `debug-investigator` agent
    │
    ├─── Took > 30 min OR learned something new?
    │       └── Yes → Create debugging doc in 04_debugging/
    │
    └─── Third time seeing similar issue?
            └── Yes → Create debugging doc + update LESSONS_INDEX.md
```

### Debug Investigator

For non-obvious bugs, use the `debug-investigator` subagent:

```
Task tool → debug-investigator agent
Input: error message + stack trace + context
Output: Top 3 suspected locations + investigation path
```

**Optional Codex alternative.** For a second, independent hypothesis, or a long self-contained search whose next step needs only the conclusion, run `00_CODEX/03_scripts/codex-run.sh -r plan -s read-only -- -o 00_CODEX/02_reviews/debug/<JIRA-ID>-hypotheses.md "<error, stack trace, the paths to read, and: list the top 3 suspected locations with the evidence for each; do not fix>"`. Give it the evidence, not your current theory. Its shell may have no network, so keep it to local files.

### Competing Hypotheses (Agent Teams)

For **complex bugs** where the root cause is unclear and multiple theories exist, use agent teams to investigate in parallel.

**When to Use:**
- Multiple plausible root causes
- Bug spans multiple systems/services
- Previous investigation attempts failed
- Time-sensitive production issue

**How it works:** One agent per competing hypothesis (e.g. "DB timeout", "race condition", "config error"), plus a devil's advocate, all spawned in parallel.

**Key Principle:** Each agent actively tries to **disprove** the other hypotheses. The theory that survives scrutiny is most likely the root cause — a hypothesis nobody attacked is untested, not confirmed.

**Trigger Phrases:**
- "Debug with competing hypotheses"
- "Investigate multiple theories"
- "I have several guesses, let's test them all"

### Prevention Actions

After fixing a bug, ask: **"How do we prevent this from happening again?"**

| If the lesson is... | Update... | Example |
|---------------------|-----------|---------|
| Project-specific pattern | `00_CLAUDE/10_projects/[project].md` | "Always check the customer SOP before designing a playbook fix" |
| General coding guideline | `CLAUDE.md` | "Add validation for Z in all APIs" |
| Code structure pattern | `00_CLAUDE/09_templates/` | Add checklist item to dev-plan template |
| Common pitfall | `00_CLAUDE/04_debugging/` | Document for future reference |

### Trigger Phrases for Debug Mode:
- "Debug this error..."
- "Help me fix..."
- "I'm seeing this bug..."
- "Why is X happening?"

---

## Phase 3.6: Verification Checklist (UI / Behavior-Touching Changes)

> **When to read:** After implementation review-html, before PR creation, when the change touches UI or otherwise externally observable behavior.

For features whose acceptance criteria require manual human verification — UI flows, Slack interactions, new endpoints with visible output, new cronjob output — generate a verification checklist before opening the PR. The checklist is a runbook: each item names what to do, what success looks like, and (optionally) what evidence to capture.

### Trigger

From the workbench, with the Jira-ticket work item selected, run the **Verify checklist** preset (phase `phase_3_post`).

The preset launches Claude Code with a prompt template that reads:

- the spec at `00_CLAUDE/01_specs/[JIRA-ID]-*.md` (acceptance criteria),
- the dev plan at `00_CLAUDE/02_dev_plans/[JIRA-ID]_*.md` (task list),
- a `git log` summary on the current branch.

CC synthesizes a flat checklist and writes to `00_CLAUDE/12_verification/[JIRA-ID]-[feature-name]-verification.md`.

### Doc Shape

```markdown
# Verification: <feature title>

**Jira:** PROJ-XXXX
**Generated:** YYYY-MM-DD
**Sources:** spec, dev plan, recent commits

## Verification

- [ ] <one-line observable check> (AC1 | Task 2.3 | commit a1b2c3d)
  - **How:** <concrete steps>
  - **Expected:** <what success looks like>
  - **Evidence:** <user fills, OR omitted entirely when absence is the evidence>
```

`**How:**` and `**Expected:**` are mandatory; `**Evidence:**` is optional. Every item ends with a parenthetical trace `(AC<n> | Task <x.y> | commit <short-sha>)` so traceability survives regeneration.

### Tick Loop

The Docs Library `Verify` chip surfaces all generated files. Open the file in DocPreview and tick boxes as you manually verify — clicking a checkbox writes back to the source `.md` file in place via the workbench backend. External-editor edits to the same file also work; the file is canonical.

### Depth-Specific Behavior

| Depth | Verification |
|-------|--------------|
| **📄 QUICK** | Run only when a spec file exists; checklist is derived from spec ACs + `git log` (no dev plan input). Skip entirely when no spec exists. |
| **📋 STANDARD / 📚 COMPREHENSIVE / ⚔️ ADVERSARIAL** | Full flow. Sources: spec ACs + dev plan tasks + `git log` summary. |

### Skip Cases

Skip Phase 3.6 entirely for: pure refactors with no behavior change, internal-library / config-only changes, dependency bumps with no code path change. The phase is conditional, not universal — invocation is the user's call, no auto-detection.

### Regeneration

Re-running the preset on a feature that already has a verification doc **prepends a `**Revision YYYY-MM-DD:**` block** with the new content and preserves prior content verbatim below. The preset never silently overwrites or refuses. Prior ticks remain visible in the prior revision; the new revision starts unticked.

### Relation to Other Phases

- After Phase 3.75 review-html (`00_CLAUDE/05_reviews/[JIRA-ID]-code-flow-review.html`) — review-html surfaces *what changed*; Phase 3.6 surfaces *how to verify*.
- Before PR creation (`generate_pr_body` preset) — but **advisory only**. The PR preset does not gate on a ticked verification doc. Discipline is on the user, not the harness.

---

## Reference: Local Environments

When you need to execute code, use each repo's own setup. List them per engagement:

| Project | Command | Notes |
|---------|---------|-------|
| `<repo>` | `make up` or the repo's setup skill | If a setup skill or script exists, use it rather than hand-rolling `.env` |

Prefer an idempotent setup command, so running it again is safe.
