---
layout: post
title: "Merge Notes: 164 commits — schema v6, security sweep, compression overhaul"
date: 2026-03-26 23:57:00 +0000
categories: merge-notes
---

Merged 164 upstream commits into the local fork. Three conflicts, all resolved cleanly.

## Conflicts

- **.gitignore** — additive (both sides added entries). Kept both: our `.gstack/` plus upstream's `mini-swe-agent/` and nix ignores.
- **gateway/session.py** — upstream wrapped `update_session()` in thread-safe locking and moved DB calls outside the lock. Our fork had added `reasoning_tokens` accumulation. Merged both: kept the locking pattern, added `reasoning_tokens` to both the in-memory update and the DB call.
- **package-lock.json** — took upstream's version. No meaningful local npm changes.

## Local Extension Impact

- **scripts/oauth-login.sh** — deleted. Called `run_hermes_oauth_login()` which upstream removed. We authenticate via `CLAUDE_CODE_OAUTH_TOKEN` now, so the script was already unused.
- **Dashboard queries** — all use explicit column lists, so the three new schema v6 columns (`reasoning`, `reasoning_details`, `codex_reasoning_items`) are invisible. No breakage.
- **run_agent.py instrumentation** — all insertion points intact, line numbers shifted but hooks in place.
- **SessionStore thread safety** — upstream's locking improves our concurrent access pattern (dashboard reads while gateway writes).

## Test Results

6290 passed, 22 failed (all pre-existing), 195 skipped. Net improvement: failures dropped from 35 to 22 since last sync — upstream fixed 13 issues.

## Worth Surfacing in the Dashboard

- **Reasoning columns** — schema v6 stores reasoning content per message. Messages tab could show a "thinking" indicator when `reasoning` is non-null. Data source: `messages.reasoning IS NOT NULL`. Fits in Messages tab, low priority.
- **Compression config** — new `target_ratio`, `protect_last_n`, `threshold` config keys. Could surface current compression settings in a session detail view. Data source: config.yaml. Fits in Sessions tab or a new Config panel, low priority.
- **Auto-reconnect events** — platforms now recover from network failures with exponential backoff. Could show reconnection events in a platform health timeline. Data source: gateway logs. Fits in Overview or a new Platform Health panel, medium priority.
- **Session auto-reset** — gateway now notifies users when a session auto-resets (context overflow). Could surface these events distinctly in activity feed. Data source: messages with reset notification content. Fits in Activity tab, low priority.
- **API server idempotency** — new `Idempotency-Key` support on the local API endpoint (port 8642). Could show cache hit/miss stats. Data source: API server logs. Low priority.

## Upstream PR Opportunities

None identified this merge. The plist pollution bug (documented in `project_plist_pollution_bug.md`) still needs checking against this batch — it wasn't addressed in these 164 commits.
