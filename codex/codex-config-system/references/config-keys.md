# Config Keys

## Common Top-Level Keys

- `model`: default model, for example `gpt-5.4`
- `model_provider`: named provider from `[model_providers.<name>]`
- `approval_policy`: `untrusted`, `on-request`, `never`, or a granular reject policy
- `sandbox_mode`: `workspace-write`, `danger-full-access`, and other supported modes in the active environment
- `web_search`: `"cached"`, `"live"`, or `"disabled"`
- `model_reasoning_effort`: reasoning level when supported
- `personality`: communication style such as `"friendly"`, `"pragmatic"`, or `"none"`
- `log_dir`: directory for local logs
- `project_root_markers`: files or directories that define the project root
- `hide_agent_reasoning` and `show_raw_agent_reasoning`: reasoning visibility controls
- `notify`: external command run for supported notifications
- `file_opener`: URI scheme choice for clickable file citations

## Key Tables

### `shell_environment_policy`

Use this table to control what subprocesses inherit:

```toml
[shell_environment_policy]
inherit = "none"
set = { PATH = "/usr/bin", MY_FLAG = "1" }
ignore_default_excludes = false
exclude = ["AWS_*", "AZURE_*"]
include_only = ["PATH", "HOME"]
```

### `features`

Use `[features]` for optional capabilities. Common examples:

- `collaboration_modes = true`
- `personality = true`
- `request_rule = true`
- `shell_snapshot = true`
- `multi_agent = false`
- `remote_models = false`
- `unified_exec = false`

Treat these keys carefully:

- `web_search`, `web_search_cached`, and `web_search_request` under `[features]` are legacy or deprecated toggles.
- Prefer the top-level `web_search` setting for current behavior.

### `model_providers`

Use custom providers when the user needs a different base URL, headers, or auth env var.

```toml
[model_providers.proxy]
name = "OpenAI using LLM proxy"
base_url = "http://proxy.example.com"
env_key = "OPENAI_API_KEY"
```

Useful related keys:

- `oss_provider`
- `model_catalog_json`
- provider-level retry and timeout settings
- provider headers and query params

### `sandbox_workspace_write`

Use nested keys like:

```toml
[sandbox_workspace_write]
exclude_tmpdir_env_var = false
exclude_slash_tmp = false
writable_roots = ["/Users/YOU/.pyenv/shims"]
network_access = false
```

### `windows`

For native Windows sandbox selection:

```toml
[windows]
sandbox = "elevated"
```

### `otel`

Use `[otel]` to opt into OpenTelemetry export:

```toml
[otel]
environment = "staging"
exporter = "none"
log_user_prompt = false
```

### Other Common Tables

- `[analytics]` to disable anonymous metrics with `enabled = false`
- `[feedback]` to disable `/feedback` with `enabled = false`
- `[history]` to disable transcript persistence or cap file size
- `[tui]` for notifications, alternate screen, animations, and tooltips
- `[profiles.<name>]` for named configuration overrides

## Notes

- If the user only needs to redirect the built-in OpenAI provider, prefer `OPENAI_BASE_URL` over defining a new provider.
- `model_verbosity` applies only to Responses API providers.
- Some sandboxed environments keep `.git/` and `.codex/` protected even in workspace-write mode.
