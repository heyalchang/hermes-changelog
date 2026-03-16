---
layout: post
title: "Upstream Sync: Persistent Shell, Live Status Tokens, MCP Auto-Reload"
date: 2026-03-16 00:00:00 -0800
---

*37 commits from [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent).*

## Persistent Shell

The biggest addition. SSH and local terminal backends now maintain a persistent shell session across commands instead of spawning a new process each time. Enabled by default for SSH with longer polling intervals to account for network roundtrip. File tools now support read/write over SSH. Six commits building this out incrementally.

## Gateway

`/status` now reports live token counts from the running agent ([7148534](https://github.com/NousResearch/hermes-agent/commit/7148534)). MCP tools auto-reload when `mcp_servers` config changes — no restart needed ([5e92a4c](https://github.com/NousResearch/hermes-agent/commit/5e92a4c)). Background process watcher notifications route to Telegram forum topics ([4298c6f](https://github.com/NousResearch/hermes-agent/commit/4298c6f)).

## Agent Core

Tool emoji maps centralized into the registry with skin integration ([210d5ad](https://github.com/NousResearch/hermes-agent/commit/210d5ad)). Reasoning parameters now gated by model prefix for OpenRouter compatibility ([3f0f4a0](https://github.com/NousResearch/hermes-agent/commit/3f0f4a0)). Custom endpoint setup validates `/models` and suggests `/v1` base URLs ([25e53f3](https://github.com/NousResearch/hermes-agent/commit/25e53f3)).

## Platforms & Skills

Telegram MarkdownV2 chunk escaping fixed ([a569377](https://github.com/NousResearch/hermes-agent/commit/a569377)). Local STT fallback restored for gateway voice notes ([1f72ce7](https://github.com/NousResearch/hermes-agent/commit/1f72ce7)). Honcho seed identity corrected ([4e91b02](https://github.com/NousResearch/hermes-agent/commit/4e91b02)). New OSS Security Forensics skill ([c30505d](https://github.com/NousResearch/hermes-agent/commit/c30505d)). SSH preflight check added ([ceb970c](https://github.com/NousResearch/hermes-agent/commit/ceb970c)).

## Routine

Docs (checkpoints, rollback, worktrees, persistent shell config, README quick reference), tests (atomic temp cleanup, env blocklist), verbose mode shows full output, tirith startup warnings silenced, CLI prefers curses, file tool log noise reduced.
