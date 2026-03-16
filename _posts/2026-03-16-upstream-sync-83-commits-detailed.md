---
layout: post
title: "Upstream Sync: 83 Commits — Detailed with Commit Links"
date: 2026-03-16 08:01:00 -0800
---

*Detailed briefing on 83 commits from [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) with links to each referenced commit.*

## Agent Core & CLI

Plugin system added through `model_tools.py` with six lifecycle hooks (startup, shutdown, pre-tool, post-tool, pre-message, post-message) and per-plugin tool namespaces to prevent collisions. Plugins self-register at import time, consistent with the existing tool registry pattern.

Smart approvals added as a third command review mode — LLM submits flagged commands to a reviewer that returns APPROVE, DENY, or ESCALATE. ESCALATE routes to the user for decision.

`/rollback` overhauled and enabled by default: diff preview before restore, file-level selection, conversation undo (removes messages without touching files), auto-checkpoint before destructive terminal commands. Prior implementation was opt-in with coarser granularity.

Status bar pricing extracted to `usage_pricing.py`. Cost display hidden by default. Memory prioritization updated to favor user preferences over procedural knowledge. Auto-detect provider on `/model` selection. File path autocomplete in CLI.

CLI token persistence to session DB ([eb4f034](https://github.com/NousResearch/hermes-agent/commit/eb4f034)) — CLI sessions now accumulate to `state.db`, enabling `/insights` across both gateway and CLI usage.

## Persistent Shell

`PersistentShellMixin` added to keep a bash process alive between tool calls via file-based IPC. Working directory, exported variables, and shell functions persist across calls.

SSH backend enables persistence by default (0.15s poll). Local backend is opt-in (0.01s poll). Both backends implement the same mixin; poll interval is the primary difference.

## Browser

`/browser connect` attaches to Chrome via CDP on `localhost:9222`. Auto-launches Chrome with `--remote-debugging-port=9222` if no debugger is detected. Model waits for user instruction after connection rather than acting autonomously.

## Gateway & Platforms

Group session isolation keyed per-participant ([06a7d19](https://github.com/NousResearch/hermes-agent/commit/06a7d19)), configurable via `group_sessions_per_user` flag ([38b4fd3](https://github.com/NousResearch/hermes-agent/commit/38b4fd3)). Honcho session routing fixed for explicit threading vs. process-global state ([dd7921d](https://github.com/NousResearch/hermes-agent/commit/dd7921d)).

`/status` reports live token counts from in-flight agents ([7148534](https://github.com/NousResearch/hermes-agent/commit/7148534)). Gateway restart on retryable failures. Systemd service scoped to `HERMES_HOME`.

Telegram: TLS retry added, MarkdownV2 escaping fixed ([a569377](https://github.com/NousResearch/hermes-agent/commit/a569377)). Email: `skip_attachments` option. PII redaction opt-in for system prompts.

## Skills & Tools

Tool emoji centralized in registry with skin theming ([210d5ad](https://github.com/NousResearch/hermes-agent/commit/210d5ad)). Block anchor fuzzy matching improved. ACP slash command support. Blender MCP skill. MCP auto-reload on config change without restart ([5e92a4c](https://github.com/NousResearch/hermes-agent/commit/5e92a4c)). Docker CWD mount gated behind explicit flag.

## Routine

Docs (checkpoints, rollback, worktrees, persistent shell, README quick reference), tests (atomic temp cleanup, env blocklist), verbose mode full output, tirith warnings silenced, CLI curses preference, Honcho seed identity fix, file tool log noise reduced.
