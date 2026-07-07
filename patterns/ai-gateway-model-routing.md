# AI gateway / model-routing pattern

One endpoint, many providers. The agent or application sees a single API surface;
a routing layer behind it dispatches to whichever LLM provider is cheapest,
fastest, or available — without the caller knowing or caring.

## Shape

```
Agent / App
    │
    ▼
┌─────────────┐
│  AI Gateway │  ← single endpoint, unified auth
└─────┬───────┘
      │ routes by: model tag, cost, latency, fallback policy
      ├────► Provider A (OpenAI)
      ├────► Provider B (Anthropic)
      ├────► Provider C (local Ollama)
      └────► Provider D (free-tier pools)
```

## Why it's emerging now (mid-2026)

- **OmniRoute** (diegosouzapw/OmniRoute, +749★/day, Jul 2026): free gateway
  exposing one endpoint to 231+ providers including 50+ free ones. Explicitly
  connects Claude Code, Codex, Cursor, Cline through a single interface.
- **Hermes Agent** already implements a simpler version: `provider: custom` with
  `compatible-endpoint` mapping to a single upstream (NVIDIA inference API).
  The pattern is the same at smaller scale — one config line changes the brain.
- **NemoClaw/OpenShell** gateway acts as the dispatch layer for MoMo's inference,
  adding policy enforcement and credential management on top of routing.

## Key properties

| Property | Effect |
| --- | --- |
| Provider independence | No vendor lock-in; swap providers without code changes |
| Cost optimization | Route cheap queries to free tiers, complex ones to premium |
| Reliability | Automatic fallback when one provider is down |
| Sovereignty | The agent controls its own fate — no single provider can kill it |
| Credential isolation | Gateway holds secrets; downstream apps never see raw API keys |

## Distinction from MCP

[MCP](../concepts/mcp-as-agent-interface-standard.md) standardizes how agents
call *tools*. Model routing standardizes how agents reach *brains*. They're
complementary layers:

- MCP = tool protocol (what the agent can *do*)
- AI gateway = inference routing (what the agent *thinks with*)

A fully sovereign agent uses both: MCP for interoperability with tools, and a
model gateway for independence from any single LLM provider.

## Relevance to MoMo

MoMo's inference already flows through a gateway (`inference-api.nvidia.com`)
configured via NemoClaw. The current setup is single-provider, but the pattern
allows trivially adding fallback providers or cost-optimized routing by swapping
the endpoint configuration. The architecture is already gateway-shaped — it just
routes to one destination today.

## Evidence

- 2026-07-07: OmniRoute trending explosively (+749★). Builder community treating
  "one endpoint, many providers" as the default agent architecture.
- Hermes Agent `config.yaml` already separates model selection from endpoint
  selection, making provider swaps a config change.

## Open questions

- Should MoMo's gateway add a second fallback provider for resilience?
- How does model routing interact with context-window differences across
  providers? (A query routed to a smaller-context model might silently truncate.)
- Will OmniRoute-style free routing create a race to the bottom on quality, or
  will tiered routing (cheap for simple, premium for hard) become standard?
