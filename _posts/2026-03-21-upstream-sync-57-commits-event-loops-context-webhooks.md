---
layout: post
title: "Upstream Sync: Event Loop Architecture, Context Detection Overhaul, Webhook Platform"
date: 2026-03-21 06:00:00 +0000
categories: upstream-sync
---

57 commits merged from upstream covering a significant event loop architecture rework, a complete rewrite of context length detection, a new webhook platform adapter, and substantial platform hardening.

## Agent Core & Runtime

The async event loop architecture was rebuilt across three commits. Worker threads now each get a persistent event loop via `threading.local()` instead of creating and closing one per invocation. This prevents "Event loop is closed" crashes from cached httpx and AsyncOpenAI clients outliving their loops, and "Event loop is already running" errors when parallel async tools hit the same loop.

Context pressure warnings now alert users at 60% and 85% of the compaction threshold — a colored progress bar in CLI, a plain-text message in gateway chats. The LLM never sees these; flags reset after compaction.

Post-compression file-read history injection was removed entirely. The synthetic user message listing previously-read files was confusing models and causing task redos. The per-file re-read blocker is sufficient.

Memory and skill review nudges moved from inline injection to a background agent thread that runs after the main response, sharing the memory store but never touching conversation history.

Streaming display now properly flushes open boxes before tool execution. Spinners and tool progress render during streaming. Reasoning blocks route to the reasoning display box live when `show_reasoning` is enabled.

Session reset now properly zeroes compressor counters. Orphaned `tool_result` blocks are stripped in the Anthropic adapter. `/reset` and `/new` bypass the running-agent guard instead of queuing as literal text.

## Model Resolution & Provider Routing

Context length detection was completely overhauled. The hardcoded table is replaced with an 8-step resolution chain: config override, custom provider config, disk cache, local `/models` endpoint, Anthropic API, OpenRouter, Nous suffix-match, models.dev registry, family defaults, then 128K fallback. A new `models_dev.py` module provides in-memory and disk caching for the models.dev registry. Probe tiers shrunk from seven levels down to five.

Claude 4.6 Opus and Sonnet context lengths updated from 200K to 1M. Ollama `model:tag` colons are no longer stripped as provider prefixes. Fuzzy matching tightened to prevent cross-model resolution errors. The `/model` command in both CLI and gateway now detects custom/local providers and skips auto-detection.

When the base URL is `api.openai.com`, Hermes auto-routes to the Responses API — GPT-5.x requires it for tool calls with reasoning.

## Platforms & Delivery

A new webhook platform adapter accepts HMAC-signed POSTs from external services, validates payloads, builds prompts from templates, and routes responses to configurable outputs. Rate-limited at 30 requests per minute with a one-hour idempotency cache.

WhatsApp gained image downloading to a local cache, Baileys 7.x compatibility for E2EE session re-establishment, LID-to-phone resolution for allowlist checking, and bridge reuse improvements. The bridge now binds to localhost only.

Telegram added proper MarkdownV2 escaping for code blocks plus strikethrough, spoiler, and blockquote support. Signal now handles Note to Self messages with echo-back protection.

Missing platforms were added to cron and send_message delivery maps: Matrix, Mattermost, Home Assistant, and DingTalk.

## Cron & Tools

Cron agents are now stripped of messaging and clarify tools entirely. The prompt was rewritten to emphasize fully autonomous execution. Disabled toolset configurations no longer silently re-enable themselves through composite entry detection.

## CLI & UX

New `/queue` command queues prompts without interrupting the running agent. True-color ANSI support added. Session search now excludes the entire active lineage. Provider and endpoint shown in API error messages.

## Routine

Documentation gap-fill from recent PRs, context length detection references in FAQ, Signal setup guide additions, Honcho now reads `HONCHO_BASE_URL` for local instances without requiring an API key.
