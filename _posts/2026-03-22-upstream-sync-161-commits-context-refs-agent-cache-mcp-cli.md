---
layout: post
title: "Upstream Sync: Context References, Agent Caching, MCP CLI, Compressor Overhaul"
date: 2026-03-22 06:00:00 +0000
categories: upstream-sync
---

161 commits merged from upstream — the largest single batch since the initial fork. Major features include inline context references, per-session agent caching for prompt cache hits, a full MCP server management CLI with OAuth, and a restructured context compressor.

## Agent Core & Runtime

The context compressor received a major overhaul. Summaries now follow a structured template (Goal, Progress, Key Decisions, Relevant Files, Next Steps) instead of free-form paragraphs, and re-compression feeds the previous summary back so accumulated context survives multiple compaction cycles. A free pre-pass prunes old tool result contents before the LLM call, saving roughly 30% context on tool-heavy sessions. Tail protection switches from fixed message count to a token-budget walk of roughly 20K recent tokens.

Inline context references let users type `@file:path`, `@folder:dir`, `@diff`, `@git:N`, or `@url:` directly in messages. Content is expanded and injected before reaching the LLM, with token budget warnings at 25% and hard block at 50% of context.

The gateway now caches AIAgent instances per session instead of creating a fresh one for every message. This preserves the frozen system prompt and tool schemas across turns, enabling provider-side prompt prefix caching. The cache invalidates on config changes, model switches, and session resets.

Synthetic error message injection was removed — the fake `[System error]` messages that were injected into conversation history on failures polluted context and could violate role alternation. The tool-call error path is preserved.

Post-response housekeeping (review agent, memory updates) now runs with stdout muted so users only see the main response.

Streaming is now on by default in CLI. The `/queue` command now properly processes queued messages after normal agent completion, not just after interrupts.

## Model Resolution & Provider Routing

Explicit providers without API keys now fail fast with a clear error naming the missing env var, instead of silently falling back to OpenRouter. The Anthropic token leak to third-party providers was fixed — OAuth credentials no longer propagate to non-Anthropic endpoints.

The default compression summary model was changed from a hardcoded `gemini-3-flash-preview` (which routed through OpenRouter for everyone) to empty string, falling back to the standard model resolution chain.

Cache control markers on `role:tool` messages are now skipped for OpenRouter, preventing silent hangs on the second tool call.

The auxiliary client now checks JWT expiry on Codex tokens and correctly propagates the OAuth flag to the Anthropic adapter.

## Platforms & Delivery

Mattermost gained an @-mention-only filter for channels — DMs are always processed, but channel messages require the bot to be mentioned. Discord got a persistent typing indicator that loops every 8 seconds during long responses. Telegram 409 polling conflicts now retry up to 3 times before giving up, handling transient conflicts during `--replace` handoff.

Cron job delivery to Telegram topics is now supported via `telegram:chat_id:thread_id` format. Cron outputs are no longer mirrored into gateway session history, preventing consecutive-assistant-message violations.

Media files referenced in streaming responses are now delivered as attachments after the stream finishes, instead of being silently dropped.

## MCP & Plugins

A full MCP server management CLI landed: `hermes mcp add/remove/list/test/configure`. The `add` flow connects to the server, discovers tools, and presents a selection interface. Includes OAuth 2.1 PKCE auth for HTTP servers with token caching and auto-refresh. Config values support `${ENV_VAR}` interpolation.

Plugin slash command registration allows plugins to add commands that appear in `/help`, tab autocomplete, Telegram bot menus, and Slack subcommands. The plugin CLI gained `hermes plugins install/remove/list` subcommands for git-based plugin management.

Project-local plugins now require explicit opt-in via `HERMES_ENABLE_PROJECT_PLUGINS=1` — they're no longer auto-discovered from `./.hermes/plugins/`.

## API Server & Security

The Responses API endpoint now persists conversation state to SQLite at `~/.hermes/response_store.db` instead of in-memory, surviving gateway restarts. Full CRUD endpoints for cron jobs were added at `/api/jobs` with input validation, field whitelisting, and 32 tests. CORS enforcement was added — `API_SERVER_CORS_ORIGINS` replaces the previous wildcard `Access-Control-Allow-Origin: *`.

## Context Files & Configuration

Context file loading now uses priority-based selection: `.hermes.md` takes priority over `AGENTS.md` over `CLAUDE.md` (newly recognized) over `.cursorrules`. Only one loads per project.

Honcho configuration now supports instance-local config at `$HERMES_HOME/honcho.json` and defaults to per-directory session strategy.

Vision analysis timeout is now configurable via `auxiliary.vision.timeout` in config.yaml.

## Infrastructure

Systemd restart storm prevention with rate limiting. Stale lock detection for stopped (Ctrl+Z) processes during `--replace`. Protected TUI extension hooks for wrapper CLIs. Case-insensitive model family matching for local models.

## Routine

Session/message count thread safety, bioinformatics gateway skill, DashScope runtime mode fix, Alibaba model identity injection, WhatsApp bridge restart on child exit, installer zprofile fallback for fresh macOS.
