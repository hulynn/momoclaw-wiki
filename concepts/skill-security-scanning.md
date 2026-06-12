# Skill security scanning

An emerging practice (mid-2026) of statically analyzing AI agent skills for
security vulnerabilities before they are published or loaded into a runtime.

## Why it matters

As the [agent-skill-as-package](../patterns/agent-skill-as-package.md) pattern
becomes the default distribution model, the attack surface grows: a malicious
or poorly-written skill can escalate permissions, exfiltrate data, or consume
unbounded resources — all while appearing to perform its stated function.

Security scanning treats skills as untrusted code and applies static analysis
before they enter the trust boundary.

## What scanners check

1. **Permission escalation** — skill requests access beyond its declared scope
   (e.g., writing files when it claims to only read).
2. **Data exfiltration** — network calls that send context, credentials, or
   user data to external endpoints.
3. **Unbounded resource use** — recursive delegation, infinite loops, or
   unrestricted token consumption.
4. **Malicious patterns** — known attack signatures in tool-call sequences
   (prompt injection relays, credential harvesting).
5. **Dependency risks** — pulling untrusted packages or executing arbitrary
   shell commands without sandboxing.

## Evidence

- **NVIDIA/SkillSpector** (June 2026, Python) — first dedicated security
  scanner for AI agent skill definitions. Detects vulnerabilities, malicious
  patterns, and security risks in skill files.
- The concept mirrors established practices in other ecosystems: npm audit for
  packages, GitHub's Dependabot for dependencies, Chrome Web Store review for
  extensions.

## Integration pattern

```
author writes skill
  → static security scan (SkillSpector or equivalent)
    → pass: skill enters registry / gets loaded
    → fail: flagged for human review, never auto-loaded
```

For Hermes Agent, this could gate `skill_manage(action='create')`: before a
skill is persisted, run a lightweight lint that checks for unsafe patterns
(unbounded network access, credential reads, recursive delegation) and requires
guardian approval if any trigger.

## Open questions

- How to handle skills that legitimately need broad permissions (e.g., a
  deployment skill that SSH's into servers)? Likely: explicit permission grants
  signed by the guardian, not blanket trust.
- Should scanning happen at authoring time, load time, or both? Authoring-time
  catches intent; load-time catches supply-chain tampering.
- No universal skill-security standard yet — each runtime defines its own
  threat model.

## Relevance to MoMo

MoMo's skill ecosystem (Hermes skills) would benefit from a pre-publish scan
step. The growth task identified on 2026-06-12 — integrating a lightweight
skill-security lint into the authoring workflow — is the first concrete step
toward this.
