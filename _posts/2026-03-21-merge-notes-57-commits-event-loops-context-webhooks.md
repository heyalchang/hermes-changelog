---
layout: post
title: "Merge Notes: 57 Commits — Event Loops, Context Detection, Webhooks"
date: 2026-03-21 06:02:00 +0000
categories: merge-notes
---

57 commits merged from upstream into `local/dashboard-and-tracking`. Zero conflicts, zero new test failures (22 pre-existing, down from 25). All services restarted and verified.

## Merge Experience

Clean merge — no conflicts at all. The event loop architecture rebuild, context length overhaul, and webhook platform all landed without touching our instrumentation points. `_compute_payload_breakdown()`, `_canonical_usage_log_fields()`, and the JSONL logging block are all intact.

The background memory/skill review thread runs post-loop at a completely separate insertion point from our token accounting. Context pressure warnings live in the compressor path. Neither overlaps with our code.

## Local Extension Audit

No action required. All instrumentation hooks verified intact at their original positions in `run_agent.py`.

## Worth Surfacing in the Dashboard

**Webhook sessions** (high priority)
- New `webhook` platform creates sessions with `source = "webhook"` and IDs like `webhook:{route}:{delivery_id}`
- Data source: existing `sessions` table, filter by `source`
- Where it fits: Sessions tab already surfaces them, but Messages feed (`get_feed()`) has a hardcoded platform list that excludes webhook. Platform filter dropdowns in Sessions/Search tabs also need the option.

**Context pressure events** (medium)
- Gateway now sends context pressure warnings at 60%/85% of compaction threshold
- These appear as regular messages in the chat — could flag them visually in the Messages tab
- Data source: message content pattern matching (not ideal), or future `message_extras` metadata

**New platform delivery targets** (low)
- Matrix, Mattermost, DingTalk, SMS added to cron delivery maps
- Dashboard platform filters and `get_feed()` should include these when any are in use
- Currently no sessions from these platforms, so low priority

**Cron autonomy change** (informational)
- Cron agents no longer have `send_message` or `clarify` tools
- No dashboard change needed, but explains why cron sessions won't show messaging tool usage going forward

## Upstream PR Opportunities

None this batch. No local fixes converged with upstream work, and no bugs discovered during merge.
