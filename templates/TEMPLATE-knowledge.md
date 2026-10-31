# [Feature Name]: Execution Flow

**JIRA:** [JIRA-ID](https://<your-org>.atlassian.net/browse/JIRA-ID)

## Git References

| Repository | Commit Hash | Short Message |
|------------|-------------|---------------|
| repo-name | `abc1234` | commit message |

---

## Overview Architecture

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Component A   │────▶│   Component B   │────▶│   Component C   │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

---

## Phase 1: [Phase Name]

### 1.1 [Step Name] (`file/path.py:line-range`)

<!-- What this code does -->

```python
def example_function():
    # Key logic
    pass
```

### 1.2 [Step Name] (`file/path.py:line-range`)

```python
# Code snippet
```

---

## Phase 2: [Phase Name]

### 2.1 [Step Name] (`file/path.py:line-range`)

```python
# Code snippet
```

---

## Execution Timeline

```
┌────────────────────────────────────────────────────────────────┐
│                        EXECUTION TIMELINE                       │
├────────────────────────────────────────────────────────────────┤
│  T+0     Step 1 description                                    │
│  T+1     Step 2 description                                    │
│  T+N     Step N description                                    │
└────────────────────────────────────────────────────────────────┘
```

---

## Key Files Summary

| File | Purpose |
|------|---------|
| `path/to/file.py` | What this file does |

---

## Configuration Reference

| Parameter | Required | Description |
|-----------|----------|-------------|
| `paramName` | Yes/No | What it controls |

---

## Behavior Notes

<!-- Edge cases, optional features, error handling -->
-

---

## Context Checkpoint

> **Purpose:** Capture key context for session continuity after `/compact`

### Files Created

| File | Purpose |
|------|---------|
| `00_CLAUDE/03_knowledges/[this-file].md` | This knowledge doc |

### Related Documents

| Document | Location |
|----------|----------|
| Spec | `00_CLAUDE/01_specs/[JIRA-ID]-*.md` |
| Dev Plan | `00_CLAUDE/02_dev_plans/[JIRA-ID]_*.md` |
| Debugging (if any) | `00_CLAUDE/04_debugging/[JIRA-ID]-*.md` |

### Gotchas Discovered

<!-- Edge cases or surprises worth adding to LESSONS_INDEX.md -->
-

### Next Phase Entry

**Ready for:** Phase 5 (Update Lessons Index)

**Start next session with:**
```
Read 00_CLAUDE/03_knowledges/[JIRA-ID]-[feature].md and update LESSONS_INDEX.md with any lessons learned.
```
