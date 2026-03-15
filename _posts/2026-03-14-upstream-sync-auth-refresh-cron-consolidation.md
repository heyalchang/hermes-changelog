---
layout: post
title: "Upstream Sync: Auth Refresh, Cron Consolidation, and Telegram Hardening"
date: 2026-03-14 20:00:00 -0800
---

25 additional commits merged from upstream NousResearch/hermes-agent, on top of the 218-commit batch from earlier today.

## Anthropic Auth Refresh

The biggest behavioral change: Hermes now auto-refreshes Anthropic OAuth tokens at runtime. A new `_try_refresh_anthropic_client_credentials()` method checks if the token on disk has changed before every API call, and rebuilds the Anthropic client if so. The token resolver was reworked to prefer Claude Code's refreshable credential file (`~/.claude/.credentials.json`) over a static `ANTHROPIC_TOKEN` environment variable. This means long-running gateway processes can survive token rotations without manual `.env` edits or restarts.

## Cron Tool Consolidation

Three separate cron tools (`schedule_cronjob`, `list_cronjobs`, `remove_cronjob`) collapsed into a single `cronjob` tool with subcommands. Multi-skill cron editing landed alongside, letting cron jobs reference multiple skills. Duplicate send suppression prevents cron jobs from sending to auto-delivery targets twice.

## Telegram Hardening

Three fixes for Telegram reliability. Media group buffering prevents the bot from interrupting itself when sending multi-photo messages. Pending media groups are now cleaned up on disconnect. Polling conflict handling was hardened to recover more gracefully from 409 errors.

## Gateway Resilience

Clean exit state management added — the gateway now writes runtime status (`starting`, `running`, `stopped`) to disk and handles fatal adapter errors gracefully. If all adapters fail post-startup, it exits cleanly instead of hanging. A fallback to `sys.executable -m hermes_cli.main` was added for environments where `hermes` isn't on PATH.

## Other Changes

- CLI skills are now preloaded on launch for faster first-use
- Vision tool surfaces actual error reasons instead of generic messages
- Dangerous command approval UI repaired
- Systemd linger auto-enabled during gateway install on headless servers
- Memory and session recall guidance tightened in prompt builder
- Docs: Anthropic auth flow, voice mode completion, cron terminology, provider contribution guide

## Merge Stats

5 conflicts resolved (run_agent.py, gateway/run.py, hermes_state.py, test_quick_commands.py, send_message_tool.py). Model backfill PR #997 officially merged upstream — local duplicate code eliminated. 4,265 tests passed.
