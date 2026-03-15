---
layout: post
title: "Upstream Sync: 120 Commits — Detailed with Commit Links"
date: 2026-03-15 02:01:00 -0800
---

*Detailed briefing on 120 commits from [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent) with links to each referenced commit.*

## Agent Core

The biggest structural change is the OpenAI client lifecycle refactor ([00c5e77](https://github.com/NousResearch/hermes-agent/commit/00c5e77)). Each API call now creates a per-request client and closes it in `finally`, instead of sharing a single client across retries. Interrupt handling no longer closes the shared client — only the worker-local request client.

Two commits add native Anthropic image content block conversion. First pass ([db362db](https://github.com/NousResearch/hermes-agent/commit/db362db)) adds structured conversion for base64, HTTPS URLs, and native Anthropic blocks. Second pass ([735a6e7](https://github.com/NousResearch/hermes-agent/commit/735a6e7)) adds async fallback to auxiliary vision when the native path can't handle an image, injecting a text description instead.

Codex Responses API arguments fix ([6f85283](https://github.com/NousResearch/hermes-agent/commit/6f85283)) — `str()` replaced with `json.dumps()` for proper serialization. Dict tool call arguments from local/vLLM backends now handled gracefully ([93a0c0c](https://github.com/NousResearch/hermes-agent/commit/93a0c0c)).

DeepSeek V3 parser updated to support multiple parallel tool calls ([26bedf9](https://github.com/NousResearch/hermes-agent/commit/26bedf9)).

## Anthropic Auth

Managed key fallback added ([db9e512](https://github.com/NousResearch/hermes-agent/commit/db9e512)) then removed ([f4e8772](https://github.com/NousResearch/hermes-agent/commit/f4e8772)) in sequential commits. Net result: `read_claude_code_credentials()` now only checks `~/.claude/.credentials.json`, not `~/.claude.json`.

`.env` loading switched to `override=True` semantics ([f24c00a](https://github.com/NousResearch/hermes-agent/commit/f24c00a)) so the file wins over stale shell exports.

## Gateway & Platforms

New `/plan` command ([ff3473a](https://github.com/NousResearch/hermes-agent/commit/ff3473a)) invokes a structured planning skill and saves markdown output to workspace ([b14a073](https://github.com/NousResearch/hermes-agent/commit/b14a073)).

System gateway service mode ([6c24d76](https://github.com/NousResearch/hermes-agent/commit/6c24d76)) with install scope prompts ([168a8e2](https://github.com/NousResearch/hermes-agent/commit/168a8e2)).

Session key format changed ([34e120b](https://github.com/NousResearch/hermes-agent/commit/34e120b)) — all DM platforms now include `chat_id` in the key, not just WhatsApp. Existing DM sessions will appear as new sessions after merge.

Active agent runs cancelled during gateway shutdown ([21c20ae](https://github.com/NousResearch/hermes-agent/commit/21c20ae)). Telegram photo bursts no longer interrupt running agents ([4ae1334](https://github.com/NousResearch/hermes-agent/commit/4ae1334)).

Direct endpoint overrides for auxiliary and delegation ([9f6bccd](https://github.com/NousResearch/hermes-agent/commit/9f6bccd)). Custom endpoint resolution restored ([53d1043](https://github.com/NousResearch/hermes-agent/commit/53d1043)).

Discord auto-threading on @mention ([23e8fdd](https://github.com/NousResearch/hermes-agent/commit/23e8fdd)). Retry without reply reference for system messages ([8ce66a0](https://github.com/NousResearch/hermes-agent/commit/8ce66a0)). Native document and video attachment support preserved ([9a177d6](https://github.com/NousResearch/hermes-agent/commit/9a177d6)).

Telegram updater/app state checked before disconnect ([0c18221](https://github.com/NousResearch/hermes-agent/commit/0c18221)). Thread context preserved for cronjob delivery ([20f381c](https://github.com/NousResearch/hermes-agent/commit/20f381c)).

Vision backend gating unified ([dc11b86](https://github.com/NousResearch/hermes-agent/commit/dc11b86)). Moonshot model selection excludes Coding Plan-only models ([2a6dbb2](https://github.com/NousResearch/hermes-agent/commit/2a6dbb2)).

## Voice

UDP keepalive prevents Discord from dropping voice after silence ([0cc7840](https://github.com/NousResearch/hermes-agent/commit/0cc7840)). DAVE protocol passthrough and auto SSRC remapping after bot rejoin ([773f3c1](https://github.com/NousResearch/hermes-agent/commit/773f3c1)). Play TTS fixed to actually play in voice channel ([f1b4d0b](https://github.com/NousResearch/hermes-agent/commit/f1b4d0b)). Clear errors for missing voice dependencies ([1cacacc](https://github.com/NousResearch/hermes-agent/commit/1cacacc)). Discord voice diagnostic script ([5f32fd8](https://github.com/NousResearch/hermes-agent/commit/5f32fd8)). STT disable propagated through shared config ([f8ceadb](https://github.com/NousResearch/hermes-agent/commit/f8ceadb), [c361360](https://github.com/NousResearch/hermes-agent/commit/c361360)).

## Security

Gateway and tool env vars blocked in subprocesses ([b177b4a](https://github.com/NousResearch/hermes-agent/commit/b177b4a)). Terminal env blocklist expanded ([9e3752d](https://github.com/NousResearch/hermes-agent/commit/9e3752d)). Path traversal prevention in `.worktreeinclude` ([12bc86d](https://github.com/NousResearch/hermes-agent/commit/12bc86d)). Fork bomb regex fixed ([e6417cb](https://github.com/NousResearch/hermes-agent/commit/e6417cb)). Approval pattern keys use description to prevent collisions ([4a93cfd](https://github.com/NousResearch/hermes-agent/commit/4a93cfd)). Legacy approval keys preserved after migration ([d5b64eb](https://github.com/NousResearch/hermes-agent/commit/d5b64eb)).

## Cron & Skills

Per-job runtime overrides ([28b3764](https://github.com/NousResearch/hermes-agent/commit/28b3764)). Cron auto-delivery target resolution fixed after dotenv reload ([0fd0eb9](https://github.com/NousResearch/hermes-agent/commit/0fd0eb9)). Clawhub search matching improved ([8ccd14a](https://github.com/NousResearch/hermes-agent/commit/8ccd14a), [df9020d](https://github.com/NousResearch/hermes-agent/commit/df9020d)). Crontab binary check hardened ([861869c](https://github.com/NousResearch/hermes-agent/commit/861869c), [f6ff663](https://github.com/NousResearch/hermes-agent/commit/f6ff663)). MCP toolsets preserved when saving platform tool config ([633488e](https://github.com/NousResearch/hermes-agent/commit/633488e)). Google OAuth PKCE persisted for headless auth ([4524cdd](https://github.com/NousResearch/hermes-agent/commit/4524cdd)).

## Routine

CLI non-blocking startup update check ([b891776](https://github.com/NousResearch/hermes-agent/commit/b891776)). Session action accepts ID prefixes ([621fd80](https://github.com/NousResearch/hermes-agent/commit/621fd80)). Execute_code sandbox adds project root to PYTHONPATH ([23bc642](https://github.com/NousResearch/hermes-agent/commit/23bc642)). Worktree include checks hardened ([f4c0128](https://github.com/NousResearch/hermes-agent/commit/f4c0128)). Autostash cleanup improved ([47c5c97](https://github.com/NousResearch/hermes-agent/commit/47c5c97), [f882dab](https://github.com/NousResearch/hermes-agent/commit/f882dab)). Docs: Slack threads ([678e0bd](https://github.com/NousResearch/hermes-agent/commit/678e0bd)), Discord behavior ([214827a](https://github.com/NousResearch/hermes-agent/commit/214827a)), gateway scopes ([95939a1](https://github.com/NousResearch/hermes-agent/commit/95939a1)), endpoint routing ([282df10](https://github.com/NousResearch/hermes-agent/commit/282df10)), fallback providers ([463239e](https://github.com/NousResearch/hermes-agent/commit/463239e)), website diagrams ([259208b](https://github.com/NousResearch/hermes-agent/commit/259208b)), Slack setup ([2119b68](https://github.com/NousResearch/hermes-agent/commit/2119b68), [fd687d0](https://github.com/NousResearch/hermes-agent/commit/fd687d0)).
