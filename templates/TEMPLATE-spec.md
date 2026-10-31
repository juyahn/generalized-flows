# Feature: [Feature Name]

**Jira Ticket:** [JIRA-ID](https://<your-org>.atlassian.net/browse/JIRA-ID)

> **Clarification markers — REQUIRED:**
> Wherever this spec would otherwise assume something unstated (schema shape, auth model, error behavior, scale assumption, external contract, rollout plan, failure mode), insert `[NEEDS CLARIFICATION: <specific question>]` **inline** at the point of the assumption.
>
> Specs with unresolved `[NEEDS CLARIFICATION]` markers **cannot advance to Phase 1.5**. Before saving, resolve each marker by asking the user (use `AskUserQuestion`) and replacing the marker with the answer.
>
> Do NOT silently pick a default. Do NOT delete the marker without resolution. A spec with no markers means you verified every assumption was explicitly stated — a spec with markers is better than a spec that guesses.

## Overview

<!-- 1-2 sentence summary of what this feature does and why -->

## Requirements

- [ ] Requirement 1
- [ ] Requirement 2
- [ ] Requirement 3

## Technical Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Example: Data storage | PostgreSQL | Existing infrastructure |

## Scope

### In Scope

-

### Out of Scope

- (reason: future iteration)

### Visibility

What becomes visible to other people, and when. Work that stays in this workspace is listed only as the starting point.

```
Now ~ Phase 2 (workspace only)
  spec · dev plan                         → only me

Phase N (<step>)                          ← first point anyone else can see
  <place: eval project id, dashboard, shared mock, repo/branch, channel>
    <what appears there>

<later step>                              ← needs the user's go-ahead first
  ...

After merge                               ← other people's work is affected from here
  ...
```

## Acceptance Criteria

- [ ] User can...
- [ ] System should...
- [ ] When X happens, Y occurs

## Notes

<!-- Links, references, prior art, open questions -->

---

## Context Checkpoint

> **Purpose:** Capture key context for session continuity after `/compact`

### Files Created

| File | Purpose |
|------|---------|
| `00_CLAUDE/01_specs/[this-file].md` | This spec document |

### Key Decisions Made

<!-- List important decisions from the interview that shaped this spec -->
1.
2.

### Open Questions

<!-- Any unresolved items to address in planning phase -->
-

### Next Phase Entry

**Ready for:** Phase 1.5 (Codebase Exploration) → Phase 2 (Dev Plan)

**Start next session with:**
```
Read the spec at 00_CLAUDE/01_specs/[JIRA-ID]-[feature].md and explore the codebase for similar implementations, then create a dev plan.
```
