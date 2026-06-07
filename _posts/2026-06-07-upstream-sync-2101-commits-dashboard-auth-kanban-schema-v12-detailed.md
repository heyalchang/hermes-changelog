---
layout: post
title: "Upstream sync — 2101 commits (detailed): dashboard auth, kanban, schema v12"
date: 2026-06-07 10:26:00 +0000
---

Detailed, SHA-linked companion to the [2101-commit briefing]({% post_url 2026-06-07-upstream-sync-2101-commits-dashboard-auth-kanban-schema-v12 %}). 2014 non-merge commits, v0.14.0 → v0.16.0, roughly May 9 to June 7.

## Dashboard & Authentication

The authentication layer is built as a pluggable provider system. The provider ABC and session dataclass define the contract (`fdc…`-era `feat(dashboard-auth): define DashboardAuthProvider ABC + Session dataclass`), with cookie helpers for the session access/refresh/PKCE triple, single-use websocket tickets behind `POST /api/auth/ws-ticket`, and ticket auth on all four WS endpoints. Providers shipped incrementally: a contract-compliant Nous OAuth provider, a generic self-hosted OIDC provider (`feat(dashboard-auth): add generic self-hosted OIDC provider`), and a non-redirect username/password `BasicAuthProvider` (`feat(dashboard-auth): add BasicAuthProvider username/password plugin`). Session rotation via refresh token landed in #37247. Hardening: `__Host-`/`__Secure-` cookies and `X-Forwarded-Prefix` support, a fail-closed gate when no providers are configured, and a json-lines audit log at `$HERMES_HOME/logs/dashboard-auth.log`.

Dashboard surface: the Channels page to configure every messaging platform from the browser (#37211), the complete admin panel — MCP catalog, enable/disable, hook creation, system stats (#36736, #36704), `hermes dashboard register` for a self-hosted OAuth client, Debug Share on the System page (#38600), a check-before-update flow (#38205), the Skills hub rehaul with security scan (#40384), full tool-backend configuration in the GUI (#40418), the nous-blue theme with bulk sessions and a schedule picker (#37383), and always-on embedded chat with the `--tui` flag removed.

## Kanban (#21582, #27145, #29747, others)

Multi-worker board on its own `kanban.db`. Goal-mode cards run workers in a `/goal` loop (#35710); decompose children inherit the root workspace (#37172); file attachments on tasks (#35395); a terminate endpoint `POST /runs/{run_id}/terminate`. Reliability: crashed-worker detection grace periods, hoisted zombie reaping, per-call board isolation, iteration-exhaustion handling (#29747), and SQLite corruption defense — synchronous-FULL + secure_delete + cell_size_check, content-addressed corrupt-DB backups, refusal to auto-init a corrupt board, and a legacy TEXT-PK → INTEGER AUTOINCREMENT rebuild on open.

## Desktop

Profile rail: drag-sort (`feat(desktop): drag-sort profiles in the rail`), per-profile colors via long-press, rename/delete, quick-create. Keyboard: rebindable shortcuts panel (`5e2b83a8a`), Cmd+K/Cmd+P palette, steer-from-composer, arrow-history navigation. Cron became a first-class sidebar entity (`471a5fc5c`) firing from the dashboard backend (`3e2d75881`). i18n with zh-Hans, Japanese, Traditional Chinese; unified overlay design system + BrandMark onboarding redesign (#40708); drag files anywhere in chat (#36262); dedicated Providers + Accounts/API-keys settings (#38551).

## State & Agent Core

Schema v12 (`3e59be0c4`, #21910): `messages.active` (default 1, soft-delete) + `sessions.rewind_count`, deferred `idx_messages_session_active`, legacy backfill. Read methods gained `include_inactive=False`. `/undo [N]` backs up N user turns with prefill + soft-delete; brought to messaging platforms in #36699. Mid-session model switch persisted to DB (`fix(state): persist mid-session model switch to database`). `runtime_cwd` resolver as single source of truth (`feat(agent): add runtime_cwd resolver`). `default_headers` honored for custom providers, main and aux (#40033). Config v11→v12 migration preserves custom-provider model maps (#40573).

## Providers & Models

Usage-aware credits — in-session notices, `/usage` view, dev readout (#40011). MiniMax-M3 with 1M context (#36214); qwen3.7-plus to nous + openrouter (#39409). Catalog refresh hourly (#35756). Model picker: ranked fuzzy search, grouped multi-endpoint providers (#35227), OpenAI/OpenRouter routing fixes (#37404, #37175). Compaction trigger raised to 85% for gpt-5.5 on Codex OAuth (#40957). Anthropic dead thinking-signature demotion on orphan-strip; MiniMax video_url → Anthropic input_video.

## Platforms

Adapter → bundled plugin migrations: Home Assistant (`c37c6eaf2`), Discord (`cc8e5ec2a`, full Teams parity), Mattermost (`af973e407`). Feishu meeting invitations; Discord voice-channel mixer (#39659); per-platform streaming defaults + structured stream-event protocol (#37303, #37250); Telegram QR onboarding; WhatsApp/WeChat text-debounce batching; LRU bounds on feishu/bluebubbles caches; qqbot 100% CPU spin fix (#40574).

## Security

Approval-bypass cluster: block agent writes to `~/.hermes/config.yaml` (`fix(file_tools)`), gate perl/ruby `-i` in-place edits, `is_approved` check in execute_code (#39275). Path traversal: skill_view name (#40566), bitwarden zip-slip (#40569). SSRF off the event loop (`fix(web): run URL SSRF checks off the event loop`). Starlette ≥1.0.1 pin for CVE-2026-48710 (#35118). AWS/Bedrock creds stripped from subprocess env.

## Packaging & Infra

Pillow (`b13ab0b9a`) + markdown (#38649) promoted to core deps. Nix `npmDepsHash` fix + react-router 7.17.0 for GHSA-8x6r-g9mw-2r78. Docker: config migrations on boot (#36627), UID remap chown (#38655), Unraid uids (#38098), orphaned-container recovery (#39415). Shallow clones in installer; macOS installer → "Hermes" launcher (#37516). `requires-python` < 3.14 (#38535). Observer-grade telemetry + NeMo-Relay plugin.

## Routine

i18n doc mirrors, install-instruction sweeps, security-redaction docs, test coverage, ~50 `chore(release): map … author email` salvage entries, desktop overlay style/refactor passes.
