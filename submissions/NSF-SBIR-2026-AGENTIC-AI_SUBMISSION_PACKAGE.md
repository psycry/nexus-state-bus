# 📦 NSF Project Pitch Package: `NSF-SBIR-2026-AGENTIC-AI`

- **Opportunity ID:** `sbir_civic_rfp:NSF-SBIR-2026-AGENTIC-AI`
- **Lane:** `HIGH_TICKET_GRANT_RFP`
- **Solicitation:** **NSF 26-510** — America's Seed Fund, SBIR/STTR Phase I / Phase II / Fast-Track
- **Title:** Secure, Sandboxed Hybrid AI Orchestration (Rust MCP + pgvector) for Small-Business Automation
- **Source / Portal URL:** https://seedfund.nsf.gov
- **Listing Type:** `live_solicitation` (placeholder tag removed, T-029)
- **Matched Capability Anchor:** `agent_smith` (`fit_score = 0.2741`)
- **Grant Award Ceiling:** `\$305,000.00` (Phase I)
- **Close Date:** 03/04/2027 (next window; 5:00 PM local time). Other windows: 11/04/2026 (passed), 07/07/2027.
- **Full Technical Dossier:** [`dossiers/HIGH_TICKET_GRANT_RFP/NSF-SBIR-2026-AGENTIC-AI.md`](../dossiers/HIGH_TICKET_GRANT_RFP/NSF-SBIR-2026-AGENTIC-AI.md)
- **Package Status:** `PROJECT_PITCH_DRAFT` (`T-029`) — **a Project Pitch is required, and NSF must invite a full proposal before one can be submitted.**

> [!NOTE]
> Solicitation number, ceiling, deadlines and the Project Pitch requirement were supplied by the applicant from seedfund.nsf.gov on 2026-11-05. The seedfund.nsf.gov pages independently confirmed only that Project Pitches are required. Verify pitch section names, word limits and the "Topic Code" field in the NSF questionnaire against the live pitch form before submitting.

---

## 1. Executive Project Pitch (Draft Text for the NSF Project Pitch Form)

### 1.1 Technology Innovation
Small businesses want AI agents that act on their systems, but today's agent stacks run tools with broad host access, depend entirely on cloud models, and keep no durable shared memory. **Digital Nomad LLC** is building a secure orchestration layer that addresses all three:

- **Deterministic sandboxed tool execution (Rust):** every tool call passes path canonicalization and a command allowlist before it runs, so an agent cannot read or write outside declared roots.
- **Hybrid inference routing:** a local model (Ollama, `qwen2.5-coder:1.5b`) handles low-cost work and cloud foundation models (Gemini Flash family) handle the rest, with automatic fallback.
- **Collective vector memory:** PostgreSQL 16 + `pgvector` 768-dimension semantic memory shared by the Rust control plane and a Python worker plane under a fixed embedding contract.

The working base is **Agent Smith** (`claw-mcp-server`, a Rust Model Context Protocol server exposing 17 tools). The innovation to fund is the part that is not yet built or proven: measurable security guarantees, routing policy that optimizes cost against quality, and multi-agent coordination over shared memory.

### 1.2 Technical Objectives and Challenges
1. **Sandbox assurance:** define a tool-call policy model and test it with adversarial prompt-injection and path-traversal suites. Target: zero boundary violations across the test corpus.
2. **Cost-aware routing:** learn a routing policy between local and cloud models. Target: a defined reduction in cloud-token spend at equal task success on a benchmark task set.
3. **Multi-agent memory coordination:** concurrent agents reading and writing shared `pgvector` memory without stale or conflicting recall. Target: measured recall precision and latency under concurrent load.

Principal technical risks: prompt-injection that survives allowlisting, and quality loss from routing to small local models. Phase I exists to quantify both.

### 1.3 Market Opportunity
Customers are small and mid-sized businesses and the integrators who serve them, who need agent automation but cannot accept unrestricted tool access or per-token cloud costs. Initial go-to-market is fixed-scope integration engagements (the Lane 1 contract work this engine already tracks), moving to a licensed or hosted orchestration product. *(Applicant to add sourced market-size figures and named design-partner conversations before submission.)*

### 1.4 Company and Team
**Digital Nomad LLC** — principal: Jasper (PsyCry), Principal Systems Architect. Relevant existing work: Agent Smith (Rust MCP server, hybrid routing, `pgvector` memory), plus Phronarch and Shelter Source as supporting engineering evidence. *(Applicant to add biography, SBIR eligibility facts, and SAM.gov / SBA registration status.)*

---

## 2. Specific Aims 1–3 & Quantitative Verification Matrix

| Specific Aim | Target TRL Progression | Core R&D Objective | Quantitative Phase I KPI | Existing Prior Art |
| :--- | :--- | :--- | :--- | :--- |
| **Aim 1: Verified Sandbox Policy** | `TRL-4 -> TRL-6` | Formal tool-call policy plus adversarial test harness | 0 boundary violations on the adversarial corpus | `claw-mcp-server` sandbox (`resolve_safe_path`, command allowlist) |
| **Aim 2: Cost-Aware Hybrid Routing** | `TRL-4 -> TRL-6` | Routing policy between local and cloud models | Cloud-token reduction at equal task success on benchmark set | Ollama + Gemini fallback router in `claw-mcp-server` |
| **Aim 3: Multi-Agent Shared Memory** | `TRL-4 -> TRL-6` | Concurrency-safe shared `pgvector` memory | Recall precision and p95 latency under N concurrent agents | `store_memory` / `recall_memory`, 768-dim HNSW index |

*(TRL starting points are conservative estimates for the unbuilt research components; adjust to your own assessment.)*

---

## 3. Indicative Phase I Budget (For the Full Proposal, After Invitation)

A Project Pitch does not include a budget. This table is a planning draft against the `\$305,000` ceiling.

| Budget Category | Allocation (USD) | Share (%) | Scope & Justification |
| :--- | :--- | :--- | :--- |
| **Senior Personnel / PI Labor** | `\$183,000.00` | `60.0%` | PI / lead architect across the Phase I period |
| **R&D Engineering & Verification Harness** | `\$48,800.00` | `16.0%` | Adversarial test suites, routing benchmarks, integration testing |
| **Compute, Vector DB & Cloud API Testbed** | `\$24,400.00` | `8.0%` | Concurrency testbed, multi-model evaluation |
| **Indirect / F&A & Compliance** | `\$27,450.00` | `9.0%` | Reporting, audit, accounting, IP |
| **Statutory Small Business Fee (7%)** | `\$21,350.00` | `7.0%` | Fee permitted under SBIR rules; confirm against the solicitation |
| **Total Phase I Request** | **`\$305,000.00`** | **`100.0%`** | **Phase I** |

---

## 4. Pre-Submission Checklist

- [x] **Technical Dossier Drafted:** [`dossiers/HIGH_TICKET_GRANT_RFP/NSF-SBIR-2026-AGENTIC-AI.md`](../dossiers/HIGH_TICKET_GRANT_RFP/NSF-SBIR-2026-AGENTIC-AI.md) (generated before NSF 26-510 was confirmed; refresh to match the pitch).
- [x] **Solicitation details recorded:** NSF 26-510, ceiling `\$305,000`, windows 11/04/2026, 03/04/2027, 07/07/2027.
- [ ] **Verify** pitch form section names, word limits and topic-code field on the live NSF pitch form.
- [ ] **Add** sourced market data and design-partner evidence to §1.3.
- [ ] **Add** founder biography and SBIR eligibility facts to §1.4.
- [ ] **Confirm** SAM.gov, SBA company registry and Research.gov registrations.
- [ ] **Submit** Project Pitch via the NSF pitch portal, then wait for an invitation.
- [ ] **Refresh dossier** to the pitch narrative and the NSF 26-510 proposal format.
