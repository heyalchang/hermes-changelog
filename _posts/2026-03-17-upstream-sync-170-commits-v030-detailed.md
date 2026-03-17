---
layout: post
title: "Upstream Sync: 170 Commits (v0.3.0) — Detailed"
date: 2026-03-17 23:31:00 +0000
categories: upstream-sync
---

170 commits from `origin/main` (v0.3.0, tagged `v2026.3.17`).

## Agent Core

- **d417ba2a** — Route-aware pricing. New session-level accumulators, schema migration v4→v5 with 11 new columns (cache_read_tokens, cache_write_tokens, reasoning_tokens, billing_provider, estimated_cost_usd, etc.). Full rewrite of usage_pricing.py.
- **85993fbb** — Pre-call sanitization + post-call guardrails. Three new methods: _sanitize_api_messages, _cap_delegate_task_calls, _deduplicate_tool_calls.
- **96dac221** — Context overflow infinite loop fix. Heuristic detection (>40% context or >80 messages), triggers compression instead of retrying. Failed messages no longer persisted.
- **9f81c11b** — Eager fallback to backup model on 429/quota errors.
- **1264275c** — compression_attempts counter reset each iteration (was unlimited).
- **cd6dc4ef** — Message role violations in JSON recovery and error handler.
- **d1d17f4f** — Compression config to YAML-only. CONTEXT_COMPRESSION_* env vars removed.
- **8d0a96a8** — Context counter shows cached token count (was showing ~3 instead of ~18K).
- **548cedb8** — Compression consecutive same-role violation fix.

## Features

- **dd60bcbf** — OpenAI-compatible API server on port 8642. Chat Completions + Responses API.
- **e5fc9168** — Auto-generate session titles after first exchange.
- **695eb042** — .hermes.md per-repo project config discovery.
- **49043b7b** — /tools disable/enable/list slash commands with session reset.
- **4433b833** — Parallel as web search backend.
- **d2b10545** — Tavily as web search/extract/crawl backend.
- **d44b6b7f** — Cloud browser multi-provider + Browser Use integration.
- **37441183** — /model two-stage autocomplete with ghost text.
- **693f5786** — Ripgrep for file search (200x faster).
- **ce7418e2** — Interactive MCP tool configuration.
- **9ece1ce2** — Reply-to message context injection for out-of-session replies.
- **4920c594** — Auto-detect local file paths for native media delivery.
- **d0faf772** — /model shows active fallback model.
- **1314b4b5** — session:end lifecycle event.
- **d50e0711** — NeuTTS as built-in TTS provider.

## New Platforms

- **a6dcc231** — DingTalk adapter.
- **cd67f60e** — Mattermost + Matrix adapters.
- **07549c96** — SMS (Twilio) adapter.
- **dd60bcbf** — API server adapter (OpenAI-compatible).
- **ef67037f** / **fd61ae13** — SMS (Telnyx) added then reverted.

## New Providers

- **35d948b6** — Kilo Code (500+ models).
- **7042a748** — Alibaba Cloud / DashScope.
- **40e2f8d9** — OpenCode Zen + Go.

## Gateway & Platforms

- **7ac9088d** — Telegram streaming: config bridge, not-modified handling, flood control.
- **2fa33dde** — Streaming message length overflow splitting.
- **67546746** — Overwrite stale PID in gateway_state.json.
- **d87655af** — Persist watcher metadata in checkpoint for crash recovery.
- **6832d60b** — SMS persistent HTTP session + Matrix MIME media types.
- **b111f2a7** — Matrix/Mattermost never reported as connected.
- **b16186a3** — Telegram auto-detect HTML tags for parse_mode.
- **d1569424** — Telegram aggregate split text messages.
- **8e20a7e0** — Strip MEDIA: and audio_as_voice tags from message body.

## Defensive Hardening

- **efa778a0** — Thread locks on 4 SessionDB methods.
- **847ee203** — Logging, dedup, locks, dead code cleanup.
- **d81de2f3** — Memory file-lock read-modify-write.
- **70219104** — Skip corrupt transcript lines.
- **344f3771** — Compression summary role collision (partial, superseded by 548cedb8).
- **8b411b23** — Anthropic adapter: merge consecutive assistant messages with mixed content.
- **9db75fcf** — Fuzzy context length match prefers longest key.
- **b5cf0f0a** — Preserve parent agent's tool list after subagent delegation.
- **1d5a39e0** — Thread safety for concurrent subagent delegation.
- **1f0bb874** — Cron get_due_jobs race window.
- **5301c017** — Cron naive ISO timestamps timezone-aware.
- **0de75505** — Anthropic tool_choice 'none' still allowed tool calls.
- **24282dce** — Reset length_continue_retries after successful continuation.
- **8cd4a966** — Browser session creation race condition.

## Auth / Security

- **d9d937b7** — Detect Claude Code version dynamically for OAuth user-agent.
- **b6a51c95** / **e9f1a8e3** / **1c61ab6b** — ANTHROPIC_TOKEN migration trilogy (unconditional clear on v8→v9).
- **634c1f67** — Sanitize corrupted .env files.
- **e2e53d49** — Recognize Claude Code OAuth in startup gate.
- **30c417fe** / **6fc76ef9** — Website blocklist enforcement.
- **2c7c30be** — Terminal safety + sandbox file writes.
- **6a320e8b** — Block sandbox backend creds from subprocess env.

## Routine

- **37862f74** — v0.3.0 release.
- **d9b9987a** / **ba728f3e** / **1ae1e361** — Docs updates.
- **c881209b** — Revert skin-aware theme.
- **6d1c5d44** — Refactor fuzzy_match position logic.
- **766f4aae** — Tie api_mode to provider config (remove HERMES_API_MODE env var).
- **6405d389** — Test alignment.
- Various test fixes, mock alignment, doc commits.
