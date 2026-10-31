# Workflow: Delivery, D4 Verify

> **When to read:** at D4 of `workflow-04-delivery.md`, before the first run. Rules only; the incidents behind them are in `LESSONS_INDEX.md` and `03_knowledges/learnings-eval.md`. Generic names (eval platform, sim profile, scenario case, staged run) are defined in the Vocabulary table of `workflow-04-delivery.md`.

**Spend the cheapest evidence that can still prove you wrong.** Nothing deploys, stages or books an eval job until the rung below it is green.

| Rung | What it exercises | Cost |
|---|---|---|
| **1. Replay** | one historical LLM call with your edit substituted into its prompt | seconds to minutes |
| **2. Local agent run** | a whole conversation against a local agent with your local playbook | minutes, local |
| **3. Staged run** | the deployed agent with your edit applied as an override, driven by a sim profile or a scenario case | costs runs, per the evidence tier |
| **4. Regression** | the regression suite scoped to the diff or the customer's golden set | one bulk job, per the evidence tier |

If the engagement lacks a rung, say so once in `team.json` and in every PR it affects. A missing rung is not a green rung.

## What each layer owes

| Layer | Rung 1 proves | Rungs 2 to 4 |
|---|---|---|
| Routing step | the edited routing file routes correctly, replayed on the historical routing call | **Often N/A.** If staged runs always read the deployed routing file, rung 1 is the only pre-deploy evidence. Say so in the PR |
| KB | the model behaves once the fact exists, with the snippet injected into the captured prompt | wait on the upload; until then the after-fix run is a recorded waiting state, not a pass |
| Main-LLM branch, playbook | the edited prompt beats the unmodified one at the same count, rendered against historical runtime values | all apply |
| Prompt executed, still wrong / behaviour change | as above; for a behaviour change the bar is the expected behaviour | all apply |
| Tool-layer code | its own tests, plus `code-reviewer` at D3 | rung 3 is integration evidence, not proof of the boundary case |
| Model / system template | captured requests resent with only the model changed, or re-rendered under the new template | rung 2 only if the local agent can call the model. Rungs 3 and 4 apply; the staged run needs the template under the new key first (D3 item 5) |

Three rung-1 rewrites that do not move the number mean the layer is wrong. Go back to D2.

**Check that a suite actually ran something.** A suite that reports `0/0` and exits 0 is not a check. Read the denominator.

## Which vehicle carries rungs 3 and 4

A **sim profile** is a simulated user that decides what to say. A **scenario case** fixes the user turns verbatim, pins the tool results, and judges by typed rules (which playbook took the turn, `says`, `must_not_say`, `tool_absent`, `handoff_absent`) with `repeats` and `pass_threshold` in the file. The agent runs free in both.

- **Take a case** when you can write the user turns down, the tools are mockable, and the claim is which playbook took the turn, what the reply had to say or must not say, a tool that must not fire, or a handoff that must not execute.
- **Take a profile** when the unknown is the user's own behaviour: any rate over phrasings, any exploratory run, and every voice or realtime flow.
- **Check whether either covers a routing-file edit.** If both only patch the playbook, a "which playbook took turn 0" rule makes routing checkable after the deploy, not before.
- **D3 does not author a case**; it re-scopes the D1 case's rules to the fix direction, as a profile's criterion is re-scoped.

## Reading a run

- **Confirm the tested candidate is the candidate** before reading any score. A staged version id plus one literal from the edit proves an override was served, not that the whole change was; bind the evidence to every branch and locale the diff touches.
- **Read the user turns before the bot turns.** A profile whose triggers misfire produces a normal conversation that never reaches the situation under test. Confirm the premise happened, then score the reply.
- **A run that does not reach the branch is not noise until proven.** When several cases miss the same way, diff the section's condition lines against your base (`git log <base>..origin/<base-branch> -- <playbook>`): sections that are first-match-wins can gain a new condition above yours that takes the traffic. Attribute it with a deployed control, the same profiles in one job with no override.
- **Lock entry routing inside the asset**: make a case's first rule "which playbook took the turn", so a case that cannot reach its branch fails and names the playbook it got. A profile has no such guard.
- **When rung 3 varies, rule out causes outside the edit first.** Routing is often the largest cause, since a staged run may never change it. A ticket that also touches routing runs the rung 1 routing replay before rung 3, because a rung 3 transcript is unreadable until you know which playbook produced it. Branch choice inside one playbook also varies.
- **Do not trust the verdict.** Judge from the raw function calls and the verbatim reply; read a rung 1 judge's reasoning, not its score. Unreadable raw evidence, PII masking included, is inconclusive.
- **Score on content, not formatting.** Check that the string you count is something the spec requires rather than a form the model tends to use; a wrong criterion also corrupts the D1 baseline.
- **Re-scope the criterion to the fix direction** before committing an asset. A before-fix criterion fails forever.
- **Acceptance is per case**: each named case, branch, locale and variant passes on its own. An aggregate `m/n` hides a failing locale.
- **Two controls, not one**: a true negative (an input the new branch must not open on) and an adjacent-flow no-regression (an existing site the edit reached).
- **A case whose utterance the prompt quotes is not evidence for the rule.** Each new condition needs a held-out paraphrase absent from the prompt and a case on the opposite side of the boundary.
- **Judge a control against the deployment**, run in the same time range, comparing each turn's playbook, tools and handoff, not only the reply. A wording-only rule list passes while the staged run drops a call the deployment makes.
- **Three failed end-to-end cycles is the cap**, counted across reclassification and separate from rung 1 rewrites. Hitting it is an *exhausted* outcome: report what moved and what did not. It does not satisfy D4.

## Failed scenario runs: optional Codex first read

When a rung 3 or 4 batch comes back with more failures than you would read one by one, you can hand the first read to Codex before opening any run. It is optional; for one or two failed runs, read them directly. Reading is what costs this session, because every run and conversation pulled into the main window is re-read on each later turn. Codex reads one case per process, in parallel, and you read one table.

```
bundle   per failing case: the case (rules, pinned tool results, earlier reviewer feedback),
         that batch's failed runs, each conversation with every tool call's actual response,
         and the agent config history between the last pass and the failure
run      one Codex process per case, read-only
summary  one table: case · label · category · evidence · confidence · 직접 읽기
```

- **Labels**: `regression`, `stale_case`, `tool_bug` or `needs_human`, with category, evidence and confidence.
- **Read every case marked in the `직접 읽기` column yourself**: `regression`, `needs_human`, or confidence below high. A `regression` drives an agent fix, so never act on it from the table alone. Act on `stale_case` and `tool_bug` at high confidence from the table: rewrite the case, or rerun.
- **The table is not D4 evidence.** Codex triages failures; a target still leaves D4 only on passes read off raw turns. Its label is an input to your next action, not a review registered on the eval platform.
- **A suite run on a staged PR version is judged against the wrong prompt** if the bundle carries the base branch's prompts instead of the PR's. Recheck every `regression` and `needs_human` against the PR branch before acting on it.
- **Passed runs can still be wrong.** Rules check a phrase, a tool or a handoff, so a run passes while the conversation drifts. For the final suite of a new or narrow-rule case, sample passed runs from the rarest conversation paths and have them judged against the case's intent. A `false_pass` is a rule gap to fix in the case and a finding to read yourself, not D4 evidence either way.

## Runs per case

- A case that decides a design choice and is only observable at runtime: `n ≥ 6` (rung 1 already demands `≥ 5/6`, so one run a rung higher makes the evidence thinner).
- An incidental confirmation: `n = 1`.
- A case a deterministic artifact settles (a byte-identical render, a diff): `n = 0`, and the artifact is the evidence.
- A result that flips between runs is measured for variance before the edit changes; the same profile listed n times in one job costs one job.

## Leaving D4

**Leave D4 when** every target's applicable rungs are green per case, each N/A rung says why, and the pass was read off raw turns. A target left unverified is not covered by another target's green.

**The close is provisional until the PR's automatic inline review has been read.** An inline finding re-enters D4; scan bots passing is irrelevant, only inline comments count, because they read the file rather than a transcript.
