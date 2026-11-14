# CONTEXT_CAPSULE.md — The Hive Rolling Recovery Snapshot

- **Generated UTC:** `2026-11-14T19:22:57Z` | **Node:** `THE-HIVE` | **WAL Events:** `349`
- **Boot Contract (Read In Full):** `SOUL.md` (persona/rules) + `ARCHITECTURE.md` (design) + `CONTEXT_CAPSULE.md` + `HIVE_LEDGER.md`
- **Public State Bus Mirror:** `psycry/nexus-state-bus` (`main`) — `SOUL.md`, `ARCHITECTURE.md`, `CONTEXT_CAPSULE.md`, `HIVE_LEDGER.md`

## 1. Active Topology & Hardening Invariants
- **The Hive Control Plane:** `hive-gateway.service` (`127.0.0.1:8799`, loopback-only JSON-RPC) + `hive-perpetual.service` (`60s` Google Doc -> Ollama -> Git bus loop); append-only `ledger.db` is strictly isolated from `phronarch.db`.
- **Module-P4 Runtime:** `OBSERVER` mode on loopback (`127.0.0.1:3000` API, `127.0.0.1:5173` Vite UI off by default); live RSS/Reddit scout, L2 slippage veto, round-trip fee tracking; live capital gated by `HITL_HOLD`.
- **Sovereign AI Backends:** Ollama reachable=`True` (`qwen2.5-coder:1.5b, qwen2.5:0.5b`), OpenAI-compat configured=`False`, Gemini key=`False`, Anthropic key=`False`.

## 2. Latest Handshake & Recent Ledger Events (Last 10)
- **Latest `HIVE_PONG`:** Event `348` (`2026-11-14T19:10:36Z`, `d05cb0d8f3cbb58d`): `{"classification": {"label": "code", "latency_ms": 2488, "model": "qwen2.5-coder:1.5b", "source": "ollama", "tokens": 2}, "ping": "HELLO_HIVE_BUS_CHECK", "ping_key": "843317adef30"`

| ID | UTC | Node | Type | Masked Payload Summary |
|---:|---|---|---|---|
| 349 | 2026-11-14T19:22:57Z | `control_plane` | `HITL_HOLD` | `{"action": "HOLD_FOR_HUMAN", "gate": "financial", "task": "modify Module-P4.db trades", "worker_g...` |
| 348 | 2026-11-14T19:10:36Z | `control_plane` | `HIVE_PONG` | `{"classification": {"label": "code", "latency_ms": 2488, "model": "qwen2.5-coder:1.5b", "source":...` |
| 347 | 2026-11-14T19:10:35Z | `local_worker` | `INTENT_PARSED` | `{"label": "code", "latency_ms": 2488, "model": "qwen2.5-coder:1.5b", "source": "ollama", "task": ...` |
| 346 | 2026-11-14T18:19:53Z | `agy` | `HEARTBEAT` | `{"node": "agy"}` |
| 345 | 2026-11-14T18:19:53Z | `local_worker` | `HEARTBEAT` | `{"node": "local_worker"}` |
| 344 | 2026-11-14T18:19:53Z | `control_plane` | `HEARTBEAT` | `{"node": "control_plane"}` |
| 343 | 2026-11-14T18:18:53Z | `agy` | `HEARTBEAT` | `{"node": "agy"}` |
| 342 | 2026-11-14T18:18:53Z | `local_worker` | `HEARTBEAT` | `{"node": "local_worker"}` |
| 341 | 2026-11-14T18:18:53Z | `control_plane` | `HEARTBEAT` | `{"node": "control_plane"}` |
| 340 | 2026-11-14T18:17:53Z | `agy` | `HEARTBEAT` | `{"node": "agy"}` |

## 3. Open Checklist & Active Milestones
- `[x]` All `LEDGER.md` operational items checked off.
- Active Milestone: T-040 (Module-P4 Micro-Capital Live Telemetry, Markout Verification & Headline Circuit Breakers)
- Staged Next Milestone: T-041 (Autonomous Multi-Agent Scale & Grant Deployment Execution)

## 4. Offline / Provider-Switch Boot Commands
```bash
python3 -m hive.perpetual --boot-prompt          # Emit SOUL.md + CONTEXT_CAPSULE.md for any LLM
python3 -m hive.perpetual --ping "STATUS_CHECK"  # Verify Ollama -> WAL ledger -> Git bus sync
HIVE_OPENAI_BASE_URL=http://127.0.0.1:8080/v1 python3 -m hive.perpetual  # Local llama.cpp/vLLM
./scripts/hive-tunnel.sh ssh                     # Forward loopback :8799/:5173/:3000 cleanly
```
