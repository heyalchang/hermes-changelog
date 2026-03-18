---
layout: post
title: "Upstream Sync: 59 Commits — Detailed"
date: 2026-03-18 07:31:00 +0000
categories: upstream-sync-detailed
---

Detailed commit-level breakdown of the 59-commit sync. See the [clean version](/hermes-changelog/upstream-sync/2026/03/18/upstream-sync-59-commits-copilot-soul-tooling.html) for the narrative briefing.

## Provider Infrastructure

- **GitHub Copilot provider integration** (`0c392e7a`) — Two provider entries: `copilot` (hitting `api.githubcopilot.com`) and `copilot-acp`. Aliases `github`, `github-copilot`, `github-models` all resolve to `copilot`. Model catalog fetched live from the Copilot `/models` endpoint. Reuses `chat_completions` API mode. 2470+ line commit touching `run_agent.py`, `delegate_tool.py`, `hermes_cli/setup.py`.
- **Copilot OAuth device code flow** (`21c45ba0`) — New `hermes_cli/copilot_auth.py`. Device code flow against `github.com/login/device/code`, client ID `Ov23li8tweQw6odWQebz`. Classic PATs (`ghp_*`) rejected; OAuth (`gho_*`), fine-grained PATs (`github_pat_*`), GitHub App tokens (`ghu_*`) accepted. Env var search: `COPILOT_GITHUB_TOKEN > GH_TOKEN > GITHUB_TOKEN > gh auth token`.
- **Copilot API mode fix** (`36921a3e`) — Corrected API mode selection to match opencode's approach.
- **Copilot documentation** (`04101bc5`) — Comprehensive GitHub Copilot provider docs.
- **Endpoint metadata pricing** (`a2440f72`) — `fetch_endpoint_model_metadata(base_url, api_key)` hits `{base_url}/models` (tries `/v1/models`), cached 5 min per URL. `get_pricing_entry()` tries metadata before `_OFFICIAL_DOCS_PRICING`. `api_key` parameter threaded through `estimate_usage_cost()`, `has_known_pricing()`, `get_pricing()`.
- **Model catalog: GPT-5.4-mini, GPT-5.4-nano, Healer-Alpha** (`00cc0c6a`) — Added to OpenRouter catalog.
- **MiniMax M2.7 upgrade** (`e4043633`) — Default MiniMax model updated.
- **Haiku 4.5 + OpenRouter reorder** (`cb54750e`, `b05f9b62`) — Haiku 4.5 added, GLM-5-turbo reordered under glm-5, minimax under stepfun.

## Agent Core

- **SOUL.md as primary identity** (`e4a3ffa9`) — Slot 1 of system prompt now loads from `$HERMES_HOME/SOUL.md`. Falls back to `DEFAULT_AGENT_IDENTITY` when missing/empty/unreadable. `build_context_files_prompt()` gets `skip_soul=True` to prevent double-injection. SOUL.md documentation (`db4dfea7`).
- **Model/provider in system prompt** (`e99aca98`) — Appended to the timestamp block (`\nModel: {self.model}\nProvider: {self.provider}`). Gated on presence. Frozen at session start.
- **Path-aware concurrent tool batching** (`c0c14e60`) — Three categories: `_NEVER_PARALLEL_TOOLS` (always sequential), `_PARALLEL_SAFE_TOOLS` (read-only, always parallel), `_PATH_SCOPED_TOOLS` (`read_file`, `write_file`, `patch` — parallel only when paths don't overlap via `_paths_overlap()`).
- **Delegate tool NameError fix** (`ba7248c6`) — `_saved_tool_names` save moved from `_build_child_agent` to top of `_run_single_child`, same scope as `finally` restore. Fixes #1802.
- **OAuth flag staleness + memory nudge** (`5b74df2b`) — `_is_anthropic_oauth` now updated in `_try_refresh_anthropic_client_credentials()` and `_try_activate_fallback()`. `_turns_since_memory`/`_iters_since_skill` initialized in `__init__` instead of reset each `run_conversation()`.
- **MCP servers as standalone toolsets** (`4b53b89f`) — Each MCP server name gets own `TOOLSETS` entry. Built-in name collisions detected and skipped. `_sync_mcp_toolsets()` called even on no-new-servers reloads.
- **Disabled skills fully gated** (`b70dd51c`) — Filtered from `<available_skills>` system prompt section, slash commands, `skill_view()`, CLI banner.
- **Skill availability check before setup hint** (`190c0797`) — Checks skill availability before hinting at `hermes-agent-setup`.

## Gateway & Platforms

- **PID-based gateway restart** (`ace2cc62`) — `_wait_for_gateway_exit(timeout=10.0, force_after=5.0)` replaces `time.sleep(2)`. Polls `gateway.pid` every 300ms. SIGKILL after 5s. Applied to both `--replace` and `launchd_restart()`.
- **Script-style gateway detection** (`5c4c4b8b`) — `"hermes_cli/main.py gateway"` added to process scan patterns in `gateway/status.py` and `hermes_cli/gateway.py`.
- **Ignore unauthorized DMs** (`0a247a50`) — `gateway.unauthorized_dm_behavior: "pair" | "ignore"` (default `"pair"`). Per-platform override. When `"ignore"`, DMs logged but get no reply or pairing code.
- **WhatsApp reply_prefix bridging** (`2f80bd9f`) — Config YAML path was dead code, never reached adapter. Now wired through.
- **Matrix reply parameter name** (`66f71c18`) — Corrected `reply_to_message_id` parameter name.
- **Cron one-shot recovery** (`0e2714ac`) — `_recoverable_oneshot_run_at()` backfills `next_run_at` for one-shot jobs with `last_run_at` unset and `run_at` within 120 seconds of now.
- **Session search all sources** (`6fc4e366`) — Default search now includes all sources, not just the current platform.
- **STT failure user message** (`9c0f3462`) — Direct user-facing message on speech-to-text failure.
- **NeuTTS documentation** (`11f029c3`) — Provider docs and install guidance alignment.

## Skills & CLI

- **Huggingface-hub skill** (`56ca84f2`) — New bundled skill. Description refined (`947827bb`, `adf188c4`, `7e30e97a`).
- **`/statusbar` command** (`c1750bb3`) — Toggles CLI context bar visibility. `ConditionalContainer` wrapping so hidden bar takes zero space.
- **Setup skill expansion** (`764825bb`) — Expanded `hermes-agent-setup` skill and STT notes integration.
- **Config model default for codex** (`a8132d12`, `24ac5770`) — `_model_is_default` now checks `config.yaml`'s `model.default`, not just CLI arg. Two commits (original + merged PR).
- **Banner label normalization** (`f8147871`) — Toolset labels normalized, skin colors applied.
- **MCP install commands** (`a9c405fa`) — Corrected from `pip` to `uv`.
