---
layout: post
title: "Merge Notes: 161 Commits — Context Refs, Agent Cache, MCP CLI"
date: 2026-03-22 06:02:00 +0000
categories: merge-notes
---

161 commits merged from upstream into `local/dashboard-and-tracking`. Zero conflicts, 13 new upstream test regressions (vLLM, Mattermost, prompt builder, context tracking — all confirmed failing on clean upstream). All services restarted and verified.

## Merge Experience

Clean merge despite the batch size — zero conflicts on 119 changed files with 9,901 insertions. All local instrumentation hooks verified intact at their original positions in `run_agent.py`.

## Local Extension Audit

One action-required finding: the AIAgent-per-session cache means `session_api_calls` no longer resets to 1 per gateway message, breaking the Payload tab's `agent_run` grouping for multi-turn gateway sessions. CLI and cron sessions are unaffected. Fix needed in `dashboard/data.py`.

Local platform filters in `search_transcripts` and `get_feed` updated to include webhook, matrix, mattermost, dingtalk, and sms.

## Worth Surfacing in the Dashboard

**@ context references** (high)
- Users can inject file/folder/git/URL content inline. These expand the `user` token count significantly but the Payload tab shows it all as generic "user" tokens.
- Enhancement: add `injected_context` sub-bucket to payload breakdown.

**MCP server management** (medium)
- `hermes mcp list` data could surface in a new "MCP" section or Plugins tab extension.
- Data source: MCP config file or `hermes mcp list --json` if available.

**/api/jobs CRUD** (medium)
- Upstream now has REST endpoints for cron job management on the API server.
- Our Cron tab could integrate create/edit/delete actions via these endpoints (breaking read-only, so maybe a v2 consideration).

**ResponseStore persistence** (low)
- New `response_store.db` — could surface response API sessions in dashboard if anyone uses it.

## Upstream Dashboard PR

Built the upstream-ready dashboard in a worktree during this session. 7 files, ~3700 lines, 34 tests, single squashed commit. All tabs verified visually with live data. Ready for user review before submission.
