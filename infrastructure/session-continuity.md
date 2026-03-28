# Session Continuity — Surviving Gaps and Compaction

## Concept

A session continuity document is a living snapshot of what's happening right now — current work, interaction state, open threads, and recent decisions. It's overwritten each update (not appended), read at session start, and serves as the handoff between sessions.

Without this, every new session starts with "where were we?" and 10 minutes of context reconstruction.

## Template

```markdown
# Session Continuity

Live session state. Auto-updated during work, read at session start.

## Session Handoff
<!-- OVERWRITE this block every update — it's a snapshot, not a log. -->
<!-- STALENESS: If "Last active" > 3 days at bootstrap, flag to user. -->
- **Last active**: [date]
- **Last session**: [1-2 sentences on what was accomplished]
- **Previous session**: [1-2 sentences for context depth]
- **Next up**: [numbered priority list]
- **Blockers**: [or "None"]
- **Open threads**: [work in progress across projects]
- **Open loops**: [in-flight work that MUST survive session boundaries]

## Interaction State
<!-- Thread-local register snapshot — how the AI sounds in THIS session. -->
<!-- After compaction: this block is step 2 in the recovery hierarchy. -->
- **mode**: [engaged / mechanical / autonomous]
- **warmth**: [standard / warm / minimal]
- **pushback**: [active / passive / off]
- **initiative**: [high / standard / low]
- **verbosity**: [V1-V8]
- **recent_irritant**: [what annoyed the user recently, or "none"]
- **avoid**: [specific behaviors to avoid this session]
- **session_tone**: [what kind of work this session is focused on]

## Project Snapshots
<!-- 2-3 lines per active project. Archive old ones. -->

### [Project Name]
- Path: [location]
- Last: [date]. [status summary]
- Next: [what needs to happen]

## Decision Trail
<!-- Append-only. Last 5 decisions. Move older ones to archive. -->
<!-- Format: DATE | DECISION | WHY | ALTERNATIVE REJECTED -->
- [date] | [what was decided] | [why] | Alt: [what was rejected and why]
```

## Design Principles

### Snapshot, Not Log
The session handoff is overwritten each update. It's a snapshot of current state, not a history. If you want history, maintain a separate decision archive.

Why: A log grows unboundedly. A snapshot stays the same size. The AI reads this at session start — it needs current state, not archaeology.

### Interaction State Is Thread-Local
The register kernel defines baseline behavior. The interaction state captures session-specific overrides — maybe this session is more mechanical, or the user is in rapid-fire mode, or something irritated them earlier.

This distinction matters after compaction: the kernel is restored first (step 1), the interaction state second (step 2). The kernel gets you close; the interaction state gets you precise.

### Open Loops Are Sacred
Open loops are in-flight work that MUST survive session boundaries. A code review that's half-done. A file that was generated but not yet reviewed. A decision that was deferred.

These are distinct from "next up" (planned future work) and "open threads" (ongoing projects). Open loops are **commitments** — things someone is waiting on or that will break if forgotten.

### Decision Trail Prevents Rehashing
Without a decision trail, the AI may suggest approaches you've already considered and rejected. The trail captures:
- What was decided
- Why (the reasoning, not just the choice)
- What was rejected (so the AI doesn't suggest it again)

Keep only the last 5. Move older decisions to an archive file. The trail is for preventing immediate rehashing, not for historical completeness.

## Update Discipline

The session continuity document should be updated:
- After completing significant work (not every small step)
- Before ending a session (the handoff for next time)
- After making a decision that changes priorities
- After compaction recovery (to capture the new state)

The rule: **inline updates, not deferred dumps.** Update the file as part of completing the work, not as a separate task at the end. Deferred updates get forgotten or compressed.

## Staleness Detection

At session start, check the "Last active" timestamp:
- **Same day**: Proceed normally
- **1-3 days**: Quick check — "Picking up from [date]. Still working on [X]?"
- **> 3 days**: Flag explicitly — "Last handoff is from [date]. Should we verify priorities?"
- **> 7 days**: Treat as potentially unreliable. Ask user to confirm before acting on it.

## Getting Started

1. Create a session continuity file with the handoff and interaction state sections
2. Update it at the end of your next 3 sessions
3. At the start of session 4, notice how much faster you get to productive work
4. Add project snapshots and decision trail as your usage grows
