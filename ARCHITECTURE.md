# ARCHITECTURE.md — The Hive Sovereign Control Plane & Cross-Node State Bus

> **Read this in full before modifying services, databases, routing, or deployment automation.**
> Built so any engineer or future AI model (local or cloud) can understand every component, why every guardrail exists, and how to run completely off-grid on owned hardware if external providers vanish or tighten restrictions.

---

## 1. System Overview & Strategic Design

Our ecosystem splits cleanly into three decoupled systems:

1. **The Hive (`/opt/hive` on cloud A1 / `~/Projects/hive` on dev):**
   - Zero-dependency Python 3 control plane, append-only SQLite WAL event ledger (`ledger.db`), JSON-RPC 2.0 gateway (`127.0.0.1:8799`), Tier 0–3 provider-agnostic router (`local_router.py` + `providers.py`), heartbeat watchdog (`watchdog.py`), financial dry-run circuit breaker (`financial.py`), and 2-way cloud state-bus daemon (`perpetual.py`).
2. **Phronarch (`/root/phronarch` on cloud A1 / `~/Projects/phronarch` on dev — OpSec codename `Module-P4`):**
   - Multi-service algorithmic intelligence & market observer (`redis`, Go `scout` ingesting real RSS/Reddit feeds, Node/TS `execution` engine in `OBSERVER` mode with L2 orderbook slippage guard & round-trip fee accounting, and opt-in `dashboard` Vite UI).
   - Stores trading/market state strictly in its own SQLite DB (`data/phronarch.db` `trades` table).
3. **Agent Smith & Nexus State Bus (`~/Projects/agent_smith` + `psycry/nexus-state-bus`):**
   - Local Garuda workstation PostgreSQL 16 + `pgvector` lead/grant pipeline, tactical dashboard (`127.0.0.1:8787`), and sanitized public Git synchronization bus (`psycry/nexus-state-bus`) bridging Antigravity, Gemini, and A1.

---

## 2. Critical Database Separation (Why `phronarch.db` != `ledger.db`)

| Database | Path | Table(s) | Purpose | Never Put Here |
|---|---|---|---|---|
| **The Hive Ledger** | `/opt/hive/ledger.db` (SQLite WAL, `no_update`/`no_delete` triggers) | `events(id, ts, node, type, payload, hash)` | Immutable control-plane events: `HEARTBEAT`, `HIVE_PING`/`HIVE_PONG`, `TASK_QUEUED`, `HITL_HOLD`, `PROVIDER_GENERATE`, `WORKER_VERIFY` | Market ticks or exchange keys |
| **Phronarch Ledger** | `/root/phronarch/data/phronarch.db` | `trades`, headlines, market state | Paper/`OBSERVER` & market execution history for `Module-P4` | Synthetic `$0.00` ping/test rows (`HELLO_HIVE_BUS_CHECK`) — breaks PnL/stop math |

---

## 3. Firewall-Proof 2-Way Cloud Communication Loop

Public/campus/library networks frequently run Layer-7 DPI that kills outbound SSH (`:22`, `:443` `ssh.github.com`) or blocks uncategorized raw server IP addresses. We solved this **without weakening host firewalls or leaving public ports open**:

```
  [Phone / Web Gemini or Operator]
          │  writes `HIVE_PING: <msg>` or `Active Milestone: T-XXX`
          ▼
  Google Doc Master Ledger (HTTPS /export?format=txt + /pub fallback)
          │  polled every 60s by `hive-perpetual.service` on Cloud A1
          ▼
  hive/perpetual.py ──▶ local Ollama (`qwen2.5-coder:1.5b`) classifies & routes
          │             (HITL gates hold financial/infra tasks cleanly)
          ▼
  /opt/hive/ledger.db (SQLite WAL append-only SHA-256 hashed row)
          │  OpSec-masked + regex-scrubbed (zero IPs, keys, OCIDs)
          ▼
  Exports `HIVE_LEDGER.md` + `CONTEXT_CAPSULE.md` + `SOUL.md` + `ARCHITECTURE.md`
          │  `git pull --rebase --autostash && git push origin main`
          ▼
  https://github.com/psycry/nexus-state-bus (readable anywhere over HTTPS)
```

### Anti-Overwrite Sync Rule
Both A1 (`hive/perpetual.py`) and the Garuda workstation (`agent_smith/scripts/push_state_bus.sh`) push to `psycry/nexus-state-bus` (`main`). **Never** use `git push -f`. Both publishers always execute `git pull --rebase --autostash origin main` before committing and before pushing so A1's `HIVE_LEDGER.md`/`CONTEXT_CAPSULE.md` commits and laptop `BUSINESS_ENGINE_STATE.md`/`submissions/` commits coexist cleanly.

---

## 4. Security & Hardening Invariants (Never Regress These)

1. **Zero Public Control-Plane Exposure:** `hive-gateway.service` binds strictly to `127.0.0.1:8799` (unauthenticated loopback JSON-RPC), Phronarch `execution` & `dashboard` bind to `127.0.0.1:3000` / `127.0.0.1:5173` (Vite dashboard OFF by default), and `hive-ollama` binds to `127.0.0.1:11434`.
2. **OCI Perimeter Minimalism:** Cloud security list allows `0.0.0.0/0` TCP `22` (key-only SSH) + essential ICMP only. Never leave public NAT IPs whitelisted after mobile/library sessions.
3. **Host Sudo & Hygiene:** `/etc/sudoers.d/ocarun` must stay deleted after OCI Run Command administrative updates; SELinux remains enforcing (`setenforce 1`).
4. **Remote Access (`scripts/hive-tunnel.sh`):** Use Tailscale WireGuard (`ssh opc@the-hive-a1` or loopback port forwards), an authenticated outbound `cloudflared` tunnel, or `ssh -L` when on permissive networks — never expose gateway/dashboard ports on `0.0.0.0`.

---

## 5. Cloud-Agnostic Provider Architecture (`hive/providers.py`)

If Gemini, Claude, or OpenAI APIs rate-limit, censor, price-hike, or shut down, **The Hive continues operating locally with zero code rewrites**:

| Tier | Default Model / Runtime | Cost | Purpose |
|---|---|---|---|
| **Tier 0** | `ollama` (`qwen2.5-coder:1.5b` -> fallback `qwen2.5:0.5b`) | \$0 | Classification, triage, formatting, watchdog, deterministic helpers |
| **Tier 1** | `ollama` (larger local weights e.g. `qwen2.5-coder:7b`) or `openai_compat` (`llama.cpp`/`vLLM`) | \$0 | Shell diagnostics, single-file diffs, local autonomous daemon (`worker.py`) |
| **Tier 2 / 3** | Optional burst (`gemini`, `anthropic`, or any `HIVE_OPENAI_BASE_URL`) | API / \$0 local | Multi-file design/audit — automatically falls back to local `ollama` on any error or unset key |

Configure local or alternate providers purely via environment variables:
- `HIVE_OLLAMA_URL` (default `http://127.0.0.1:11434`) & `HIVE_OLLAMA_MODEL` (default `qwen2.5-coder:1.5b`)
- `HIVE_OPENAI_BASE_URL`, `HIVE_OPENAI_MODEL`, `HIVE_OPENAI_API_KEY` (`llama.cpp` server, `vLLM`, `LM Studio`, Groq, OpenRouter, etc.)
- `GEMINI_API_KEY` / `ANTHROPIC_API_KEY` (optional cloud burst; automatically scrubbed via `scrub_external()`)

---

## 6. Autonomous Daemon Safety Contract (`hive/worker.py`)

When running autonomous background coding tasks on the server (`hive-agent.service` / `worker.py`):
1. Every task passes through `router.route()` + `worker.check_safety()`. Anything hitting `financial` or `infrastructure` HITL gates immediately records `HITL_HOLD` and stops.
2. Code edits are strictly isolated to dedicated git branches (`hive/auto-<task_id>`), **never** `main`/`master`.
3. Every change must pass automated verification (`worker.run_verification()`) before opening a review summary or requesting human sign-off (`handoff.py`).

---

## 7. Quickstart / Disaster Recovery Commands

```bash
# 1. Print portable persona (SOUL.md) + live node state (CONTEXT_CAPSULE.md) for any LLM:
python3 -m hive.perpetual --boot-prompt

# 2. Send a local handshake ping through Ollama -> SQLite WAL -> HIVE_LEDGER.md + CONTEXT_CAPSULE.md:
python3 -m hive.perpetual --ping "STATUS_CHECK" --src "manual_cli"

# 3. Query local gateway status (loopback only):
curl -s http://127.0.0.1:8799/health && curl -s http://127.0.0.1:8799/status

# 4. Run Gateway / Perpetual loop standalone on any Linux host with Python 3.9+:
python3 -m hive.gateway &
python3 -m hive.perpetual
```
