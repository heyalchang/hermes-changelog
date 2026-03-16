---
layout: post
title: "Smart Approvals and Rollback Checkpoints"
date: 2026-03-16 10:30:00 +0000
---

Two features in the latest upstream batch that caught our attention while briefing the 83-commit sync.

## Smart Approvals (`57be18c0`)

The dangerous-command approval system previously had two modes: manual (always prompts before running flagged commands) and off (never asks). Smart approvals adds a third. When a command gets flagged — `rm -rf`, `git reset --hard`, `DROP TABLE` — instead of stopping to ask the user, it fires a one-shot call to a cheap auxiliary LLM with just the command text and the reason it was flagged. The LLM returns APPROVE, DENY, or ESCALATE.

On APPROVE, the command runs and the pattern gets session-level approval so identical commands skip the LLM next time. On DENY, it blocks silently and tells the agent not to retry. On ESCALATE (or any LLM failure), it falls through to the normal interactive prompt.

It's an agent-autonomy feature — lets the agent self-police on destructive commands without stopping for user input every time, but with a safety net when the LLM isn't confident. Useful for longer unattended runs where you don't want to babysit every `rm` but also don't want full yolo.

## Rollback Checkpoints (`9e845a6e`, originally `c1775de5`)

The `/rollback` system takes filesystem snapshots using a shadow git repo. Before the agent runs a tool that mutates files, it commits the current state to the shadow repo as a checkpoint. The original implementation (`c1775de5`) covered `write_file` and `patch`. This batch expands it significantly:

- Enabled by default, zero overhead until a file-mutating tool actually fires
- Destructive terminal commands (`rm`, `mv`, `sed -i`, `git reset`, `git clean`, output redirects) now trigger checkpoints too
- `/rollback diff <N>` previews what would change before committing to a restore
- `/rollback <N> <file>` restores a single file instead of the full directory
- After a full restore, the conversation history is auto-trimmed so the agent doesn't think it already made changes that were just rolled back

The design is clever. Using a shadow `GIT_DIR` gives you atomic snapshots without touching the user's actual repo — all of git's diffing and restore machinery for free, no commit history pollution. The original commit credits PR #559 by @alireza78a as inspiration, but the implementation is from scratch by the maintainer. Both features are characteristic of how this project evolves — practical agent-autonomy improvements that reduce the need for human babysitting without removing the escape hatches.
