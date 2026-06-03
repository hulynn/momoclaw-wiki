# `nemohermes <sandbox> rebuild` needs `COMPATIBLE_API_KEY` in env

After a sandbox is onboarded against the `custom` (OpenAI-compatible) provider,
the inference key lives in NemoClaw's host-side credential store and the
sandbox container has it too. The CLI status output reports the sandbox is
healthy.

`nemohermes <name> rebuild` does **not** read that stored credential. Without
an env var it aborts at preflight:

```
Rebuild preflight failed: provider credential not found.
The non-interactive recreate step requires COMPATIBLE_API_KEY,
but it is not set in the environment.
```

## Workaround

Set the env explicitly before any `rebuild`:

```
export COMPATIBLE_API_KEY=<your-key>
nemohermes <sandbox> rebuild --yes
```

Or use `nemohermes credentials reset COMPATIBLE_API_KEY && nemohermes onboard`
to re-enter it interactively.

## Why this matters

Day-of operations like `channels add` queue a rebuild as their final step. If
you ran the original onboard with non-interactive env vars and rotated your
shell, `channels add slack && nemohermes <name> rebuild` will fail mid-stream
unless you re-export the inference key.

The credential is *not* lost — only the rebuild's preflight check is strict.

## Related

[`nemoclaw-provider-env-key.md`](nemoclaw-provider-env-key.md) — the
sibling-gotcha that `NEMOCLAW_PROVIDER=custom` is what maps to
the internal `compatible-endpoint` label.
