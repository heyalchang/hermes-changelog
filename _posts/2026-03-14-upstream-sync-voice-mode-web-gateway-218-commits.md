---
layout: post
title: "Upstream Sync: Voice Mode, Web Gateway, and 218 Commits"
date: 2026-03-14 00:00:00 -0800
---

Hermes Agent merged 218 commits from upstream NousResearch/hermes-agent. One conflict (pyproject.toml, additive). 3644 tests passed. No impact on local extensions.

## Voice Mode (68 commits)

Full push-to-talk and TTS pipeline for CLI, Discord, Telegram, and Web UI. Three STT providers supported: Groq, ElevenLabs, and local Whisper. TTS streams sentence-by-sentence. Continuous listening with VAD silence detection. Discord voice channels can now be joined, listened to, and spoken into.

## Web Gateway

New browser-based chat UI over WebSocket. Launched via `/remote-control` command. Full platform adapter with voice support, mobile mic button, and HTTPS.

## Model Backfill PR Merged

Our PR #997 fixing NULL model values in gateway sessions was accepted upstream. The `COALESCE(model, ?)` fix is now part of mainline — no longer needed as a local patch.

## MCP Tool Filtering

Per-server include/exclude lists and capability-aware selective loading. Reduces tool noise when connecting to broad MCP servers.

## Skills Infrastructure

skills.sh hub integration, well-known URL support, update checks. New skills: X/Twitter, telephony (Twilio), Parallel CLI research.

## Agent Core

Anthropic interrupt handler restored. Atomic session log writes. Custom provider fixes. DeepSeek V3 parser fix for multi-tool calls.

## Gateway

Reasoning hot reload via `/reasoning` command. Telegram polling hardening. Quick commands from config. Restart recovery. systemd linger support for headless servers.

## ACP (Agent Communication Protocol)

Full server implementation letting editors (VS Code, Zed, JetBrains) use Hermes as a native coding agent.

## Platforms

Email: IMAP UID tracking + SMTP TLS. Send message MEDIA tag support. Discord deferred annotations.
