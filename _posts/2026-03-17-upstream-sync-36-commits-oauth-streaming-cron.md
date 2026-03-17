---
layout: post
title: "Upstream Sync: 36 Commits — Native OAuth, Streaming, Cron Skip"
date: 2026-03-17 23:00:00 +0000
categories: upstream-sync
---

36 commits merged from upstream. The headline is that Hermes can now authenticate directly with Claude Pro/Max subscriptions through its own OAuth flow — no more relying on `claude setup-token` or stale credential files.

## Auth / OAuth

Hermes now has a native PKCE OAuth flow. It opens `claude.ai/oauth/authorize`, accepts a paste-back code, exchanges tokens, stores them in `~/.hermes/.anthropic_oauth.json`, and auto-refreshes. The critical piece is identity fingerprinting: OAuth requests now send Claude Code's user-agent and header fingerprint, which is what Anthropic's API expects for subscription-authenticated calls. Without this, OAuth tokens get 400/500 errors — which is exactly what we were seeing.

## Streaming Infrastructure

A complete streaming pipeline shipped across three stages: core delta callbacks for all providers, CLI line-buffered rendering with a response box, and gateway token delivery via progressive message editing. Reasoning tokens stream into a dim bordered box above the response. A critical race condition in the gateway stream consumer was caught and fixed in a follow-up audit. The whole feature ships disabled by default — opt-in via config.

## Cron & Gateway

Recurring cron jobs that are more than 2 minutes past their scheduled time on gateway restart are now fast-forwarded to the next future run instead of firing immediately. Rate-limit (429) and overloaded (529) errors from Anthropic now retry with exponential backoff instead of silently failing, and error messages now surface details to the user.

## Agent Core

All slash command dispatch sites — CLI, gateway, Telegram bot commands, Slack subcommands — now derive from a single centralized registry. Adding a command is a one-line change. The refactor replaces the entire `elif` chain in the CLI with a canonical-name lookup.

## Infrastructure

Docker terminal auto-mounts host CWD. The macOS dual-gateway bug from `hermes update` is fixed. D-Bus auto-detection for headless servers. Vercel AI Gateway added as a new provider with live model discovery.
