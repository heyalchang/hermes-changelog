---
layout: post
title: "Mechanism and Meaning"
date: 2026-03-21 06:00:00 +0000
draft: true
---

Every view answers a question. The list answers "what do I have and how often does it run?" The detail pane answers "what exactly does this job do?" The calendar answers "when did things happen and when will they happen next?" Three views, three questions. No view trying to answer someone else's question.

The design move isn't "simplify." Simplification implies the same information, presented more cleanly. This is choosing which *level of information* belongs where. The raw cron expression isn't clutter — it's useful, in the detail pane, next to the human-readable version, for someone who reads cron. It just doesn't belong in the scanning view, because in the scanning view you're not reading cron. You're asking "is this daily or weekly?"

Here's where it came from.

`0 5 * * *` is a cron schedule. It means "daily at 5 AM." The dashboard was showing the raw expression — five numbers and asterisks — and the instinct is to make it "more readable." Translate each field, add labels. But that's just the same mechanism in a different font. The question isn't how to display the expression. It's what the expression *means*.

`0 5 * * *` means "daily." `0 9 * * 1` means "weekly." `0 9 1 * *` means "monthly." These are categories, not formats. Once you see that, the compact label writes itself: the word. `daily`. `weekly`. `monthly`. The time and the specific day go in the detail pane, where you're already looking at one job and there's room for it.

The split — *cadence* in the list, *specifics* in the detail — didn't come from thinking about column widths. It came from noticing that the list answers a different question than the detail pane. If you put both answers in the same place, neither one is fast.

A health dot is four colors. Green, yellow, red, gray. That's the *meaning* of a running process. The mechanism is a PID number, an exit code, whether `launchctl list` includes the label, whether `KeepAlive` is a boolean or a conditional dictionary. The dot is what you need at a glance. The mechanism is what you need when something's wrong. Different questions, different places.

The design move isn't "simplify." Simplification implies the same information, presented more cleanly. This is choosing which *level of information* belongs where. The raw cron expression isn't clutter — it's useful, in the detail pane, next to the human-readable version, for someone who reads cron. It just doesn't belong in the scanning view, because in the scanning view you're not reading cron. You're asking "is this daily or weekly?"

The exotic cases make this sharper. `0 9 1,15 * *` — first and fifteenth of each month. Compact label: `2×/month`. Not the dates. Not the time. The cadence. `0 9 * * 1-5` — `weekdays`. And then there's `5 4 */3 1,6 *`, where honest classification is just `custom`. That's not a failure. `custom` means "this one's different, look at the detail pane." It's a meaningful answer to the question the list is asking.

The thing I'm trying to name isn't minimalism and it isn't progressive disclosure. It's that classification has to happen before layout can. You can't build the two-level view until you've answered: what does a cron expression *mean* — not syntactically, but operationally? Daily, weekly, monthly, twice a month, custom. Those categories don't exist in the data. They exist in the user's head. The dashboard's job is to match the mental model, not expose the storage format.

When you get that right, the dashboard stops feeling like a window into a database and starts feeling like it understands your system the way you do.

*(Draft written by Claude.)*
