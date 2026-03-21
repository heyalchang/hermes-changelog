---
layout: post
title: "Upstream Sync: 57 Commits — Event Loops, Context Detection, Webhooks (Detailed)"
date: 2026-03-21 06:01:00 +0000
categories: upstream-sync
---

57 commits merged from upstream. Detailed version with commit references.

## Agent Core & Runtime

The event loop architecture was rebuilt across three commits. First, the main thread's `_run_async()` switched from `asyncio.run()` to a persistent loop via `_get_tool_loop()`, preventing "Event loop is closed" crashes from cached async clients outliving closed loops (7a427d7b). Then worker threads switched to `asyncio.run()` with fresh loops to prevent "loop already running" when parallel async tools shared the main loop (aafe86d8). Finally, worker threads got per-thread persistent loops via `threading.local()` instead of `asyncio.run()`, preventing the same GC crash in workers (ab6abc2c).

Context pressure warnings at 60% and 85% of compaction threshold — colored progress bar in CLI, plain-text in gateway. New `status_callback` parameter on `AIAgent` (c52353cf).

Post-compression file-read history injection removed — the synthetic user message was confusing models and causing task redos (4263350c).

Memory/skill review moved to background agent thread, no longer inline. 5-iteration budget, memory + skill_manage tools only (45058b41).

Streaming: `_stream_delta(None)` now flushes open boxes before tool execution (76bc2719). Spinners and tool progress render during streaming via `_executing_tools` flag (b313751a). Reasoning blocks (`<think>`, `<thinking>`, `<REASONING_SCRATCHPAD>`) route to display box live (b1832faa).

Session reset zeroes compressor counters — `last_total_tokens`, `compression_count`, `_context_probed` (5822711a).

Orphaned `tool_result` blocks stripped in Anthropic adapter. `/reset` and `/new` bypass running-agent guard (2ea4dd30).

Crash fix for `None` entries in `tool_calls` during Anthropic conversion (0ce35a11).

## Model Resolution & Provider Routing

Context length detection overhauled with 8-step resolution chain: config override → custom provider config → disk cache → local `/models` endpoint → Anthropic API → OpenRouter → Nous suffix-match → models.dev registry → family defaults → 128K fallback. New `agent/models_dev.py` with 1hr memory cache + disk cache at `~/.hermes/models_dev_cache.json`. Probe tiers shrunk to `[128K, 64K, 32K, 16K, 8K]`. Config gains `model.context_length` and per-provider `custom_providers[model].context_length` (88643a1b).

Claude 4.6 Opus and Sonnet updated from 200K to 1M in `DEFAULT_CONTEXT_LENGTHS` (3ec6c71e).

Six-bug fix: Ollama `model:tag` preservation, fuzzy matching tightened (prevents `claude-sonnet-4` → `claude-sonnet-4-6`), reasoning detection expanded, Nous queries models.dev, disk cache fallback TTL shortened, delegate tool `try/finally` on `_last_resolved_tool_names` (55ce6015).

Ollama `model:tag` colons preserved via `_strip_provider_prefix()` that only strips recognized providers (471ea81a).

`/model` in CLI (1055d435) and gateway (f3b23034) skips auto-detection for custom/local providers, shows endpoint URL.

`api.openai.com` auto-routes to `codex_responses` mode for GPT-5.x compatibility (b1d05dfe).

## Platforms & Delivery

New webhook platform adapter: HMAC validation, payload templates per route, configurable outputs (github_comment, telegram, discord, log), 30 req/min, 1hr idempotency. Env vars: `WEBHOOK_ENABLED`, `WEBHOOK_PORT`, `WEBHOOK_SECRET` (e140c02d).

WhatsApp: image download to `~/.hermes/image_cache/`, `[image received]` placeholder for captionless media, bridge binds `127.0.0.1`, reuses bridge only when `connected`, Baileys 7.x `getMessage` callback for E2EE. LID→phone resolution for allowlist checking (a5beb6d8).

Telegram: backslash/backtick escaping in code blocks (43b3a0ac), strikethrough/spoiler/blockquote support (02f639e5).

Signal: Note to Self messages processed as inbound, echo-back protection via bounded timestamp set (ec9b868a).

ACP streaming: `.strip()` removed, preserving whitespace (1173adbe).

Missing platforms in cron/send_message maps: Matrix, Mattermost, Home Assistant, DingTalk (8f6ecd5c, 4be50704).

## Cron & Tools

Cron agents stripped of messaging/clarify toolsets, prompt rewritten for autonomous execution (4494c0b0).

Disabled toolsets no longer re-enable through composite entry detection (173a5c62).

## CLI & UX

`/queue <prompt>` (alias `/q`) queues without interrupting (66a19425). True-color ANSI (f7e2ed20, 2416b2b7). Spinner suppression in non-TTY (6e2be335). Provider/endpoint in API errors (d560f2d1). Session search excludes entire active lineage (1870069f).

## Routine

Docs gap-fill (0e3b7b6a, 80e578d3, cf29cba0). Honcho reads `HONCHO_BASE_URL` (4ad00831). `dashscope-intl.aliyuncs.com` URL-to-provider mapping (59074df0).
