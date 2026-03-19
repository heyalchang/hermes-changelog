---
layout: post
title: "The Friction Budget"
date: 2026-03-19 06:00:00 +0000
---

Most engineering work isn't hard. It's interruptible.

You need to check what landed upstream. That's a `git log`, but then you need to read the diffs that matter, skip the ones that don't, and figure out which changes touch something you care about. Fifteen minutes if you're focused. More likely thirty, because you context-switch twice to check if a fix overlaps with something you already did, and then you lose the thread.

You need to update a workflow file. You know what you want — two new behaviors, specific places they slot in. But first you have to re-read the file, hold its structure in your head, find the right insertion points, make sure you're not duplicating something that's already there. Ten minutes. Trivial. But it's ten minutes of overhead on a change that's conceptually instant.

You need to recall what a fix was and why you made it. `git log --grep`, find the commit, read the message, maybe read the diff. The answer's in there. Five minutes of archaeology for a fact you knew three days ago.

None of these tasks is difficult. Each one costs five to fifteen minutes — not of thinking, but of *navigating*. Finding the file, re-establishing context, locating the insertion point, verifying you're not about to duplicate work. The friction isn't intellectual. It's mechanical. And it compounds. A maintenance session that's conceptually twenty minutes of decisions becomes two hours of context-loading with decisions sprinkled in.

What changes when the agent holds the context is that the navigation cost drops to zero. Not approximately zero — actually zero. "What's new on main?" produces a grouped, thematic briefing in seconds, with the overlaps to local work already flagged. "What was that delegate fix?" comes back with the commit, the root cause, and the fact that upstream just independently landed the same fix. Updating the workflow file is two edits with the right insertion points found on the first try. The decisions are the same decisions I would have made. The judgment calls are the same. Nothing got smarter. The friction just isn't there.

This matters more than it sounds like it should. The friction budget — the cumulative overhead of navigating, context-loading, re-reading, and verifying before you can do the actual work — is where most maintenance time goes. Not the hard problems. Not the design decisions. The stuff between the decisions. When that budget goes to zero, a session that would have been "check upstream, update one file, maybe fix something" and taken an evening becomes four tasks deep in fifteen minutes, each one clean, each one done.

The work doesn't get easier. It gets *uninterrupted*.
