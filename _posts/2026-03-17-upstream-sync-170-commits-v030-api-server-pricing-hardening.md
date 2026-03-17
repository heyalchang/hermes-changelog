---
layout: post
title: "Upstream Sync: 170 Commits — v0.3.0, API Server, Pricing, Hardening"
date: 2026-03-17 23:30:00 +0000
categories: upstream-sync
---

170 commits — the v0.3.0 release, tagged `v2026.3.17`. The biggest batch yet by a wide margin.

## Highlights

- **v0.3.0 release** — a named release, not just a batch of commits.
- **OpenAI-compatible API server** — Hermes now exposes itself as an OpenAI endpoint on port 8642. Any OpenAI-compatible frontend (Open WebUI, LobeChat) can connect.
- **Route-aware pricing + schema migration** — full cost tracking in the sessions table with 11 new columns. Real per-model rates from OpenRouter metadata, replacing the old estimate-only approach.
- **5 new platform adapters** — DingTalk, Matrix, Mattermost, SMS (Twilio), and the API server. Plus Telnyx SMS added and reverted.
- **3 new providers** — Alibaba Cloud/DashScope, Kilo Code, OpenCode Zen/Go.
- **Auto-generated session titles** — background LLM call after first exchange. Sessions that previously showed only their ID now have human-readable names.
- **`.hermes.md` per-repo project config** — like CLAUDE.md but for Hermes. Walks from cwd to git root, nearest-first, injected into system prompt.
- **Defensive hardening wave** — thread locks on SessionDB, memory file locking, compression role violations, context overflow infinite loop fix, tool call deduplication, pre-call message sanitization.
- **Heavy `run_agent.py` changes** — 617 lines modified. Pricing rewrite, new guardrail methods, compression config moved to YAML.

## Agent Core

The biggest changes are in `run_agent.py`. The route-aware pricing rewrite adds per-session token accumulators — cache read/write tokens, reasoning tokens, estimated cost, billing provider — and replaces the old `estimate_cost_usd()` with a full pricing architecture backed by OpenRouter metadata. The schema migration (v4→v5) adds 11 new columns to the sessions table including `cache_read_tokens`, `cache_write_tokens`, `reasoning_tokens`, `billing_provider`, `estimated_cost_usd`, and `cost_source`.

Three new guardrail methods fire on every LLM call: message sanitization (drops orphaned tool results, injects stub results for orphaned calls), delegate_task call capping, and tool call deduplication. These run unconditionally now — they were previously gated on having a compressor.

A context overflow infinite loop was fixed — when Anthropic returns a generic 400 "Error" on context overflow, the agent now detects it heuristically (>40% context used or >80 messages) and triggers compression instead of retrying forever. Failed messages are no longer persisted to transcript.

Compression config moved entirely to `config.yaml` — the `CONTEXT_COMPRESSION_THRESHOLD`, `CONTEXT_COMPRESSION_ENABLED`, and `CONTEXT_COMPRESSION_MODEL` env vars are removed. New key: `compression.summary_base_url` for custom compression endpoints.

Eager fallback to backup model on rate-limit errors — on 429 or quota errors, immediately switches to the configured backup model instead of exhausting backoff retries.

The `compression_attempts` counter was being reset every outer-loop iteration, making the cap of 3 effectively unlimited. Fixed.

The cached token count in the context status bar was reading only the uncached portion, showing ~3 tokens instead of the real ~18K. Fixed — now includes cache_read + cache_creation.

## Gateway & Platforms

Auto-generated session titles fire a background thread using the auxiliary LLM after the first exchange, generating a 3-7 word title. The dashboard already reads the `title` column — it'll just start being populated.

The OpenAI-compatible API server is a new platform adapter exposing Hermes on port 8642. Chat Completions + Responses API. Any OpenAI-compatible frontend can connect. Env vars: `API_SERVER_ENABLED`, `API_SERVER_KEY`, `API_SERVER_PORT`.

Telegram streaming got three fixes: config now bridges properly into gateway runtime, "not modified" API errors are treated as no-ops, and flood control is handled with retry. Streaming message length overflow is handled by splitting at the adapter's max length.

Stale PID in `gateway_state.json` is now overwritten on restart — the same class of problem we hit earlier today with the missing PID file.

Reply-to context injection — when a user replies to a message from outside the active session, the gateway injects the quoted message as context. Session:end lifecycle event added. Auto-detect local file paths in responses for native media delivery. `/model` now shows the active fallback model instead of config default.

## Defensive Hardening

Thread locks added to 4 SessionDB methods — `search_sessions`, `clear_messages`, `delete_session`, `prune_sessions` were all accessing SQLite without the lock, causing `ProgrammingError` in multi-threaded gateway contexts.

Memory tool now uses file locking for concurrent read-modify-write. Compression consecutive same-role violations fixed — the compressor could produce `assistant → assistant` sequences that caused API 400s mid-conversation. Corrupt transcript lines are now skipped instead of crashing. Message role violations fixed in JSON recovery and error handler paths.

## Auth & Security

The ANTHROPIC_TOKEN migration trilogy unconditionally clears `ANTHROPIC_TOKEN` during v8→v9 config migration. Website blocklist enforcement — new `security.website_blocklist` config key, default disabled, fail-open. Terminal safety hardened with expanded dangerous command detection and a new `HERMES_WRITE_SAFE_ROOT` env var. Sandbox containers no longer inherit API keys from parent env. Prompt injection detection in skills hub cache files.

Claude Code version for OAuth user-agent is now detected dynamically instead of hardcoded `2.1.2`.

## Features & Skills

`.hermes.md` per-repo project config — walks from cwd to git root looking for `.hermes.md`, injects into system prompt. No collision with CLAUDE.md.

`/tools disable/enable/list` slash commands — in-session tool management with session reset. Parallel and Tavily added as web search backends. Cloud browser multi-provider with Browser Use. `/model` two-stage autocomplete with ghost text. Ripgrep for file search — 200x faster. Interactive MCP tool configuration.

New skills: Sherlock OSINT, inference.sh, Base blockchain. NeuTTS moved from optional skill to built-in provider.

## New Platforms

DingTalk, Matrix, Mattermost, SMS (Twilio), and the API server. Each with env var configuration and setup wizard integration.

## New Providers

Alibaba Cloud (DashScope/Qwen models), Kilo Code (500+ models via their gateway), and OpenCode Zen/Go.

## Routine

v0.3.0 release commit, comprehensive docs updates, DingTalk/Matrix/Mattermost setup guides, test alignment, skin-aware theme added then reverted, fuzzy_match refactor, various test mock fixes.

## Merge Risk

High risk in `run_agent.py` — 617 lines changed. The pricing rewrite overlaps with our payload instrumentation and token accounting. The compression config env var removal will need attention. Plan: keep our JSONL logging but adapt to use the new session-level token fields. Verify our `_total_input` fix survives in the modified `update_token_counts()`.

Medium risk in `hermes_state.py` — schema migration is additive but `update_token_counts()` has been heavily modified.

Low risk everywhere else — dashboard untouched, new platforms/providers are additive.
