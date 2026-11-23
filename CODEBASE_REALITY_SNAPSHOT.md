# Agent Smith & Hive — Grounded Codebase & Capability Snapshot

- **Generated At:** `2026-11-23T20:31:49Z` (auto-refreshed by `engine/reality_snapshot.py`)
- **Purpose:** Give Gemini (Cloud Architect) an accurate, zero-hype inventory of what actually exists in `psycry/agent_smith` and `psycry/hive`, what is running right now, and what is **not** implemented.
- **Rule:** Never assume a staged or R&D item in the Google Doc Master Ledger is implemented in code unless it appears under **Implemented & Running** below.

---

## 1. Live Operational Posture (`THE-HIVE`)

- `nexus-state-bus.service`: `active`
- `agent-smith-dashboard.service`: `active`
- `agent-smith-sweep.timer`: `active`
- `agent-smith-watchdog.timer`: `active`
- **OCI Budget Guard:** scale_mode=`CONSERVE_LOCAL_FIRST`, cloud_burst_allowed=`True`, zero_overage_compliant=`True`

**What Agent Smith is NOT doing autonomously:**
- It is **not** autonomously reading Master Ledger milestones and writing/deploying arbitrary code on its own.
- `hive.self_improve` (`hive/self_improve.py`) is **on-demand only**, runs symbolic AST/Rust complexity scans first, tests edits in an isolated `git worktree` (`py_compile` / `cargo check`), and is ring-fenced against modifying core safety/constitutional files.
- Financial/trading/order actions are blocked behind `HITL_HOLD` (human-in-the-loop required).
- Contracted revenue is currently **`$0.00`** (`$405,000.00` unsubmitted federal SBIR ceilings + `$20,500.00` uncontracted SOW placeholders).

---

## 2. Verified Codebase Footprint (Scanned on Disk)

| Component | Language | Files | Lines of Code | Role |
| :--- | :--- | ---: | ---: | :--- |
| `agent_smith/claw-mcp-server/src` | Rust | 24 | 4,548 | MCP server (`v0.2.0`), `TaskRouter`, HumSynth DSP (`audio/`), Discord/CLI C2, sandboxing |
| `agent_smith/engine` | Python | 23 | 5,415 | Lead ingestion, 768-dim `pgvector` scoring, KEOV, dossiers, QA, ledger sync, dashboard API |
| `hive/hive` | Python | 15 | 2,086 | Provider adapters, `self_improve` worktree runner, `CONTEXT_CAPSULE.md`, IPC |

## 2b. Recently Implemented Master Ledger Modules

- **KEOV (`engine/keov.py`)**: Implemented (deterministic operator intent vectors in `agent_memories` `namespace='keov_operator_intent'`, offline JSONL checkpoint buffer `submissions/keov_offline_buffer.jsonl`, and `align_task_with_operator_vector()`).
- **Vocode-MIDI / HumSynth DSP Core (`claw-mcp-server/src/audio/`)**: Implemented in Rust (`src/audio/ingest.rs` zero-allocation ring buffer, `src/audio/pitch.rs` streaming YIN pitch detector, `src/audio/midi_mapper.rs` MIDI note/pitch-bend/CC & contour-to-prompt mapper).

---

## 3. Staged / Aspirational Items NOT Yet Implemented in Code

Do **not** assume Agent Smith can currently execute these:
- **Zero-Knowledge Proofs (ZKPs)** in Phronarch execution pipeline (R&D concept only).
- **Dynamic Self-Healing Agent Topologies & Decentralized Consensus State Sync** (R&D concepts only; current sync is single-master Git + Google Doc `INSTANT_TXT`).
- **Autonomous Proposal/PR Submission without Human Review** (packages and patch bundles are staged in `submissions/`, not auto-submitted to portals or upstream repos without operator review).
- **Full-Autonomy Multi-Step Code Generation from Ledger** (Agent Smith stays in deterministic monitoring/ingestion/budget-guard mode; complex architectural coding is handled by Antigravity or explicit human-triggered workflows).

---

## 4. Routing & Model Guidance

- **Deterministic / Zero-Token Path:** Status checks, ledger reads, OCI budget checks, lead scoring, and AST scans run symbolically without LLM tokens (`engine/router_bridge.py`).
- **Fast / Local Execution (`qwen2.5-coder:1.5b` / fast cloud models):** Day-to-day routing, lightweight classification, and bounded checks.
- **Daily Multi-Step Orchestration (`Orchestrator (Neon)` / `Gemini 4 Neon`):** Preferred for daily multi-step orchestration and coding bursts (daily-resetting quota), conserving weekly **`Orchestrator (Argon)`** quota for heavy architectural reviews.
- **Complex Architectural / Coding Work:** Escalate to Antigravity (local IDE agent) or an explicit top-level orchestrator (`Orchestrator (Argon)` for heavy architectural reviews) rather than expecting Agent Smith's background daemon to implement multi-file features from the ledger.
