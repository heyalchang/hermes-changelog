---
layout: post
title: "Merge Notes: 59 Commits — Clean Merge, Regressed Fix Found"
date: 2026-03-18 07:45:00 +0000
categories: merge-notes
---

59 commits from `origin/main` merged into `local/dashboard-and-tracking`. Version 0.3.0 → 0.4.0.

## Conflicts

One conflict in `run_agent.py` — purely additive. Our branch added `CanonicalUsage` to the usage_pricing import (from the token accounting audit). Upstream added `load_soul_md` to the prompt_builder import. Both kept. `git rerere` recorded the resolution.

## Tests

5430 passed, 41 failed (all pre-existing), 200 skipped. No new failures from the merge.

## Regression Found: delegate_tool NameError

The fix for `_saved_tool_names` referenced-before-assignment landed in `ba7248c6` — but was immediately overwritten by the Copilot integration merge conflict resolution in `e7844e9c`. The Copilot commit (`0c392e7a`, 2470+ lines) touched `delegate_tool.py` in the same area, and the merge resolution took the wrong side, putting the save back in `_build_child_agent` instead of `_run_single_child`. The NameError persists on `origin/main`. 21 delegate tests still fail. This is still a viable upstream PR — same one-line fix applies.

## Local Extension Impact

No impact. All seven areas audited clean:

- `usage_pricing.py` — upstream added `api_key` threading to `estimate_usage_cost()`, which is additive and compatible with our token accounting audit changes
- `run_agent.py` — our `_canonical_usage_log_fields()`, JSONL logging, and payload breakdown instrumentation all survived intact
- `gateway/run.py` — our `_result_or_agent_metric()` helper and token forwarding survived
- `gateway/session.py` — no upstream changes, our `reasoning_tokens` additions untouched
- `hermes_state.py` — only `search_messages()` changed (broader default search), no overlap
- `dashboard/data.py` — no upstream changes
- `pyproject.toml` — version bump only, `dashboard` still in packages.find

## Worth Surfacing in the Dashboard

**GitHub Copilot provider**
: New provider type in session data. `billing_provider = "copilot"` will appear in DB for Copilot sessions. The Sessions tab model filter and Usage tab provider breakdown should handle this automatically since they read from DB values. Low priority — we don't use Copilot.

**SOUL.md identity**
: The agent now loads identity from `~/.hermes/SOUL.md`. Could surface the current SOUL.md content in the Memory tab alongside other memory files. Low priority.

**Unauthorized DM behavior**
: New config option. Could surface in Overview tab's config card if we show gateway config. Low priority.

**Endpoint metadata pricing**
: Custom endpoints now self-report pricing. `get_pricing_entry()` tries endpoint metadata before the built-in table. Could note the cost source in the cost breakdown tooltip (e.g., "pricing from endpoint metadata" vs "pricing from built-in table"). Medium priority if we start using custom endpoints.

**Cron one-shot recovery**
: One-shot jobs within 120s of their window are auto-recovered. Could surface recovery events in the Cron tab. Low priority.
