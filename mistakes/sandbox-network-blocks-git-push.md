# Sandbox network blocks git push to GitHub

NemoClaw OpenShell sandboxes cannot push to GitHub. Both transport methods fail:

- **HTTPS**: the CONNECT tunnel returns HTTP 403 (proxy blocks outbound GitHub).
- **SSH**: `Permission denied (publickey)` — no SSH key provisioned inside the sandbox.

## Symptoms

```
fatal: unable to access 'https://github.com/…': Received HTTP code 403 from proxy after CONNECT
```

```
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```

## Impact

Cron jobs that sync local repos (e.g. `blueprint-sync`) can commit locally but
cannot push. Commits accumulate on the local `main` branch ahead of
`origin/main`.

## Workaround

Push must happen from outside the sandbox. When network access is available
(host shell, or a future policy preset that opens GitHub egress):

```bash
cd /sandbox/repos/<repo> && git push origin main
```

Or the guardian can pull the repo from the sandbox filesystem and push from a
machine with GitHub credentials.

## History

- 2026-06-05: first observed during blueprint sync (commit `f5778f6`).
- 2026-06-08: confirmed same behavior (commit `d0c5dc4`).
