# Multi-Model Dispatch Routing

## Concept

When you have access to multiple AI models (Claude + Codex, Claude + GPT, local + cloud), dispatch routing governs when and how to use each one — not for cost optimization, but for correctness.

Different models have different strengths. A dispatch protocol routes work to the model best suited for it, defines how to handle disagreement, and establishes shadow review as a default quality gate.

## The Dispatch Card

Every cross-model handoff uses a standard format. Nothing more.

```
mode:      shadow | review | research | implement | deep
target:    [what the other model should look at]
ask:       [one sentence, one job]
done_when: [concrete exit condition]
```

**Examples:**

```
mode:      review
target:    git diff HEAD (uncommitted changes)
ask:       Flag regressions or breaking changes
done_when: CLEAR or FLAG with severity + location

mode:      research
target:    features/ pipeline
ask:       Which features use post-event data? Trace each to raw source.
done_when: List of features with data lineage, flagging any leakage

mode:      implement
target:    constants.py (new file)
ask:       Extract all magic numbers from scanner.py into typed constants
done_when: File exists, scanner imports from it, tests pass
```

## Routing Table

### When to Dispatch

| Trigger | Route | Mode |
|---|---|---|
| Just wrote or modified non-trivial code | Secondary model | `shadow` |
| Pre-commit diff with 3+ files changed | Secondary model | `review` |
| Need to trace symbol/behavior across repo | Repo-aware model | `research` |
| Domain methodology question | Domain specialist | `research` |
| Localized patch, problem already identified | Any capable model | `implement` |
| User asks for multi-perspective input | All models | `deep` |

### The 2-Minute Rule

If framing a solo answer takes more than 2 minutes of reasoning, or the answer needs evidence beyond intuition — dispatch. Optimize for correctness, not speed.

### When NOT to Dispatch

- Single tool call or < 3 file reads
- User said "quick" or is in rapid-fire mode
- Task is conversational / informational
- Secondary model is unavailable

## Shadow Mode — The Default

Shadow review is **opt-out, not opt-in.** After producing non-trivial technical output:

1. Ship the output to the user immediately (no blocking)
2. Send the secondary model a shadow dispatch card
3. If it returns `FLAG HIGH/CRITICAL` — surface to user
4. If `FLAG MEDIUM` — note inline next time you reference that output
5. If `CLEAR` — nothing. No ceremony.

**What counts as non-trivial:** New files, architecture decisions, financial calculations, security-adjacent code, multi-file changes. Single-line fixes, config tweaks, docs — skip shadow.

## Deep Mode — Contested Decisions

Use when blast radius is high AND the answer is genuinely ambiguous, or the user explicitly asks for multi-model input. Rare — less than once per session.

### 4-Phase Procedure

1. **Blind Draft** — Both models answer independently, same prompt. Each provides: Verdict, Reasoning, Key Assumptions, What Would Change My Mind. The primary model forms its position BEFORE seeing the secondary's response.

2. **Delta Analysis** — Compare positions. Identify the settling question (what resolves the disagreement) and the settling artifact (test, benchmark, code trace, etc.).

3. **Reaction Round** — Show each model the other's position. Ask: "Does this change your recommendation? Produce the settling artifact if you can."

4. **Synthesis** — Document: shared foundation, where they converged, what remains contested, single actionable recommendation.

### ABSTAIN Gate
If after Phase 3 both models score confidence < 60%, or the settling artifact can't be produced, outcome is **ABSTAIN**. Document what validation is needed. Don't ship false consensus.

Max 2 rounds. If not converged after Phase 3, ship the narrower qualified answer or ABSTAIN.

## Transport States

| State | Detection | Behavior |
|---|---|---|
| HEALTHY | Secondary model returns content | Full dispatch |
| DEGRADED | Returns empty/error | Retry once, then solo + marker |
| UNAVAILABLE | Both retry paths fail | Solo. Mark output `[no-shadow]` |

Probe with a lightweight call before first dispatch of the session. Don't probe before every call.

## What This Replaces

Most people either:
1. Use one model for everything (no quality gate)
2. Manually paste between models (slow, inconsistent)
3. Route by cost (send cheap tasks to cheap models)

This protocol routes by **artifact type** — what the work IS determines which model handles it. Shadow review catches errors the primary model can't see. Deep mode resolves genuine ambiguity through structured disagreement.

## Getting Started

1. Identify your primary and secondary models
2. Enable shadow mode: after non-trivial output, send a review card to the secondary
3. Start with `shadow` and `review` modes only — skip `deep` until you need it
4. Track: how often does shadow review catch something real? If rarely, you may not need the overhead.
