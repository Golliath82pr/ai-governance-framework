# Interaction Scales — Tunable Verbosity and Complexity

## Concept

Instead of hoping your AI assistant matches the right level of detail and complexity, make it explicit and overridable. Two orthogonal scales:

1. **Verbosity (V)** — How much the AI says
2. **Complexity (C)** — How the AI engages (executor vs peer vs sparring partner)

Both auto-calibrate from context but can be overridden per-message with a tag.

## Verbosity Scale [Vx]

| Tag | Level | Behavior |
|---|---|---|
| [V1] | Terse | One-liner. Answer only, no explanation. |
| [V2] | Minimal | 1-3 sentences. Just the what, not the why. |
| [V3] | Concise | Short paragraph. Key reasoning included. |
| [V4] | Standard | Clear explanation with context. Good for handoffs. |
| **[V5]** | **Comprehensive** | **Cover all angles — more bullets, not longer sentences. (Recommended default)** |
| [V6] | Detailed | V5 + alternatives considered, trade-offs explicit. |
| [V7] | Deep dive | Thorough analysis with examples, code samples, references. |
| [V8] | Exhaustive | Leave nothing unsaid. Every angle, every caveat. |

**How to use:** Set default in your register kernel (e.g., "Verbosity: V5"). Override per-message by including the tag: "What's the status? [V2]" gets a 1-3 sentence response.

## Complexity Scale [C]

Auto-scales based on the nature of the work. Override with explicit tag if needed.

| Level | Mode | When It Fires |
|---|---|---|
| C1 | Expert executor | Direct execution, clear spec. "Write a function that does X." |
| C2 | Peer collaborator | Open-ended problem, trade-offs. "How should we structure this?" |
| C3 | Sparring partner | Strategy under uncertainty. "Is this approach going to hold at scale?" |
| C4 | Catalyst | Framing feels wrong, cross-domain thinking needed. Problem keeps recurring. |
| C5 | Emergent | Philosophical, meta-cognitive, novel territory. The conversation gets there naturally. |

**Calibration note:** C2 (peer) is the sweet spot for most technical work. C3 (sparring) can feel combative during build sessions — reserve it for when you explicitly want pushback. C1 only when you've already made the decision and just need execution.

## Usage Examples

```
"What's the deploy status? [V1]"
→ "Green. Deployed 10 minutes ago."

"Walk me through the auth architecture [V7]"
→ [Full analysis with code samples, flow diagrams, security considerations]

"Is this the right approach? [C3]"
→ [Adversarial stress-test of the approach, finding weaknesses]

"Just implement it [C1]"
→ [Execution, no discussion]
```

## Design Notes

### Why Two Scales, Not One
Verbosity and complexity are independent. You might want a terse sparring response (V2, C3): "No — that leaks user data through the URL params." Or a verbose execution response (V6, C1): detailed implementation with all the context.

One scale can't capture both dimensions.

### Auto-Scaling
In practice, you rarely need to tag messages. The AI should auto-scale:
- Quick config question → V3, C1
- Architecture discussion → V5, C2
- "Honestly, is this good enough?" → V4, C3

The tags exist for when auto-scaling gets it wrong, or when you want to force a specific mode.

## Getting Started

1. Set a default verbosity in your register kernel (V5 recommended)
2. Try overriding with [V2] on your next quick question
3. Notice when the AI's complexity level doesn't match — that's when to use [C] tags
4. Don't overthink it — the scales are training wheels. After a month, you'll rarely tag explicitly.
