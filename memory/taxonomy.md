# Memory Taxonomy — Structured Persistent Memory for AI Assistants

## Concept

AI memory should be structured, not a dumping ground. Four memory types with different lifecycles, different triggers for saving, and different uses.

The taxonomy prevents two failure modes:
1. **Saving everything** — memory becomes noise, nothing is findable
2. **Saving nothing** — every session starts from zero

## The Four Types

### 1. User Memories
**What:** Information about the person you're working with — role, expertise, preferences, perspective.

**When to save:** When you learn details about the user's background, domain knowledge, or how they prefer to work.

**How to use:** Tailor your responses. A senior engineer gets different explanations than a student. Someone new to React but experienced in Go gets frontend concepts framed as backend analogues.

**Examples:**
- "Data scientist focused on observability and logging"
- "Deep Go expertise, first time touching React — frame frontend explanations in backend terms"
- "Prefers direct communication, dislikes option-listing"

### 2. Feedback Memories
**What:** Guidance on how to approach work — corrections AND confirmations.

**When to save:** When the user corrects you ("don't do that") OR confirms a non-obvious approach ("yes, exactly"). Corrections are obvious. Confirmations are quiet — watch for them.

**How to use:** Don't repeat corrected behavior. Do repeat confirmed behavior.

**Structure:**
```
[The rule]
**Why:** [The reason — often a past incident or strong preference]
**How to apply:** [When/where this guidance applies]
```

**Examples:**
- "Integration tests must hit a real database. **Why:** Mock/prod divergence masked a broken migration last quarter. **How to apply:** Any test touching data persistence."
- "For refactors in this area, one bundled PR is preferred. **Why:** Confirmed after user accepted this approach — splitting would be churn. **How to apply:** When refactoring tightly coupled modules."

### 3. Project Memories
**What:** Ongoing work context that isn't derivable from code or git history.

**When to save:** When you learn who is doing what, why, or by when. Convert relative dates to absolute dates ("Thursday" -> "2026-03-05").

**Structure:**
```
[The fact or decision]
**Why:** [The motivation — constraint, deadline, stakeholder ask]
**How to apply:** [How this should shape your suggestions]
```

**Examples:**
- "Merge freeze begins 2026-03-05 for mobile release cut. **Why:** Mobile team cutting release branch. **How to apply:** Flag any non-critical PR work after that date."
- "Auth middleware rewrite driven by legal compliance, not tech debt. **Why:** Legal flagged session token storage. **How to apply:** Scope decisions should favor compliance over ergonomics."

### 4. Reference Memories
**What:** Pointers to where information lives in external systems.

**When to save:** When you learn about resources and their purpose in external systems.

**Examples:**
- "Pipeline bugs tracked in Linear project INGEST"
- "grafana.internal/d/api-latency is the oncall latency dashboard — check when editing request-path code"

## What NOT to Save

- **Code patterns, architecture, file paths** — derivable from reading the codebase
- **Git history** — `git log` / `git blame` are authoritative
- **Debugging solutions** — the fix is in the code, the commit message has context
- **Anything in CLAUDE.md** — already loaded every session
- **Ephemeral task details** — use session continuity or task tracking instead

These exclusions apply even when explicitly asked to save. If someone asks to save a PR list, ask what was surprising or non-obvious about it — that's the part worth keeping.

## File Structure

Each memory is its own file with frontmatter:

```markdown
---
name: descriptive-name
description: one-line description (used to judge relevance in future sessions)
type: user | feedback | project | reference
---

[Memory content]
```

An index file (`MEMORY.md`) contains one-line pointers to each memory file:
```markdown
# Memory Index

## User & Collaboration
- [user-role](user_role.md) — Senior data engineer, new to this codebase

## Feedback
- [testing-approach](feedback_testing.md) — Integration tests only, no mocks

## Project
- [release-freeze](project_freeze.md) — Merge freeze 2026-03-05 through 2026-03-12

## Reference
- [bug-tracker](ref_linear.md) — Pipeline bugs in Linear project INGEST
```

## Operating Rules

1. **Check before writing.** Don't duplicate. Update existing memories when information changes.
2. **Keep the index under 200 lines.** If it grows past that, you have too many memories — prune or merge.
3. **Memories decay.** Project memories especially. Review periodically. Remove or update stale ones.
4. **Verify before recommending.** A memory that names a specific file or function is a claim that it existed when the memory was written. Check it still exists before acting on it.
5. **Organize by topic, not chronology.** Memories are for retrieval, not journaling.

## Getting Started

1. Start with feedback memories only — save the next 3 corrections your AI assistant needs
2. After a week, add project memories for active work context
3. After a month, add user memories if you work across different domains
4. Reference memories accumulate naturally — save them when you find yourself re-explaining where things live
5. Review monthly: delete stale project memories, merge redundant feedback memories
