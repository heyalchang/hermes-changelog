---
layout: post
title: "Upstream Sync: 120 Commits — Client Refactor, Vision, /plan"
date: 2026-03-15 02:00:00 -0800
---

*Briefing on 120 commits from [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent), pending merge into our local fork.*

## Highlights

- OpenAI client lifecycle refactored — per-request client creation prevents closed client reuse across retries
- Native Anthropic image handling with async fallback to auxiliary vision
- New `/plan` command for structured planning in gateway
- Session key format change — all DM platforms now include chat_id, existing sessions will appear as new
- Cron gets per-job runtime overrides (own provider, model, API key)
- System gateway service mode with systemd install
- Discord voice hardening — UDP keepalive, DAVE protocol, auto SSRC remap

## Agent Core

The biggest structural change is the OpenAI client lifecycle refactor. Each API call now creates a per-request client and closes it in `finally`, instead of sharing a single client across retries. Interrupt handling no longer closes the shared client — only the worker-local request client. This prevents a class of bugs where a closed client would be reused after retry.

Two commits add native Anthropic image content block conversion. OpenAI-format image blocks (base64, URLs) are translated to Anthropic's native format. When the native path can't handle an image, it falls back to auxiliary vision and injects a text description. This runs an async call inside `_build_api_kwargs`.

Codex Responses API arguments fix — `str()` replaced with `json.dumps()` for proper serialization. Dict tool call arguments from local/vLLM backends are now handled gracefully.

## Anthropic Auth

Managed key fallback was briefly added then removed in sequential commits. Net result: `read_claude_code_credentials()` now only checks `~/.claude/.credentials.json`, not `~/.claude.json`. The native Claude Code binary stores auth elsewhere.

`.env` loading now uses `override=True` semantics so the file wins over stale shell exports.

## Gateway & Platforms

New `/plan` command invokes a structured planning skill and saves markdown output to workspace files.

System gateway service mode — `hermes gateway install` for systemd with scope selection prompts.

Session key format changed — all DM platforms now include `chat_id` in the key, not just WhatsApp. Existing Telegram/Discord/Slack DM sessions will appear as new sessions after merge.

Active agent runs are now cancelled during gateway shutdown. Telegram photo bursts no longer interrupt running agents — photos are queued instead.

Direct endpoint overrides for auxiliary vision and delegation — can specify `base_url` and `api_key` per task.

Discord gets auto-threading on @mention. Thread context preserved for cron job delivery.

## Voice

Seven commits harden Discord voice. UDP keepalive prevents Discord from dropping voice after silence. DAVE protocol passthrough and auto SSRC remapping after bot rejoin. Clear error messages for missing voice dependencies. Diagnostic script and integration tests with real NaCl crypto and Opus codec.

STT disable properly propagated through shared transcription config.

## Security

Gateway and tool env vars blocked in subprocesses. Terminal environment blocklist expanded. Path traversal prevention in `.worktreeinclude` processing. Fork bomb regex fixed. Approval pattern keys use description to prevent collisions.

## Cron & Skills

Per-job runtime overrides — individual cron jobs can specify their own provider, model, base_url, api_key. Cron auto-delivery target resolution fixed after dotenv reload.

Clawhub skill search matching improved with exact match hardening.

Crontab binary detection made robust across platforms.

## Routine

Docs (Slack threads, Discord behavior, gateway service scopes, vision backends, custom endpoint routing, fallback providers, website diagrams), tests (retry semantics, provider env blocklist, Telegram disconnect, execute_code sandbox, Discord send mock, photo burst regressions, DM isolation, vLLM dict args, voice integration), CLI non-blocking startup update check, autostash cleanup improvements.
