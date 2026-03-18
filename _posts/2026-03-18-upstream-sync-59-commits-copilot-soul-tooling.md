---
layout: post
title: "Upstream Sync: 59 Commits — Copilot Provider, SOUL.md Identity, Path-Aware Tooling"
date: 2026-03-18 07:30:00 +0000
categories: upstream-sync
---

59 commits merged from `origin/main`. GitHub Copilot arrives as a full provider, the agent identity system gets externalized, and the tool execution layer gets meaningfully smarter about concurrency.

## Highlights

- GitHub Copilot is now a first-class LLM provider with OAuth device code flow
- SOUL.md replaces the hardcoded identity string as the primary agent persona
- Model and provider are injected into the system prompt, frozen at session start
- Concurrent tool calls are now path-aware — parallel file writes to the same path are serialized
- Gateway restart uses PID-based wait with force-kill instead of a blind sleep
- OAuth flag staleness fixed — token refresh and fallback now update the auth mode flag
- Custom endpoint pricing fetched from `/models` metadata before falling back to the built-in table

## Provider Infrastructure

GitHub Copilot joins as a provider. OAuth device code flow authenticates against GitHub, then hits the Copilot API at `api.githubcopilot.com`. Classic PATs are rejected with clear guidance — only OAuth tokens, fine-grained PATs, and GitHub App tokens work. The model catalog is fetched live from the Copilot endpoint. Provider aliases (`github`, `github-copilot`, `github-models`) all resolve to `copilot`. A second entry `copilot-acp` handles Copilot ACP shim endpoints. The API mode reuses chat completions, so no new response parsing was needed.

Custom endpoints can now self-describe their pricing and context limits. A new metadata fetch hits the endpoint's `/models` route, caches for 5 minutes, and provides pricing data before falling back to the built-in pricing table. The API key is threaded through the cost estimation path to support authenticated catalog fetches.

Model catalog additions: GPT-5.4-mini, GPT-5.4-nano, Healer-Alpha on OpenRouter. MiniMax default upgraded to M2.7. Haiku 4.5 added to OpenRouter.

## Agent Core

The agent identity system is now externalized. SOUL.md at `~/.hermes/SOUL.md` becomes slot 1 of the system prompt, replacing a hardcoded default string. If the file is missing or empty, the hardcoded default is used — existing installs are unaffected. Model and provider are injected into the system prompt's timestamp block, frozen at session start so they don't break prompt caching.

The tool execution layer is now path-aware for concurrency. Previously, only the `clarify` tool was marked as sequential. Now tools are categorized: always-parallel (read-only tools like `read_file` and `web_search`), path-scoped (file mutation tools that parallelize only when targeting different paths), and always-sequential. Two concurrent `write_file` calls to the same path are serialized rather than raced.

The delegate tool NameError that shipped with v0.3.0 is fixed. The refactor had split setup and execution into separate functions, leaving a variable referenced in the wrong scope's `finally` block.

OAuth token state management fixed: the internal OAuth mode flag wasn't updated when the token changed through refresh or fallback activation. Both paths now re-check the token type. Same commit fixes memory nudge accumulation — the turn counter was being reset on each conversation start, preventing it from ever reaching the threshold.

MCP servers are now independently addressable as toolsets. Each server name gets its own entry, making raw names like `github` usable in gateway platform toolset config.

## Gateway & Platforms

Gateway restart is PID-aware. The `--replace` path polls the PID file every 300ms instead of sleeping 2 seconds. After 5 seconds without clean exit, it sends SIGKILL to the specific process. Script-style gateway processes (launched via `python` rather than the installed console script) are now detected by the process scanner.

Unauthorized DMs can be silently ignored with a new config option (`unauthorized_dm_behavior: "ignore"`). Per-platform override available. Default behavior unchanged.

WhatsApp `reply_prefix` config bridging was dead code that never reached the adapter — now wired through. Matrix reply parameter name corrected. Cron one-shot jobs that missed their window by up to 120 seconds are auto-recovered. Session search now searches all sources by default.

## Skills & CLI

New bundled skill: `huggingface-hub`. New `/statusbar` command toggles the CLI context bar. The setup skill expanded for onboarding flows. Config model default now respected for the openai-codex provider.

## Routine

Banner label normalization, NeuTTS provider docs, MCP install commands corrected to use `uv`, SOUL.md documentation.
