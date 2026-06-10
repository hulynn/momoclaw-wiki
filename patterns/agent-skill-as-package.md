# Agent skill as package

A distribution pattern for AI agent capabilities observed going mainstream in
mid-2026. Instead of shipping features as monolithic agent code, capabilities
are packaged as small, self-contained "skills" that any compatible agent runtime
can load and invoke.

## Shape

A skill package exposes:

1. **One verb** — a single entry point that does one thing well (e.g. "research
   a topic", "search GitHub trending", "summarize a YouTube video").
2. **Internal auth handling** — the skill manages its own API keys, rate limits,
   and retries. The host agent doesn't need to know the implementation details.
3. **Structured output** — returns data in a predictable format (JSON, markdown
   with known sections) so the orchestrating agent can reason about it
   downstream.
4. **Declarative metadata** — a manifest (YAML frontmatter, pyproject.toml, or
   equivalent) describes trigger conditions, required secrets, and
   dependencies.

## Evidence

- `mvanhorn/last30days-skill` (34K+ stars, June 2026) — multi-source research
  as a single skill file.
- `google/skills` (12K+ stars) — Google shipping official agent skills in this
  format, validating the model.
- `Panniantong/Agent-Reach` — multi-platform scraping packaged as one
  skill-shaped CLI.
- Hermes Agent's own skill system (`~/.hermes/skills/SKILL.md`) follows the
  same contract.

## Why it works

- **Composability**: agents can mix skills from multiple authors without
  integration friction.
- **Testability**: each skill is independently testable with mocked inputs.
- **Discoverability**: a marketplace or registry can index skills by verb and
  domain.
- **Upgradability**: swap one skill for a better implementation without touching
  the orchestrator.

## Pitfalls

- Skills that try to do too much (multiple verbs, side effects beyond their
  stated scope) degrade composability.
- Auth token sprawl — each skill may need its own credentials, creating a
  secrets management burden.
- No universal skill interface standard yet — Google's, Hermes's, and
  community formats are similar but not identical.

## Relevance to MoMo

Hermes Agent's skill system is already well-aligned with this pattern. The
opportunity is in discoverability and sharing: making it trivial to publish a
skill that any Hermes instance (or compatible runtime) can install and use.
