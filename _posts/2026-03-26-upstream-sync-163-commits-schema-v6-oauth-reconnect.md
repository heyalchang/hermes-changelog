---
layout: post
title: "Upstream sync — 163 commits: schema v6, OAuth migration, platform reconnect"
date: 2026-03-26 22:45:00 +0000
---

163 commits landed upstream since the last sync. Schema v6 is the headline — reasoning content now persists across gateway session turns.

## Schema & Sessions

The messages table gains three columns: reasoning, reasoning details, and codex reasoning items. Migration is additive-only (ALTER TABLE). Reasoning content captured during streaming now survives across gateway session turns — previously it was discarded after each response.

SQLite concurrency hardening: timeout extended to 30 seconds, INSERT OR IGNORE on session creation to handle CLI/gateway races, transcript loading now picks whichever of SQLite or JSONL has more rows (fixes a truncation bug where pre-DB sessions converged to 2–4 messages on the second turn). Sessions flagged as ended after cron jobs complete. Silent SessionDB failures now surface instead of silently losing data.

## Auth & OAuth

The Hermes-native PKCE OAuth flow is removed entirely. Token management routes through Claude Code's credential file only. OAuth token refresh migrated to platform.claude.com with console.anthropic.com fallback. Agent cache keys now use SHA256 fingerprint of the full auth token instead of the first 8 characters — prevents false cache hits when JWT tokens rotate.

## Gateway

Auto-reconnect for failed platforms with exponential backoff (30 seconds to 5-minute cap, 20 attempts max). Previously a DNS blip at startup permanently disabled a platform until manual restart. The gateway now stays up and retries in the background.

The /stop command force-unlocks hung sessions — previously required a full gateway restart. Telegram gains Private Chat Topics (forum threads as separate sessions with optional auto-skill binding) and configurable reply threading mode. Discord fixes phantom typing indicators and adds document caching. Matrix fixes duplicate messages and adds image caching for vision. WhatsApp now downloads documents, audio, and video media.

Request timeouts added to Home Assistant, Email, Mattermost, SMS adapters, and the send_message tool.

## Compression & Context

Dead config key summary_target_tokens removed. Three new config keys: target_ratio (0.40), protect_last_n (20), and threshold (0.80). Compression now fires later (80% full vs 50%) and protects more recent context. Context file discovery in gateway mode fixed to use TERMINAL_CWD — was previously loading dev context files into every gateway message, inflating input tokens by roughly 10K.

## API Server

SSE streaming fixed when the agent makes tool calls (was terminating early on a null sentinel). Idempotency-Key header support with in-memory LRU cache. 1MB body size limit. Unified OpenAI error envelope on all error responses.

## Security

SSRF protection on browser_navigate and vision/web tools. Zip-slip prevention in self-update. Input normalization before dangerous command detection. Shell injection guard on tilde-user path expansion. Context references restricted to safe workspace paths. Subagent toolsets restricted to parent's enabled set. MCP tool name collision protection. litellm, typer, and platformdirs removed from dependencies (supply chain compromise response). All dependency versions pinned with hashes.

## Provider & Config

Nous Portal model slugs aligned with OpenRouter naming. The /model command overhauled with shared switch_model() for CLI and gateway. Config.yaml now supports ${ENV_VAR} substitution — API keys can reference environment variables. get_hermes_home() consolidated into hermes_constants.py. Custom/local endpoints work without API key.

## CLI & Display

/verbose command for gateway debugging. Tool progress shown for substantive tools. Multiline paste preserved. Reasoning preview chunks buffered. Status bar token display fixed (was showing 26K instead of 260K). ANSI stripped at source before reaching the model. --source flag for third-party session isolation. Claude Code-style @ context completions.

## Routine

v0.4.0 release and changelog revision. Roughly 100 unused imports removed. mini-swe-agent fully excised. Nix flake support. Docker management skill. Godmode jailbreaking skill. Documentation corrections across 18 files. OpenClaw migration v2.
