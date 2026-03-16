---
layout: post
title: "Upstream Sync: 83 Commits — Plugins, Persistent Shell, Browser CDP, Smart Approvals"
date: 2026-03-16 08:00:00 -0800
---

*83 commits from [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent), pending merge.*

## Highlights

- First-class plugin architecture with lifecycle hooks, tool registration, and namespace isolation
- Persistent shell — terminal keeps a bash process alive across calls; `cd` and `export` persist
- Browser CDP attach — `/browser connect` attaches to a running Chrome via DevTools Protocol
- Smart approvals — LLM-powered dangerous-command review as a third mode between always-ask and yolo
- `/rollback` overhauled — enabled by default, diff preview, file-level restore, conversation undo, auto-checkpoints

## Agent Core & CLI

The largest structural addition is a plugin system threaded through `model_tools.py`. Plugins self-register with six lifecycle hooks: startup, shutdown, pre-tool, post-tool, pre-message, and post-message. Each plugin gets its own tool namespace to prevent collisions. This is the foundation for distributing capabilities independently of the core repo.

Smart approvals introduce a third command review mode. Rather than blanket always-ask or blanket yolo, the agent submits flagged commands to an LLM reviewer that returns APPROVE, DENY, or ESCALATE. Escalate routes to the user. The prior approval system already had pattern-matching; this adds judgment where patterns are ambiguous.

`/rollback` ships enabled by default. It now shows a diff preview before restoring, supports file-level selection rather than full conversation revert, adds conversation undo (message removal without file changes), and auto-creates checkpoints before destructive terminal commands. The prior implementation was opt-in and coarser.

Status bar pricing extracted to `usage_pricing.py` and cost display hidden by default. Memory prioritization updated to favor user preferences over procedural knowledge. Auto-detect provider on `/model` selection. File path autocomplete in CLI.

## Persistent Shell

Six commits build out a persistent shell subsystem. A `PersistentShellMixin` keeps a bash process alive between tool calls using file-based IPC rather than stdin/stdout pipes. SSH and local backends both support it. State — working directory, exported variables, shell functions — survives between calls.

SSH backend enables persistence by default with a 0.15s poll interval sized for network roundtrip. Local backend is opt-in with a 0.01s poll interval. The prior behavior spawned a fresh subprocess per command; this eliminates that overhead and makes multi-step shell workflows reliable.

## Browser

`/browser connect` attaches to a running Chrome instance via CDP on `localhost:9222`. If no debugger port is detected, Chrome auto-launches with the flag set. After connecting, the model waits for an explicit instruction rather than acting immediately.

## Gateway & Platforms

Group session isolation now keys per-participant rather than per-group, configurable via `group_sessions_per_user`. Honcho session routing fixed for explicit threading versus process-global state in multi-user gateway scenarios.

`/status` reports live token counts from in-flight agents. Gateway restarts on retryable failures rather than halting. Systemd service mode scoped to `HERMES_HOME`. Telegram adds TLS retry and MarkdownV2 escaping. Email adapter gets a `skip_attachments` option. PII redaction available as opt-in for system prompts.

## Skills & Tools

Tool emoji centralized in the registry with skin theming. Block anchor fuzzy matching improved for better skill resolution. ACP slash command support. Blender MCP skill. MCP auto-reload on config change without gateway restart. Docker CWD mount gated behind an explicit flag.

## Routine

Docs (checkpoints, rollback, worktrees, persistent shell, README), tests (atomic temp cleanup, env blocklist), verbose mode full output, tirith warnings silenced, CLI curses preference, Honcho seed identity fix, file tool log noise reduced.
