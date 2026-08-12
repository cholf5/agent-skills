# Troubleshooting

## A Setting Is Ignored

Check in this order:

1. Is there a CLI flag or `--config` override winning?
2. Is an active profile overriding the top-level value?
3. Is a closer trusted project `.codex/config.toml` overriding a broader one?
4. Is the project untrusted, causing project config to be skipped?
5. Is a managed `requirements.toml` disallowing the requested value?

## The Project Config Does Nothing

- Verify the project is trusted.
- Verify the file path is exactly `.codex/config.toml`.
- Verify the user is running Codex from a directory inside that project tree.
- Verify a nearer `.codex/config.toml` is not overriding the same key.

## `--config` Does Not Behave As Expected

- Remember that values are parsed as TOML.
- Quote strings explicitly when needed.
- Use dot notation for nested keys.

Examples:

```shell
codex --config model='"gpt-5.4"'
codex --config mcp_servers.context7.enabled=false
```

## Profiles Cause Confusion

- Check whether `profile = "<name>"` is set at the top level.
- Check whether the CLI also passes `--profile`.
- Remember that profile values override top-level config values.
- Remember that profiles are not currently supported in the IDE extension.

## Sandbox Or Approval Changes Still Fail

- Managed requirements may block unsafe values.
- Workspace-write does not guarantee write access to protected `.git/` or `.codex/` paths.
- Network access may still be disabled inside the selected sandbox mode unless explicitly enabled.

## Web Search Behavior Looks Different Than Expected

- Top-level `web_search` is the current control.
- Cached mode uses OpenAI-maintained indexed results instead of live fetches.
- Full-access runs such as `--yolo` can default to live search behavior.
- Legacy feature flags may still appear in configs, but they are not the preferred control surface.

## Relative Paths Break

- In project config, resolve relative paths from the `.codex/` directory containing the file.
- In user config, resolve paths according to the normal process working directory rules for the consuming feature.

## OTel Or Metrics Expectations Do Not Match

- `otel.exporter = "none"` records events locally but sends nothing.
- OTel export and anonymous analytics are separate systems.
- Disabling `[analytics].enabled` does not configure or disable `[otel]`, and vice versa.
