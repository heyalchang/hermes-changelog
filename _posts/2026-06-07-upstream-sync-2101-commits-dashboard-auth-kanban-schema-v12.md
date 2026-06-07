---
layout: post
title: "Upstream sync — 2101 commits: dashboard auth, kanban orchestration, schema v12"
date: 2026-06-07 10:25:00 +0000
---

The largest batch yet — 2101 commits (2014 non-merge) landed upstream over roughly a month, taking the project from v0.14.0 to v0.16.0. The headline is that upstream built a complete, authenticated web dashboard — the exact surface this fork was created to provide. Alongside it: a full kanban orchestration system and a flagship desktop app.

## Dashboard & Authentication

The web dashboard matured from a read-only viewer into an administered, authenticated product. There is now a pluggable authentication layer with single-use websocket tickets, refresh-token rotation, hardened cookies, reverse-proxy prefix support, and a json-lines audit log. Authentication providers ship as plugins: a Nous OAuth provider, a generic self-hosted OIDC provider, and a username/password provider for setups without an identity provider. A registration command provisions a self-hosted OAuth client.

The dashboard gained a Channels page that configures every gateway messaging platform from the browser, a complete admin panel (MCP catalog with enable/disable toggles, hook creation, pairing, credentials, memory, gateway operations, system stats), always-on embedded chat (the terminal-mode flag was removed), a Skills hub browser with featured listings and a security scan, a profiles dashboard, bulk session operations, a schedule picker, and multi-profile support over a single global-remote backend.

## Kanban Orchestration

A multi-worker task board, persisted to its own SQLite database. Cards decompose into children, specify endpoints break work down, and goal-mode cards run workers in a continuous loop. Tasks carry file attachments and inherit their spawning worker's workspace. The system grew substantial reliability work: crashed-worker detection with grace periods, zombie reaping, per-call board isolation, iteration-budget exhaustion handling, and heavy SQLite corruption defense (synchronous-full writes, cell-size checks, content-addressed corrupt-DB backups, refusal to auto-init a corrupt board).

## Desktop App

The Electron desktop app became the flagship surface. A profile rail supports drag-sort reordering, per-profile colors, rename and delete, and quick-create. Keyboard work is extensive: a rebindable shortcuts panel, a command palette on Cmd+K and Cmd+P, steer-the-live-run from the composer, and arrow-key history navigation. Cron jobs became a first-class sidebar entity that can be fired from the dashboard backend. The app added internationalization (Simplified Chinese, Japanese, Traditional Chinese), a unified overlay design system, drag-and-drop of files and sessions into chat, and dedicated Providers and Accounts settings.

## State & Agent Core

Schema advanced to v12, adding a soft-delete flag to messages and a rewind counter to sessions, supporting new rewind and undo primitives. Undo backs up N user turns with prefill and soft-deletion; rewound rows are excluded from reads by default and from search unless explicitly opted in. Mid-session model switches now persist to the database. A runtime working-directory resolver became the single source of truth for the agent's current directory. Custom OpenAI-compatible providers now honor configured default headers on both the main and auxiliary clients.

## Providers & Models

Usage-aware credits arrived: in-session credit notices, a usage view, and a developer readout. MiniMax-M3 was added with a 1M-token context window, and qwen3.7-plus joined the catalogs. The model catalog now refreshes hourly. The model picker saw heavy work — fuzzy search, grouped multi-endpoint providers, and routing fixes between OpenAI and OpenRouter. Compression's compaction trigger was raised to 85% for one model family on Codex OAuth.

## Platforms

The Home Assistant, Discord, and Mattermost adapters migrated to bundled plugins. Feishu gained meeting-invitation handling; Discord gained a voice-channel mixer that overlaps acknowledgements with text-to-speech. Streaming defaults are now per-platform (on for Telegram, off for Discord) over a structured stream-event protocol. Telegram added QR onboarding. WhatsApp and WeChat gained text-debounce batching. Several adapters bounded previously-unbounded caches with LRU eviction.

## Security

A cluster of approval-bypass fixes closed paths through config-file writes, in-place edits via interpreter flags, and the code-execution tool. Path-traversal was blocked in the skill viewer and in archive extraction. Server-side request forgery checks moved off the event loop in async paths. A patched Starlette was pinned for a published CVE, and cloud-provider credentials are now stripped from subprocess environments.

## Packaging & Infrastructure

Pillow and markdown were promoted to core dependencies so rich output works out of the box. Nix dependency hashing was corrected and a router dependency bumped for a CVE. Docker hardening covered config migrations on container boot, UID remapping, Unraid uid mappings, and orphaned-container recovery. The installer now does shallow clones, and the macOS installer became a renamed launcher. Observer-grade telemetry hooks and a relay plugin were added.

## Routine

Documentation (internationalization mirrors, install instructions, security redaction), test coverage, roughly fifty release author-mapping chores, and style passes across the desktop overlays.
