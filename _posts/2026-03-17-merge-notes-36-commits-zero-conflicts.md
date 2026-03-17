---
layout: post
title: "Merge Notes: 36 Commits — Zero Conflicts, OAuth Fix Lands"
date: 2026-03-17 23:02:00 +0000
categories: merge-notes
---

36 commits merged from upstream into `local/dashboard-and-tracking`. Zero conflicts. All 4717 tests pass (13 pre-existing failures, 1 new upstream regression in model metadata tests).

## The Merge

Cleanest merge yet. Despite heavy changes to `run_agent.py` (streaming infrastructure, OAuth fingerprinting, 5-bug audit fix, 140 lines of dead code removal), `cli.py` (command registry rewrite), and `gateway/run.py` (streaming consumer wiring), none of it conflicted with our instrumentation. Our payload breakdown, token accounting fix, and JSONL logging all sit in different sections of the file.

## What Matters

The OAuth identity fingerprinting fix is the big one for us. We spent time earlier today trying to use OAuth tokens and getting 400 errors. Upstream fixed this by sending Claude Code's user-agent and headers when authenticating via OAuth. Now `hermes login` should give us a working Max subscription token without the `claude setup-token` workaround.

The stale cron skip is a quality-of-life fix we've wanted — recurring jobs that are past their scheduled time on gateway restart now fast-forward instead of firing immediately and burning tokens.

## Local Extension Audit

No impact on local extensions. All instrumentation insertion points survived:
- `_compute_payload_breakdown()` intact
- `_total_input` and `update_token_counts()` fix intact
- JSONL token logging intact
- Dashboard queries and data layer untouched

## Worth Surfacing in the Dashboard

From this batch, several features could enhance the dashboard:

- **Auth method indicator** — Show whether gateway is using API key, OAuth, or subscription billing. Data: `get_anthropic_token_source()` in `anthropic_adapter.py`. Fits in Overview tab.
- **Streaming status** — Show if streaming is enabled and active on sessions. Data: `StreamingConfig` in `gateway/config.py`. Fits in Sessions tab.
- **Cron skip events** — Log when stale cron jobs are fast-forwarded on restart. Data: `cron/jobs.py` log messages. Fits in Cron tab.
- **Error retry counts** — Show 429/529 retry attempts per session. Data: would need instrumentation in `run_agent.py`. Fits in Sessions detail.
- **Vercel AI Gateway** — New provider, sessions would appear naturally if configured. No dashboard work needed.

Priority: auth method indicator (quick win, useful now) > cron skip events (small, informative) > streaming status (only matters once we enable streaming).
