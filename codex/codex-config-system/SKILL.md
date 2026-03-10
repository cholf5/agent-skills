---
name: codex-config-system
description: Use when explaining, comparing, editing, or troubleshooting Codex configuration, including config.toml layers, precedence, profiles, trusted-project behavior, approvals, sandboxing, providers, features, MCP, OTel, notifications, history, project-root detection, or one-off --config overrides.
---

# Codex Config System

## Overview

Explain how Codex resolves configuration, choose the correct config layer to change, make the smallest safe edit, and verify that the intended value will actually win.

## Workflow

1. Classify the request.
   - For "why is this value applied?", start with `references/precedence-and-layers.md`.
   - For "how do I set X?", start with `references/common-recipes.md`.
   - For "what keys control X?", start with `references/config-keys.md`.
   - For "this still does not work", start with `references/troubleshooting.md`.
2. Identify the active scope before proposing edits.
   - Check whether the user wants a one-off CLI override, a profile value, a project setting, or a user default.
   - Prefer the narrowest scope that solves the request.
3. Account for trust and precedence.
   - Remember that untrusted projects ignore project `.codex/config.toml`.
   - Remember that CLI flags and `--config` overrides beat profile, project, user, system, and built-in defaults.
4. Edit conservatively.
   - Change only the relevant key.
   - Keep examples minimal and valid TOML.
   - Mention when a setting is experimental, deprecated, or platform-specific.
5. Verify the outcome.
   - State which layer should win after the change.
   - Call out cases where admin-enforced `requirements.toml` can still block the requested value.

## Safe Edit Rules

- Prefer user config at `~/.codex/config.toml` for personal defaults.
- Prefer project `.codex/config.toml` only for repo-scoped behavior that teammates should share.
- Prefer `--config` or a dedicated CLI flag for one-off runs.
- Keep profile values focused on the settings that differ from the top level.
- Resolve relative paths in project config relative to the `.codex/` directory that contains the file.
- If the user is changing approvals or sandboxing, explicitly mention security impact and managed-policy constraints.

## Common Pitfalls

- Assuming a project config applies when the project is untrusted.
- Editing user config when a closer project config or active profile will override it.
- Forgetting that `--config` values are parsed as TOML, not JSON.
- Forgetting that some legacy feature flags map to newer top-level settings.
- Expecting protected paths like `.git/` or `.codex/` to become writable just because workspace-write mode is enabled.

## Reference Map

- Read `references/precedence-and-layers.md` for load order, trust gating, project root detection, and path resolution.
- Read `references/config-keys.md` for the main settings and feature flags users change most often.
- Read `references/common-recipes.md` for short, copyable examples.
- Read `references/troubleshooting.md` for precedence conflicts, profile surprises, trusted-project issues, and managed-policy blockers.
