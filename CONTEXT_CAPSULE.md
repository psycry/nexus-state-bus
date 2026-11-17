# CONTEXT_CAPSULE.md — The Hive Rolling Recovery Snapshot

- **Generated UTC:** `2026-11-17T20:09:39Z` | **Node:** `THE-HIVE` | **WAL Events:** `424`
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
| 424 | 2026-11-17T19:33:36Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |
| 423 | 2026-11-17T19:33:36Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |
| 422 | 2026-11-16T22:06:27Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |
| 421 | 2026-11-16T22:06:27Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |
| 420 | 2026-11-16T21:13:04Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |
| 419 | 2026-11-16T21:13:04Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |
| 418 | 2026-11-16T21:05:05Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |
| 417 | 2026-11-16T21:05:05Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |
| 416 | 2026-11-16T20:28:57Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |
| 415 | 2026-11-16T20:28:57Z | `local_worker` | `SELF_IMPROVE_SCAN` | `{"hotspots_found": 20, "python_files": 19, "repo": "agent_smith", "rust_files": 20, "top_hotspot"...` |

## 3. Open Checklist & Active Milestones
- `[x]` All `LEDGER.md` operational items checked off.
- Active Phase: Phase 5 (T-047 COMPLETED — Closed-Loop Auto-Sync Active)
- Latest Milestone Flip:** `T-047` $\rightarrow$ **`DONE`** (Completed: `T-001, T-002, T-003, T-004, T-005, T-006, T-007, T-008, T-009, T-010,

## 4. Offline / Provider-Switch Boot Commands
```bash
python3 -m hive.perpetual --boot-prompt          # Emit SOUL.md + CONTEXT_CAPSULE.md for any LLM
python3 -m hive.perpetual --ping "STATUS_CHECK"  # Verify Ollama -> WAL ledger -> Git bus sync
HIVE_OPENAI_BASE_URL=http://127.0.0.1:8080/v1 python3 -m hive.perpetual  # Local llama.cpp/vLLM
./scripts/hive-tunnel.sh ssh                     # Forward loopback :8799/:5173/:3000 cleanly
```
