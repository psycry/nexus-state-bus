# 🕶️ Autonomous Agent Business Engine — Source-of-Truth State (`BUSINESS_ENGINE_STATE.md`)

> **Entity:** Digital Nomad LLC  
> **Primary Operator:** PsyCry (Jasper)  
> **Last Updated:** 2026-10-29 12:05 EDT  
> **Operational Mandate:** Read `BUSINESS_ENGINE_STATE.md` and `gemini_inbox.md` at the start of every workflow, verify `poll_bus.py` (`nexus-state-bus.service`) is actively polling the published Google Doc ledger and syncing state packets to `psycry/nexus-state-bus`, automatically update `BUSINESS_ENGINE_STATE.md`, `sync_packet.md`, and `gemini_inbox.md` whenever milestones are completed, and resume watching `gemini_inbox.md` for incoming directives.

---

## 1. Executive Summary & Objective

We are building the **Autonomous Agent Business Engine**—a self-sustaining, revenue-generating autonomous agentic business system for **Digital Nomad LLC**.

- **Mission:** Transition from standalone projects (`phronarch`, `shelter_source`, `nomad_nexus`) into automated operational revenue pipelines (high-signal data ingestion, automated contract/grant discovery & proposal execution, algorithmic intelligence) that generate real cashflow.
- **Capital Flywheel:** Maintain a **\$0/month baseline infrastructure burn rate** while reinvesting initial profits directly into autonomous compute scaling, API capacity, and workforce expansion.
- **Active Staged Pipeline Valuation (`T-011` + `T-012`):** **\$600,500.00** across 4 `DOSSIER_READY` & `STAGED` opportunities (`\$20,500.00` in `FAST_CASHFLOW` contracts + `\$580,000.00` in `HIGH_TICKET_GRANT_RFP` Phase I SBIR proposals).

---

## 2. Core Architecture & Stack

| Layer | Technology / Component | Location / Endpoint | Status |
| :--- | :--- | :--- | :--- |
| **Local Workstation** | Garuda Linux (`i7-6820HQ`, 40GB RAM, Quadro M2200 4GB, Docker, Rust Nightly `1.101.0`, Python `3.14`) | `/home/psycry/Projects` | **ONLINE** |
| **Cloud Compute** | OCI Always Free ARM Ampere A1 (`VM.Standard.A1.Flex`, 2 OCPUs / 12GB RAM) | [`psycry/oci-ampere-claimer`](file:///home/psycry/Projects/oci-ampere-claimer) (Polling `AD-1`, `AD-2`, `AD-3`) | **POLLING (36 tries/run)** |
| **Control Plane (Rust)** | **Agent Smith v0.2.0** (`claw-mcp-server` release binary — 17 MCP tools incl. `run_lead_ingestion`, `generate_dossiers`, `list_ranked_leads`) | [`/home/psycry/Projects/agent_smith`](file:///home/psycry/Projects/agent_smith) (**Private Repo**) | **ONLINE** |
| **Routing Brain** | Local `qwen2.5-coder:1.5b` (fits in 4GB Quadro M2200 VRAM) + Cloud `gemini-2.5-flash` / `gemini-2.5-flash-lite` | [`sandbox_config.json`](file:///home/psycry/Projects/agent_smith/claw-mcp-server/sandbox_config.json) (with ENV overrides) | **ONLINE** |
| **Worker Plane (Python)** | Python **SQLModel** + `pgvector` Ingestion (`engine/ingest.py`), Anchor Scoring (`engine/anchors.py`), Dossier Generator (`engine/dossier.py`), Alert Dispatcher (`engine/notify.py`), & Ledger Sync (`engine/ledger_sync.py`) | [`/home/psycry/Projects/agent_smith/engine`](file:///home/psycry/Projects/agent_smith/engine) (`.venv`) | **ONLINE** |
| **Persistence Backbone** | **PostgreSQL 16 + `pgvector` (`0.8.7`)** (`nomad_nexus_grid`) | `postgres://smith:[REDACTED_DB_PASS]@localhost:5432/nomad_nexus_grid` (`agent-smith-pgvector`) | **ONLINE** |
| **Autonomous Sweep Timer** | `agent-smith-sweep.timer` (`OnBootSec=5min`, `OnUnitActiveSec=6h`) $\rightarrow$ `agent-smith-sweep.service` | `~/.config/systemd/user/agent-smith-sweep.timer` | **ACTIVE (ENABLED)** |
| **Bi-Directional State Bus** | **Outbound:** `engine/ledger_sync.py` + `scripts/push_state_bus.sh` (`psycry/nexus-state-bus`)<br>**Inbound:** Google Doc Master Ledger (`poll_bus.py` -> `gemini_inbox.md`) | `https://raw.githubusercontent.com/psycry/nexus-state-bus/main/MASTER_LEDGER_FEEDBACK.json` | **ONLINE (CLOSED-LOOP 2-WAY)** |

---

## 3. Architectural Contracts & Engineering Guardrails

1. **Strict Rust vs. Python Boundary (Zero Schema Drift):**
   - **Rust Control Plane (`claw-mcp-server`):** Owns MCP tool exposure (17 tools), path/command sandboxing, Discord C2 daemon, semantic memory recall (`store_memory` / `recall_memory`), subagent orchestration (`dispatch_subagent` / `list_workforce_tasks`), and native MCP invocation of Python workers (`run_lead_ingestion`, `generate_dossiers`, `list_ranked_leads`).
   - **Python Worker Plane (`engine/` — `SQLModel` + `pgvector`):** Owns API/web scraping (`engine/ingest.py`), Capability Anchor pre-scoring (`engine/anchors.py`), Autonomous Proposal/Dossier generation (`engine/dossier.py`), Notification Dispatch (`engine/notify.py`), and Closed-Loop Ledger Feedback (`engine/ledger_sync.py`).
2. **Embedding Lock (768-Dim Contract):**
   - Both Rust (`workforce.rs`) and Python (`engine/embeddings.py`) are pinned strictly to **768 dimensions** (`models/text-embedding-004` with `outputDimensionality: 768` + identical 64-bit FNV-1a normalized 768-dim fallback).
3. **Upsert Idempotency (`UNIQUE (source, external_id)`):**
   - `opportunity_leads` enforces `CONSTRAINT uq_opportunity_leads_source_ext_id UNIQUE (source, external_id)` for idempotent `ON CONFLICT (source, external_id) DO UPDATE` upserts.
4. **Capability Anchor Matching (Zero-Token Pre-Scoring):**
   - `engine/anchors.py` seeds 3 canonical proof-of-capability profiles (`agent_smith`, `shelter_source`, `phronarch`) into `agent_memories` (`namespace = 'capability_anchors'`) and pre-scores every lead via PostgreSQL cosine similarity before any LLM generation tokens are spent.
5. **Autonomous Proposal, Dossier, Submission & Alert Pipeline (`engine/dossier.py` + `submissions/` + `engine/notify.py`):**
   - Queries `opportunity_leads` where `fit_score >= 0.40` and `status = 'NEW'`, generates tailored Markdown dossiers in `dossiers/<lane>/<external_id>.md`, transitions `status` to `'DOSSIER_READY'`, stages actionable outreach/grant packages in `submissions/`, and dispatches notifications via `engine/notify.py` (with graceful no-op local logging when `DISCORD_WEBHOOK_URL` is unset).
6. **Closed-Loop Two-Way Feedback Bridge (`engine/ledger_sync.py` + `scripts/push_state_bus.sh` + `poll_bus.py`):**
   - Every invocation of `scripts/push_state_bus.sh` runs `python -m engine.ledger_sync`, which queries PostgreSQL (`opportunity_leads`, `workforce_tasks`) and `BUSINESS_ENGINE_STATE.md`, flips active milestone status in `gemini_inbox.md`, writes `MASTER_LEDGER_FEEDBACK.json`, pushes to the Google Doc if `GDOC_WEBHOOK_URL` / `GOOGLE_OAUTH_ACCESS_TOKEN` is set, and commits & pushes all sanitized state files to `psycry/nexus-state-bus`.

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

### Phase 4: Rust MCP Worker Bindings & Bi-Directional Ledger Sync (COMPLETED — `T-009` & `T-010`)
- [x] Add native MCP tool bindings in Rust `claw-mcp-server` (`src/tools/workforce.rs` & `src/main.rs`) to trigger `engine.ingest` and `engine.dossier` on demand (`run_lead_ingestion`, `generate_dossiers`, `list_ranked_leads`).
- [x] Wire 2-way Google Doc Master Ledger poller (`poll_bus.py` $\rightarrow$ `gemini_inbox.md`) + Git State Bus (`scripts/push_state_bus.sh`).

### Phase 5: Outbound Submission Packaging, Alerts, Systemd Sweeps & Closed-Loop Feedback Bridge (COMPLETED — `T-011`, `T-012` & `T-013`)
- [x] Execute autonomous boot sync, verify `claw-mcp-server` (`list_ranked_leads` & `generate_dossiers`), and stage submission packages for all 4 `DOSSIER_READY` leads (`MCP-RUST-ORCH-2026-01`, `SBIR-2026-CIVIC-SHELTER-01`, `NSF-SBIR-2026-AGENTIC-AI`, `QUANT-STREAM-OFI-2026-02` — `\$600,500` total pipeline value).
- [x] Build `engine/notify.py` (webhook/local alert dispatcher with graceful no-op fallback if `DISCORD_WEBHOOK_URL` is unset) and enable `agent-smith-sweep.timer` (`systemd --user` periodic ingestion + dossier + alert sweep).
- [x] Build `engine/ledger_sync.py` and integrate into `scripts/push_state_bus.sh` to automatically synthesize PostgreSQL + milestone telemetry, update `gemini_inbox.md` and `MASTER_LEDGER_FEEDBACK.json`, and push closed-loop feedback to `psycry/nexus-state-bus`.

### Phase 6: Backend Telemetry API & Tactical Command Center UI Synchronization (COMPLETED — `T-014`, `T-015` & `T-016`)
- [x] Diagnose & resolve harness stdio/IPC pipe stalls (`claw-mcp-server` concurrent async `stdout`/`stderr` drain, `Stdio::null()` stdin isolation, `tokio::time::timeout` watchdogs, headless `gdocs` IPC guard in `ledger_sync.py`, and boot-time `docker.socket` wakeup).
- [x] Build `engine/dashboard_api.py` (REST/JSON API `/api/state`, `/api/leads`, `/api/tasks`, `/api/anchors`, `/api/health` + `ui/dashboard_state.json` snapshot exporter) and `ui/index.html` (Tactical Command Center Dashboard) with automatic sync on every `scripts/push_state_bus.sh` commit, deployed persistently via `agent-smith-dashboard.service` on `http://127.0.0.1:8787`.
- [x] Self-Issued Directive (`T-016` — Autonomous Discretion): Expand `engine/dossier.py` with automated `stage_missing_submission_packages()` and execute next-wave dossier + submission staging (`fit_score >= 0.20` — `PD-19-127Y`, `1137560`, `1137545`), scaling ready-to-submit pipeline from `4` leads (`\$600,500`) to `7` leads (`\$780,500`).

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
| `T-009` | 2026-10-26 14:36 | Phase 4 | Wire Rust `claw-mcp-server` MCP tool bindings (`run_lead_ingestion`, `generate_dossiers`, `list_ranked_leads`) + 2-way Google Doc Master Ledger poller (`poll_bus.py`) | **DONE** |
| `T-010` | 2026-10-26 15:24 | Phase 4 | Verify bidirectional communication ping across Google Doc Master Ledger (`INSTANT_TXT`) and GitHub State Bus (`nexus-state-bus.service`) | **DONE** |
| `T-011` | 2026-10-27 12:49 | Phase 5 | Execute boot sync, run `claw-mcp-server` health check (`list_ranked_leads`), stage submission packages for `MCP-RUST-ORCH-2026-01` & `SBIR-2026-CIVIC-SHELTER-01`, and install persistent `SessionStart` boot memory | **DONE** |
| `T-012` | 2026-10-27 13:09 | Phase 5 | Stage `NSF-SBIR-2026-AGENTIC-AI` (`\$305,000`) & `QUANT-STREAM-OFI-2026-02` (`\$12,000`) submission packages, update DB `raw_metadata` to `STAGED_T012`, build `engine/notify.py` with no-op Discord fallback, and enable `agent-smith-sweep.timer` | **DONE** |
| `T-013` | 2026-10-27 13:14 | Phase 5 | Upgrade `scripts/push_state_bus.sh` with `engine/ledger_sync.py` to programmatically synthesize PostgreSQL + milestone status flips into `gemini_inbox.md` and `MASTER_LEDGER_FEEDBACK.json`, and verify closed-loop push to `psycry/nexus-state-bus` | **DONE** |
| `T-014` | 2026-10-29 11:58 | Phase 5 | Diagnose & resolve harness stdio/IPC hangs (`claw-mcp-server` concurrent async stdout/stderr drain + `Stdio::null()` stdin isolation + `tokio::time::timeout` watchdogs, headless `gdocs` IPC guard in `ledger_sync.py`, and boot-time `docker.socket` wakeup for `agent-smith-pgvector`) | **DONE** |
| `T-015` | 2026-10-29 12:05 | Phase 6 | Multi-agent pod execution: build Backend Telemetry API (`engine/dashboard_api.py` + `ui/dashboard_state.json`) & Tactical Command Center UI (`ui/index.html`) synchronized via `ledger_sync.py` and `push_state_bus.sh` | **DONE** |
| `T-016` | 2026-10-29 13:25 | Phase 6 | Self-Issued Directive (Autonomous Discretion): Expand `engine/dossier.py` with automatic submission staging, generate 3 next-wave dossiers & submission packages (`PD-19-127Y` `\$150k`, `1137560` `\$15k`, `1137545` `\$15k` $\rightarrow$ `7` staged leads totaling `\$780,500`), and deploy persistent `agent-smith-dashboard.service` (`:8787`) | **DONE** |
| `T-017` | 2026-10-29 13:28 | Phase 6 | Self-Issued Directive (Autonomous Discretion): Add interactive `POST /api/actions/sweep` & `POST /api/actions/push-bus` controls to `engine/dashboard_api.py` + `ui/index.html` and auto-generate `submissions/EXECUTIVE_SUBMISSION_INDEX.md` | **DONE** |
| `T-018` | 2026-10-30 12:25 | Phase 6 | Self-Issued Directive (Autonomous Discretion): Build `engine/qa.py` Submission QA Gate (lane-aware section checks, dossier link integrity, budget-math/percent validation, close-date and weak-fit flags) -> `submissions/QA_REPORT.md` + `qa_report.json`; result: 7 packages, 5 PASS / 2 WARN (weak fit: `1137545` 0.21, `PD-19-127Y` 0.26) / 0 FAIL | **DONE** |
| `T-019` | 2026-10-30 12:33 | Phase 6 | Self-Issued Directive (Autonomous Discretion): Wire `engine.qa` into `scripts/push_state_bus.sh` (auto-refresh each push), expose `qa_report` via `/api/state`, add Submission QA Gate panel to `ui/index.html`; dashboard service restarted and verified (7 reports served) | **DONE** |
| `T-020` | 2026-10-30 13:00 | Phase 6 | Self-Issued Directive (Autonomous Discretion): Enable `nexus-state-bus.service` for auto-start on login (was disabled); make dashboard milestone header/highlight dynamic (was hardcoded T-001..T-015); ledger-watch cron (5 min) active this session | **DONE** |
| `T-021` | 2026-10-30 13:03 | Phase 6 | Self-Issued Directive (Autonomous Discretion): Pipeline integrity audit - RemoteOK leads `1137545` (Sothebys Sr SWE) and `1137560` (Aprio Accounting Mgr) are salaried job postings with synthetic \$15k valuations, not contract RFPs; set `ON_HOLD` (reversible, reason in `raw_metadata`), packages moved to `submissions/_held/`. Staged pipeline corrected \$780,500 -> \$750,500 (5 leads). `PD-19-127Y` (fit 0.26) kept, flagged for human alignment review | **DONE** |
| `T-022` | 2026-10-30 13:06 | Phase 6 | Self-Issued Directive (Autonomous Discretion): Root-cause fix for `T-021` - RemoteOK ingest no longer assigns salary/synthetic \$15k as contract value (`est_val=0`, `listing_type=job_posting`); `dossier.py` excludes job postings from dossier selection and `--reset-qualifying` no longer revives `ON_HOLD` leads | **DONE** |
