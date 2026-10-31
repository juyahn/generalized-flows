# Workflow: Delivery, D5 Ship and record

> **When to read:** at D5 of `workflow-04-delivery.md`, after D4 is green. Two steps, in this order: settle the test assets, then open the PR, because the PR body lists what was committed. Rules only; the evidence is in `03_knowledges/learnings-eval.md`.

## Test assets, before the PR

Every sim profile and scenario case D1 to D4 produced leaves in one of three states: not committed, committed, or golden. Run this per target. A team procedure skill or PR skill may not ask this question, so nothing downstream catches a skipped step.

| | Sim profile | Scenario case |
|---|---|---|
| States | scratch (gitignored) → shared (committed) → golden | temporary (eval platform only) → committed under the customer's scenario directory |
| Committing | puts the file in the repo; golden is a separate, explicit promotion | is the promotion; there is no middle state |

**List first**, per target: every new profile, every temporary case, and every existing profile or case whose expectation the diff changed.

**Profiles**

```
sim profile
 │
 ├─ Does the PR body cite this run? (a before/after link, a results row, a control)
 │    ├─ no  → do not commit; it stays in scratch (reverted designs, exploratory runs)
 │    └─ yes → commit to shared/<ticket>-<slug>/
 │
 ├─ Is the criterion re-scoped to the fix direction?
 │    └─ no  → fix it first
 │
 └─ Golden only if all four hold
      · a production conversation id      at least one source conversation recorded
      · a recurrence the customer sees    wrong coupon, handoff or amount; tone-only stays in shared/
      · must_not is filled
      · a case cannot carry the claim     varying user phrasing is under test, or the flow is voice/realtime
      yes → one representative per fix, by the platform's promote command, never a hand move
      no  → stays in shared/
```

A cited profile is committed so a reviewer can re-run it; an uncited one is not, because every committed profile is one more that later PRs may pull into their scan. A case comes before a golden profile because it judges the same claim more steadily.

**Cases**

```
temporary case (eval platform only)
 │
 ├─ Does it guard a branch or a flow, not only wording?
 │    └─ wording only (🗣 Copy) → do not merge; it would break on every copy change
 │
 ├─ If it is a regression case, does it name the production conversation it broke in?
 │    └─ no → it is not a regression case
 │
 └─ Rules re-scoped to the fix, run at repeats=5: does every rule hold?
      ├─ every run → merge; that is the golden
      ├─ some runs → do not merge; report the spread as a finding
      └─ no run    → the rule or the mocks are wrong; fix and re-run
```

Author at `repeats=1`, validate once at `repeats=5`. Each repeat is a real conversation.

**A routing edit inverts the last question.** If staged runs never see the new routing file, a "which playbook took turn 0" rule fails before the deploy for a correct change. Commit such a case as a post-deploy gate (RED before, GREEN after) and say so in the PR. Every other rule still has to hold on the staged run.

**Existing assets the diff made stale**

```
playbooks the diff touched, wording it changed
 │
 ├─ cases     run the engagement's impact check over <base>..HEAD
 │            <base> is the PR's real base (the default branch, or the release branch it stacks on)
 ├─ profiles  grep the old wording across the customer's profile directory
 │
 └─ a hit that changed the way the fix intended → update its expectation in this PR
```

**By tier**

| Tier | Profiles committed | Scenario case | Golden profile |
|---|---|---|---|
| 🗣 Copy | cited only | none | none |
| 🔀 Branch | cited only | recommended; if none, one line saying why | only when a case cannot carry the claim |
| 🧭 Flow | cited only | one representative per branch | only those meeting all four conditions |

**Format checks on anything committed**: follow the agent config repo's naming and tagging rules for profiles and cases. At minimum, a profile lists every playbook the run passes through and has both a must-reach and a must-not expectation; a case uses a fresh user id, starts with a routing rule, and gives every rule a note.

**What the PR carries.** List the committed assets in the PR body in the repo's format. A 🔀 ticket with no case says why and that the regression cell is empty for this PR. Uncommitted assets go in the diagnosis note if there is one.

## PR and record

- **PR**: use the agent config repo's PR format; the repo's own format document wins on conflict. Its body carries 이슈 · 원인 · 기대 결과 · before-fix link · after-fix link, so that five-line report is the PR itself. The before column comes from the production log; if it came from a staged run, say so. An automatic inline review, if the repo has one, is a D4 close condition.
- **Ask for the review.** A PR nobody was asked to look at is not shipped. The message is a pointer to the PR in the channel's own style, not a second report.
- **Not every ticket ends in a PR.** A KB hand-off, a "재현 불가" close and an exhausted D4 still owe the five-line report.
- **Whatever carries the report carries the minimum record**: the D1 outcome and provenance, the tier and what set it, per-case results for the applicable rungs, why each N/A rung was N/A, any antipattern added with its record, and an owner for anything deferred (a KB upload, a release-time check).
- **Diagnosis note**: `00_CLAUDE/04_debugging/[JIRA-ID]-[slug].md` when the work took over 30 minutes or taught something new. It holds only what the report cannot: dead ends, drift findings, per-run raw evidence, unresolved items. Template: `09_templates/TEMPLATE-behavior-fix.md`.
- **Knowledge doc**, when the fix required drawing the flow rather than editing a line: `00_CLAUDE/03_knowledges/[JIRA-ID]-[flow-name].md` from `09_templates/TEMPLATE-behavior-knowledge.md` (turn map, what changes the path, which layer owns each decision, model contracts). The trigger is "I had to map this flow", not ticket size.
- **Lessons**: `workflow-03-documentation.md` Phase 5. Anything true only while this flow exists goes in the knowledge doc; anything that applies to the next ticket goes in `learnings-*.md`, with a line in `LESSONS_INDEX.md`.
- `12_verification/` is T-track only; in D the eval run links are the verification.
