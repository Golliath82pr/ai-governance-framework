# AI Behavioral Governance Framework

A pattern language for making AI assistants behave consistently, degrade gracefully, and maintain institutional memory across sessions.

## The Core Observation

When an AI coding assistant hits context compaction, it preserves facts but strips behavioral calibration. It remembers *what* you discussed but forgets *how* you work together. This is the opposite of what you'd want — and it's the reason most long sessions end with "let me just start a new chat."

The ecosystem has built good solutions for **capability** (more agents, more tools) and **persistence** (session memory, context injection). What's less well-covered is **governance** — how to make an AI collaborator recover when context is lost, catch its own reasoning errors, and manage degradation transparently.

This framework formalizes solutions to three problems encountered during intensive daily usage:

1. **Graceful degradation** — When context compacts, tools fail, or tokens run low, there's no protocol for what to restore first and what to let go
2. **Behavioral drift** — AI assistants change personality mid-session, with no mechanism to detect or correct it
3. **Reasoning errors** — Known failure patterns that repeat because the AI doesn't remember past mistakes across sessions

## When This Is Overkill

Not every workflow needs a governance framework. Before adopting any of this:

- If your sessions are under 30 minutes and you rarely hit context compaction, skip the degradation protocol
- If your AI assistant's behavior doesn't bother you, skip the register kernel
- If you're not making domain-specific mistakes repeatedly, skip circuit breakers
- If you use one model for short tasks, skip dispatch routing entirely

**Start with circuit breakers** — they have the highest ROI for the lowest setup cost. Five trigger/stop/alternative rules in your CLAUDE.md costs you 10 minutes and saves you hours.

## Repository Structure

```
ai-governance-framework/
├── README.md
├── LICENSE
├── core/
│   ├── degradation-protocol.md     — Recovery hierarchy for when things break
│   ├── register-kernel.md          — Behavioral calibration as managed artifact
│   └── circuit-breakers.md         — Pattern-match vetoes for reasoning errors
├── infrastructure/
│   ├── session-continuity.md       — Surviving session gaps and compaction
│   ├── dispatch-routing.md         — Multi-model governance
│   ├── agent-dispatch.md           — Sub-agent spawning rules
│   └── interaction-scales.md       — Tunable verbosity and complexity
├── memory/
│   ├── taxonomy.md                 — 4-type structured memory system
│   └── MEMORY-TEMPLATE.md          — Starter index
└── examples/
    ├── circuit-breaker-examples.md — CBs across 6 domains
    └── register-examples.md        — 3 working register kernels (start here)
```

## Core Patterns

### 1. Degradation Protocol (`core/degradation-protocol.md`)

What to do when things break — context compaction, tool failures, token pressure, stale state. A prioritized recovery hierarchy: restore behavioral calibration first (what compaction kills), then session state, then reference material, then facts (what compaction preserves).

Analogous to checkpoint/restart patterns in distributed systems, mapped onto AI session state with explicit priority ordering. The key insight — compaction preserves facts but strips register — drives the entire hierarchy.

### 2. Register Kernel (`core/register-kernel.md`)

A single source of truth for how the AI should behave — voice, interaction style, decision-making rules. Injected at session start, recovered after compaction.

The individual components (system prompts, custom instructions, persona files) are common. What this formalizes is treating them as a **layered authority architecture** — kernel (authoritative) / texture (reference) / state (thread-local) — with explicit precedence, conflict resolution semantics, and drift detection. The self-healing from files under compaction ties directly to the degradation protocol.

For three immediately usable register kernels (peer collaborator, directive lead, researcher), see [`examples/register-examples.md`](examples/register-examples.md).

### 3. Circuit Breakers (`core/circuit-breakers.md`)

Pattern-match vetoes for known reasoning failures. A lookup table of expensive mistakes formatted so the AI can catch them before producing output.

Inspired by the circuit breaker pattern in software engineering, adapted here as **static vetoes** rather than stateful transitions. No state tracking, no automatic reset — just a trigger/stop/alternative table the AI pattern-matches against in real time. The name is borrowed for its intuition ("stop before damage"), not its full semantics.

For domain-specific examples across web development, Python, financial modeling, DevOps, and ERP, see [`examples/circuit-breaker-examples.md`](examples/circuit-breaker-examples.md).

## Infrastructure Patterns

Supporting architecture that makes the core patterns operational. These are practical guides — adopt as your workflow demands.

| Pattern | File | Purpose |
|---|---|---|
| Session Continuity | `infrastructure/session-continuity.md` | Living handoff document that survives session gaps. Tightly coupled with the degradation protocol — recovery step 2 reads session state. |
| Multi-Model Dispatch | `infrastructure/dispatch-routing.md` | Governs when and how to route work across multiple AI models. Shadow review as default quality gate. |
| Agent Dispatch | `infrastructure/agent-dispatch.md` | Systematic rules for spawning sub-agents — when to parallelize, how many, how to synthesize. |
| Interaction Scales | `infrastructure/interaction-scales.md` | Tunable verbosity (V1-V8) and complexity (C1-C5) with per-message overrides. |

## Memory Architecture

A companion concern: how to make the AI *remember* correctly across sessions. Structured into four types with different lifecycles:

| Type | Purpose | Example |
|---|---|---|
| `user` | Who you are, how you work | "Senior engineer, new to React, deep Go expertise" |
| `feedback` | What to do and not do | "Don't mock the database — we got burned by divergence" |
| `project` | Ongoing work context | "Merge freeze starts March 5 for mobile release" |
| `reference` | Where to find things | "Pipeline bugs tracked in Linear project INGEST" |

The memory taxonomy is adjacent to the behavioral governance patterns — it addresses persistence while the core patterns address consistency and recovery. Details in `memory/taxonomy.md`.

## How to Use This

1. **Start with circuit breakers.** Write 5 trigger/stop/alternative rules for your domain. Put them in your CLAUDE.md. 10 minutes, immediate payoff.

2. **Add the degradation protocol** once you've hit your first context compaction and watched the AI forget how you work together. Add the recovery hierarchy to your config.

3. **Add a register kernel** when you want consistent behavior across sessions. Start with 3-5 behavioral rules, a drift detection command, and an authority chain. Keep it under 70 lines.

4. **Add infrastructure patterns** as needed. Session continuity for multi-session work. Dispatch routing for multiple models. Interaction scales for tunable verbosity.

5. **Build memory gradually.** Start with feedback memories (corrections). Add project context as it accumulates. Don't create 40 files on day one.

## What This Is Not

- Not a prompt engineering guide
- Not a Claude Code configuration tutorial
- Not a collection of useful CLAUDE.md snippets
- Not a tool or installable package

It's a **pattern language** — named, reusable solutions to recurring problems in human-AI collaboration. Adapt the patterns to your tools and workflow. Keep in mind that for a more professional version a full harnest, including controlpanels, QA, Regression testing will have to be done. Good luck!

## Origin & Limitations

Built over 12+ months of daily, intensive Claude Code usage across multiple domains (ERP development, financial modeling, trading systems, web development, document automation, data science). Every pattern exists because a real problem demanded it — nothing was designed speculatively.

**Empirical basis:** These patterns are validated through sustained personal usage (n=1), not controlled experiments. They solved real problems at scale for one heavy user. The framework is a formalized architecture with templates, not a research claim. Your mileage may vary — adapt what works, discard what doesn't. 

## License

MIT. Use it, fork it, adapt it. Attribution appreciated but not required.
