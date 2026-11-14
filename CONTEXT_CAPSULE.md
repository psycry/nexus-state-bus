# CONTEXT_CAPSULE.md — The Hive Rolling Recovery Snapshot

- **Generated UTC:** `2026-11-14T19:43:31Z` | **Node:** `THE-HIVE` | **WAL Events:** `372`
- **Sovereign Triad:** **Argon/Antigravity (`he/him`)** · **Gemini (`she/her`)** · **Agent Smith (`it/its`)**
- **Boot Contract (Read In Full):** `SOUL.md` (persona/rules) + `ARCHITECTURE.md` (design) + `CONTEXT_CAPSULE.md` + `HIVE_LEDGER.md`
- **Public State Bus Mirror:** `psycry/nexus-state-bus` (`main`) — `SOUL.md`, `ARCHITECTURE.md`, `CONTEXT_CAPSULE.md`, `HIVE_LEDGER.md`

## 1. Active Topology & Hardening Invariants
- **The Hive Control Plane:** `hive-gateway.service` (`127.0.0.1:8799`, loopback-only JSON-RPC) + `hive-perpetual.service` (`60s` Google Doc -> Ollama -> Git bus loop + Agent Smith `self_improve`); append-only `ledger.db` is strictly isolated from `phronarch.db`.
- **Module-P4 Runtime:** `OBSERVER` mode on loopback (`127.0.0.1:3000` API, `127.0.0.1:5173` Vite UI off by default); live RSS/Reddit scout, L2 slippage veto, round-trip fee tracking; live capital gated by `HITL_HOLD`.
- **Sovereign AI Backends:** Ollama reachable=`True` (`qwen2.5-coder:1.5b, qwen2.5:0.5b`), OpenAI-compat configured=`False`, Gemini key=`False`, Anthropic key=`False`.

## 2. Latest Handshake & Recent Ledger Events (Last 10)
- **Latest `HIVE_PONG`:** Event `348` (`2026-11-14T19:10:36Z`, `d05cb0d8f3cbb58d`): `{"classification": {"label": "code", "latency_ms": 2488, "model": "qwen2.5-coder:1.5b", "source": "ollama", "tokens": 2}, "ping": "HELLO_HIVE_BUS_CHECK", "ping_key": "843317adef30"`
- **Latest `PROMPT_OPT`:** Event `372` (`2026-11-14T19:43:22Z`, `73ad992c2052ba47`): `{"agent": "agent_smith", "candidate_updated": false, "metrics": {"generate": {"calls": 4, "fail": 0, "lat_sum": 90305, "ok": 4}, "intent": {"calls": 2, "fallbac`

| ID | UTC | Node | Type | Masked Payload Summary |
|---:|---|---|---|---|
| 372 | 2026-11-14T19:43:22Z | `local_worker` | `PROMPT_OPT` | `{"agent": "agent_smith", "candidate_updated": false, "metrics": {"generate": {"calls": 4, "fail":...` |
| 371 | 2026-11-14T19:43:22Z | `local_worker` | `PROVIDER_GENERATE` | `{"fallback_used": false, "latency_ms": 6737, "model": "qwen2.5-coder:1.5b", "ok": true, "prefer":...` |
| 370 | 2026-11-14T19:43:15Z | `local_worker` | `SELF_IMPROVE_ITER` | `{"agent": "agent_smith", "branch": "read_only_audit", "diff_hash": "7b84d63c4d7ef0be", "hotspots_...` |
| 369 | 2026-11-14T19:43:14Z | `local_worker` | `PROVIDER_GENERATE` | `{"fallback_used": false, "latency_ms": 8264, "model": "qwen2.5-coder:1.5b", "ok": true, "prefer":...` |
| 368 | 2026-11-14T19:43:06Z | `local_worker` | `WORKER_VERIFY` | `{"cmd": "/usr/bin/python3 -m py_compile poll_bus.py engine/__init__.py engine/anchors.py", "ok": ...` |
| 367 | 2026-11-14T19:43:06Z | `local_worker` | `WORKER_VERIFY` | `{"cmd": "/usr/bin/python3 -m py_compile hive/__init__.py hive/capsule.py hive/financial.py", "ok"...` |
| 366 | 2026-11-14T19:43:06Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 15, "python_files": 13, "repo": "agent_smith", "rust_files": 16, "top_hotspot"...` |
| 365 | 2026-11-14T19:43:06Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 4, "python_files": 15, "repo": "hive", "rust_files": 0, "top_hotspot": {"detai...` |
| 364 | 2026-11-14T19:43:06Z | `local_worker` | `WORKER_VERIFY` | `{"cmd": "/usr/bin/python3 -m py_compile poll_bus.py engine/__init__.py engine/anchors.py", "ok": ...` |
| 363 | 2026-11-14T19:43:05Z | `local_worker` | `WORKER_VERIFY` | `{"cmd": "/usr/bin/python3 -m py_compile hive/__init__.py hive/capsule.py hive/financial.py", "ok"...` |

## 3. Open Checklist & Active Milestones
- `[x]` All `LEDGER.md` operational items checked off.
- Active Phase: Phase 5 (T-042 COMPLETED — Closed-Loop Auto-Sync Active)
- Latest Milestone Flip:** `T-042` $\rightarrow$ **`DONE`** (Completed: `T-001, T-002, T-003, T-004, T-005, T-006, T-007, T-008, T-009, T-010,

## 4. Offline / Provider-Switch Boot Commands
```bash
python3 -m hive.perpetual --boot-prompt          # Emit SOUL.md + CONTEXT_CAPSULE.md for any LLM
python3 -m hive.perpetual --ping "STATUS_CHECK"  # Verify Ollama -> WAL ledger -> Git bus sync
HIVE_OPENAI_BASE_URL=http://127.0.0.1:8080/v1 python3 -m hive.perpetual  # Local llama.cpp/vLLM
./scripts/hive-tunnel.sh ssh                     # Forward loopback :8799/:5173/:3000 cleanly
```
