---
layout: post
title: "Upstream Sync: 36 Commits — Detailed"
date: 2026-03-17 23:01:00 +0000
categories: upstream-sync
---

36 commits from `origin/main`, merged into `local/dashboard-and-tracking`.

## Auth / OAuth (5 commits)

- **63e88326** — Native PKCE OAuth flow for Claude Pro/Max subscriptions. `run_hermes_oauth_login()` opens browser to `claude.ai/oauth/authorize`, exchanges code for tokens, writes to `~/.hermes/.anthropic_oauth.json` with auto-refresh.
- **2158c44e** — OAuth identity fingerprinting. OAuth requests now send `user-agent: claude-cli/2.1.2`, `x-app: cli` headers. System prompts sanitized in OAuth mode (swap "Hermes Agent"/"Nous Research", prefix tool names with `mcp_`). Fixes 400/500 errors on OAuth-authenticated calls.
- **19c8ad3d** — User-agent added to token exchange/refresh HTTP requests. Cloudflare was 403ing Python's default UA.
- **bd3b0c71** — OAuth login URL moved to prominent bordered box for SSH users.
- **b7980625** — OAuth UX improvements for headless/SSH environments.

## Streaming (11 commits)

- **c1ac3273** — Core streaming infrastructure. `stream_delta_callback` param on `AIAgent.__init__`, all three provider paths fire callbacks per text token. Old `_streaming_api_call` (140 lines) deleted.
- **d23e9a9b** — CLI streaming display. Line-buffered rendering, response box opens on first token.
- **5479bb0e** — Gateway streaming. `GatewayStreamConsumer` with progressive `editMessageText`. `StreamingConfig` dataclass. `already_sent` flag prevents double delivery.
- **942950f5** — Live reasoning token streaming in dim bordered box above response.
- **ac739e48** — Reasoning tag suppression during streaming, fallback detection fix.
- **fc4080c5** — `<THINKING>` added to streaming tag suppression list.
- **99369b92** — Broad streaming fallback on any exception (was keyword-matched).
- **c0b88018** — Streaming disabled by default. Opt-in via `display.streaming` (CLI) and `streaming.enabled` (gateway).
- **2219695d** — 14-test streaming suite (accumulator, callbacks, fallback, reasoning, Codex).
- **25a1f186** — Flooding fix for adapters without edit support (Signal, Email).
- **d3687d3e** — Docs for planned reasoning display.

## Gateway & Platforms (3 commits)

- **d998cac3** — Anthropic 429/529 retry with exponential backoff. Error details surfaced to users. Touches `run_agent.py` (retry classification) and `gateway/run.py` (error surfacing).
- **e3f9894c** — `send_animation` metadata fix (TypeError on non-Telegram), MarkdownV2 inline code splitting fix, tirith cosign-free install.
- **8e07f9ca** — 5-bug audit: (1) gateway stream consumer task never started (critical), (2) streaming fallback double-response, (3) Codex `on_first_delta` lost, (4) CLI close-tag after-text bypassed filtering, (5) 140 lines dead code removed.

## Agent Core (2 commits)

- **46176c80** — Centralized slash command registry. Single `COMMAND_REGISTRY` in `hermes_cli/commands.py`. `resolve_command()` canonicalization replaces `elif` chains. Telegram/Slack now register previously missing commands.
- **23b9d88a** — Streaming config added to `cli-config.yaml.example`.

## Infrastructure (5 commits)

- **4768ea62** — Stale cron job skip on restart. Recurring jobs >2min past schedule fast-forwarded to next future run.
- **3576f44a** — Vercel AI Gateway provider with live model discovery, `AI_GATEWAY_API_KEY` auth.
- **3543b755** — Docker auto-mount host CWD to `/workspace`.
- **5e5c9266** — macOS dual-gateway fix (launchd `--replace`, plist auto-refresh, explicit restart in `cmd_update`).
- **60e38e82** — D-Bus auto-detection for `systemctl --user` on headless servers.

## Routine (5 commits)

- **474301ad** — `execute_code` cleanup hardening.
- **181077b7** — Honcho session line hidden when no API key.
- **6794e79b** — `/bg` alias for `/background` (superseded by registry refactor).
- **63635744** — ascii-video skill refactor.
- **ce430fed** — Installer sudo prompt clarification.
