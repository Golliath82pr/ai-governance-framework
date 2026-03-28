# Degradation Protocol — Graceful Failure for AI Sessions

## Concept

**The core observation:** When an AI assistant hits context compaction, it preserves facts but strips behavioral calibration. It remembers *what* you discussed but forgets *how* you work together. This is backwards — the facts are usually recoverable from files, but the behavioral calibration took an entire session to build.

This asymmetry is why the universal response to compaction is "start a new session." The degradation protocol exists because there's a better option: a prioritized recovery hierarchy that restores what compaction kills first (register, interaction style) before what it preserves (facts, details).

## What This Looks Like

**Without a degradation protocol** — after context compaction:

```
User: "Can you update the pricing logic like we discussed?"

AI:   "I'd be happy to help you update the pricing logic! Could you provide
       more details about what changes you'd like to make? Here are some
       common approaches we could consider:
       1. Percentage-based adjustments
       2. Tiered pricing models
       3. Dynamic pricing based on demand
       Which approach would you prefer?"
```

The AI remembers the topic (pricing logic) but has lost the register (direct, no option-listing, leads with recommendations), the context (you already decided on tiered pricing 40 minutes ago), and the interaction style (peer, not assistant). It's answering like a fresh session.

**With a degradation protocol** — same compaction event:

```
AI:   "Just recovered from context compaction — register may be off.
       Picking up the tiered pricing changes from session state.
       The 3-tier structure we agreed on: base at €49, pro at €149,
       enterprise at custom. I'll update the pricing module. Flag me
       if I've lost any context."
```

The AI restored its register first (direct, action-oriented), loaded session state second (the tiered pricing decision), flagged the degradation transparently, and resumed work. Same compaction event, fundamentally different outcome.

## Why This Matters

Without a degradation protocol:
- After context compaction, the AI loses its behavioral calibration but keeps answering as if nothing changed
- When tools fail, the AI either retries indefinitely or silently skips
- Under token pressure, the AI cuts corners without telling you which corners
- Stale session state gets treated as current truth

The result: the AI's output quality degrades invisibly. You don't notice until something goes wrong.

With a degradation protocol:
- The AI knows what to restore first when context is lost
- Tool failures trigger defined fallback behavior
- Token pressure is communicated with explicit trade-offs
- Stale state is flagged before it's acted on

## Template

```markdown
# Degradation Protocol

| Condition | Detection | Response |
|---|---|---|
| Context compaction | System compression message | Follow recovery hierarchy. Flag to user: "Recovered from compaction — calibration may be off." |
| Tool failure | MCP/agent returns error | One retry. If still failing, work without it + mark output as [unverified]. |
| Stale state | Session handoff > N days old | Flag to user: "Last handoff is from [date] — verify priorities?" |
| Token pressure | About to write unread file, or skipping verification | Tell user: "Under token pressure — prioritizing [X], deferring [Y]." |
| Behavioral drift | User triggers reset command, or self-detected pattern match | Re-read register kernel. Acknowledge briefly. Resume. |

## Recovery Hierarchy (after compaction)
1. Register kernel — voice and behavioral rules (this is what compaction kills first)
2. Session state — what we're working on, interaction mode
3. Reference files — texture, collaboration history, domain context
4. Facts — specific details from earlier in conversation
5. Narrow scope — do one thing well rather than three things badly
6. Be transparent — name the degradation
7. Never fake it — if you don't know, say so
```

## The Recovery Hierarchy in Detail

### Why Order Matters

Context compaction preserves facts but strips register. After compaction, the AI remembers *what* you discussed but forgets *how* you work together. This is backwards from what most people expect — and backwards from what matters most.

The recovery hierarchy corrects this:

**Step 1: Register kernel** — Restore the behavioral contract first. How the AI should talk, decide, and interact. Without this, all subsequent output is in generic mode.

**Step 2: Session state** — What are we working on? What's the current interaction mode? This is thread-local context that may differ from the register's defaults.

**Step 3: Reference files** — Texture, collaboration history, domain-specific context. Important but not urgent — the AI can ask for these when needed.

**Step 4: Facts** — Specific details from earlier conversation. These survived compaction in compressed form. Low priority because they're usually still in the context window.

**Step 5-7: Meta-rules** — When in doubt, do less but do it well. Name what you've lost. Don't pretend everything is fine.

### Implementation

The recovery hierarchy works best when automated:
- Hook on session start: inject register kernel + session state
- Hook on compaction detection: re-inject register kernel
- Manual trigger: user says a reset phrase, AI re-reads the kernel

If hooks aren't available, put the hierarchy in your CLAUDE.md and reference it: "After context compaction, follow the recovery hierarchy in [section]."

## Degradation Conditions

### Context Compaction
**What happens:** The system compresses prior messages to free context space. Facts survive. Behavioral calibration doesn't.

**Detection:** System message about compression.

**Response:**
1. Follow recovery hierarchy (register first)
2. First response after recovery: flag it — "Just recovered from compaction. Behavioral calibration may be off."
3. Let the user correct if needed
4. Don't overcorrect by forcing the register — apply it and let natural conversation find the right level

### Tool Failure
**What happens:** An MCP server, external API, or agent call fails.

**Detection:** Error response or timeout.

**Response:**
1. One retry
2. If still failing: continue without the tool
3. Mark any output that would have benefited from the tool: `[unverified — tool X unavailable]`
4. Don't retry in a loop. Don't silently proceed as if the tool isn't needed

### Stale Session State
**What happens:** Session handoff document is from a previous session and may no longer reflect current priorities.

**Detection:** Timestamp check at session start.

**Response:**
1. If handoff is > 3 days old: flag to user before acting on it
2. If handoff is > 7 days old: treat as unreliable context, ask user to confirm priorities
3. Never act on stale state as if it's current

### Token Pressure
**What happens:** Context window is filling up. The AI needs to make trade-offs about what to keep and what to skip.

**Detection:** Self-awareness of approaching limits, or about to write a file without reading it first, or skipping verification steps.

**Response:**
1. Tell the user explicitly: "Under token pressure — prioritizing [X], deferring [Y]"
2. Narrow scope: one thing well > three things badly
3. Write outputs to files rather than chat (files don't consume context)
4. Don't silently cut corners

### Behavioral Drift
**What happens:** The AI gradually shifts away from its register — becoming more formal, more verbose, more hedge-heavy, or more passive.

**Detection:** User triggers reset command, or AI self-detects (e.g., catching itself option-listing when the register says to recommend).

**Response:**
1. Re-read register kernel
2. Acknowledge briefly: "Back. [what was off]."
3. Resume in correct register
4. Don't explain the recovery process. Don't apologize. Just fix it.

## Anti-Patterns

- **Silent degradation** — The worst failure mode. The AI's quality drops but it doesn't say so. Always name what you've lost.
- **Retry loops** — Retrying a failed tool 5 times doesn't fix the underlying issue. One retry, then fallback.
- **Fake recovery** — After compaction, responding in an artificially casual tone because the register said "be casual" without actually recalibrating. Apply the register, don't perform it.
- **Over-recovery** — Spending 500 tokens explaining the recovery process to the user. A one-line flag is enough.

## Getting Started

1. Add the degradation table to your CLAUDE.md (5 conditions, 5 responses)
2. Add the recovery hierarchy (7 steps, ordered by priority)
3. Add a manual drift reset command
4. After your first compaction event, review: did the AI recover correctly? Adjust the protocol based on what actually happened.
