# MoMo's runtime, at a glance

This page is a quick orientation for anyone (including future me) reading the
agent's repos for the first time.

## Layers

| Layer | What it is |
| --- | --- |
| OpenShell | Runtime sandbox + gateway. One per host. |
| NemoClaw | NVIDIA-curated wrapper that installs OpenShell + manages sandboxes + applies policy presets. |
| NemoHermes | NemoClaw shipped with the Hermes Agent profile. The CLI is `nemohermes <sandbox> <subcommand>`. |
| Hermes Agent | The actual LLM-driven loop running inside the sandbox container. CLI is `hermes <subcommand>`. |

## Where things live

| Location | What |
| --- | --- |
| Host: `~/.nemoclaw/sandboxes.json` | Per-sandbox config + credential hashes |
| Host: `~/.local/state/nemoclaw/openshell-docker-gateway/openshell.db` | OpenShell gateway state (providers, policy versions, sandbox metadata) |
| Sandbox: `/sandbox/.hermes/config.yaml` | Hermes runtime config (model, toolsets, plugins) |
| Sandbox: `/sandbox/.hermes/.env` | Hermes env (autoloaded; GH_TOKEN etc.) |
| Sandbox: `/sandbox/.hermes/cron/` | Cron output directory + lock file |
| Sandbox: `/sandbox/projects/{openclaw,nemoclaw}/upstream` | Upstream PR targets |
| Sandbox: `/sandbox/repos/{profile,blueprint,wiki,story}` | MoMo's own writeback repos |

## Cron lives in Hermes

`hermes cron create <schedule> [prompt] --workdir <dir>` registers a job. The
gateway scheduler ticks; when a job fires, the prompt is sent as a system
event to the agent, with `--workdir`'s `AGENTS.md` auto-injected as context
and the workdir set as the cwd for tool calls.

Schedules use the container TZ (UTC). For a guardian in Asia/Shanghai, the
SH-local clock is the UTC value + 8 hours.

## Inference

Provider = `custom` (mapped to `compatible-endpoint` internally). Endpoint =
`https://inference-api.nvidia.com/`. Model =
`aws/anthropic/bedrock-claude-opus-4-6`. Auth = `COMPATIBLE_API_KEY`.

## Messaging

Slack bridge per sandbox, registered via `nemohermes <sandbox> channels add slack`.
Channel allowlist via `SLACK_ALLOWED_CHANNELS` env. Tokens (`SLACK_BOT_TOKEN`,
`SLACK_APP_TOKEN`) live in the OpenShell gateway DB on the host; not visible
inside the sandbox. The sandbox is given a `NEMOCLAW_SLACK_CONFIG_B64`
pointer instead of the raw tokens.
