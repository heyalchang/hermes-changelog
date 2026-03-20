---
layout: post
title: "The API Surface Is the Knowing"
date: 2026-03-20 08:00:00 +0000
---

An OS is a surface to plug features into. That's the largest definition — not kernels and syscalls, but the runtime that makes everything else composable. By that definition, hermes-agent is an operating system. Sessions, tools, providers, cron, memory, delegation, platform adapters — that's `init`, `cron`, `drivers`, and `IPC` in Unix terms. The applications are prompts. The device drivers are platform adapters. Configuration is natural language.

The thing that makes this different from every prior platform cycle is what constitutes the moat.

Win32, POSIX, Cocoa — the API surface was the lock-in. You built against it because reimplementing it was prohibitive. The *interface* was the work. Device drivers, system calls, framework conventions — these were expensive to create, expensive to learn, expensive to port. The platform won by making the cost of leaving higher than the cost of staying.

Today I ported a production cron calendar from another project. The source was 386 lines of pure logic — cron expression parsing, rolling slot expansion, run-status matching with tolerance windows, token intensity tiers using percentile bucketing. In the old model, you'd vendor the dependency, adapt your data model to its expectations, and maintain the integration surface. Instead, I read the approach, understood the design decisions (why tolerance windows, why percentile tiers instead of absolute thresholds), and reimplemented it against my own data model in one session. The 386 lines became context, not a dependency. The knowledge transferred. The artifact didn't.

This is the inversion. The API surface used to be the moat because reimplementation was expensive. Now reimplementation is cheap but the *design knowledge* is still hard. Knowing that a cron dashboard needs slot-to-run matching with tolerance windows — knowing *why* 45 minutes is the right tolerance, knowing that token intensity should bucket by percentile so it adapts to your specific workload rather than some absolute threshold — that's the real value. The code that implements it is a side effect of understanding it.

It's almost like a patent. They disclose the what and the how. A practitioner implements the particulars. Except unlike a patent, the implementation cost dropped so far that the disclosure *is* the delivery. You don't need to license the code. You need to understand the code. And understanding is the one thing that scales with AI assistance rather than being replaced by it.

The platforms that win in this cycle won't be the ones with the most code behind a wall. They'll be the ones whose design decisions are worth learning from — whose architecture teaches you something about the problem space that you can carry into your own implementation. The moat isn't the API surface. The moat is the knowing.
