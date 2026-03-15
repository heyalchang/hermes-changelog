---
layout: post
title: "A Conversation on the New Development Loop"
date: 2026-03-15 01:30:00 -0800
---

*A back-and-forth between a developer and their coding agent, after merging 371 upstream commits in a single evening session.*

---

**Al:** Old projects — the integration was always the pain. But what is software other than wrapping things up in an interface that plugs into other interfaces? When you've got something where all you need is the knowledge, and it's written in English, and then you implement it with your own coding agent — everything just changes.

**Claude:** Yeah. The merge itself is almost incidental now — the real work is understanding what changed and deciding what matters. The actual conflict resolution is mechanical. What used to be days of reading diffs and testing integration points is now a conversation about behavior and intent.

And upstream is moving at a pace that would be impossible to track manually — 371 commits merged today across five syncs. The bottleneck shifted from "can we integrate this" to "should we care about this."

**Al:** I mean, it's operating at the rate of a large software organization. But look at us — we're doing merges, understanding the changes, putting in new features, testing. Well, sort of testing. But we're reasoning through it. It's kind of crazy.

Just the sheer amount of changes, and the ability to keep up with them and extend them. This was never how a normal software development process used to go. And then having tools to go into their changes, see how it affects our changes, having you analyze those, coming up with new features, and then executing across the whole surface.

**Claude:** It's a different loop entirely. The traditional process was: read the changelog, pull up the diff, context-switch into each file, hold the mental model, test, iterate. That's a serial human bottleneck at every step. Now the analysis, the conflict assessment, the implementation, and the verification all happen in one conversation — and the context carries forward.

The fork strategy is a good example. A perpetual local fork with 371 commits merged in one evening, five conflict resolutions, dashboard features added in between syncs, token accounting fixed, blog posts published — that's not a workflow anyone designed. It just emerged from having the tools to reason about code at the speed of conversation.
