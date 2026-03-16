---
layout: post
title: "Merge Notes: 83 Commits, Anthropic Accounting Fix, and What's Next for the Dashboard"
date: 2026-03-16 11:00:00 +0000
---

*Notes from merging 83 upstream commits into our local fork on 2026-03-16. This covers the conflict analysis, a bug we found and fixed during merge, and upstream features worth surfacing locally.*

## The Merge

One conflict out of 83 commits. `run_agent.py` — both sides added a new method in the gap between `_build_api_kwargs()` and `_build_assistant_message()`. We added `_compute_payload_breakdown()` (tiktoken estimates per payload component). Upstream added `_supports_reasoning_extra_body()` (gates reasoning extra_body to known-capable model families on OpenRouter). Pure additive — keep both, done.

Everything else auto-merged: `gateway/run.py` (five upstream additions — Honcho key threading, /status bypass, PII redact arg, approval config, retryable startup), `model_tools.py` (plugin architecture + Honcho params), `gateway/session.py` (group session isolation), `agent/insights.py` (pricing extraction to `usage_pricing.py`). `git rerere` recorded the resolution.

4626 passed, 12 failed (all pre-existing — 2 known, 10 new upstream transcription regressions confirmed on clean `origin/main`), 200 skipped.

## The Bug We Fixed

Commit [eb4f034](https://github.com/NousResearch/hermes-agent/commit/eb4f034) adds CLI token persistence — after each API call, it writes `prompt_tokens` and `completion_tokens` to the session DB so `/insights` works for CLI sessions (previously only gateway sessions persisted token counts).

The problem: for Anthropic, `prompt_tokens` (from `response.usage.input_tokens`) reports only *uncached* input tokens. The true total is `input_tokens + cache_read_input_tokens + cache_creation_input_tokens`. With heavy caching — which is typical — the DB could record 5K when the real input was 50K.

OpenAI and OpenRouter are unaffected. Their `prompt_tokens` already includes cached tokens. The discrepancy is Anthropic-specific: they chose to report `input_tokens` as the *billed* input (excluding cache hits), while OpenAI reports the *total* input and breaks out cache details separately.

We'd already fixed this exact issue in our JSONL instrumentation two days ago. We compute `_total_input` — the true total regardless of provider:

```python
_total_input = (prompt_tokens + cached + written
                if self.api_mode == "anthropic_messages"
                else prompt_tokens)
```

The fix: hoist the cache extraction and `_total_input` computation above upstream's `update_token_counts()` call, then change `input_tokens=prompt_tokens` to `input_tokens=_total_input`. Both the SQLite session totals and our JSONL log now report the correct number from the same variable. One source of truth, computed once.

This is a clean upstream PR candidate — the `_total_input` computation plus a one-line change to the DB write. Small, testable, clear rationale.

## Policy Decisions

**Group session isolation** (`group_sessions_per_user`) — upstream defaults to `true`, giving each participant their own session in group chats. We're keeping the default. Our single-user install isn't affected (DM keys are unchanged), but if we ever add group chat support, per-user isolation is the right default.

**`has_known_pricing()` semantic change** — upstream tightened the check so zero-priced models (GLM, Kimi, MiniMax) return `False` instead of `True`. The dashboard's cost column now shows "N/A" instead of "$0.00" for these models. More accurate, no action needed.

**Transcription test regressions** — 10 new failures in `test_transcription.py` and `test_transcription_tools.py`, confirmed failing on clean `origin/main`. Added to our known-failures baseline. Not our problem, but noted.

## What We Should Think About for the Dashboard

These upstream features don't break anything locally, but they create opportunities worth considering:

**Plugin system** — if users install plugins, we could surface them: installed plugins, hook activity, errors. Not urgent — the plugin ecosystem doesn't exist yet — but the infrastructure is there when it matters. A Plugins tab or a section in the Overview tab.

**Persistent shell sessions** — these keep a bash process alive across tool calls. If a persistent shell crashes or accumulates state, it could be confusing to debug remotely. A shell status indicator (alive/dead, command count, session age) would help. Probably a row in the Sessions or Overview tab rather than its own tab.

**Smart approvals** — the auxiliary LLM call goes through `tools/approval.py`, not the main agent loop. It doesn't show up in our token JSONL or dashboard usage tracking. If we care about total cost accounting, we'd need to instrument that path. Low priority — the approval call is cheap (fast model, tiny prompt) — but it's a blind spot.

**Rollback checkpoints** — now enabled by default, auto-triggered on destructive terminal commands, creating shadow git state in `~/.hermes/checkpoints/`. Not a dashboard concern, but if we ever build a session replay or audit view, checkpoint history would be a natural inclusion.

**`/status` live tokens** — upstream now extracts `session_prompt_tokens`/`session_completion_tokens` from the agent during an active run and writes them to the session store. This means our dashboard's session token counts will update mid-conversation rather than only at the end. The data just got more useful without us doing anything.
