# Workflow: Documentation (Phase 4, 5)

> **When to read:** After completing a feature, creating knowledge docs, updating lessons.

---

## Phase 4: Knowledge Documentation (Post-Implementation)

After completing a complex feature or bugfix, create or **update** knowledge documentation in `00_CLAUDE/03_knowledges/`:

1. **Check for existing knowledge doc** at `00_CLAUDE/03_knowledges/[JIRA-ID]-*.md`
   - If exists → update it with the new findings (e.g., add to Behavior Notes, Gotchas)
   - If not → create at `00_CLAUDE/03_knowledges/[JIRA-ID]-[feature-name].md`
   - Example: `PROJ-123-playbook-lint-flow.md`
2. **Update `03_knowledges/learnings-[project].md`** — this is the ground-truth reference with generous detail (Problem/Solution/Why with full context, root cause analysis, code examples)
3. **Include in knowledge docs** (concise — only broadly reusable patterns):
   - Git references (commit hashes, repos)
   - Overview architecture diagram
   - Phase-by-phase execution flow with code references (file:line)
   - Key files summary table
   - Configuration/parameter reference
   - A few key issues/patterns that apply across features (not duplicating learnings)
4. **Optional: Codex first draft.** For a long knowledge doc or a KR summary, hand the first draft to Codex: `00_CODEX/03_scripts/codex-run.sh -r implement -s read-only -- -o 00_CODEX/02_reviews/docs/[JIRA-ID]-knowledge-draft.md "<dev plan path, commit range, template path, and what the doc must cover>"`. The Codex-side skills `kr-workflow-summary-generator` and `review-html-generator` (`00_CODEX/05_skills/`) cover the KR summary and the code-flow HTML. A draft is never saved as is: rewrite it to the 한국어 작성 원칙, check every `file:line` it cites, then save under `03_knowledges/`.

### Knowledge Doc Template:
```markdown
# [Feature Name]: Detailed Execution Flow

**JIRA:** [JIRA-ID](https://<your-org>.atlassian.net/browse/JIRA-ID)

## Git References
| Repository | Commit Hash | Short Message |
|------------|-------------|---------------|
| repo-name | `hash` | commit message |

---

## Overview Architecture

```
[ASCII diagram]
```

---

## Phase 1: [Phase Name]

### 1.1 [Step Name] (`file/path.py:line-range`)

```python
# Relevant code with explanation
```

---

## Key Files Summary

| File | Purpose |
|------|---------|
| `path/to/file` | What this file does |
```

**Template:** Use `00_CLAUDE/09_templates/TEMPLATE-knowledge.md`

### Trigger Phrases for Knowledge Mode:
- "Document this feature..."
- "Create knowledge doc for..."
- "Write up the execution flow..."

---

## Phase 5: Update Lessons (Two-Tier System)

Lessons are stored in two tiers:

| Tier | File | Purpose |
|------|------|---------|
| **Index** | `LESSONS_INDEX.md` | One-liner summaries with `→ See:` links. Read before Phase 2. |
| **Detail** | `03_knowledges/learnings-[project].md` | Full problem/solution/why writeups. |

### Step-by-step

1. **Read** `LESSONS_INDEX.md` and the relevant `learnings-*.md` file
2. **Always add detailed entry** to `03_knowledges/learnings-[project].md` with date, context, problem/solution/why
3. **Only add to `LESSONS_INDEX.md`** if the lesson is broadly reusable — something that would save time across multiple future features. Keep entries simple one-liners.
   - **DO add:** Architectural patterns, cross-service integration gotchas, general principles
   - **DON'T add:** Narrow/specific fixes (e.g., a particular SQL filter), obvious best practices, one-off debugging findings
   - Sections: `General`, then one per area of the engagement. For an LLM agent engagement, for example: `playbook` (authoring, lint, SOP sync), `eval` (scenarios, judges, regression), `tools` (tool calls, deterministic checks), `runtime` (prompt assembly, routing, RAG), `observability` (metrics, alerts)
4. **Cross-project lessons** go in both `LESSONS_INDEX.md > General` and `learnings-general.md`

### LESSONS_INDEX.md format (one-liner):
```markdown
- **[Short summary]** - [Brief explanation] → See: `[source-file.md]`
```

### learnings-[project].md format (detailed):
```markdown
## [Title]

**Date:** YYYY-MM-DD
**Context:** JIRA-ID

### Problem
[What went wrong / what was missing]

### Solution
[What we did]

### Why
[Root cause / principle]
```

### Where to categorize

| If the lesson is about... | Index section | Detail file |
|---------------------------|---------------|-------------|
| Claude API patterns, LLM prompts | General | `learnings-general.md` |
| Playbook authoring, SOP sync, lint rules | playbook | `learnings-playbook.md` |
| Eval scenarios, judges, regression scoping | eval | `learnings-eval.md` |
| Tool calls, deterministic checks, mocks | tools | `learnings-tools.md` |
| Prompt assembly, routing, RAG | runtime | `learnings-runtime.md` |
| Flow metrics, alerts, drilldown | observability | `learnings-observability.md` |

### Trigger Phrases:
- "Update lessons index"
- "Add this to lessons learned"
- "What did we learn from this?"
