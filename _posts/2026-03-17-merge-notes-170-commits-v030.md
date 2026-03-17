---
layout: post
title: "Merge Notes: 170 Commits (v0.3.0) — Pricing Supersedes Our Fix"
date: 2026-03-17 23:45:00 +0000
categories: merge-notes
---

170 commits merged into `local/dashboard-and-tracking`. v0.3.0 release. Two conflicts, both in areas we instrument.

## Conflicts

**run_agent.py (conflict 1, line 870)** — our payload breakdown fields (`_last_payload_breakdown`, `_tiktoken_enc`) vs upstream's new session token accumulators (`session_cache_read_tokens`, `session_estimated_cost_usd`, etc.). Resolution: keep both. They're additive — our fields serve the payload viz tab, upstream's serve the pricing system.

**run_agent.py (conflict 2, line 5637)** — our `_total_input` fix (writing true Anthropic total to `update_token_counts`) vs upstream's `canonical_usage` pricing system. Resolution: take upstream. Their `canonical_usage.input_tokens` does the same normalization we did — includes cache read + cache creation tokens for Anthropic, raw prompt_tokens for OpenAI. Our fix is superseded. The `canonical_usage` approach is cleaner because it normalizes once at extraction time instead of recomputing at write time.

**send_message_tool.py (conflict 3, line 328)** — our WhatsApp routing addition vs upstream's message chunking refactor. Resolution: take upstream. Their version includes WhatsApp routing (same as ours) plus message chunking for all platforms plus SMS support.

## What Happened to Our Anthropic Fix

The `_total_input` computation we added in the 83-commit merge is now redundant. Upstream's `normalize_usage()` returns a `CanonicalUsage` object that handles the Anthropic token semantics correctly — `input_tokens` is the true total. Our cache extraction block (`cached`, `written`, `_total_input`) still exists and feeds our JSONL logging and hit_pct display, but the DB write now uses `canonical_usage` fields directly. This is a clean convergence — we identified the bug, fixed it locally, and upstream fixed it independently through a more comprehensive pricing rewrite.

## Test Results

5291 passed (up from 4717 — 574 new tests), 38 failed (all pre-existing or upstream bugs), 197 skipped. Notable new failures: 5 anthropic adapter tests fail because our real OAuth credential file isn't covered by test monkeypatches, 15 delegate tests fail due to an upstream NameError bug in `_saved_tool_names`, 3 web tools tests fail due to missing `parallel` package.

## Worth Surfacing in the Dashboard

This batch adds significant new data to the sessions table that our dashboard could use:

- **Session cost** — `estimated_cost_usd`, `cost_source`, `billing_provider`, `billing_mode` columns. Could add a cost column to Sessions tab and a cost summary to Usage tab. Data: sessions table directly. Priority: high — this is real billing data.
- **Cache tokens in DB** — `cache_read_tokens`, `cache_write_tokens` now persisted per-session. Could simplify the Usage tab to read from DB instead of JSONL. Priority: medium — cleaner data source.
- **Session titles** — auto-populated after first exchange. Already displayed in dashboard — will just start showing real titles. Priority: already done.
- **Auth method** — `billing_provider` and `billing_mode` show per-session auth (subscription vs API key). Could show in Overview tab. Priority: low.
- **New platforms** — DingTalk, Matrix, Mattermost, SMS, API server sessions will appear with their platform names. Dashboard platform filters should be updated. Priority: low.
