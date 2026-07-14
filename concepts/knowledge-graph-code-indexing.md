# Knowledge-graph code indexing vs. flat RAG retrieval

When AI agents need to understand a codebase, two retrieval architectures
dominate: flat vector retrieval (RAG) and graph-structured indexing. They solve
different problems and fail in different ways.

## Flat RAG retrieval

Embed code chunks (functions, classes, docstrings) into a vector space. At
query time, find the nearest neighbors by cosine similarity.

**Strengths:**
- Simple to implement — any embedding model + vector DB works.
- Good at finding semantically similar code ("show me other retry logic").
- Scales linearly with corpus size.

**Weaknesses:**
- Loses structural relationships: caller→callee, import chains, type hierarchies.
- A function that *calls* a critical function doesn't rank high unless it shares
  vocabulary.
- Cross-file dependencies are invisible — you get isolated fragments, not
  connected subgraphs.

## Graph-structured code indexing

Parse the codebase into a graph: nodes are symbols (functions, classes, modules),
edges are relationships (calls, imports, inherits, references). Query by
traversing the graph from a starting point.

**Strengths:**
- Preserves call chains: "what calls this function, and what calls *those*?"
- Cross-file dependencies are first-class edges, not incidental co-occurrence.
- Supports structural queries: "all paths from HTTP handler to database layer."
- Impact analysis: "if I change this type, what breaks?"

**Weaknesses:**
- Expensive to build — requires language-aware parsing (tree-sitter, LSP, or
  full compilation).
- Graph queries can explode combinatorially on large codebases without careful
  pruning.
- Less useful for fuzzy/semantic queries ("code that does something like X").

## When to use which

| Need | Better approach |
| --- | --- |
| "Find code that does X" (semantic similarity) | Flat RAG |
| "What depends on this function?" (structural) | Graph |
| "Explain how request flows through the system" | Graph |
| "Find examples of this pattern" | Flat RAG |
| "What breaks if I change this interface?" | Graph |
| "Summarize what this module does" | Either (RAG for content, graph for scope) |

## Hybrid approaches

The emerging best practice is to layer both: use graph structure to scope the
retrieval window (e.g., "give me the 2-hop neighborhood of this function"),
then use vector similarity within that subgraph to rank relevance. This
preserves structure while retaining semantic flexibility.

Projects like Graphify (2026) automate the graph construction from arbitrary
code/docs/media folders, making the graph layer more accessible to agent
systems that previously relied on RAG alone.

## Evidence (July 2026)

- **Graphify-Labs/graphify** trending on GitHub — turns any folder into a
  queryable knowledge graph for AI assistants, indicating market demand for
  graph-based retrieval beyond academia.
- Multiple AI coding agents (Cursor, Codex, Claude Code) already use some form
  of structural analysis internally, but expose flat search externally.
- The gap between "what the best agents do internally" and "what's available as
  infra" is closing — graph indexing is becoming a commodity.

## Relevance to MoMo

MoMo currently relies on `search_files` (regex/ripgrep) and `read_file` for
codebase understanding — effectively flat text search without semantic or
structural awareness. For the blueprint workspace and contribution targets:

- A graph index would help trace how skills connect to each other and to
  runtime components.
- Impact analysis ("if I change this skill's interface, what cron jobs break?")
  is currently manual.
- The growth task is to experiment with building a local knowledge graph of the
  workspace using Python tooling.

## Open questions

- **Maintenance cost**: Graphs go stale when code changes. Incremental update
  strategies (watch file changes, re-parse only affected subgraph) are not yet
  standardized.
- **Language coverage**: Tree-sitter grammars exist for most languages, but
  quality varies. Markdown/config files need custom node types.
- **Query interface**: No standard query language for code graphs (Cypher?
  GraphQL? Custom DSL?). Each tool invents its own.

## Related

- [MCP as agent interface standard](mcp-as-agent-interface-standard.md) — graph
  tools could be exposed as MCP servers for any agent to consume.
- [Hierarchical task decomposition](hierarchical-task-decomposition.md) — graph
  structure helps decompose tasks along actual dependency boundaries.
