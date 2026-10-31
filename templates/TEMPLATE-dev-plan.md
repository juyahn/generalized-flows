# [Feature Name] Plan

**JIRA:** [JIRA-ID]
**Date:** [YYYY-MM-DD]
**Goal:** <!-- 1-sentence implementation goal -->

---

## Summary

<!-- Key points of what this plan accomplishes -->
-
-

## Context

<!-- Relationship with existing systems, plans, or dependencies -->

---

## Spec Corrections (Apply to All Tasks)

<!-- Phase 1.5 architect-validate output goes here. List each spec/code mismatch the architect found. If none, write "None — all spec assertions verified against code." Treat the table below as authoritative for downstream tasks; do NOT rely on the spec wording where it conflicts with this table. -->

| Spec said | Reality (file:line) | Impact |
|-----------|---------------------|--------|
| | | |

<!-- Example:
| Route value is `"tools"` | `session.go:671` → `SetRoute("scripts")` | YAML cases use `route: scripts`; extractor filters on `"scripts"` |
| Bot token (`xoxb-`) for posting | `slack.go:567` drops `event.BotID != ""` | Runner MUST use user token (`xoxp-`) |
-->

---

## Gates

> **Architect must tick every gate or write an explicit justification.** A plan that fails a gate without justification cannot advance to user approval. Gates fight over-engineering — the default is "simpler."

| # | Gate | Check | Status / Justification |
|---|------|-------|-------------------------|
| 1 | **Simplicity** | Does this add more than 1 new abstraction layer (new service, new interface type, new framework)? | ☐ Pass / ☐ Justify below |
| 2 | **Anti-speculation** | Any tasks labeled "for future X," "to support eventual Y," or adding fields/flags not required by *this* spec? | ☐ Pass / ☐ Justify below |
| 3 | **Integration-first** | Are API / schema / interface contracts fully defined before implementation tasks begin? | ☐ Pass / ☐ Justify below |
| 4 | **Reuse** | Is there existing code in the target repo that already does 80%+ of this? If yes, is this plan extending it or duplicating it? | ☐ Pass / ☐ Justify below |
| 5 | **Framework directness** | Are we using the framework/library directly, or building a wrapper/abstraction on top? If wrapping, why? | ☐ Pass / ☐ Justify below |

**Justifications (only fill for gates that did not pass):**

- **Gate N:** <specific reason this plan violates the gate and why the violation is necessary>

---

## Architecture Approaches

<!-- Evaluate 2-3 approaches before choosing -->

| Approach | Description | Pros | Cons |
|----------|-------------|------|------|
| **Minimal** | <!-- Smallest change --> | Fast | May add tech debt |
| **Clean** | <!-- Best abstractions --> | Maintainable | More work |
| **Pragmatic** | <!-- Balanced --> | Good default | - |

**Chosen Approach:** <!-- Which approach and why -->

---

## Architecture

```
<!-- ASCII diagram showing component interactions -->
┌─────────────────┐     ┌─────────────────┐
│                 │────▶│                 │
└─────────────────┘     └─────────────────┘
```

<!-- Optional: Status/data flow -->
```
STATE_A → STATE_B → STATE_C
```

---

## Implementation

> **One task = one logical commit.** Size each task to a single coherent diff, with exact file paths and complete code. Split at real seams (schema vs its consumer, behavior vs its test); don't fragment below commit granularity.

### 1. [Task Name]

**File:** `path/to/file.py`

```python
# Complete code (not pseudocode)
```

### 1.1 [Subtask if needed]

**File:** `path/to/file.py`

```python
# Complete code
```

### 2. [Task Name]

**File:** `path/to/file.py`

```python
# Complete code
```

---

## Files to Modify

| Repository | File | Changes |
|------------|------|---------|
| | `path/to/file` | |

---

## Implementation Order

| Task | Description | Commit |
|------|-------------|--------|
| 1 | | `feat: ` |
| 1.1 | | |
| 2 | | `feat: ` |

> **Workflow:** Implement → Review (Claude) → Debug & Verify (User) → Commit

---

## Reality Log

> **Purpose:** Track deviations from plan during implementation. Fill this in as you code.

| Step | Planned | Actual | Why | Impact |
|------|---------|--------|-----|--------|
| | | | | |

<!-- Example:
| 3 | Add field to Model A | Also needed Model B | Foreign key dependency | +1 file |
| 5 | Simple validation | Complex edge cases | API returned unexpected format | +30 min |
-->

**Lessons for Future Planning:**
<!-- What would you do differently next time? -->
-

---

## Verification (User)

<!-- How user will verify this works -->
1.
2.

---

## Context Checkpoint

> **Purpose:** Capture key context for session continuity after `/compact`

### Files Created

| File | Purpose |
|------|---------|
| `00_CLAUDE/01_specs/[JIRA-ID]-*.md` | Requirements spec |
| `00_CLAUDE/02_dev_plans/[this-file].md` | This dev plan |

### Key Decisions Made

<!-- Architecture choice, trade-offs accepted, scope boundaries -->
1. **Approach chosen:** <!-- Minimal/Clean/Pragmatic and why -->
2.

### Dependencies Identified

<!-- External systems, APIs, or code that must exist -->
-

### Next Phase Entry

**Ready for:** Phase 3 (Implementation)

**Branch:** `[JIRA-ID]`

**Start next session with:**
```
Read the dev plan at 00_CLAUDE/02_dev_plans/[JIRA-ID]_[feature]_[date].md and implement starting from Step 1. Verify branch is [JIRA-ID] before coding.
```
