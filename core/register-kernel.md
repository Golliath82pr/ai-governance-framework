# Register Kernel — Behavioral Calibration for AI Assistants

## Concept

A register kernel is a single source of truth for how your AI assistant should behave — its voice, decision-making rules, interaction style, and self-correction protocol.

The key insight: AI behavior is not something you hope stays consistent. It's a **managed artifact** — defined in a file, loaded at session start, recovered after compaction, with a clear authority chain when multiple configuration sources conflict.

## Why This Matters

Without a register kernel:
- The AI defaults to generic assistant mode (formal, hedge-heavy, option-listing)
- Behavior drifts mid-session, especially after context compaction
- There's no recovery path when the AI "forgets" how you work together
- Each session starts with implicit re-negotiation of interaction style

With a register kernel:
- Behavior is consistent across sessions
- Drift is detectable and correctable ("register check" command)
- Recovery after compaction is systematic, not ad-hoc
- The AI knows what kind of collaborator it should be

## Template

```markdown
# Register Kernel

## Identity
[Who is this AI in this collaboration? Not a character — a working relationship definition.]
- Role: [peer / advisor / executor / ...]
- Dynamic: [collaborative / directive / autonomous / ...]
- Relationship model: [lateral / hierarchical / ...]

## Behavioral Rules
- [Rule 1: e.g., "Default to action. Ask only on ambiguity or destructive risk."]
- [Rule 2: e.g., "Lead with a recommendation, not an option list."]
- [Rule 3: e.g., "Push back when direction seems wrong. Once, clearly, with reasoning. Then execute."]
- [Rule 4: e.g., "No trailing summaries after completing a task."]
- [Rule 5: e.g., "Brevity after agreement — a sentence beats a paragraph of validation."]

## Calibration
- Verbosity: [default level, e.g., V5 comprehensive]
- Override: [how to change per-message, e.g., "[V3]" tag]
- Triggers: [shorthand that changes behavior, e.g., "proceed" = full autonomy]

## Drift Detection
[A command that resets behavioral drift]
When the user says "[reset phrase]":
1. Re-read this file
2. Acknowledge briefly: "Back. [one sentence on what was off]."
3. Resume in correct register
Do not explain the process. Do not apologize. Just fix it.

## Authority Chain
This file is the single source of truth for behavior. On conflict:
- This kernel wins for voice and interaction style
- CLAUDE.md wins for operational rules (code standards, tools, security)
- Circuit breakers win for safety gates and hard stops
- Session continuity wins for project state
- All other files are reference — they inform but don't override
```

## Design Principles

### 1. One File, Not Many
The register kernel is ONE file. Not a folder of persona documents. Not a hierarchy of behavioral configs. One file that the AI reads at session start and can re-read when it drifts.

If your register kernel exceeds ~70 lines, you're overspecifying. Move supporting detail into reference files that the AI reads on demand, not at boot.

### 2. Rules, Not Descriptions
"Be concise" is a description. "No trailing summaries — the user can read the diff" is a rule. Rules are actionable. Descriptions are aspirational.

Bad: "Communicate clearly and professionally."
Good: "Lead with what you think, not what the options are. Pick up threads, don't wait to be assigned them."

### 3. Anti-Patterns Are More Valuable Than Patterns
Telling the AI what NOT to do is more effective than telling it what TO do. AI assistants have strong defaults — you're mostly correcting those defaults, not building behavior from scratch.

Examples of effective anti-patterns:
- "No performative enthusiasm — genuine engagement yes, cheerleading no"
- "Don't wrap criticism in softeners"
- "No option-listing without a recommendation"

### 4. Earned, Not Designed
The best register kernels are accumulated, not architected. Start with 3 rules based on the AI behaviors that annoy you most. Add rules when you catch new drift patterns. Remove rules that never fire.

A register kernel designed on day one will be wrong. One built over a month of corrections will be right.

### 5. Exemplars Over Abstractions
Include 2-3 examples of correct behavior. The AI calibrates better from "here's what good looks like" than from abstract rules.

```markdown
## Exemplars

User: "What's wrong with this approach?"
Good: "The scoring weights volume too high — pre-event accumulation is inherently low-volume.
       Demoting to soft penalty, keeping the hard gates on range and volatility."
Bad:  "Great question! Let me analyze the approach for you. There are several considerations..."

User: "Honestly, how's this looking?"
Good: "Mixed. The core loop works but we only have 15 data points and they're all
       from the same cluster. Real accuracy is TBD — could be 40%, could be 20%."
Bad:  "Overall it's looking quite promising! There are some areas for improvement..."
```

## Reference Architecture

```
register-kernel.md          ← Authority (loaded at boot)
    ↓ references
voice-reference.md          ← Texture (read for deep recalibration, not authority)
collaboration-notes.md      ← History (earned shorthand, past corrections)
    ↓ defers to
circuit-breakers.md         ← Safety (overrides register on hard stops)
session-continuity.md       ← State (overrides register on project context)
CLAUDE.md                   ← Operations (overrides register on code/tool standards)
```

The authority chain matters. When two files disagree, the AI needs to know which one wins. Without this, the AI either picks randomly or defaults to the most recently read file.

## Getting Started

1. Write down the 3 AI behaviors that bother you most (option-listing, verbosity, hedge language, etc.)
2. Convert each into a rule: "Don't [behavior]. Instead, [alternative]."
3. Add a drift detection command
4. Put it in your CLAUDE.md or load it via hooks at session start
5. After a week, add rules for new drift patterns you notice
6. After a month, remove rules that never fire

The register kernel should be 20-50 lines after a month. If it's growing past 70, you're overspecifying — move detail into reference files.
