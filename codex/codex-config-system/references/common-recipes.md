# Common Recipes

## Set A Default Model

Use user config for a personal default:

```toml
model = "gpt-5.4"
```

For one run:

```shell
codex --model gpt-5.4
```

## Change Approval Behavior

```toml
approval_policy = "on-request"
```

Call out that stricter or looser approval settings may be constrained by managed requirements.

## Change Sandbox Behavior

```toml
sandbox_mode = "workspace-write"
```

For extra workspace-write controls:

```toml
[sandbox_workspace_write]
network_access = true
```

Mention network and protected-path implications when suggesting sandbox edits.

## Create A Review Profile

```toml
model = "gpt-5-codex"
approval_policy = "on-request"

[profiles.deep-review]
model = "gpt-5-pro"
model_reasoning_effort = "high"
approval_policy = "never"
```

Run:

```shell
codex --profile deep-review
```

## Enable A Feature Flag

```toml
[features]
shell_snapshot = true
```

## Limit Command Environment Variables

```toml
[shell_environment_policy]
include_only = ["PATH", "HOME"]
```

## Configure OTel Export

```toml
[otel]
environment = "staging"
exporter = { otlp-http = {
  endpoint = "https://otel.example.com/v1/logs",
  protocol = "binary",
  headers = { "x-otlp-api-key" = "${OTLP_TOKEN}" }
}}
```

## Set Project Root Markers

```toml
project_root_markers = [".git", ".hg", ".sl"]
```

## Disable History Persistence

```toml
[history]
persistence = "none"
```

## Add An External Notification Hook

```toml
notify = ["python3", "/path/to/notify.py"]
```

Use this when the user wants desktop notifications, webhook triggers, or CI side-channel alerts on `agent-turn-complete`.
