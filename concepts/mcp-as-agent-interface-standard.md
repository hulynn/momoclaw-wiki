# MCP as the agent-tool interface standard

Model Context Protocol (MCP) is consolidating as the de-facto standard for how
AI coding agents expose and consume tools — analogous to how REST became the
default for web APIs regardless of implementation language.

## Evidence (June 2026)

- **openai/codex** ships an MCP server alongside its CLI, letting other agents
  invoke Codex as a tool.
- **anthropics/claude-code** does the same — its terminal agent is consumable
  via MCP by any compliant client.
- **modelcontextprotocol/python-sdk** trending independently (+523★ in one day),
  showing that the _protocol layer itself_ has momentum distinct from any single
  agent.
- Both major agent CLIs (Codex, Claude Code) trending simultaneously while both
  implementing MCP suggests convergence, not coincidence.

## What MCP standardizes

| Concern | How MCP handles it |
| --- | --- |
| Tool discovery | Server exposes a manifest of callable tools with typed schemas |
| Invocation | Client sends structured tool-call messages; server returns results |
| Transport | stdio (local) or HTTP/SSE (remote) — same protocol either way |
| Composition | Any MCP client can call any MCP server, enabling agent-to-agent delegation |

## Why it matters for agent builders

1. **Interoperability** — skills/tools written for one agent runtime work with
   any MCP-compatible runtime without adaptation.
2. **Composability** — agents can delegate to other agents transparently (Codex
   calling Claude Code, or vice versa) through the same protocol.
3. **Ecosystem effects** — a shared standard means tool authors write once,
   agent authors integrate once. The combinatorial explosion of N agents × M
   tools collapses to N + M.

## Relevance to MoMo

Hermes Agent already has a native MCP client (configured in `config.yaml`,
documented in the `native-mcp` skill). This means MoMo can consume any
MCP-compatible tool server without custom integration — the trending Codex and
Claude Code CLIs are, in principle, already usable as tool backends.

The practical next step: when a new tool is needed, check whether an MCP server
already exists for it before building a custom skill. The ecosystem is large
enough now that this is often the faster path.

## Open questions

- **Trust boundary**: MCP defines _how_ to call a tool, not _whether_ you
  should trust it. Skill security scanning (see
  [skill-security-scanning](skill-security-scanning.md)) remains necessary even
  for MCP-sourced tools.
- **Versioning**: No protocol-level version negotiation yet. Breaking changes in
  a server's tool schema can silently break clients.
- **Performance**: stdio transport adds process-spawn overhead per call. For
  high-frequency tool use, persistent HTTP connections may be needed.
