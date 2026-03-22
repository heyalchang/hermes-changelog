---
layout: post
title: "Upstream Sync: 161 Commits — Context Refs, Agent Cache, MCP CLI (Detailed)"
date: 2026-03-22 06:01:00 +0000
categories: upstream-sync
---

161 commits merged from upstream. Detailed version with commit references.

## Agent Core & Runtime

Context compressor overhauled: structured summary template, re-compression with previous summary fed back, token-budget tail protection (~20K), free pre-pass pruning tool results, 20% context budget for summaries (e75f5842).

Inline `@` context references: `@file:`, `@folder:`, `@diff`, `@staged`, `@git:N`, `@url:` — content expanded into messages with 25%/50% budget guard. New file: `agent/context_references.py` (da44c196).

Gateway caches AIAgent per session for prompt prefix caching. Cache invalidates on config/model/toolset changes. Per-message state re-injected on reuse. New test file (342096b4).

Synthetic error message injection removed — fake `[System error]` messages no longer pollute history. Session resume after iteration limit fixed (779619f7).

Post-response housekeeping stdout muted via `_mute_post_response` flag (885f88fb). Review agent emits compact summary of successful tool actions (96a5e9fc).

Streaming default on in CLI (c4e787d4). `/queue` processes messages after normal completion, not just interrupts. Bundled DashScope auto-detect fix (ff071fc7).

Empty assistant message stripping added (5407d12b) then reverted (34be3f8b) — `_empty_content_retries` mechanism sufficient.

Session/message count thread lock added to `hermes_state.py` (2de42ba6). `/stop` crash and streaming media `UnboundLocalError` fixed (f69c47d9).

## Model Resolution & Provider Routing

Explicit provider without API key now fails fast with clear error, not silent OpenRouter fallback (306e67f3). Anthropic token no longer leaks to third-party `anthropic_messages` providers (525caadd).

Default compression model changed from hardcoded `gemini-3-flash-preview` to empty string (0698ddb4). Cache control on `role:tool` skipped for OpenRouter path to prevent hangs (bd49bce2).

Auxiliary client: JWT expiry check on Codex tokens, OAuth flag propagation fix (dbc25a38). Case-insensitive model family matching for local models + compressor init logging (292d12be).

## Platforms & Delivery

Mattermost @-mention-only filter for channels (fbbe9e60). Discord persistent typing indicator — 8s loop during responses (ab3cbfc9). Telegram 409 polling retry — 3 attempts with 10s delay (488a30e8).

Cron Telegram topic delivery via `platform:chat_id:thread_id` format (89befcaf). Cron outputs no longer mirrored into gateway session history (37a99794). Streaming media files delivered as attachments after stream (f5890281).

WhatsApp bridge restart on child exit (8304a771). Signal attachment RPC fix (f9052d7e). Telegram MarkdownV2 bare parentheses/braces escaping (febfe1c2).

## MCP & Plugins

`hermes mcp add/remove/list/test/configure` — discovery-first install, OAuth 2.1 PKCE, `${ENV_VAR}` interpolation. New files: `hermes_cli/mcp_config.py`, `tools/mcp_oauth.py` (b7091f93).

Plugin slash command registration via `ctx.register_command()` — appears in `/help`, autocomplete, platform menus (8da410ed).

`hermes plugins install/remove/list` — git-based plugin management with security guards. New file: `hermes_cli/plugins_cmd.py` (5a9ab09b).

Project-local plugin discovery requires `HERMES_ENABLE_PROJECT_PLUGINS=1` (10d719ac). Skills guard: agent-created dangerous skills now `ask` instead of `block` (0b370f2d).

## API Server & Security

ResponseStore persists to `~/.hermes/response_store.db` (SQLite, WAL mode) (8d528e00). Full CRUD `/api/jobs` endpoints for cron (7cd9f9ed) with input limits, field whitelist, 32 tests (0f1c9701).

CORS enforcement: `API_SERVER_CORS_ORIGINS` env var replaces wildcard (e109a8b5).

## Context Files & Configuration

Priority-based context file selection: `.hermes.md` → `AGENTS.md` → `CLAUDE.md` (new) → `.cursorrules`. Only one loads (2da79b13).

Honcho instance-local config at `$HERMES_HOME/honcho.json`, default strategy per-directory (e183744c). Honcho banner hidden when not explicitly configured (f4a74d3a).

Vision timeout configurable via `auxiliary.vision.timeout` (6435d69a). Minimax-m2.7 added to model catalogs (0510ee05).

## Infrastructure

Systemd restart storm prevention: `StartLimitBurst=5`, `RestartSec=30` (326b146d). Stale lock detection for stopped processes during `--replace` (4bded44b).

Protected TUI extension hooks: `_get_extra_tui_widgets`, `_register_extra_tui_keybindings`, `_build_tui_layout_children` (d70e07fc). Shift+Enter keybindings for Ghostty/WezTerm added (356122e9) then reverted (29520df4).

Streaming deferred linebreak to prevent blank line stacking (f9c2ad48, 669c60a6). Spinner animation suppressed during streaming tool execution (b313751a, already in prior merge).

## Routine

Docs: Honcho self-hosted section (ebd0291e), OCR skill examples (f8423052), Discord voice intent (fb6d4123), Signal Note to Self guide (cf29cba0). Opencode-Go config corruption fix (e0ca46cd). Firecrawl error message improvement (c42a18e9). PortAudio missing library error (e8048913). Logger replaces print in rl_training_tool (027fc1a8). Disk warning debug level (0ea7d0ec). Redact safely handles non-strings (40c9a134). JSON parse error returned to model (aefcdd6f). DingTalk stream optional dep (a9f9c60e). Python 3.12 test compat (189214a6). Git pull --ff-only in update (870ebb88).
