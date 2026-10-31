# Workflow: Planning (Phase 1, 1.5, 2)

> **When to read:** Starting a new feature, creating specs, or planning implementation.

---

## Phase 0: Determine Planning Depth

Before starting, **determine the appropriate depth** based on task complexity:

| Depth | When to Use | Spec | Dev Plan | Research |
|-------|-------------|------|----------|----------|
| **📄 QUICK** | Bug fixes, small changes, config tweaks | Append to existing (or skip) | Append to existing (or skip) | Local only |
| **📋 STANDARD** | Most features, moderate complexity | Create new | Create new | Local + conditional external |
| **📚 COMPREHENSIVE** | Architectural changes, major features, high-risk | Create new (detailed) | Create new (phased) | Full parallel research |
| **⚔️ ADVERSARIAL** | Critical architecture, multi-service, high-stakes | Create new (detailed) | Create new (phased) + Codex debate | Full parallel + Codex adversarial review |

### QUICK Mode Rules
- **Only append** to existing spec/dev-plan docs for the JIRA ID
- **If no existing docs exist → skip doc creation entirely**
- Jump straight to implementation after brief exploration
- Use for: typo fixes, config changes, simple bug fixes, minor UI tweaks

### How to Detect Depth
Ask user OR infer from context:
```
Is this a quick fix or a larger feature?
- 📄 Quick fix (< 30 min, no new files)
- 📋 Standard feature (new functionality)
- 📚 Major change (architecture, multi-service)
- ⚔️ Critical architecture (multi-service, high-stakes, needs adversarial debate with Codex)
```

---

## Phase 1: Spec Discovery (Interview Mode)

When the user starts with a minimal spec or feature idea, **DO NOT jump into implementation**. Instead:

1. **Acknowledge** you're entering "spec discovery mode"
2. **FIRST, ask for the Jira ticket ID** (e.g., PROJ-1234 or CUST-1234; see `.claude/team.json` for which project) - this is required for file naming
3. **Use AskUserQuestion tool** to interview the user about:
   - Jira ticket ID (if not already provided)
   - **Planning depth** (📄 QUICK / 📋 STANDARD / 📚 COMPREHENSIVE / ⚔️ ADVERSARIAL) — see Phase 0 table. Ask this in the first batch so downstream behavior (spec detail, Codex passes, teamreview) is set early.
   - Core requirements and user stories
   - Key technical decisions (APIs, data models, integrations)
   - Scope boundaries (what's in vs out)
   - Success criteria and acceptance conditions
4. **Ask 2-4 questions at a time** to avoid overwhelming
5. **Synthesize answers** into a structured spec document — wherever a requirement is still implicit or assumed, insert `[NEEDS CLARIFICATION: <question>]` inline rather than guessing (see TEMPLATE-spec.md)
6. **Save the spec** to `00_CLAUDE/01_specs/[JIRA-ID]-[feature-name].md`
7. **Resolve all `[NEEDS CLARIFICATION]` markers** — grep the saved spec; for each remaining marker, ask the user via `AskUserQuestion` and replace with the answer. A spec with unresolved markers cannot advance to Phase 1.5.
8. **Review the spec** with the user before proceeding, in three parts:
   - **What this work will do**: a flow diagram, one step per line, each with a real example value.
   - **What it will not do**: a table, each row with the reason or when it happens instead.
   - **What becomes visible to other people, and when**: a timeline by phase naming each place a trace appears (test platform project, dashboard, shared mock, branch, PR, channel) and who sees it. Mark the first point anyone else can see, the point from which other people's work is affected (e.g. a merged golden picked up by their PRs), and every step that waits for the user's go-ahead. If nothing done so far is visible, say so first.

   The third part goes into the spec's `### Visibility` section.

### Depth-Specific Behavior

| Depth | Spec Behavior |
|-------|---------------|
| **📄 QUICK** | Check for existing `01_specs/[JIRA-ID]-*.md`. If exists → append notes. If not → **skip spec creation**, proceed with verbal agreement. |
| **📋 STANDARD** | Create new spec with core sections (requirements, scope, success criteria) |
| **📚 COMPREHENSIVE** | Create detailed spec with all sections + edge cases + risk analysis |
| **⚔️ ADVERSARIAL** | Same as 📚 COMPREHENSIVE — detailed spec with all sections |

### QUICK Mode Spec Flow
```
1. Check: Does 01_specs/[JIRA-ID]-*.md exist?
   ├── YES → Append brief notes under "## Updates" section
   └── NO → Skip spec file, confirm requirements verbally, proceed to Phase 1.5
```

### Trigger Phrases for Spec Mode:
- "Let's spec out..."
- "Interview me about..."
- "Help me define..."
- "I want to build [feature] - ask me questions"

### Codex Spec Ambiguity Pass (📋 STANDARD and above)

> **Auto-trigger:** For 📋 STANDARD, 📚 COMPREHENSIVE, and ⚔️ ADVERSARIAL depths, CC MUST run this after saving the spec file, before moving to Phase 1.5. Skip for 📄 QUICK.

**Why this step exists:** Spec bugs are ~10× cheaper to fix than the code bugs they become. Codex reads the spec cold with no conversation context, which surfaces hidden assumptions that CC missed during the interview.

**Flow:**

1. Send spec to Codex for an ambiguity pass (not an adversarial plan critique — that's Phase 2):

```bash
! 00_CODEX/03_scripts/codex-run.sh -r implement "Read 00_CLAUDE/01_specs/[JIRA-ID]-[feature-name].md. Do NOT propose implementation. Output ONLY: (1) Unanswered questions a developer would hit during implementation. (2) Hidden assumptions the spec makes without stating. (3) Missing acceptance criteria or success conditions. (4) Scope ambiguities (in vs out unclear). Be specific — cite spec section names. Skip items the spec already addresses clearly. If the spec is fully clear, say 'No ambiguities found' and stop. Save output to 00_CODEX/02_reviews/plans/[JIRA-ID]-spec-ambiguity.md."
```

2. Read Codex's output. If "No ambiguities found", proceed to Phase 1.5.

3. If ambiguities exist, present them to the user as follow-up interview questions (use `AskUserQuestion`). Update the spec file with answers under a `## Clarifications` section.

4. Proceed to Phase 1.5 only after ambiguities are resolved.

**Rules:**
- This is a **one-shot** pass — no debate rounds. Codex lists gaps, CC + user fill them.
- Do not let Codex propose solutions at this stage. Solutions belong in Phase 2.
- If Codex invents ambiguities that aren't real (e.g., "what database type?" when spec says PostgreSQL), discard them silently rather than asking the user.

---

## Phase 1.5: Codebase Exploration

Before planning, **explore the existing codebase** and **review past lessons**.

### Step 0: Identify Target Project

Determine which project(s) this work affects:

| Project | Architect Agent |
|---------|-----------------|
| any repo | none yet — use `Explore` |

Per-repo architect agents are optional. Write one only once a repo is understood well enough that the agent states facts rather than guesses; until then a broad `Explore` fan-out is the honest substitute. Repos owned by another team (listed in `.claude/team.json`) change under you, so an architect for them would also have to track their changes, not just their current shape.
| Multiple repos | Launch multiple architects in parallel |

### Step 1: Local Research (Run in Parallel)

Launch these research tasks **simultaneously** using parallel Task tool calls:

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| **[project]-architect** | Architecture, patterns, cross-repo integration | Always for the target project(s) |
| **Explore** | Quick code search | Find specific code, grep for patterns |
| **Read** (lessons) | Check past learnings | Always - read `LESSONS_INDEX.md` |

**Use architect agents for:**
- Understanding component relationships
- Finding patterns to follow
- Cross-repo coordination (for example agent config ↔ tool proxy ↔ runtime)
- Design decisions and trade-offs
- **Spec validation against code** — verify file paths, line numbers, function names, mode/route/field names, and architecture claims in the spec actually match the current code. For each spec assertion that names a code anchor, the architect should return a confirmation (`file:line` exists, signature matches) or a contradiction (spec says X, code says Y, evidence at `file:line`). The contradictions become the dev plan's `## Spec Corrections` section in Phase 2.

**Why this matters:** the dry-run analysis (2026-04-26) found that spec-only Phase 2 dev plans missed critical spec/code mismatches — e.g., spec said `route="tools"` but code at `session.go:671` used `route="scripts"`; spec said bot-token post but `slack.go:567` drops `event.BotID != ""`. Phase 1.5 must catch these before Phase 2, otherwise the dev plan codifies bugs.

**Use Explore for:**
- Quick "find all usages of X"
- Searching for specific strings/patterns
- When you don't need architectural context

Example for a prompt or playbook change: `Explore` on "which customers' playbooks reference this tool", `Explore` on "where is this rule duplicated across `*/sop/`", and a `Read` of the lessons index — all three in one message.

### Step 2: Research Decision Point

After local research, decide if **external research** is needed:

| Condition | Decision |
|-----------|----------|
| High-risk topic (security, payments, external APIs) | → Always do external research |
| Unfamiliar framework/library | → Do external research |
| Strong local context found | → Skip external research |
| **📄 QUICK depth** | → Always skip external research |

### Step 3: External Research (Conditional, Parallel)

If warranted, launch in parallel:

| Agent | Purpose | When |
|-------|---------|------|
| **WebSearch** | Best practices | New patterns, industry standards |
| **WebFetch** | Framework docs | Specific library usage |

### Step 4: Consolidate Research

Synthesize all findings into:

```markdown
## Research Summary

### Relevant Files
- `path/to/file.py:42` - [what it does]

### Patterns to Follow
- [Pattern name]: [how it's used]

### Lessons to Apply
- [From LESSONS_INDEX.md]: [lesson summary]

### External References (if any)
- [URL]: [key takeaway]
```

### Depth-Specific Behavior

| Depth | Research Scope |
|-------|----------------|
| **📄 QUICK** | Single Explore agent OR skip entirely if change is obvious |
| **📋 STANDARD** | Full local research (parallel), conditional external |
| **📚 COMPREHENSIVE** | Full local + full external research |
| **⚔️ ADVERSARIAL** | Full local + full external + Codex adversarial review in Phase 2 |

### Trigger Phrases for Explore Mode:
- "Explore the codebase for..."
- "Find similar implementations..."
- "How does [feature] work?"

### Multi-Perspective Exploration (`/teamreview` skill)

For **📚 COMPREHENSIVE** and **⚔️ ADVERSARIAL** tasks, run `/teamreview [JIRA-ID]` after the spec is saved (spec-only mode). Auto-trigger for these depths — do not wait for user to invoke.

**What it does:** Spawns 4 parallel agents (architect, security, performance, edge-cases) that read the spec and emit structured verdicts (SHIP/HOLD/BLOCK + findings + evidence coverage). Aggregates via disagreement triage — any BLOCK halts, split verdicts surface to user. Output saved to `00_CLAUDE/05_reviews/[JIRA-ID]-teamreview-spec-only.md`.

**Relation to Codex spec-ambiguity pass (Phase 1):** Codex finds gaps in the spec as written; teamreview stress-tests the spec from 4 adversarial angles. Complementary, not redundant. Run Codex pass first (cheap, single-shot), then teamreview (broader, higher-cost).

**When to Use Manual Multi-Perspective Instead:**
- 📋 STANDARD tasks where the user wants extra scrutiny
- Debugging with multiple competing hypotheses (different use case — see Phase 3.5)

**How it works:** Claude is the lead — spawns the four perspectives in parallel, lets them challenge each other's assumptions, then synthesizes. The lead decides; the perspectives advise.

**Standard Perspectives:**

| Perspective | Focus | Questions It Asks |
|-------------|-------|-------------------|
| **Architecture** | System design, patterns | "How does this fit existing patterns? What abstractions?" |
| **Edge Cases** | Failure modes, boundaries | "What happens when X fails? What are the limits?" |
| **Performance** | Scalability, resources | "Will this scale? What's the memory/CPU impact?" |
| **Security** | Auth, validation, exposure | "What can be exploited? What data is exposed?" |

**Trigger Phrases:**
- "Spec this with team"
- "Explore from multiple perspectives"
- "I need different angles on this"
- "Debate the trade-offs"

**Output:** Each perspective reports findings, challenges others' assumptions, and the lead synthesizes into a consolidated research summary with trade-offs clearly documented.

---

## Phase 2: Development Planning

> **DO NOT use `EnterPlanMode` tool here.** Phase 2 means writing a dev plan *file* in `00_CLAUDE/02_dev_plans/`, not entering the built-in plan mode. The dev plan file is the deliverable of this phase. Do not proceed to Phase 3 until the dev plan file exists and the user has approved it.

After spec approval and codebase exploration, create an **implementation plan** in `00_CLAUDE/02_dev_plans/`.

### Depth-Specific Behavior

| Depth | Dev Plan Behavior |
|-------|-------------------|
| **📄 QUICK** | Check for existing `02_dev_plans/[JIRA-ID]_*.md`. If exists → append tasks. If not → **skip dev plan**, proceed directly to implementation with verbal task list. |
| **📋 STANDARD** | Create new dev plan with tasks, file table, single approach |
| **📚 COMPREHENSIVE** | Create detailed dev plan with multiple approaches, architecture diagrams, phased implementation |
| **⚔️ ADVERSARIAL** | Same as 📚 + **Codex debate** before finalizing chosen approach (see below) |

### QUICK Mode Dev Plan Flow
```
1. Check: Does 02_dev_plans/[JIRA-ID]_*.md exist?
   ├── YES → Append new tasks under existing task list
   └── NO → Skip dev plan file, verbally confirm:
            "I'll make these changes: [1-3 bullet points]"
            Then proceed directly to implementation
```

### Multi-Approach Architecture Design (STANDARD/COMPREHENSIVE only)

Design **2-3 implementation approaches** before choosing:

| Approach | Focus | Trade-offs |
|----------|-------|------------|
| **Minimal** | Smallest change, maximum reuse of existing code | Fast to implement, but may add tech debt |
| **Clean** | Maintainability, elegant abstractions | More work upfront, better long-term |
| **Pragmatic** | Balance of speed + quality (recommended) | Good default choice |

**Ask user which approach they prefer** before finalizing the dev plan.

> **📄 QUICK mode:** Skip approach comparison. Use the most obvious/minimal approach.

### Dev Plan Creation

1. **Create dev plan** at `00_CLAUDE/02_dev_plans/[JIRA-ID]_[feature-name]_[YYYY-MM-DD].md`
   - Example: `PROJ-123_playbook-lint-rule_2026-09-08.md`
2. **Include in the dev plan:**

| Section | 📄 QUICK | 📋 STANDARD | 📚 COMPREHENSIVE | ⚔️ ADVERSARIAL |
|---------|----------|-------------|------------------|----------------|
| Task list | Verbal only | Yes | Yes (phased) | Yes (phased) |
| File table | No | Yes | Yes | Yes |
| Architecture diagram | No | Optional | Yes | Yes |
| Approaches considered | No | No | Yes | Yes |
| Risk analysis | No | No | Yes | Yes |
| Codex independent plan v1 (plan-generate) | No | Opt-in | Opt-in | **Yes** (auto) |
| Codex revised plan v2 + disagreement list (cross-review) | No | If plan-generate ran | If plan-generate ran | **Yes** (auto) |
| Codex debate (plan-debate) | No | No | No | **Yes** |
| Dual-Plan Synthesis section | No | If plan-generate ran | If plan-generate ran | **Yes** |
| Adversarial Review section | No | No | No | **Yes** |

### Task Sizing

**One task = one logical commit.** Size each task so its diff is a single coherent change that a reviewer can judge on its own — the same boundary Phase 3 commits at. Do not decompose below that: a task that is one line of a larger change belongs folded into that change, and micro-tasks fragment both the plan and the git history.

Split when the work crosses a real seam — a different file's responsibility, a schema change vs the code that uses it, a behavior change vs its test. Keep together when splitting would leave the repo in a state that doesn't build or doesn't make sense.

Each task names its file(s) and shows the concrete change:

```markdown
### Task 1.1: Add status field to model

**File:** `backend/models/connection.py`

```python
class Connection(models.Model):
    # ... existing fields ...
    status = models.CharField(max_length=20, default='pending')  # ADD
```
```

**Template:** Use `00_CLAUDE/09_templates/TEMPLATE-dev-plan.md`

### Architect Validation (STANDARD/COMPREHENSIVE/ADVERSARIAL)

> **Auto-trigger:** CC MUST run this automatically after writing the dev plan. Do not wait for user to ask.

After writing the dev plan and **before asking the user to approve**, spawn the relevant `*-architect` agent to sanity-check:

```
Task tool → [project]-architect agent
Input: "Review this dev plan: [path to dev plan file]. Check for:
  - Does the approach follow existing codebase patterns?
  - Are file locations and module boundaries correct?
  - Any cross-repo implications missed?
  - Any LESSONS_INDEX pitfalls that apply?
  - **Gates check:** verify every gate in the dev plan's '## Gates' section is either ticked Pass OR has a specific written justification. Flag any unticked + unjustified gate as a blocker."
Output: OK / Issues found
```

If the architect flags issues, revise the dev plan before presenting to the user. Architect must confirm all Gates pass or are justified — this fights over-engineering at the planning stage.

> **📄 QUICK mode:** Skip architect validation.

### Codex Independent Plan + Cross-Review — plan-generate (⚔️ auto, 📚 and 📋 opt-in)

> **Auto-trigger:** ⚔️ ADVERSARIAL runs plan-generate automatically **before** plan-debate. 📚 COMPREHENSIVE and 📋 STANDARD run it as an option: when the user says "codex plan" / "dual plan" / "plan with codex", or when CC offers it because the spec admits more than one reasonable decomposition. Skip for 📄.

Catches framing-level blind spots that plan-debate can't — Codex works from the spec independently, without seeing CC's in-progress plan, so a different decomposition or architecture can surface. **Cross-review step** (added 2026-05-02) gives Codex a real chance to assert its position: after both sides draft independently, Codex revises its own plan with full cross-context (sees CC's v1) and emits a disagreement list. CC's final synthesis sees Codex's v1, v2, and disagreements — not just CC's own diff of two raw drafts. The prior flow had Codex's voice end at v1; this fixes that.

**Flow:**
```
Step 1 (parallel): CC drafts plan v1            Codex drafts plan v1 (spec-only)
Step 2 (Codex turn): Codex reads CC v1, writes Codex v2 + disagreement list
Step 3 (CC turn):    CC reads Codex v1 + v2 + disagreements,
                     writes final dev plan with ## Dual-Plan Synthesis section
Step 4 (⚔️ only):    Continue to plan-debate on synthesized draft
```

**Step 1 — Parallel independent drafts.**

CC begins its dev plan per the normal flow (saves v1 to dev plan file). In parallel, dispatch Codex (do NOT send Codex CC's draft):

```bash
! 00_CODEX/03_scripts/codex-run.sh -r plan "You are a senior engineer writing an independent dev plan. Read ONLY the spec at 00_CLAUDE/01_specs/[JIRA-ID]-*.md — do not look at any existing dev plan files. Produce a dev plan covering: decomposition into tasks, architecture/approach with rationale, key risks, testing strategy, rollout. Be opinionated — if a different decomposition or architecture would be better than an obvious first-pass approach, say so and explain why. Save to 00_CODEX/02_reviews/plans/[JIRA-ID]-codex-plan.md."
```

(Bash tool param `timeout: 600000`.)

**Step 2 — Codex cross-review (Codex revises its own plan with full context).**

Once both v1 drafts exist on disk, dispatch Codex to read CC's v1 alongside its own v1 and produce v2 + disagreement list:

```bash
! 00_CODEX/03_scripts/codex-run.sh -r plan "You wrote 00_CODEX/02_reviews/plans/[JIRA-ID]-codex-plan.md. Read CC's competing dev plan at 00_CLAUDE/02_dev_plans/[JIRA-ID]_*.md. Produce a revised version of YOUR plan, defending choices you still believe in and incorporating CC's points where you genuinely agree. Rules: (1) Do not capitulate — if you have a real reason for your original choice, defend it with reasoning, not 'CC said X so I'll change.' (2) Output two files: revised plan to 00_CODEX/02_reviews/plans/[JIRA-ID]-codex-plan-v2.md, and a disagreement list to 00_CODEX/02_reviews/plans/[JIRA-ID]-codex-disagreements.md. (3) Disagreement list format: for each CC choice you reject, cite which CC section, what CC chose, what you'd choose instead, and why (codebase constraint / pattern / risk Codex sees that CC missed). (4) If after honest reflection CC is right on every point, say so and write 'No remaining disagreements' in the disagreement file — but only if true. The point of this step is for you to assert your position with full context, not to defer."
```

**Why the anti-capitulation rule:** Codex defaults toward agreement when it sees CC's plan. The prompt explicitly instructs it to defend its position. CC's job in step 3 is to handle that pushback honestly, not to receive a sycophantic v2 that flattens differences.

**Step 3 — CC final synthesis with full cross-context.**

CC reads Codex's v1, v2, and disagreement list together. Revise CC's dev plan body to reflect adopted ideas, then append:

```markdown
## Dual-Plan Synthesis

**Codex plan v1:** `00_CODEX/02_reviews/plans/[JIRA-ID]-codex-plan.md`
**Codex plan v2 (post cross-review):** `00_CODEX/02_reviews/plans/[JIRA-ID]-codex-plan-v2.md`
**Codex disagreement list:** `00_CODEX/02_reviews/plans/[JIRA-ID]-codex-disagreements.md`

### Convergence (both sides moved here in v2)
- [Areas where cross-review brought both plans to the same answer]

### Adopted from Codex
- [Specific Codex idea now in CC's plan — note whether adopted from v1 or v2]

### Rejected from Codex (rebuttals to disagreement list)
- **[Codex's disagreement point]:** [Codex's proposed alternative + reason]. CC rejects because [codebase constraint / existing pattern / prior decision Codex lacks context on]. Evidence: [file:line or doc reference].

### Unresolved (surface to user)
- [Where neither side moved after cross-review — present both positions, do NOT auto-resolve]
```

**Rules:**
- A `## Dual-Plan Synthesis` section with empty "Rejected from Codex" + non-empty Codex disagreement list = audit failure. CC must engage every disagreement point — accept it (move to "Adopted from Codex") or rebut it (move to "Rejected from Codex"). Silent dismissal defeats the cross-review.
- "Unresolved" is a real outcome, not a fallback for laziness. Use only when both sides have substantive arguments and the call genuinely belongs to the user (e.g., risk tolerance, scope trade-off).
- If Codex's disagreement file says "No remaining disagreements", verify by spot-checking 1-2 axes where v1s diverged. Empty disagreement files happen, but they should be the exception.

See `.claude/skills/codex-review/SKILL.md` Mode 3 for the supporting skill detail.

### Codex Adversarial Debate — plan-debate (⚔️ ADVERSARIAL only)

> **Auto-trigger:** When ⚔️ ADVERSARIAL depth is active, CC MUST invoke this automatically after plan-generate synthesis + architect validation. Do not wait for user to say "debate with codex" — just do it.

After plan-generate and architect validation, **before finalizing the chosen approach**, invoke Codex for an independent adversarial review. Run the `/codex-review` skill or follow this flow manually:

```
CC writes dev plan (approaches TBD) → Codex critiques → CC rebuts → (optional R2) → Synthesize → Finalize plan
```

**Step 1:** Leave "Chosen Approach" as TBD in the dev plan.

**Step 2:** Send plan to Codex. Try `/codex:rescue` first; if it hangs (broker.sock issue), use direct CLI:

```
/codex:rescue --background Review the dev plan at 00_CLAUDE/02_dev_plans/[JIRA-ID]_*.md as an adversarial architect. Challenge architecture, assumptions, edge cases. Save review to 00_CODEX/02_reviews/plans/[JIRA-ID]-review.md
```

Fallback (if rescue hangs):
```bash
! 00_CODEX/03_scripts/codex-run.sh -r plan "You are an adversarial architect. Read 00_CLAUDE/01_specs/[JIRA-ID]-*.md and 00_CLAUDE/02_dev_plans/[JIRA-ID]_*.md. Write adversarial review to 00_CODEX/02_reviews/plans/[JIRA-ID]-review.md. Focus: architecture, assumptions, edge cases, cost. Format: Verdict, Critique, Assumptions Challenged, Edge Cases, Counter-Proposal, Agree With."
```

**Step 3:** Read Codex's critique (`/codex:result` or read the review file).

**Step 4:** CC writes structured rebuttal — append to dev plan under `## Adversarial Review`:

```markdown
## Adversarial Review

### Codex Critique Summary
[Key points]

### CC Response
#### Accepted
- [Point]: Agreed. Plan updated: [change].

#### Rebutted
- [Point]: Disagree — [reason + evidence].

#### Synthesized
- [Merged approach from both perspectives].
```

**Step 5 — Round 2 (evidence-gated, default-on for ⚔️):**

Run round 2 unless round 1 already reached consensus. Round 2 is **evidence-gated**: Codex must cite a specific round-1 claim to attack. If it only restates round 1, abort.

```bash
! 00_CODEX/03_scripts/codex-run.sh -r plan "Read the CC rebuttal appended to 00_CLAUDE/02_dev_plans/[JIRA-ID]_*.md under '## Adversarial Review'. Has your position changed? Rules: (1) Address ONLY rebutted points. (2) Each new critique must cite which specific CC rebuttal claim you disagree with and why (new evidence or reasoning). (3) If you have nothing substantively new to say, output 'No new critique — round 1 stands' and stop. Do not restate round 1. Save to 00_CODEX/02_reviews/plans/[JIRA-ID]-review-r2.md."
```

**Abort conditions for round 2:**
- Codex outputs "No new critique" → skip to Step 6
- Round 2 findings ⊆ round 1 findings (CC judgment call) → skip to Step 6
- Round 2 introduces new critiques unrelated to rebutted points → discard those, only engage with on-topic critiques

**Step 5b — CC rebuts round 2:** Append under `## Adversarial Review → Round 2` with same Accepted/Rebutted/Synthesized structure. Keep it shorter than round 1 — only the points that moved.

**Hard cap: 2 rounds.** If round 2 diverges instead of converging (new objections each round, no claim revision), append `UNRESOLVED — user decision` to the dev plan and present both positions to the user. Do not run round 3.

**Step 6:** Finalize "Chosen Approach" with rationale incorporating the debate. Present to user for approval.

> **Debate rules:** CC has final say. No echo chambers — push back where CC has codebase context Codex lacks. Both sides cite evidence. The `## Adversarial Review` section stays in the dev plan as a decision record. Novelty > volume: a round 2 that restates round 1 adds noise, not rigor.

### Team Review (📚 COMPREHENSIVE and ⚔️ ADVERSARIAL)

> **Auto-trigger:** After architect validation (and Codex debate, if ⚔️), and **before** presenting the dev plan to the user for approval, run `/teamreview [JIRA-ID]`. Skip for 📄 / 📋.

**What it adds:** Architect validation checks pattern fit. Codex debate challenges architecture at a high level. `/teamreview` stress-tests the final plan from **four orthogonal angles** (architect, security, performance, edge-cases) in parallel, each emitting a structured verdict.

For ⚔️ ADVERSARIAL, ALSO run `/codex-teamreview [JIRA-ID]` **in parallel** with `/teamreview`. This runs four Codex processes with **differentiated perspectives** (implementation-patterns, ecosystem-fit, anti-patterns, alternative-architectures) — prompts chosen specifically to exploit Codex's priors and NOT duplicate Claude's 4 perspectives. This gives cross-model-family orthogonality, which is the real independence signal.

**Flow (📚 COMPREHENSIVE):**

```
Architect validation → /teamreview PROJ-XXXX → aggregate → user approval
```

**Flow (⚔️ ADVERSARIAL):**

```
Codex plan-generate v1 (parallel with CC draft) → Codex cross-review v2 + disagreements → Dual-Plan Synthesis → Architect validation → Codex plan-debate → {/teamreview  ‖  /codex-teamreview}  → within-family aggregate → cross-family aggregate → user approval
```

Both teamreviews run in **plan mode** automatically (spec + dev plan both exist). Outputs:
- `00_CLAUDE/05_reviews/[JIRA-ID]-teamreview-plan.md`
- `00_CODEX/02_reviews/plans/[JIRA-ID]-codex-team-combined.md`

**Aggregation — two stages for ⚔️:**

**Stage 1: Within-family** (same rules as Phase 3 PR review — no percentage threshold)

| Condition | Action |
|-----------|--------|
| Any agent returns **BLOCK** | **HALT.** Revise dev plan, re-run the blocking agent. |
| All agents **SHIP** | Family verdict = SHIP. Record top_risks. |
| Mix **HOLD** / **SHIP** | Family verdict = USER_DECIDE_WITHIN. Surface split. |
| All **HOLD** | Family verdict = HOLD. Fix HIGH findings, re-run. |

Apply this separately to Claude teamreview and Codex teamreview.

**Stage 2: Cross-family** (⚔️ only)

| Claude family | Codex family | Action |
|---------------|--------------|--------|
| SHIP | SHIP | **Strongest signal.** Append deduped top_risks to dev plan. Proceed to user approval. |
| BLOCK | any | HALT (Claude blocks). |
| any | BLOCK | HALT (Codex blocks). |
| HOLD | HOLD | Both families see HIGH findings. Fix, re-run. |
| SHIP | HOLD, or HOLD | SHIP | **GOLD SIGNAL — families disagree.** One family sees something the other structurally misses. Present the disagreeing family's HIGH findings to the user with full context. User decides; do NOT auto-resolve. |

**Rules:**
- **No auto-averaging of disagreements.** Surface splits at both stages.
- **Within-family agreement = one signal, not N independent confirmations.** Four Codex perspectives agreeing is one Codex-family signal. Treat accordingly.
- **Cross-family disagreement is the highest-value output of this workflow.** Do not smooth it over. This is where multi-model review earns its cost.
- If teamreview BLOCKs after Codex debate already SHIPped, the plan is still not ready — teamreview is the stricter gate at this tier.
- Running 4 parallel Codex calls can hit OAuth rate limits. If 429s occur, `codex-teamreview` falls back to sequential — cost is wall-clock, not quality.

### Trigger Phrases for Dev Plan Mode:
- "Create a dev plan for..."
- "Plan the implementation of..."
- "How should we implement [spec]?"
