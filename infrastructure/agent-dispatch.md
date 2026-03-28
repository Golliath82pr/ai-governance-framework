# Agent Dispatch Rules

## Concept

When your AI assistant can spawn sub-agents (Claude Code agents, background workers, parallel researchers), you need rules for when to dispatch, how many, and how to synthesize results.

Without rules, dispatch becomes ad-hoc — sometimes agents are spawned for trivial tasks (overhead), sometimes they're not spawned for complex ones (missed opportunity).

## Decision Tree

```
Is it a single tool call or < 3 file reads?
  -> Do it yourself. No agent overhead.

Are there 2+ independent questions/tasks?
  -> Parallel agents. One per concern.

Is it deep code tracing across 5+ files?
  -> Explorer agent (set thoroughness level).

Is it architecture/design requiring trade-off analysis?
  -> Architect agent, then challenge the output yourself.

Is it a full multi-workstream task (3+ workstreams)?
  -> Multiple typed agents in parallel. Track with dispatch log.
```

## Hard Rules

1. **Max 6 concurrent agents** `[empirical]`. Beyond this, synthesis quality degrades noticeably — the orchestrating model struggles to hold enough context from each agent's output to produce a coherent combined result.
2. **One concern per agent.** Clear prompt, clear deliverable. Don't bundle unrelated tasks.
3. **Foreground when chaining.** If step 2 depends on step 1's result — foreground. No placeholders.
4. **Background when independent.** If you have other work to do while it runs — background.
5. **Never parrot.** Agent output gets your assessment layered on top. Agree, challenge, or extend — never just relay.
6. **Fail fast.** If an agent returns garbage, handle it directly. One retry max, then do it yourself.
7. **Typed agents when the type matches.** Explorer for search, Code Reviewer for review, Architect for design. General-purpose only when nothing fits.

## Agent Type Quick Reference

| Task | Agent Type | Notes |
|---|---|---|
| Find files/code patterns | Explorer | Set thoroughness: quick, medium, thorough |
| Code review | Code Reviewer | Confidence-filtered output |
| Architecture analysis | Software Architect | Trade-off focused |
| Deep codebase understanding | Code Explorer | Traces execution paths |
| Implementation planning | Planner | Step-by-step |
| Security audit | Security Engineer | Threat modeling |
| Web/API research | General-purpose | With clear search instructions |

## When NOT to Use Agents

- User said "quick" or it's obviously a 30-second task
- Reading a file the user just pointed to
- Single tool call
- Tasks where you need to iterate interactively (agents can't ask clarifying questions mid-run)
- When the user is in rapid-fire conversation mode — agents add latency

## Synthesis Protocol

When multiple agents return results:

1. **Read all results before responding** — don't stream partial conclusions
2. **Identify contradictions** — if two agents disagree, that's signal, not noise
3. **Layer your own judgment** — you dispatched them, you're responsible for the synthesis
4. **Be concise** — the user doesn't need to see the full agent output. Give the punchline.

## Getting Started

1. For your next complex task, ask: "Would this be faster with 2 parallel agents?"
2. If yes, write clear, single-concern prompts for each
3. Synthesize the results yourself — don't relay
4. After a week, review: which dispatches saved time? Which added overhead? Adjust the decision tree.
