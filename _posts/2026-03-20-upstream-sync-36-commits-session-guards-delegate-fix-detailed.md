---
layout: post
title: "Upstream Sync: 36 Commits — Detailed"
date: 2026-03-20 09:01:00 +0000
---

Detailed breakdown of 36 upstream commits merged on 2026-03-20.

### Agent Core
- Token counters reset on new session in same CLI process (fe331ed9) — in-memory only, no state.db impact
- Codex handles reasoning-only responses and replay path (e84d952d)
- Unavailable tool names no longer leak into model schemas (36a44811)
- MCP object schemas without properties normalized (4d2c93a0)

### Delegation
- Parent tool names saved before child construction, restored after (816d1344) — fixes NameError
- Stray _saved_tool_names reference removed from _build_child_agent (ae8059ca)

### Gateway & Platforms
- Concurrent session guard via sentinel object (aaa96713) — queues second messages during media processing
- /stop hardened for sentinel state and shutdown (fc061c2f)
- ACP sessions persist to state.db (388130a1) — source='acp', CWD in model_config JSON
- Chrome CDP endpoint normalization (bb59057d)

### Provider Infrastructure
- MiniMax 401 fix — default to anthropic_messages mode (6bcec1ac)
- Local server context window probing: LM Studio, Ollama, llama.cpp, vLLM (ec5fdb8b, d223f738, c030ac1d)
- Context length fuzzy matching + config override for custom endpoints (d76fa7fc)
- /model command: bare provider names, custom endpoint display (4c0c7f4c)

### Cron & Skills
- Missing skills warn and skip instead of crashing job (defbe0f9)
- FastMCP skill for building MCP servers (02954c1a)

### Security
- SQL string formatting eliminated in insights.py and hermes_state.py (219af757)

### CLI
- Tab accepts auto-suggestions / ghost text (04b6ecad)
- Session list columns expanded for full ID visibility (2f07df31)

### Routine
- Docs: venv path alignment (672e9752)
- TTS base_url support (116984fe)
- Daytona sandbox migration (18862145)
- pyproject.toml module fix (3959e3ca)

### Local Fix Convergence
- Delegate NameError: our fix ba7248c6 (one-liner), upstream fix 816d1344 + ae8059ca (more complete, saves at delegate_task level). No PR opportunity — they fixed independently.
