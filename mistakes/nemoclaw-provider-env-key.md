# NemoClaw `NEMOCLAW_PROVIDER` does not accept the internal label

When you script a non-interactive NemoClaw onboard against the
OpenAI-compatible custom endpoint, the env var to set is:

```
NEMOCLAW_PROVIDER=custom
```

**Not** `compatible-endpoint`, even though `compatible-endpoint` is the value
you will see in:

- `~/.nemoclaw/sandboxes.json` → `sandboxes.<name>.provider`
- `~/.nemoclaw/onboard-session.json` → `provider`

If you pass `compatible-endpoint` as the env var, NemoClaw aborts at step 3/8
with:

```
Unsupported NEMOCLAW_PROVIDER: compatible-endpoint
Valid values: build, openai, anthropic, anthropicCompatible, gemini,
              hermes-provider, ollama, custom, nim-local, vllm, routed, ...
```

## Why

The internal `providerName` and the env-var key are different namespaces. The
mapping lives in `src/lib/onboard/providers.ts → REMOTE_PROVIDER_CONFIG`:

| Env value (`NEMOCLAW_PROVIDER`) | Stored `providerName` | Credential env var |
| --- | --- | --- |
| `build` | `nvidia-prod` | `NVIDIA_API_KEY` |
| `openai` | `openai-api` | `OPENAI_API_KEY` |
| `anthropic` | `anthropic-prod` | `ANTHROPIC_API_KEY` |
| `anthropicCompatible` | `compatible-anthropic-endpoint` | `COMPATIBLE_ANTHROPIC_API_KEY` |
| `gemini` | `gemini-api` | `GEMINI_API_KEY` |
| `hermesProvider` | `hermes-provider` | `OPENAI_API_KEY` |
| `custom` | `compatible-endpoint` | `COMPATIBLE_API_KEY` |

## How to figure it out next time

Read `src/lib/onboard/providers.ts` first; do not try to guess from the
interactive prompt labels.

## Related

[`nemoclaw-rebuild-requires-compatible-api-key.md`](nemoclaw-rebuild-requires-compatible-api-key.md)
covers a sibling gotcha: `nemohermes <name> rebuild` does not auto-pull the
inference key from saved credentials and requires `COMPATIBLE_API_KEY` to be
present in the environment.
