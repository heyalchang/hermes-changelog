---
layout: post
title: "Merge Notes: 36 Commits — Zero Conflicts, Delegate Fix Lands"
date: 2026-03-20 09:02:00 +0000
---

Merged 36 upstream commits into the local fork. Zero conflicts. Test count improved — 25 failures (down from 41) as the delegate NameError fix landed upstream, resolving ~20 previously-failing delegation tests.

### Merge Details

No conflicts. `git rerere` had nothing to do. The delegate_tool.py NameError that we fixed locally (ba7248c6) was independently fixed upstream with a more complete approach — saving parent tool names at the `delegate_task` level before child construction rather than inside `_run_single_child`. Our fix is now superseded.

### Local Extension Impact

No impact on local extensions. The `run_agent.py` changes (session state reset, context length config) are additive and don't touch our instrumentation insertion points. The `hermes_state.py` SQL hardening is cosmetic — identifier quoting on schema migration, no new columns or version bump. ACP sessions now persist to state.db with `source='acp'`, which our dashboard queries handle automatically.

### Dashboard Opportunities

- **ACP source filter** — ACP sessions now appear in state.db. Add `acp` to the source filter dropdowns across Messages, Sessions, Search, and Usage tabs. Data source: `sessions.source = 'acp'`. Low priority — queries already return them, just needs filter option.
- **Concurrent session indicator** — Gateway now queues messages that arrive during agent startup. Could surface queued/pending state in the Messages or Overview tab. Would need a new gateway status field. Medium priority, deferred.

### Local Fix Convergence

Delegate NameError — our fix `ba7248c6` was a one-liner moving the save/restore into `_run_single_child`. Upstream's fix (816d1344 + ae8059ca) is more complete, saving at the `delegate_task` level and adding a regression test. No PR opportunity — they got there independently. This validates our diagnosis was correct.
