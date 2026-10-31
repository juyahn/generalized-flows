# Workflow: Delivery (Phase D0-D5)

> **When to read:** any ticket, in any tracker project, whose correctness is judged by how a deployed LLM agent behaves rather than by a unit test: a customer-reported issue, a wrong bot reply, a model or system-template switch on a customer agent, any change to the agent's definition (playbooks, SOP, KB, prompts) or to a tool layer judged by the conversation it produces.

This file holds the rules and gates. Incidents that taught them live in `LESSONS_INDEX.md`, `03_knowledges/learnings-*.md` and the diagnosis notes in `04_debugging/`; a rule here points there when the evidence matters.

| File | Covers |
|---|---|
| this file | track choice, entry point, D0 to D3.5, evidence tier |
| `workflow-04-delivery-verify.md` | D4: rungs, which vehicle, how to read a run, optional Codex first read of failed runs, runs per case |
| `workflow-04-delivery-ship.md` | D5: test assets, PR, record |
| `03_knowledges/playbook-fix-patterns.md` | recommended formats for playbook edits and the antipattern review criteria D3 and D3.5 use. Write one per engagement |

## Vocabulary

These names are generic. Map each one to the engagement's real tool in `.claude/team.json` or the engagement's `CLAUDE.md` before the first ticket.

| Name here | What it is | Example of what it maps to |
|---|---|---|
| **agent config repo** | where the agent's prompts, playbooks, SOP and KB are versioned | a prompt repo per customer directory |
| **playbook** | an instruction document the agent follows for one use case | a markdown procedure with numbered steps and `- If` branches |
| **routing step** | the call that picks which playbook handles a turn | a selector prompt reading a routing file |
| **eval platform** | the service that runs the agent against test conversations | an internal conversation-test service, or a framework run locally |
| **sim profile** | a simulated user that decides what to say | an LLM user persona with a goal and success criteria |
| **scenario case** | fixed user turns, pinned tool results, typed rules | a YAML case with `says`, `must_not_say`, `tool_absent` rules |
| **staged run** | the deployed agent with your edit applied as an override | a forked agent version |
| **trace store** | where production conversations, tool calls and prompts are logged | a data warehouse table, an LLM observability tool |
| **team procedure skill** | the team's own end-to-end skill for this loop, if one exists | an issue-fix skill in the agent config repo |

---

## Which track

How correctness is judged picks the track. The tracker project does not; `.claude/team.json` says where the ticket sits and which key the PR cites.

| Correctness is judged by | Track |
|---|---|
| Conversation: sim profile, scenario case, replay, production monitoring | **D**, this workflow |
| Code tests | **T**, `workflow-01` / `-02` / `-03` |
| Both | **split**: the code part on T, each customer's rollout and verification on D |

| Boundary case | Where it goes |
|---|---|
| customer ticket whose fix lands in tool-layer code | D, and D3 borrows `code-reviewer` from T |
| customer ticket whose root cause is in the shared runtime | re-file to the runtime owner and hand it to T. It ships on a runtime release |
| ticket that switches one customer agent's model or template, or edits its playbooks | D. The ticket stays in its project |
| ticket that edits playbooks across customers | D, one target per customer at D0 |
| change to a shared template, model or default that moves bot behaviour without touching a playbook | split. Code and runtime release on T; each affected customer's verification on D, owed before the change reaches that customer, even when the change has no ticket |

---

## Entry point

If the team has a procedure skill for this loop, it owns the D0 to D4 procedure end to end and this file does not restate it; it adds routing, gates, evidence tiers and artifacts. Check which skills the session can see with `ls <dir>/.claude/skills` rather than a remembered list, because skill visibility depends on where the session launched.

| Need | Where it comes from |
|---|---|
| a production conversation's text, tool calls, system prompt | the trace store, not a re-run. Fetch it once to a file and read the file |
| runtime logs | the engagement's log store |
| the eval platform's operations | its API or the tools that wrap it. Discover the API rather than guessing endpoints |
| whether the local stack is up | the stack's own status command. Read the port rather than assuming it |

**Writing to a repo another team owns needs an explicit rule.** If the owning team's `CLAUDE.md` forbids shell writes or requires a specific tool, follow it as written. Where nothing is written, ask before shelling into a checkout you did not create. A permission given for one run is not a standing exemption.

---

## The loop

```
D0 Intake ──▶ D1 Baseline ──▶ D2 Diagnose ──▶ D3 Fix ──▶ D3.5 Read ──▶ D4 Verify ──▶ D5 Ship
 targets       STOP gate       name the layer   two forms   edited region  cheapest rung   assets, PR
 expected                           ▲                                          │
                                    └────────────── not fixed, cap 3 ──────────┘
```

**Every gate from D1 to D4 runs once per target.** A target is one customer directory and locale, or one variant copy that ships separately.

### D0 Intake and routing

- **Enumerate the targets.** A ticket often covers two locales or a variant copy. Each target gets its own baseline, tier and verification.
- Resolve each customer directory and eval project by documented paths, never from memory. Directory names can differ from the customer slug, and a project can be dead.
- **SOP gate**: grep the customer's SOP and record `SOP checked: <use case §> · <branch>` or `no matching use case`. Record any customer the engagement exempts, with where the exemption is written.
- **Write down the expected behaviour**, verbatim from the ticket where it gives one; the reporter's wording is often in the linked chat thread. If the ticket settles neither the copy nor the policy, ask the user.
- **A control's expected behaviour comes only from the ticket, the SOP or the customer's approved copy**, never from the playbook you are about to change. An input the ticket leaves open, or settles as "ask", is a confirmation item, not a control.
- **A model or template switch expects equivalence**: the same behaviour as the current deployment on the sets the ticket declares, with each defect the switch should fix listed separately.
- **When sources disagree** (approved copy, a specific tracker row, a tracker summary, the ticket's framing), the provisional order is that one, but the disagreement is itself a confirmation item and both readings go into the PR.
- Guess the evidence tier (below). D3 sets the real one.
- **Report the plan in three parts before D1**, in the format of `workflow-01-planning.md` Phase 1 step 8. The parts are what this work will do, what it will not do, and what becomes visible to other people and when. D-track leaves traces early, so the third part matters more here. The first eval run puts a job in the customer's eval project and conversations in its dashboard, and staged runs land in the same history. Mock rows may go into a shared environment. Then come the branch, the PR, the customer channel, the merge and the deploy. Mark the first point anyone else can see, the point from which other people or the customer are affected (merge, then deploy), and every step that waits for the user's go-ahead. If a diagnosis note is written, the third part goes into its `## 보이게 되는 것과 그 시점`.

**Leave D0 when** every target is listed with its project id and directory, plus the SOP line and the expected behaviour, and the three-part report has been given.

### D1 Baseline, the STOP gate

**No edit until the baseline is written down**, per target, as one of five outcomes. Writing it down is the gate, not getting a good number, and "did not reproduce" is not "fixed".

| Outcome | What it means | What follows |
|---|---|---|
| **Reproduced** `m/n` | the defect shows in the replay | D2 |
| **Already fixed** | it does not reproduce and something corroborates it, such as a later commit | report "재현 불가, 배포본에서 해소" with the corroboration; fix only what still fails |
| **Inconclusive** | the replay cannot carry the defect (an image turn, unstable mock state, an expired customer state) | say what it could not carry; diagnosis may proceed, verification stays owed |
| **Blocked** | no valid run: dead project, stack down, infrastructure failures | record the blocker; D4 stays unsatisfied and nothing ships on it |
| **Captured** | no defect; the ticket asks for new behaviour or copy | capture the unmodified deployment as the before; D4 judges against the expected behaviour. Say "capture" in the PR |

- **A model or template switch is Captured, run as an equivalence check** against observed production behaviour. A defect it should fix is recorded as Reproduced on the current model.
- **Record provenance**: source, project, agent version, when, and production observation versus staged replay. The PR's before column rests on it.
- **Check repo vs deployed drift here.** A staged run built from repo wording that differs from the deployed wording tests something that was never deployed.
- Verify the instrument before reading numbers. All-`failed` with an infrastructure error is the project, not the bot.
- **Preserve the unchanged candidate** (base commit or unedited files). Extra baseline runs D3 may need come from it, labelled with when they ran.
- **With a production conversation, the cheapest reproduction is a scenario case**: verbatim user turns, that conversation's tool responses pinned, `repeats=5`. A baseline measured as a rate over varied phrasings stays on sim profiles.

**Leave D1 when** every target has an outcome with its provenance.

### D2 Diagnose, classify the layer

Name the layer and the quote or tool output that proves it. `workflow-04-delivery-verify.md` says what evidence each layer owes at D4.

Layers: retrieval / KB, routing step, main-LLM branch, prompt executed but still wrong, already fixed in the deployment (a D1 outcome, not a fix), not a bug but a behaviour change, model / system template.

- **A ticket can be more than one layer.** Name the primary hypothesis and link the others; each subchange gets its own verification path.
- **A model or template switch names the model layer for the switch**, and each difference D4 finds gets a layer of its own.
- **Separate the playbook text from the layers around it**: routing, tool-layer semantics, a predefined response that bypasses the LLM, runtime re-rendering, global prompt examples, the test harness. Playbook prose may mitigate an outside-layer defect only by stating what the runtime can observe, on the path that actually fails, with the mitigation record.

**Leave D2 when** a layer is named with its proof and any second layer is linked rather than dropped.

### D3 Fix, with the pre-flight checks

Edit the smallest thing the layer allows. A tool-layer fix is code, so run `code-reviewer` before D4.

**Write two candidate forms before the first edit** and pick one with its antipattern cost stated, recorded in the diagnosis note. The aim is to add no antipattern on the lines you touch; you review against `playbook-fix-patterns.md` and list anything you did add.

Before editing, establish:

1. **The file's own convention, then its sibling copies.** Grep how the file already does the same job and follow the majority (the minority needs a reason in the commit body). Then check the copies that ship separately: variant copies and the other locale.
   - **Recount after the base moves.** A rebase can add or recut copies; re-run the sweep and re-check every exclusion reason.
   - **Changing a line breaks notes that quote it.** Grep the old wording across the whole customer tree and check what each hit points at rather than what it says. Repair a genuinely broken note as a fact naming what the model can see (a tool-result field or message, not an error code that only reaches the template context), with no ban in it and no new quote or anchor.
   - **A line you repair as collateral is a line you authored.** It gets the D3.5 questions too.
2. **The turn contract for each playbook and model you touch.** The unit is not the customer: one edit can land under two contracts when one playbook runs on a different model with its own template. Find the live template the runtime actually resolves, not the packaged default. Three rules do most of the work:
   - A numbered item is one turn's action. Exclusive branches written as sibling numbers read as sequential turns; make them bullets.
   - Indentation is parent and child: the parent `- If` must hold for its children to run.
   - Text and a tool call in one numbered item are one turn; two sibling numbers are two turns, which is how a say-then-call pair gets truncated. Check whether the locale's template limits actionable items per turn.
3. **The deployed wording, before staging.** A staged run applies edits to the deployed body; check that the text you replace occurs there exactly once, with a dry run if the platform offers one.
4. **Routing layer only: replay the original routing call before editing the routing file.** If it already returns the right playbook, the misroute was non-deterministic or already fixed and no routing text is added.
5. **Model or template switch only: put the template under the new model's key before pinning the model.** Find the order in which the runtime resolves a template per model. A template under the old key does not follow the switch: the turn falls to a default or errors. Template first, then model; remove in reverse and keep the old key as the rollback.

**Add no counterexample that settles a boundary the ticket or a decision record does not settle.** Ask instead. A counterexample that preserves existing policy is fine with its source cited; a paraphrase test does not make an unsourced one acceptable.

**Set the evidence tier now, per target, from what the diff touches**, not from the ticket. Recompute it on every substantive revision, reversions included, and keep the failure history. Every measurement the report cites (render diffs, character counts, drift probes) goes stale the same way; re-measure after the last commit.

**Run whatever lint and antipattern scanners the engagement has**, and say which instrument you ran. Require no new issue on a line you touched. Where a rule has no mechanical check, D3.5 is the check.

**Leave D3 when** the checks for this layer are done, the tier is recorded, the scanners have been run and read, and every antipattern the edit adds is listed for the PR's 판단 요청.

### D3.5 Read the edited region

D4 walks the paths you wrote a test for; this step is for the paths you did not think of. A state no branch matches has no scenario driving it, so no run reaches it, and scanner silence does not cover it either. Read what you edited and the chain that encloses it, per target:

| Question | The failure it catches |
|---|---|
| For each `- If` chain you touched, what happens when no sibling matches? | nothing is written, so the model improvises |
| Does every branch you created end in an utterance and a stop, a jump, or a handoff? | a dead end where the model continues on its own |
| Can one message satisfy two sibling branches? | routing depends on the model. Make the chain exclusive |
| Do tool-result branches cover more than success and failure? | an empty or unnamed result shape falls through |
| Does the enclosing section say how it is entered and left? | reachable with no exit, or unreachable so the edit is dead |
| Is any constraint you added an open negative? | `do NOT X` leaves the alternative to the model. Use a closed positive unless the prohibition is the point |
| Is the target of every pointer you wrote visible, unique and valid where it is used? | a target compiled out by another template branch, or a quote that no longer matches. Render the states involved; a count proves presence only |
| Does the test set rest on an example the prompt quotes, or did you add an unsourced counterexample? | the rule is tested only against its own sentence, or your reading of a boundary is written into the agent |
| If you routed a new path into an existing section, is every sentence there true on the new path, and did existing paths keep what they need? | a reminder or promise that is false on the new path |

Every hole is written down one of three ways: fixed back in D3, promoted to a D4 case, or accepted with the reason it is out of scope. **A hole is a hypothesis about cost, not a result**: state what is structurally missing and what you predict it costs separately, and let D4 settle the cost.

**Leave D3.5 when** each question has an answer for every target and every hole has a disposition.

### D4 Verify

**Spend the cheapest evidence that can still prove you wrong**: replay, then local run, then staged run, then regression, with nothing staged or booked until the rung below is green. Rungs, vehicles, reading discipline and runs per case: `workflow-04-delivery-verify.md`.

**Leave D4 when** every target's applicable rungs are green per case, each N/A rung says why, and the pass was read off raw turns rather than a verdict. The close stays provisional until the PR's automatic inline review has been read.

### D5 Ship and record

Settle the test assets first, then open the PR, since the PR body lists what was committed. Detail: `workflow-04-delivery-ship.md`.

---

## Optional: hand a step to Codex

None of these is required. Use one when the step is self-contained and its output is a file you can read back instead of a stream of tool results in this session. Run it read-only, and have it write only under a reviews directory such as `00_CODEX/02_reviews/behavior/`. Codex never writes to the agent config repo. Give it the files it needs as paths, never your own conclusion, or it agrees with you.

| Step | What Codex does |
|---|---|
| D2 | An independent layer diagnosis from the conversation and the ticket's Expected, without your hypothesis. Save the conversation from the trace store to a file first. Where its layer differs from yours, the proof decides, not either side |
| D2 | A self-contained investigation that would take ten or more calls here and whose next decision needs only the conclusion, such as every caller of a tool across repos or a count over local logs. Local files only: a query needing network or credentials stays in this session |
| D3 | A review of the playbook diff against `playbook-fix-patterns.md` and the D3.5 questions: antipatterns added on touched lines, chains with no fallthrough, pointers that no longer resolve. It does not replace your own D3.5 read |
| D4 | Held-out paraphrases and opposite-side cases for each new condition, written from the ticket and Expected only so they do not echo the prompt |
| D4 | First read of a failed scenario batch, see `workflow-04-delivery-verify.md` |

The D2 prompt, as the shape the others follow:

```
A customer-support bot behaved wrongly. Conversation: <path>. Expected, from the ticket: <quote>.
The agent is defined under <agent config repo>/<customer>/; use read-only git only.
Name the layer that caused it (retrieval/KB, routing, main-LLM branch, prompt executed but still wrong,
already fixed, behaviour change, model/template) with the exact quote or tool output that proves it,
and the strongest alternative layer with what would rule it out. Do not propose a fix.
```

Read the file it writes skeptically: a cited line that does not exist, or a file it claims to have read but could not reach, voids the finding.

---

## Evidence tier

In D the cost driver is how many runs the evidence takes, not planning depth. Guess the tier at D0 (it decides how many baseline runs D1 buys); set it at D3 from what the diff actually touches.

| Tier | What changes | D1 baseline | D4 slow loop | D4 regression |
|---|---|---|---|---|
| 🗣 **Copy** | customer-visible wording only, same branch structure | a production log quote; an eval run only when the log has none, and the PR says which | one staged run per branch, every branch correct | none |
| 🔀 **Branch** | a condition, a branch, or tool-call order | n ≥ 5, rate recorded | core cases plus both controls, each passing on its own | diff-scoped regression. A scenario case merged with the PR satisfies it for that case |
| 🧭 **Flow** | playbook boundaries, routing, or SOP | 30 | staged run plus routing replay, per case | full golden set, and a release-level run at release |

**A model or template switch is 🧭 by default.** It may be scoped down when the ticket declares the sets, both arms run the same sets, and the ticket and PR say what was left out. A scoped run is not reported as a full-golden pass.

When D3 raises the tier, the extra baseline runs come from the candidate D1 preserved, and the PR says when they ran.

Traps that cost a run are indexed in `LESSONS_INDEX.md`. Read it before D3.
