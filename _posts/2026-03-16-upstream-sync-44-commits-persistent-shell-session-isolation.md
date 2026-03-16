---
layout: post
title: "Upstream Sync: Persistent Shell, Session Isolation, CLI Token Tracking"
date: 2026-03-16 00:30:00 -0800
---

*44 commits from [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent), pending merge.*

## Persistent Shell

SSH and local terminal backends now maintain a persistent shell session across commands instead of spawning a new process each time. File tools work over SSH with read/write passthrough. Enabled by default for SSH with longer polling intervals for network roundtrip.

## Session Isolation

Group chat sessions are now isolated per user — each participant gets their own session context ([06a7d19](https://github.com/NousResearch/hermes-agent/commit/06a7d19)). Made configurable via `group_sessions_per_user` flag ([38b4fd3](https://github.com/NousResearch/hermes-agent/commit/38b4fd3)). Honcho session routing updated for multi-user gateway scenarios ([dd7921d](https://github.com/NousResearch/hermes-agent/commit/dd7921d)).

## CLI Token Tracking

CLI sessions now persist token counts to state.db ([eb4f034](https://github.com/NousResearch/hermes-agent/commit/eb4f034)), enabling `/insights` for CLI usage. Previously only gateway sessions accumulated DB counts.

## Agent Core

Tool emoji maps centralized into the registry with skin theming ([210d5ad](https://github.com/NousResearch/hermes-agent/commit/210d5ad)). `/status` reports live token counts from running agents ([7148534](https://github.com/NousResearch/hermes-agent/commit/7148534)). MCP tools auto-reload on config change without restart ([5e92a4c](https://github.com/NousResearch/hermes-agent/commit/5e92a4c)). Reasoning parameters gated by model prefix for OpenRouter ([3f0f4a0](https://github.com/NousResearch/hermes-agent/commit/3f0f4a0)).

## Platforms & Skills

Telegram MarkdownV2 chunk escaping fixed ([a569377](https://github.com/NousResearch/hermes-agent/commit/a569377)). Background process watcher routes to forum topics ([4298c6f](https://github.com/NousResearch/hermes-agent/commit/4298c6f)). Local STT fallback restored for voice notes ([1f72ce7](https://github.com/NousResearch/hermes-agent/commit/1f72ce7)). SSL certificate auto-detection for NixOS ([3801532](https://github.com/NousResearch/hermes-agent/commit/3801532)). New OSS Security Forensics skill ([c30505d](https://github.com/NousResearch/hermes-agent/commit/c30505d)). SSH preflight check ([ceb970c](https://github.com/NousResearch/hermes-agent/commit/ceb970c)). Custom endpoint `/models` validation ([25e53f3](https://github.com/NousResearch/hermes-agent/commit/25e53f3)).

## Routine

Docs (checkpoints, rollback, worktrees, persistent shell, README quick reference), tests (atomic temp cleanup, env blocklist), verbose mode full output, tirith warnings silenced, CLI curses preference, Honcho seed identity fix, file tool log noise reduced.
