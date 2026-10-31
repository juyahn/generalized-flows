# Bug: [Short Description]

**JIRA:** [JIRA-ID]
**Date:** [YYYY-MM-DD]
**Status:** Resolved / In Progress

---

## Problem

<!-- Error message, symptoms, or unexpected behavior -->

```
<!-- Paste error message or stack trace -->
```

**Environment:** <!-- prod/stg, region, service -->

---

## Reproduction Steps

1.
2.
3.

---

## Investigation

### Hypothesis 1: [What you thought was wrong]

<!-- What you checked, what you found -->

**Result:** Confirmed / Ruled out

### Hypothesis 2: [Next theory]

**Result:** Confirmed / Ruled out

---

## Root Cause

<!-- What was actually wrong -->

**Location:** `file/path.py:line`

```python
# The problematic code
```

---

## Fix

**Files Changed:**

| File | Change |
|------|--------|
| `path/to/file` | Description |

```python
# The fix
```

---

## Verification

<!-- How you confirmed the fix works -->

-

---

## Prevention

**Question:** How do we prevent this class of bug from happening again?

### Guidelines to Add

<!-- Check all that apply and fill in details -->

- [ ] **CLAUDE.md** - General workflow/coding guideline
  - Add:

- [ ] **00_CLAUDE/10_projects/[project].md** - Project-specific pattern
  - Project:
  - Add:

- [ ] **Template update** - Add checklist/section to a template
  - Template:
  - Add:

- [ ] **No guideline needed** - One-off issue, just document here

### Pattern to Watch For

<!-- Describe the anti-pattern so it can be caught in future code reviews -->

```
<!-- Example of what NOT to do -->
```

**Instead, do:**

```
<!-- Example of the correct pattern -->
```

---

## Context Checkpoint

> **Purpose:** Capture key context for session continuity after `/compact`

### Files Created/Modified

| File | Purpose |
|------|---------|
| `00_CLAUDE/04_debugging/[this-file].md` | This debugging doc |
| <!-- code files --> | <!-- fix description --> |

### Root Cause Summary

<!-- One-liner for quick recall -->


### Lesson Learned

<!-- Key takeaway to add to LESSONS_INDEX.md -->
-

### Next Phase Entry

**Ready for:** Phase 5 (Update Lessons Index) or return to Phase 3 (Implementation)

**Start next session with:**
```
Read 00_CLAUDE/04_debugging/[JIRA-ID]-[description].md and update LESSONS_INDEX.md with the lesson learned.
```
