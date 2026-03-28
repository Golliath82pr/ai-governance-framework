# Circuit Breakers — Epistemic Guardrails for AI Reasoning

## Concept

A circuit breaker is a pattern-match veto: if the AI recognizes a specific trigger in what it's about to do, it **stops before producing output** and uses a known-safe alternative instead.

This is not error handling. Error handling catches failures after they happen. Circuit breakers prevent known categories of failure from happening at all.

## Why This Works

AI assistants make the same expensive mistakes repeatedly because:
1. They don't remember past failures across sessions
2. They can't distinguish "correct-looking" from "actually correct" in domain-specific contexts
3. Confidence and correctness are uncorrelated for edge cases

Circuit breakers solve this by encoding your domain's "expensive lessons" in a format the AI reads at session start and pattern-matches against during work.

## Template

```markdown
# Circuit Breakers

Bootstrap-loaded. Pattern-match vetoes — if you recognize the TRIGGER, STOP before writing code.

| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-1 | [pattern you recognize] | [why it fails] | [correct approach] |
| CB-2 | ... | ... | ... |
```

### Column Definitions

- **TRIGGER**: The specific pattern the AI should watch for. Be concrete — "float for money" not "incorrect numeric types."
- **STOP**: Why this fails. One line explaining the failure mode. The AI needs to understand *why* to catch variants.
- **USE INSTEAD**: The correct alternative. Specific enough to act on immediately.

## Examples Across Domains

### Software Engineering
| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-1 | `float` for monetary calculation | Drift: 0.1+0.2 != 0.3 | `Decimal` type with explicit precision |
| CB-2 | Client-side validation as enforcement | Bypassed by API calls, CSV imports, devtools | Server-side validation. Client-side is UX only. |
| CB-3 | `"use client"` on data-fetching component | Disables SSR, kills SEO, double waterfall | Keep as Server Component. Extract interactive child. |

### Financial Modeling
| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-4 | IRR/NPV without scenario qualification | Single-scenario misleads decision-makers | Always: Base + Optimistic + Conservative |
| CB-5 | 20-year projection table in portrait mode | Clips in PDF, auto-sizes unpredictably | Landscape section break or split 2x10-year |
| CB-6 | Discount rate without sourcing | Assumed WACC varies wildly by context | Source: comparable transactions, CAPM, or state assumption explicitly |

### Data Engineering
| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-7 | Training model on data that includes target-derived features | Leakage: model learns the answer, not the pattern | Trace every feature to raw source. If path crosses target, remove. |
| CB-8 | Validating model on same distribution as training | Overfitting invisible in metrics | Walk-forward validation or leave-group-out. Never random split on time-series. |
| CB-9 | Upgrading database/vector store in-place on production data | Migration may be destructive | Pin version. Test migration on disposable copy first. |

### API & Integration
| # | TRIGGER | STOP | USE INSTEAD |
|---|---|---|---|
| CB-10 | API call without rate limiting | Burst traffic triggers bans, costs spiral | Minimum delay between calls. Never burst. |
| CB-11 | Search/query without result limit | Returns unbounded results, memory/governance blow | Always paginate. Set explicit page size. Use streaming callbacks. |

## Operating Rules

1. **Max 15 circuit breakers.** More than that and they stop being read. If you need more, the oldest ones should be retiring into your general knowledge.

2. **Validate periodically.** Each CB should have a last-validated date. If it hasn't been relevant in 6 months, retire it to a lessons-learned archive.

3. **Be specific, not general.** "Don't use floats for money" is a circuit breaker. "Write correct code" is not. The trigger must be pattern-matchable.

4. **Include the WHY.** The AI needs to understand the failure mode to catch variants. "Fails silently" vs "throws error" changes how urgently the CB matters.

5. **Bootstrap-load them.** Circuit breakers belong in your CLAUDE.md or equivalent — loaded at session start, not looked up on demand.

## Adversarial Pre-Ship Check

Before shipping any significant output, run these three questions:

1. What breaks if someone follows this spec verbatim with zero domain knowledge?
2. What fails silently under 10x production volume?
3. What am I most uncertain about, and what's the blast radius if wrong?

These aren't circuit breakers — they're a meta-check that catches problems your CBs don't cover yet. When a pre-ship check catches something new, consider adding it as a CB.

## Building Your First Set

Start with 5. Not 15 — five.

1. Think of the last 3 times your AI assistant produced something that looked right but was wrong
2. Think of the last 2 domain-specific gotchas that only experience teaches
3. Write each as TRIGGER / STOP / USE INSTEAD
4. Put them in your CLAUDE.md
5. After a month, review: which fired? Which never matched? Add new ones from fresh mistakes, retire stale ones.

The circuit breaker set is a living document. It should evolve with your work, not be designed once and forgotten.
