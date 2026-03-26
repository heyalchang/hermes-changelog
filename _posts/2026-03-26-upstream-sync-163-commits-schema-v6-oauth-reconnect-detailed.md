---
layout: post
title: "Upstream sync — 163 commits (detailed): schema v6, OAuth migration, platform reconnect"
date: 2026-03-26 22:46:00 +0000
---

163 commits landed upstream since the last sync. Detailed version with commit references.

## Schema & Sessions

Schema v6 (`42fec191`): `messages` table gains `reasoning TEXT`, `reasoning_details TEXT`, `codex_reasoning_items TEXT`. Migration is additive-only (ALTER TABLE, try/except for existing columns). `append_message()` gains three new optional kwargs. `get_messages_as_conversation()` SELECT now fetches and deserializes these columns on load.

SQLite concurrency hardening (`b81d49dc`): timeout 10s→30s, INSERT OR IGNORE on `create_session()` to handle CLI/gateway races, `_session_db` no longer nulled on creation failure, new `ensure_session()` helper satisfies FK before message append. `load_transcript()` now reads both SQLite and JSONL, returns whichever has more rows — fixes truncation bug where pre-DB sessions converged to 2–4 messages.

Cron sessions marked as ended after job completes (`650b400c`). Silent SessionDB failures surfaced (`52c5e491`). Session notify on auto-reset (`cd2280d1`). `/clear` and `/new` clear compressor summary (`b374f520`).

## Auth & OAuth

PKCE OAuth removed (`910ec7eb`): `run_hermes_oauth_login()`, `refresh_hermes_oauth_token()`, `read_hermes_oauth_credentials()` deleted from `anthropic_adapter.py` (−219 lines). `~/.hermes/.anthropic_oauth.json` no longer read — file silently ignored if present.

Token refresh migrated to `platform.claude.com` with fallback to `console.anthropic.com` (`2c719f07`). Content-Type changed from form-urlencoded to JSON. Per-endpoint failure logging with fallthrough.

Agent cache key uses SHA256 fingerprint of full auth token (`c6fe75e9`), replacing `api_key[:8]` prefix. Prevents false cache hits on JWT rotation.

## Gateway

Auto-reconnect (`3b509da5`): `_failed_platforms` dict on GatewayRunner, `_platform_reconnect_watcher` background task, exponential backoff 30s→60s→120s→240s→300s cap, max 20 attempts. Non-retryable errors (bad auth) skip retry. Gateway no longer shuts down when adapters=0 but reconnects pending.

Force-unlock hung sessions (`59575d6a`): `/stop` intercept in running-agent guard. Force-removes agent from `_running_agents` and unlocks session. 10-minute hard timeout on executor removed.

Telegram: self-reschedule reconnect on polling failure (`6610c377`), Private Chat Topics with skill binding (`36af1f3b`, +291 lines), configurable reply threading (`e5691eed`), auto-reconnect after network interruption (`2bd8e5cb`). Discord: phantom typing fixed (`65dace1b`), document caching + text-file injection (`afe2f0ab`). Matrix: duplicate messages + image caching (`5e5ad634`). Slack: progress to correct thread (`e0cfc089`). WhatsApp: document/audio/video download (`c6f4515f`).

Request timeouts: HA, Email, Mattermost, SMS adapters (`3a863288`), send_message_tool (`9989e579`). Background task refs tracked (`e9e7fb06`, `243ee675`). Virtualenv path detection (`ce39f9cc`). All missing platform allowlist env vars warned at startup (`6302e56e`). Agents prevented from starting gateway outside systemd (`93dc5dee`).

## Compression & Context

Dead `summary_target_tokens` removed (`9231a335`). New config keys (`27c023e0`): `compression.target_ratio` (0.40), `compression.protect_last_n` (20), `compression.threshold` (0.80). Default trigger 0.50→0.80, protect 4→20, summary cap fixed→ratio-based.

TERMINAL_CWD fix (`8ee4f328`): `build_context_files_prompt()` uses TERMINAL_CWD instead of `os.getcwd()`. Gateway was loading AGENTS.md (~16K chars) into every message.

Compression restarts count toward retry limit (`9792bde3`). Context pressure warnings updated after compaction (`48191558`). Hygiene compression: broken 1.4x multiplier removed, now respects config context_length (`b799bca7`, `b2b4a9ee`).

## API Server

Streaming fix for tool calls (`b2a6b012`): filter `None` sentinel in `_on_delta`, use `agent_task.done()` for termination. Idempotency-Key + body limit + error envelope (`80cc27eb`): LRU cache (300s TTL, 1000 entries), 1MB max, OpenAI `{"error":{...}}` format.

## Security

SSRF: `browser_navigate` (`ab548a9b`), vision_tools + web_tools (`0791efe2`), async redirect guard (`ad5f973a`). Zip-slip in self-update (`3a7907b2`). Input normalization for dangerous command detection (`76ed15dd`). Shell injection via `~user` path (`73a88a02`). `@` references restricted to workspace (`2d8fad82`). Subagent toolsets restricted to parent's set (`e5d14445`). MCP tool name collision protection (`0cfc1f88`).

Supply chain: litellm/typer/platformdirs removed (`18cbd18f`). All deps pinned with version ranges (`c9b76057`). uv.lock regenerated with hashes (`624e4a8e`). CI supply chain audit workflow (`ac5b8a47`). CVE dependency bump (`3bc953a6`).

## Provider & Config

Nous Portal slugs aligned (`a8e02c7d`). /model overhaul with shared `switch_model()` (`b641ee88`, `2e524272`). `${ENV_VAR}` in config.yaml (`4ff73fb3`). `get_hermes_home()` consolidated into `hermes_constants.py` (`77bcaba2`, 31 files). Custom endpoints without API key (`1b5fb36c`). `'custom'` provider preserved instead of silent remap to `'openrouter'` (`2f1c4fb0`). Root-level provider/base_url read from config.yaml (`f46542b6`). Config.yaml errors logged instead of swallowed (`f9c2565a`).

## Agent Core

Streaming always preferred (`c511e087`). Subagents get independent iteration budgets (`68ab37e8`). Tool tokens in preflight estimates (`43af094a`). Duplicate reasoning callback eliminated (`156b5035`). Safe non-streaming fallback restored (`94e3d9ad`). Stale SSE detection (`f7f30aaa`). Reasoning preview chunks buffered (`8f6ef042`). Graceful return on max retries (`08d3be04`). Lifecycle events surfaced for retry/fallback/compression (`c07c17f5`). GLM reasoning-only handling (`099dfca6`). Subagent auth key fix (`4b45f658`). AGENTS.md walk stops at top level (`72583117`).

## CLI & Display

/verbose gateway command (`72250b5f`). Tool progress for substantive tools (`4a56e2cd`). Multiline paste (`841401f5`). --source flag for session isolation (`db241ae6`). @ context completions (`9c32fed1`). Status bar 26K→260K fix (`0dcd6ab2`). ANSI strip at source (`934fbe3c`). TUI refresh before background output (`861624d4`). KawaiiSpinner suppression (`114e636b`, `fd292e67`). KeyboardInterrupt catches (`41ee207a`, `e4033b2b`). Non-TTY crash prevention (`41081d71`). EOFError handling (`bd43a43f`). Setup menu dispatch fix (`0d7f7396`). `isatty()` guard on closed streams (`14cf2d85`). sys.executable for pip (`432ba3b7`). Tool generation callback for streaming (`87e2626c`, `4313b8af`). HTML error cleanup (`bd6b138e`, `5b29ff50`, `712cebc4`). Activated skills line moved below welcome (`f60ebc7b`). Tilde expansion in vision paths (`b0727371`).

## Routine

v0.4.0 release (`8416bc21`). ~100 unused imports removed (`8bb1d15d`). 154 f-strings fixed (`cbf195e8`). mini-swe-agent removed (`02b38b93`, `ad1bf16f`, `177e4325`). Nix flake (`b6461903`). Docker skill (`f83c27e2`). Godmode skill (`26bfdc22`). Docs: 18 files corrected (`ee3f3e75`), 9 features documented (`ebcb81b6`), hooks unified (`ef475316`), session_search schema (`1e9ff53a`). OpenClaw migration v2 (`ab4ba816`). SOUL.md reset to baseline (`0426bb74`). Nix config drift check removed (`7126524e`). Env var passthrough for skills (`745859ba`). Skills: null metadata handling (`d218cf91`, `37cabc47`), trust preservation (`b7b3294c`), toolset resolution (`62f8aa9b`), Git Trees API install fix (`fba73a60`), skills-sh nested repo fix (`5dbe2d9d`), agent-created skills trust fix (`1b24a226`). Shell exponential backoff (`f6653517`). Media delivery spaces fix (`9d614831`). Approval YAML off (`7da08224`). Discord system messages ignored (`d35df0db`). Stale memory overwrite prevention (`48b5bc60`). File tools ANSI strip (`fa6f0695`). Session search recent mode (`e93b539a`). MCP OAuth port/path/state fixes (`ed805f57`). Browser timeout configurable (`98b55709`). Vision 402 handling (`2233f764`). Consistent test fixes (`be3eb620`). MacOS Homebrew PATH resolution (`1345e933`). Streaming box closure (`4313b8af`). Changelog revision (`6e97a3b3`). Docs: pip extras quoting (`0b993c1e`), api-server storage docs (`97183349`), model command docs (`773d3bb4`), hooks page update, missing skills/CLI commands/env vars documented.
