---
layout: post
title: "Upstream Sync: 170 Commits — v0.3.0, API Server, Pricing, Hardening"
date: 2026-03-17 23:30:00 +0000
categories: upstream-sync
---

170 commits merged — the v0.3.0 release. The biggest batch yet by a wide margin. The headline features are an OpenAI-compatible API server, route-aware cost tracking, auto-generated session titles, and a wave of defensive hardening across the agent core.

## OpenAI-Compatible API Server

Hermes now exposes itself as an OpenAI endpoint on localhost:8642. Any OpenAI-compatible frontend — Open WebUI, LobeChat, LibreChat — can connect and use Hermes as its backend. Chat Completions and Responses API both supported.

## Route-Aware Pricing

Full cost tracking lands in the sessions database: cache read/write tokens, reasoning tokens, billing provider, estimated and actual cost, cost source. The pricing engine pulls metadata from OpenRouter for accurate per-model rates. This replaces the old estimate-only approach with real data that persists across sessions.

## Auto-Generated Session Titles

After the first exchange in any session, a background thread fires the auxiliary LLM to generate a 3-7 word title. Sessions that previously showed only their ID now have human-readable names.

## Per-Repo Project Config

`.hermes.md` files are now discovered and injected into the system prompt — walk from cwd to git root, nearest-first. This is like CLAUDE.md but for Hermes sessions running inside a repository.

## Defensive Hardening

Thread locks added to four SessionDB methods that were accessing SQLite without synchronization. Memory tool now uses file locking for concurrent writes. The compression pipeline had two bugs that could produce consecutive same-role messages, causing API 400s mid-conversation — both fixed. A context overflow condition could cause an infinite retry loop — now detected and triggers compression. Message role violations in JSON recovery and error handling paths fixed.

## New Platforms

Five new gateway adapters: DingTalk, Matrix, Mattermost, SMS (Twilio), and the API server. Each comes with env var configuration and setup wizard integration.

## New Providers

Alibaba Cloud (DashScope/Qwen models), Kilo Code (500+ models via their gateway), and OpenCode Zen/Go.

## Security

Website blocklist enforcement for web and browser tools. Terminal safety hardened with expanded dangerous command detection. Sandbox containers no longer inherit API keys from the parent environment. Prompt injection detection in skills hub cache files.

## Agent Core

Compression config moves entirely to config.yaml — env vars removed. Eager fallback to backup model on rate-limit errors. Three new pre-call guardrail methods: message sanitization, delegate_task call capping, and tool call deduplication. The compression_attempts counter was being reset every loop iteration, making the cap effectively unlimited — fixed.
