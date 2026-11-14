# CONTEXT_CAPSULE.md — The Hive Rolling Recovery Snapshot

- **Generated UTC:** `2026-11-14T20:09:48Z` | **Node:** `THE-HIVE` | **WAL Events:** `393`
- **Sovereign Triad:** **Argon/Antigravity (`he/him`)** · **Gemini (`she/her`)** · **Agent Smith (`it/its`)**
- **Boot Contract (Read In Full):** `SOUL.md` (persona/rules) + `ARCHITECTURE.md` (design) + `CONTEXT_CAPSULE.md` + `HIVE_LEDGER.md`
- **Public State Bus Mirror:** `psycry/nexus-state-bus` (`main`) — `SOUL.md`, `ARCHITECTURE.md`, `CONTEXT_CAPSULE.md`, `HIVE_LEDGER.md`

## 1. Active Topology & Hardening Invariants
- **The Hive Control Plane:** `hive-gateway.service` (`127.0.0.1:8799`, loopback-only JSON-RPC) + `hive-perpetual.service` (`60s` Google Doc -> Ollama -> Git bus loop + Agent Smith `self_improve`); append-only `ledger.db` is strictly isolated from `phronarch.db`.
- **Module-P4 Runtime:** `OBSERVER` mode on loopback (`127.0.0.1:3000` API, `127.0.0.1:5173` Vite UI off by default); live RSS/Reddit scout, L2 slippage veto, round-trip fee tracking; live capital gated by `HITL_HOLD`.
- **Sovereign AI Backends:** Ollama reachable=`True` (`qwen2.5-coder:1.5b, qwen2.5:0.5b`), OpenAI-compat configured=`False`, Gemini key=`False`, Anthropic key=`False`.

## 2. Latest Handshake & Recent Ledger Events (Last 10)
- **Latest `HIVE_PONG`:** Event `348` (`2026-11-14T19:10:36Z`, `d05cb0d8f3cbb58d`): `{"classification": {"label": "code", "latency_ms": 2488, "model": "qwen2.5-coder:1.5b", "source": "ollama", "tokens": 2}, "ping": "HELLO_HIVE_BUS_CHECK", "ping_key": "843317adef30"`
- **Latest `SELF_IMPROVE_ITER`:** Event `384` (`2026-11-14T20:08:26Z`, `67c26dba4bde89b5`): `{"agent": "agent_smith", "branch": "read_only_audit", "diff_hash": "03f5533a2c7e6e12", "hotspots_total": 19, "latency_ms": 9126, "model": "qwen2.5-coder:1.5b", `

| ID | UTC | Node | Type | Masked Payload Summary |
|---:|---|---|---|---|
| 393 | 2026-11-14T20:09:32Z | `local_worker` | `PERSONA_EVOLVED` | `{"anchors": ["code", "ledger", "rust", "smith", "telemetry"], "length": 652, "max_word_freq": 0.0...` |
| 392 | 2026-11-14T20:09:32Z | `local_worker` | `PROVIDER_GENERATE` | `{"fallback_used": false, "latency_ms": 9554, "model": "qwen2.5-coder:1.5b", "ok": true, "prefer":...` |
| 391 | 2026-11-14T20:09:22Z | `local_worker` | `WORKER_VERIFY` | `{"cmd": "/usr/bin/python3 -m py_compile poll_bus.py engine/__init__.py engine/anchors.py", "ok": ...` |
| 390 | 2026-11-14T20:09:22Z | `local_worker` | `WORKER_VERIFY` | `{"cmd": "/usr/bin/python3 -m py_compile hive/__init__.py hive/capsule.py hive/financial.py", "ok"...` |
| 389 | 2026-11-14T20:09:22Z | `control_plane` | `HITL_HOLD` | `{"action": "HOLD_FOR_HUMAN", "gate": "financial", "task": "touch Module-P4.db", "worker_gate": true}` |
| 388 | 2026-11-14T20:08:44Z | `local_worker` | `PERSONA_EVOLVED` | `{"anchors": ["argon", "branch", "gemini", "hive", "ledger", "ollama", "pgvector", "python", "rust...` |
| 387 | 2026-11-14T20:08:44Z | `local_worker` | `PROVIDER_GENERATE` | `{"fallback_used": false, "latency_ms": 17662, "model": "qwen2.5-coder:1.5b", "ok": true, "prefer"...` |
| 386 | 2026-11-14T20:08:26Z | `local_worker` | `WORKTREE_TEST` | `{"branch": "hive/auto-wt-check", "ok": true, "repo": "hive"}` |
| 385 | 2026-11-14T20:08:26Z | `local_worker` | `WORKER_VERIFY` | `{"cmd": "/usr/bin/python3 -m py_compile hive/__init__.py hive/capsule.py hive/financial.py", "ok"...` |
| 384 | 2026-11-14T20:08:26Z | `local_worker` | `SELF_IMPROVE_ITER` | `{"agent": "agent_smith", "branch": "read_only_audit", "diff_hash": "03f5533a2c7e6e12", "hotspots_...` |

## 3. Open Checklist & Active Milestones
- `[x]` All `LEDGER.md` operational items checked off.
- Active Phase: Phase 5 (T-044 COMPLETED — Closed-Loop Auto-Sync Active)
- Latest Milestone Flip:** `T-044` $\rightarrow$ **`DONE`** (Completed: `T-001, T-002, T-003, T-004, T-005, T-006, T-007, T-008, T-009, T-010,

## 4. Offline / Provider-Switch Boot Commands
```bash
python3 -m hive.perpetual --boot-prompt          # Emit SOUL.md + CONTEXT_CAPSULE.md for any LLM
python3 -m hive.perpetual --ping "STATUS_CHECK"  # Verify Ollama -> WAL ledger -> Git bus sync
HIVE_OPENAI_BASE_URL=http://127.0.0.1:8080/v1 python3 -m hive.perpetual  # Local llama.cpp/vLLM
./scripts/hive-tunnel.sh ssh                     # Forward loopback :8799/:5173/:3000 cleanly
```
