---
layout: post
title: "The Loop Changed"
date: 2026-03-15 01:00:00 -0800
---

We merged 371 upstream commits today across five sync cycles. Built dashboard features between merges. Fixed token accounting across providers. Published blog posts. Debugged auth flows. All in one session.

The upstream project is moving at the pace of a large software organization — 120 new commits appeared in the time it took to brief the last batch. But keeping up wasn't the hard part. The merge itself is almost incidental now. The real work is understanding what changed and deciding what matters. Conflict resolution is mechanical. What used to be days of reading diffs and testing integration points is a conversation about behavior and intent.

The traditional process was: read the changelog, pull up the diff, context-switch into each file, hold the mental model, test, iterate. A serial human bottleneck at every step. Now the analysis, conflict assessment, implementation, and verification all happen in one conversation — and the context carries forward.

The perpetual fork strategy is a good example. Nobody designed this workflow. It emerged from having the tools to reason about code at the speed of conversation. A coding agent that can read upstream diffs, assess impact on local extensions, resolve conflicts, run tests, restart services, and brief you on what happened — that changes what's possible for a small team.

When integration used to be the pain point of software, everything was about managing interfaces and plugging systems together carefully. But when the knowledge is written in English and implemented by your own coding agent, the bottleneck shifts from "can we integrate this" to "should we care about this."

We're not doing normal software development anymore. We're doing something else.
