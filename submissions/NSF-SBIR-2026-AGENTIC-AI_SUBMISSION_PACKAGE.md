# 📦 Staged NSF SBIR Phase I Submission Package: `NSF-SBIR-2026-AGENTIC-AI`

- **Opportunity ID:** `sbir_civic_rfp:NSF-SBIR-2026-AGENTIC-AI`
- **Lane:** `HIGH_TICKET_GRANT_RFP`
- **Title:** NSF SBIR Phase I: Autonomous Multi-Agent Workflows & Secure Sandboxed MCP Infrastructure
- **Matched Capability Anchor:** `agent_smith` (`fit_score = 0.4527`)
- **Grant Award Ceiling:** `\$305,000.00` (6-Month Phase I Execution)
- **Full Technical Dossier:** [`dossiers/HIGH_TICKET_GRANT_RFP/NSF-SBIR-2026-AGENTIC-AI.md`](../../dossiers/HIGH_TICKET_GRANT_RFP/NSF-SBIR-2026-AGENTIC-AI.md)
- **Package Status:** `STAGED_T012`

---

## 1. Executive Project Pitch & Technical Abstract (NSF Project Pitch / Portal Format)

**Technical Abstract:**  
Small and medium-sized enterprises (SMEs) adopting autonomous AI agents face two systemic barriers: prohibitive cloud token expenditure for continuous multi-step workflows, and severe security vulnerabilities (arbitrary command injection, unauthorized filesystem traversal, and credential exfiltration) inherent in unsandboxed tool-calling frameworks. **Digital Nomad LLC** proposes a 6-month NSF SBIR Phase I R&D project to advance **Agent Smith (`claw-mcp-server v0.2.0`)**, our operational **TRL-6/TRL-7 Rust Hybrid Multi-Agent Orchestrator**, into an open-standards, verifiable enterprise automation runtime. Built in **Rust (Edition 2024, Tokio async, RMCP 1.7)** coupled with a **Python 3.14 `SQLModel` worker plane** and **PostgreSQL 16 + `pgvector 0.8.7` (768-dim HNSW cosine indexing)**, the proposed system introduces three core technical innovations: (1) a deterministic **Hybrid Inference & Zero-Token Pre-Scoring Router** that pairs sub-millisecond `pgvector` capability matching and local small-language-model inference (`qwen2.5-coder:1.5b`) with multi-model cloud foundation fallbacks, cutting cloud API costs by $>70\%$; (2) a zero-trust **Rust Capability Sandbox (`SandboxConfig::resolve_safe_path`)** enforcing path canonicalization, binary allowlists, and non-blocking execution checkpoints across 17+ native Model Context Protocol (MCP) tools; and (3) a shared **768-Dimensional Collective Vector Memory** uniting compiled Rust control planes and Python analytical workers with zero schema drift. Phase I will benchmark latency, sandbox containment across 10,000 adversarial tool-invocation payloads, and end-to-end autonomous workflow execution for resource-constrained SMEs.

---

## 2. Specific Aims 1–3 & Quantitative Verification Matrix

| Specific Aim | Target TRL Progression | Core R&D Objective | Quantitative Phase I KPI | Existing Digital Nomad LLC TRL-6/7 Prior Art |
| :--- | :--- | :--- | :--- | :--- |
| **Aim 1: Hybrid Inference & Zero-Token Vector Pre-Scoring** | `TRL-6 -> TRL-7` | Formalize the 768-dim `pgvector` HNSW cosine pre-scoring engine and strongly-typed JSON router splitting local SLM (`qwen2.5-coder:1.5b`) vs. Cloud LLM workloads | $< 50\text{ms}$ semantic recall & routing latency; $\ge 70\%$ reduction in cloud LLM token burn on benchmark workflows | `/home/psycry/Projects/agent_smith` (`workforce.rs` + `engine/anchors.py` 768-dim embedding lock) |
| **Aim 2: Zero-Trust Rust MCP Sandbox & Adversarial Hardening** | `TRL-6 -> TRL-7` | Harden `SandboxConfig::resolve_safe_path`, binary allowlisting, and resource-bounded subprocess execution across all 17+ JSON-RPC stdio MCP tools | $0$ unhandled sandbox escapes or path-traversal bypasses across $10,000$ automated adversarial fuzzing payloads | `claw-mcp-server v0.2.0` (`rmcp 1.7.0`) path canonicalization & command whitelist engine |
| **Aim 3: Multi-Agent Collective Memory & SME Pilot Package** | `TRL-7 -> TRL-8` | Validate multi-role subagent dispatch (`scout`, `architect`, `operator`, `analyst`) over shared PostgreSQL `pgvector` state and release containerized SME deployment kit | $\ge 99.5\%$ idempotent checkpoint recovery across burst-mode restarts; complete Phase I Final Report & Phase II commercialization plan | Operational 2-lane autonomous business engine (`engine/ingest.py`, `engine/dossier.py`, `poll_bus.py`) |

---

## 3. NSF Phase I Budget Allocation Blueprint (`\$305,000.00` — 6 Months)

| Budget Category | Allocation (USD) | Share (%) | Scope & Justification |
| :--- | :--- | :--- | :--- |
| **Senior Personnel / Principal Investigator Labor** | `\$183,000.00` | `60.0%` | Lead Systems Architect / PI (Rust MCP control plane hardening, adversarial sandbox verification harness, hybrid router optimization across Months 1–6) |
| **R&D Engineering & Security Auditing Support** | `\$48,800.00` | `16.0%` | Automated red-team fuzzing harness (10,000 adversarial MCP payloads), benchmark dataset curation, and containerization packaging |
| **Compute, Vector Database & Cloud API Testbed** | `\$24,400.00` | `8.0%` | Multi-model foundation API benchmark quota, high-concurrency PostgreSQL 16 + `pgvector` HNSW stress testing, and edge GPU validation |
| **Indirect / F&A & Grant Compliance Administration** | `\$27,450.00` | `9.0%` | NSF Research.gov financial reporting, audit compliance, accounting, and IP management |
| **NSF SBIR Statutory Fee / Small Business Profit (7%)** | `\$21,350.00` | `7.0%` | Reinvested into Digital Nomad LLC autonomous compute infrastructure & Phase II commercialization |
| **Total Phase I Request** | **`\$305,000.00`** | **`100.0%`** | **6-Month NSF SBIR Phase I Execution** |

---

## 4. Pre-Submission Portal Checklist

- [x] **Technical Dossier Complete:** [`dossiers/HIGH_TICKET_GRANT_RFP/NSF-SBIR-2026-AGENTIC-AI.md`](../../dossiers/HIGH_TICKET_GRANT_RFP/NSF-SBIR-2026-AGENTIC-AI.md).
- [x] **Operational TRL-6/7 Codebase Verified:** `/home/psycry/Projects/agent_smith` (`target/release/claw-mcp-server`, 17 MCP tools verified).
- [x] **Entity Alignment:** Digital Nomad LLC (`BUSINESS_ENGINE_STATE.md`).
