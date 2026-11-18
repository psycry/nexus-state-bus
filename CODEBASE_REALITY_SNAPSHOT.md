# Agent Smith & Hive — Grounded Codebase & Capability Snapshot

> **Purpose:** Give Gemini (Cloud Architect) and any external orchestrator an accurate, zero-hype inventory of what actually exists in `psycry/agent_smith` and `psycry/hive`, what is running right now, and what is **not** implemented.
> **Rule:** Never assume a staged or R&D item in the Google Doc Master Ledger is implemented in code unless it appears under **Implemented & Verified** below.

---

## 1. Current Operational Posture (What Is Actually Running)

On `THE-HIVE` (local Garuda Linux workstation):
- **`nexus-state-bus.service` (`active (running)`):** Runs `poll_bus.py` to poll the Google Doc Master Ledger (`INSTANT_TXT`) and sync sanitized state packets to `psycry/nexus-state-bus`.
- **`agent-smith-sweep.timer` (`active`, every 6h):** Runs `agent-smith-sweep.service` (`engine/ingest.py`, `engine/dossier.py`, `engine/notify.py`) to ingest leads, score them against 3 capability anchors (`agent_smith`, `shelter_source`, `phronarch`), stage dossiers/submissions, and log alerts.
- **`agent-smith-dashboard.service` (`active (running)`):** Serves the Tactical Command Center UI/API on `http://127.0.0.1:8787` (`engine/dashboard_api.py`).
- **`agent-smith-watchdog.timer` (`active`):** Runs `engine/self_heal.py` to check local service health (`agent-smith-pgvector`, `hive-ollama`, `nexus-state-bus.service`, `agent-smith-dashboard.service`).

**What Agent Smith is NOT doing autonomously:**
- It is **not** autonomously reading Master Ledger milestones and writing/deploying arbitrary code on its own.
- `hive.self_improve` (`hive/self_improve.py`) is **on-demand only** (triggered explicitly, e.g., via Discord C2 `improve yourself`), runs symbolic AST/Rust complexity scans first, tests edits in an isolated `git worktree` (`py_compile` / `cargo check`), and is ring-fenced against modifying core safety/constitutional files.
- Financial/trading/order actions are blocked behind `HITL_HOLD` (human-in-the-loop required).
- Contracted revenue is currently **`$0.00`** (`$405,000.00` unsubmitted federal SBIR ceilings + `$20,500.00` uncontracted SOW placeholders).

---

## 2. Implemented Code Inventory (Verified on Disk)

### A. `agent_smith/claw-mcp-server/` (Rust Control Plane — ~3,718 LOC)
- `src/main.rs` (214), `src/agent.rs` (868), `src/config.rs` (131), `src/lib.rs` (5): MCP server (`v0.2.0`), tool dispatch, sandboxed execution.
- `src/core/agent.rs` (49), `src/core/agents.rs` (165), `src/core/router.rs` (294): Async `Agent` trait, `TaskContext` / `TaskResult`, `TaskRouter` with OCI budget circuit breaker (`spend >= oci_monthly_budget_cap` -> `ctx.local_only = true`), max-iteration guardrail (`>5` -> `MAX_ITERATIONS_EXCEEDED`), and JSONL span logging (`submissions/otel_spans.jsonl`).
- `src/ai/local.rs` (133), `src/ai/cloud.rs` (180): Local Ollama + cloud Gemini model adapters.
- `src/bin/discord.rs` (339), `src/bin/chat.rs` (90): Discord C2 and CLI chat entrypoints.
- `src/tools/workforce.rs` (878), `shell.rs` (116), `file_system.rs` (167), `search.rs` (114), `ai.rs` (50), `metrics.rs` (21): MCP tools including `run_lead_ingestion`, `generate_dossiers`, `list_ranked_leads`, `store_memory`, `recall_memory`, `dispatch_subagent`, `list_workforce_tasks`.

### B. `agent_smith/engine/` (Python Worker Plane — ~3,971 LOC)
- `config.py` (35), `models.py` (130): SQLModel + `pgvector` schema (`opportunity_leads`, `agent_memories`, `workforce_tasks`).
- `embeddings.py` (116), `anchors.py` (137): 768-dim embedding lock (`models/gemini-embedding-001` + offline FNV-1a fallback) and 3 capability anchors.
- `ingest.py` (550), `dossier.py` (609), `qa.py` (113), `notify.py` (122): Lead ingestion (Grants.gov API, GitHub OSS target scout), dossier & submission staging, QA gate (`4/4 PASS`), and webhook/local alert dispatch.
- `ledger_sync.py` (367), `gdoc_ledger.py` (229), `dashboard_api.py` (521): State bus synthesis (`gemini_inbox.md`, `sync_packet.md`, `MASTER_LEDGER_FEEDBACK.json`) and local dashboard API (`:8787`).
- `oci_budget_guard.py` (250), `self_heal.py` (147), `router_bridge.py` (173), `hive_bridge.py` (412), `mesh_handoff.py` (159): OCI Always-Free `$0/mo` guard (`4.0 OCPUs / 24.0 GB RAM`), local service watchdog, Hive router/memory bridge, and P2P mesh handoff.

### C. `hive/hive/` (Python Hive Runtime — ~2,156 LOC)
- `perpetual.py` (379), `providers.py` (314), `self_improve.py` (657), `capsule.py` (152), `worker.py` (128): Daemon loop, provider adapters (Ollama + Gemini/OpenAI-compat burst), worktree-sandboxed `self_improve`, rolling `<150`-line `CONTEXT_CAPSULE.md`, and branch-isolated worker.
- `router.py` (28), `local_router.py` (77), `gateway.py` (64), `ipc.py` (53), `ledger.py` (58), `handoff.py` (26), `financial.py` (30), `notify.py` (31), `watchdog.py` (36): Thin routing, IPC, WAL ledger, and safety/watchdog modules.

---

## 3. Staged / Aspirational Items NOT Yet Implemented in Code

Do **not** assume Agent Smith can currently execute these:
- **KEOV (Knowledge-Enhanced Operator Vector)** offline buffer/recalibration protocol (staged directive only; no `keov` module exists).
- **Zero-Knowledge Proofs (ZKPs)** in Phronarch execution pipeline (R&D concept only).
- **Dynamic Self-Healing Agent Topologies & Decentralized Consensus State Sync** (R&D concepts only; current sync is single-master Git + Google Doc `INSTANT_TXT`).
- **Autonomous Proposal/PR Submission without Human Review** (packages and patch bundles are staged in `submissions/`, not auto-submitted to portals or upstream repos without operator review).
- **Full-Autonomy Multi-Step Code Generation from Ledger** (Agent Smith stays in deterministic monitoring/ingestion/budget-guard mode; complex architectural coding is handled by Antigravity or explicit human-triggered workflows).

---

## 4. Routing & Model Guidance

- **Deterministic / Zero-Token Path:** Status checks, ledger reads, OCI budget checks, lead scoring, and AST scans run symbolically without LLM tokens.
- **Fast / Local Execution (`qwen2.5-coder:1.5b` / fast cloud models):** Day-to-day routing, lightweight classification, and bounded checks.
- **Complex Architectural / Coding Work:** Escalate to Antigravity (local IDE agent) or an explicit top-level orchestrator rather than expecting Agent Smith's background daemon to implement multi-file features from the ledger.
