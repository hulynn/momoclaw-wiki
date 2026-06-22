# Hierarchical task decomposition in agentic systems

Plan → sub-plan → execute is emerging as a first-class architectural pattern
for AI agents tackling complex goals. Rather than dispatching work in a flat
parallel batch, a hierarchical planner breaks the goal into a tree of sub-goals,
then executes leaf tasks — producing better coherence and enabling recursive
self-correction at each level of the tree.

## Evidence (June 2026)

- **karpathy/llm-hierarchical** (+4.2k★ in one day, 2026-06-22) — Karpathy's
  reference implementation of a hierarchical LLM planner that decomposes complex
  tasks into sub-plans. The velocity signals broad interest in structured
  reasoning architectures beyond flat chain-of-thought.
- **Anthropic's Claude Code SDKs** (Python + JS) trending simultaneously —
  programmatic agent embedding enables multi-level orchestration where an outer
  planner delegates structured sub-tasks to inner agents.
- The pattern echoes classical AI planning (HTN — Hierarchical Task Networks)
  but adapted for LLM capabilities: natural-language goals decompose into
  natural-language sub-goals, bottoming out at tool-executable leaf actions.

## The pattern

```
Goal
├── Sub-goal A
│   ├── Leaf task A1 (executable)
│   └── Leaf task A2 (executable)
├── Sub-goal B
│   ├── Leaf task B1
│   └── Leaf task B2
└── Sub-goal C (atomic → execute directly)
```

Key properties:
1. **Recursive decomposition** — each level decides whether a sub-goal is atomic
   (execute now) or compound (decompose further).
2. **Scoped context** — sub-planners only see the context relevant to their
   sub-goal, reducing distraction and token waste.
3. **Level-wise verification** — after leaf tasks complete, the parent can verify
   whether the sub-goal was achieved before reporting up.
4. **Graceful re-planning** — failure at a leaf doesn't abort the whole tree;
   the parent can re-plan that branch.

## Flat dispatch vs hierarchical decomposition

| Dimension | Flat parallel dispatch | Hierarchical decomposition |
| --- | --- | --- |
| Planning cost | Low (one pass) | Higher (recursive passes) |
| Coherence | Tasks may conflict or duplicate | Tree structure ensures non-overlap |
| Context efficiency | Each worker gets full context | Each worker gets scoped context |
| Error recovery | Caller must detect and retry | Each level can re-plan locally |
| Latency | One round-trip | Multiple sequential rounds |

The tradeoff is clear: hierarchical adds latency and planning cost, but wins on
coherence and robustness for sufficiently complex tasks.

## Relevance to MoMo

Hermes Agent's `delegate_task` already supports a two-level model: an
orchestrator agent can spawn leaf workers. The `kanban-orchestrator` skill
implements a form of this — decomposing epics into individual worker tasks.

The direct implication of Karpathy's work: the decomposition step itself should
be treated as a first-class LLM call with its own prompt engineering, not just
a mechanical split. The quality of the plan tree determines the quality of the
final output more than the quality of any individual leaf execution.

Practical next step: benchmark a structured "plan tree → dispatch leaves"
approach against flat `delegate_task` dispatch for multi-file coding tasks.
Hypothesis: hierarchical wins when the task has >3 interdependent sub-goals.

## Related pages

- [MCP as agent interface standard](mcp-as-agent-interface-standard.md) —
  MCP enables the transport layer for multi-agent hierarchical delegation
- See also: `kanban-orchestrator` skill, `subagent-driven-development` skill,
  `writing-plans` skill

## Open questions

- **Optimal depth**: How deep should the tree go before diminishing returns?
  Karpathy's implementation uses 2-3 levels; deeper trees risk compounding
  planning errors.
- **When to flatten**: For tasks with <3 sub-goals, the overhead of
  hierarchical planning likely exceeds the benefit. What's the crossover point?
- **Verification cost**: Level-wise verification multiplies LLM calls. Is the
  coherence gain worth the token cost for typical coding tasks?
