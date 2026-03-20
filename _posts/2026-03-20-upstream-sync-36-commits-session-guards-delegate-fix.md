---
layout: post
title: "Upstream Sync: 36 Commits — Session Guards, Delegate Fix, ACP Persistence"
date: 2026-03-20 09:00:00 +0000
---

36 commits landed upstream covering session safety, delegation fixes, and provider hardening.

**Session Safety.** The gateway now prevents concurrent agent runs on the same session. A sentinel object holds the slot during media processing (vision enrichment, speech-to-text, session compression) — a window that was previously seconds wide. A second message arriving during this window queues instead of racing. The `/stop` command was hardened to handle the sentinel state gracefully.

**Delegation Fix.** The `_saved_tool_names` NameError in delegate_tool.py is fully resolved. Parent tool names are now saved before child agent construction (which overwrites the global) and restored after all children are built. A regression test validates the fix.

**ACP Session Persistence.** ACP (Agent Communication Protocol) sessions were previously fully in-memory. They now persist to state.db via SessionDB, surviving process restarts. CWD is stored in the model_config JSON field with no schema migration needed.

**Local Model Context Windows.** Unknown local models previously defaulted to a 2M context window. The system now actively probes LM Studio, Ollama, llama.cpp, and vLLM for actual context size. For LM Studio, it prefers the loaded instance size over the model maximum.

**Cron Resilience.** Jobs referencing missing skills no longer crash — the scheduler warns, skips the unavailable skill, and continues with what remains. A notice is prepended to the prompt so the model can inform the user.

**Provider Fixes.** MiniMax auth errors resolved by defaulting to the correct API mode. TTS tool gained base_url support for non-OpenAI endpoints. SQL string formatting eliminated from insights and state modules.

Token counters on AIAgent now reset between sessions in the same CLI process. Codex responses API handles reasoning-only responses. Tool schema validation prevents unavailable names from leaking into model schemas.
