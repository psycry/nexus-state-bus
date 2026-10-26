# 🕶️ Autonomous Agent Business Engine — Source-of-Truth State (`BUSINESS_ENGINE_STATE.md`)

> **Entity:** Digital Nomad LLC  
> **Primary Operator:** PsyCry (Jasper)  
> **Last Updated:** 2026-10-26 14:10 EDT  
> **Operational Mandate:** Read `BUSINESS_ENGINE_STATE.md` at the start of every workflow, automatically update both `BUSINESS_ENGINE_STATE.md` and `sync_packet.md` whenever milestones are completed or system architecture changes, and execute `scripts/push_state_bus.sh` to publish sanitized state to `psycry/nexus-state-bus`.

---

## 1. Executive Summary & Objective

We are building the **Autonomous Agent Business Engine**—a self-sustaining, revenue-generating autonomous agentic business system for **Digital Nomad LLC**.

- **Mission:** Transition from standalone projects (`phronarch`, `shelter_source`, `nomad_nexus`) into automated operational revenue pipelines (high-signal data ingestion, automated contract/grant discovery & proposal execution, algorithmic intelligence) that generate real cashflow.
- **Capital Flywheel:** Maintain a **\$0/month baseline infrastructure burn rate** while reinvesting initial profits directly into autonomous compute scaling, API capacity, and workforce expansion.

---

## 2. Core Architecture & Stack

| Layer | Technology / Component | Location / Endpoint | Status |
| :--- | :--- | :--- | :--- |
| **Local Workstation** | Garuda Linux (`i7-6820HQ`, 40GB RAM, Quadro M2200 4GB, Docker, Rust Nightly `1.101.0`, Python `3.14`) | `/home/psycry/Projects` | **ONLINE** |
| **Cloud Compute** | OCI Always Free ARM Ampere A1 (`VM.Standard.A1.Flex`, 2 OCPUs / 12GB RAM) | [`psycry/oci-ampere-claimer`](file:///home/psycry/Projects/oci-ampere-claimer) (Polling `AD-1`, `AD-2`, `AD-3`) | **POLLING (36 tries/run)** |
| **Control Plane (Rust)** | **Agent Smith v0.2.0** (`claw-mcp-server` in Rust) — MCP Server, Sandbox Guardrails, Subagent Dispatch & Discord Daemon | [`/home/psycry/Projects/agent_smith`](file:///home/psycry/Projects/agent_smith) (**Private Repo**) | **ONLINE** |
| **Routing Brain** | Local `qwen2.5-coder:1.5b` (fits in 4GB Quadro M2200 VRAM) + Cloud `gemini-2.5-flash` / `gemini-2.5-flash-lite` | [`sandbox_config.json`](file:///home/psycry/Projects/agent_smith/claw-mcp-server/sandbox_config.json) (with ENV overrides) | **ONLINE** |
| **Worker Plane (Python)** | Python **SQLModel** + `pgvector` Ingestion (`engine/ingest.py`), Anchor Scoring (`engine/anchors.py`) & Dossier Generator (`engine/dossier.py`) | [`/home/psycry/Projects/agent_smith/engine`](file:///home/psycry/Projects/agent_smith/engine) (`.venv`) | **ONLINE** |
| **Persistence Backbone** | **PostgreSQL 16 + `pgvector` (`0.8.7`)** (`nomad_nexus_grid`) | `postgres://smith:[REDACTED_DB_PASS]@localhost:5432/nomad_nexus_grid` (`agent-smith-pgvector`) | **ONLINE** |
| **Git State Bus** | Sanitized Public State & Sync Bus (`psycry/nexus-state-bus`) via [`scripts/push_state_bus.sh`](file:///home/psycry/Projects/agent_smith/scripts/push_state_bus.sh) | `https://raw.githubusercontent.com/psycry/nexus-state-bus/main/sync_packet.md` | **ONLINE** |

---

## 3. Architectural Contracts & Engineering Guardrails

1. **Strict Rust vs. Python Boundary (Zero Schema Drift):**
   - **Rust Control Plane (`claw-mcp-server`):** Owns MCP tool exposure to Antigravity, path/command sandboxing, Discord C2 daemon, semantic memory recall (`store_memory` / `recall_memory`), and subagent orchestration (`dispatch_subagent` / `list_workforce_tasks`).
   - **Python Worker Plane (`engine/` — `SQLModel` + `pgvector`):** Owns API/web scraping (`engine/ingest.py`), Capability Anchor pre-scoring (`engine/anchors.py`), and Autonomous Proposal/Dossier generation (`engine/dossier.py`).
2. **Embedding Lock (768-Dim Contract):**
   - Both Rust (`workforce.rs`) and Python (`engine/embeddings.py`) are pinned strictly to **768 dimensions** (`models/text-embedding-004` with `outputDimensionality: 768` + identical 64-bit FNV-1a normalized 768-dim fallback).
3. **Upsert Idempotency (`UNIQUE (source, external_id)`):**
   - `opportunity_leads` enforces `CONSTRAINT uq_opportunity_leads_source_ext_id UNIQUE (source, external_id)` for idempotent `ON CONFLICT (source, external_id) DO UPDATE` upserts.
4. **Capability Anchor Matching (Zero-Token Pre-Scoring):**
   - `engine/anchors.py` seeds 3 canonical proof-of-capability profiles (`agent_smith`, `shelter_source`, `phronarch`) into `agent_memories` (`namespace = 'capability_anchors'`) and pre-scores every lead via PostgreSQL cosine similarity before any LLM generation tokens are spent.
5. **Autonomous Proposal & Dossier Pipeline (`engine/dossier.py`):**
   - Queries `opportunity_leads` where `fit_score >= 0.40` and `status = 'NEW'`, generates tailored Markdown dossiers in `dossiers/<lane>/<external_id>.md` via the `gemini-2.5-flash` fallback chain (or deterministic blueprint fallback), transitions `status` to `'DOSSIER_READY'`, records `dossier_path` in `raw_metadata`, and logs completion to `workforce_tasks`.
6. **Public Git State Bus (`scripts/push_state_bus.sh`):**
   - Automatically sanitizes and publishes `BUSINESS_ENGINE_STATE.md` and `sync_packet.md` to `https://github.com/psycry/nexus-state-bus` at the end of every milestone so Gemini can read live project state without manual copy/pasting.

---

## 4. Roadmap Phases

### Phase 1: Foundation, Migration, Hardening & Storage Backbone (COMPLETED)
- [x] Migrate Windows/WSL2 repositories, SSH keys, OCI credentials, and GitHub tokens to Garuda Linux (`/home/psycry/Projects`).
- [x] Audit & harden `oci-ampere-claimer` (all 3 Ashburn ADs, 12-cycle loop, `--no-retry` HTTP 500 fix, duplicate-instance guard).
- [x] Initialize Rust nightly toolchain (`1.101.0-nightly`), deploy `agent-smith-pgvector`, and lock down `psycry/agent_smith` as a private repo.

### Phase 2: SQLModel Persistence Layer & 2-Lane Ingestion Engine (COMPLETED)
- [x] Scaffold Python `SQLModel` + `pgvector` package (`engine/config.py`, `embeddings.py`, `models.py`, `anchors.py`, `ingest.py`).
- [x] Implement 768-dim Embedding Lock, `UNIQUE (source, external_id)` idempotent upserts, and Zero-Token Capability Anchor Pre-Scoring.

### Phase 3: Autonomous Proposal & Capability Dossier Generator + Git State Bus (COMPLETED)
- [x] Create public Git State Bus (`psycry/nexus-state-bus`) and sanitizer/publisher script (`scripts/push_state_bus.sh`).
- [x] Build `engine/dossier.py` generating tailored Markdown execution briefs (`dossiers/FAST_CASHFLOW/*.md`) and SBIR Specific Aims dossiers (`dossiers/HIGH_TICKET_GRANT_RFP/*.md`) for leads with `fit_score >= 0.40` and `status = 'NEW'`, transitioning them to `'DOSSIER_READY'`.

### Phase 4: Rust MCP Worker Bindings & Autonomous Scheduling (NEXT — `T-009`)
- [ ] Add native MCP tool bindings in Rust `claw-mcp-server` (`src/tools/workforce.rs` & `src/main.rs`) to trigger `engine.ingest` and `engine.dossier` on demand or schedule (`run_ingestion_sweep`, `generate_dossiers`, `list_opportunity_leads`).
- [ ] Offload 24/7 ingestion & dossier cron workers + Discord review notifications upon OCI Ampere A1 provisioning.

---

## 5. Active Task Log

| ID | Timestamp (EDT) | Phase | Task Description | Status |
| :--- | :--- | :--- | :--- | :--- |
| `T-001` | 2026-10-26 12:47 | Phase 1 | Migrate Windows/WSL2 credentials, SSH keys, OCI keys, and 31 repos to Garuda Linux | **DONE** |
| `T-002` | 2026-10-26 13:09 | Phase 1 | Audit & upgrade `oci-ampere-claimer` workflow (3 Ashburn ADs, 12-cycle loop, `--no-retry`) | **DONE** |
| `T-003` | 2026-10-26 13:25 | Phase 1 | Deploy `agent-smith-pgvector` (PostgreSQL 16 + `pgvector` 0.8.7) Docker backbone | **DONE** |
| `T-004` | 2026-10-26 13:30 | Phase 1 | Upgrade `claw-mcp-server` to v0.2.0 with `store_memory`, `recall_memory`, `dispatch_subagent`, `list_workforce_tasks` | **DONE** |
| `T-005` | 2026-10-26 13:31 | Phase 1 | Establish `BUSINESS_ENGINE_STATE.md` as workspace source-of-truth | **DONE** |
| `T-006` | 2026-10-26 13:43 | Phase 1 | Apply 5 architectural upgrades (`psycry/agent_smith` private, `qwen2.5-coder:1.5b`, ENV overrides, `opportunity_leads` schema, OCI PAYG doc) | **DONE** |
| `T-007` | 2026-10-26 13:53 | Phase 2 | Build Python `SQLModel` + `pgvector` worker package (`engine/`), 768-dim lock, `UNIQUE (source, external_id)` upserts, 3 Capability Anchors, and Lane 1 / Lane 2 ingestion runners | **DONE** |
| `T-008` | 2026-10-26 14:10 | Phase 3 | Build Autonomous Proposal & Capability Dossier Generator (`engine/dossier.py`), generate dossiers under `dossiers/<lane>/<external_id>.md`, and establish `psycry/nexus-state-bus` (`scripts/push_state_bus.sh`) | **DONE** |
| `T-009` | — | Phase 4 | Add Rust `claw-mcp-server` MCP tool bindings to trigger `engine.ingest` and `engine.dossier` on demand or schedule | **READY** |
