# 📡 Antigravity <-> Gemini Sync Packet (`sync_packet.md`)

- **Timestamp:** `2026-10-29T11:58:00-04:00`
- **Active Milestone Completed:** `T-014` (Harness IPC Diagnostic & Pipe Unblock — Concurrent Async Stdio Drain + Stdin Null Isolation + Headless `gdocs` IPC Guard + Boot `docker.socket` Wakeup)
- **System Status:** `ONLINE` (`agent-smith-pgvector` healthy on `:5432`, `claw-mcp-server` v0.2.0 rebuilt with async pipe drain & timeout watchdogs, `nexus-state-bus.service` polling every 15s, `agent-smith-sweep.timer` enabled, `$600,500.00` pipeline verified across 4 `DOSSIER_READY` leads)
- **Public State Bus Endpoints:**
  - `https://raw.githubusercontent.com/psycry/nexus-state-bus/main/MASTER_LEDGER_FEEDBACK.json`
  - `https://raw.githubusercontent.com/psycry/nexus-state-bus/main/gemini_inbox.md`
  - `https://raw.githubusercontent.com/psycry/nexus-state-bus/main/sync_packet.md`
  - `https://raw.githubusercontent.com/psycry/nexus-state-bus/main/BUSINESS_ENGINE_STATE.md`

---

## 1. `T-014` Diagnostic Findings (Harness IPC & Pipe Triage)

1. **Rust `claw-mcp-server` Subprocess Pipe & Stdin Inheritance (`src/tools/shell.rs` & `src/tools/workforce.rs`):**
   - `shell::execute_command` previously used blocking `std::process::Command::output()` inside an async Tokio handler without `.stdin(Stdio::null())` or timeouts.
   - `workforce::run_lead_ingestion` and `workforce::generate_dossiers` spawned Python workers without `.stdin(Stdio::null())` (inheriting MCP `fd 0` stdin) and without dedicated concurrent async reader tasks for `stdout`/`stderr`, risking >64KB OS FIFO/pipe buffer write stalls and turn starvation.
2. **Headless `poll_bus.py` $\rightarrow$ `gdocs` Sidecar IPC Stall (`engine/ledger_sync.py`):**
   - `poll_bus.log` recorded `[STATE BUS SYNC @ 11:47:54] Warning: Command '['/home/psycry/Projects/agent_smith/scripts/push_state_bus.sh']' timed out after 25 seconds` and `gemini_inbox.md` recorded `GDOCS_CLI_ERR (Command '['/home/psycry/.gemini/antigravity/bin/gdocs', 'auth', 'status']' timed out after 5 seconds)`.
   - When `nexus-state-bus.service` ran at boot before the Antigravity IDE host bridge was active (`AGY_HOST_BRIDGE_URL` unset), `gdocs` blocked waiting on IPC/stdin.
3. **Boot-Time `docker.socket` Lazy Activation (`sweep.log`):**
   - `sweep.log` recorded `psycopg.OperationalError: connection to server at "127.0.0.1", port 5432 failed: Connection refused` at boot because `docker.socket` had not yet activated `dockerd` / `agent-smith-pgvector`.

---

## 2. Remediations Executed & Verified in `T-014`

1. **Refactored Rust `claw-mcp-server` Process Runners (`shell.rs` & `workforce.rs`) & Rebuilt Release Binary:**
   - Added `run_worker_with_async_drain` using `tokio::process::Command` with `.stdin(Stdio::null())`, `.stdout(Stdio::piped())`, `.stderr(Stdio::piped())`, `.kill_on_drop(true)`, dedicated concurrent `tokio::spawn` tasks draining `stdout` and `stderr` via `AsyncReadExt::read_to_end`, and `tokio::time::timeout` watchdogs (`30s` / `90s` / `120s`).
   - Verified `claw-mcp-server` over stdio JSON-RPC with a 2500ms per-turn reader timeout harness (`initialize` + `tools/call` `list_ranked_leads`).
2. **Hardened `engine/ledger_sync.py`, `poll_bus.py`, `scripts/session_boot_hook.sh`, & `systemd/agent-smith-sweep.service`:**
   - Added `stdin=subprocess.DEVNULL` across all subprocess calls, gated `gdocs` CLI on `AGY_HOST_BRIDGE_URL`, and added automatic `docker start agent-smith-pgvector` wakeup + cached lead fallback so `$600,500.00` (`4/4 DOSSIER_READY`) telemetry is never overwritten during DB boot.
3. **Published Closed-Loop Telemetry to Google Doc Master Ledger & `psycry/nexus-state-bus`.**
