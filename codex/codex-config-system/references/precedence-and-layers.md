# Precedence And Layers

## Precedence Order

Codex resolves configuration in this order, highest first:

1. CLI flags and `--config` overrides
2. Active profile values from `--profile <name>`
3. Trusted project `.codex/config.toml` files from project root down to the current working directory, with the closest file winning
4. User config at `~/.codex/config.toml`
5. System config at `/etc/codex/config.toml` on Unix when present
6. Built-in defaults

Use this order to explain why a value appears active or where to place a new default.

## Trust Gating

- Project `.codex/config.toml` files load only for trusted projects.
- If the project is untrusted, Codex ignores project-scoped `.codex/` layers and falls back to user, system, and built-in defaults.
- If a user says "my repo config is ignored", verify trust before debugging the file contents.

## Profiles

- Define profiles under `[profiles.<name>]` in `config.toml`.
- Activate them with `codex --profile <name>`.
- Use `profile = "<name>"` at the top level to make one the default.
- Treat profiles as overrides for a small subset of values, not as full duplicated configs.
- Note that profiles are experimental and not currently supported in the IDE extension.

## Project Config Files

- Codex walks upward to find the project root, then loads `.codex/config.toml` files from that root down to the current working directory.
- The closest file wins when the same key is set multiple times.
- Resolve relative paths in a project config relative to the `.codex/` directory that contains that file.

## Project Root Detection

- By default, a directory containing `.git` is the project root.
- Override this with `project_root_markers = [".git", ".hg", ".sl"]` or similar.
- Set `project_root_markers = []` to stop parent-directory discovery and treat the current directory as the project root.

## One-Off Overrides

- Prefer dedicated CLI flags like `--model` when they exist.
- Use `--config` for arbitrary keys.
- Parse `--config` values as TOML, not JSON.

Examples:

```shell
codex --model gpt-5.4
codex --config model='"gpt-5.4"'
codex --config sandbox_workspace_write.network_access=true
codex --config 'shell_environment_policy.include_only=["PATH","HOME"]'
```

## Managed Constraints

- Organization-managed machines can enforce constraints through `requirements.toml`.
- A requested value can still be rejected even if the local config is syntactically valid.
- Mention this whenever the user wants risky values like `approval_policy = "never"` or `sandbox_mode = "danger-full-access"`.
