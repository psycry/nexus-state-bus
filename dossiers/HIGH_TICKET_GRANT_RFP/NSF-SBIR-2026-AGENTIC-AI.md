# 🏛️ NSF Project Pitch Dossier: `NSF-SBIR-2026-AGENTIC-AI`

- **Solicitation / Opportunity ID:** `sbir_civic_rfp:NSF-SBIR-2026-AGENTIC-AI` (**NSF 26-510**, America's Seed Fund SBIR/STTR Phase I)
- **Title:** Secure, Sandboxed Hybrid AI Orchestration (Rust MCP + pgvector) for Small-Business Automation
- **Lane:** `HIGH_TICKET_GRANT_RFP` | **Listing Type:** `live_solicitation`
- **Phase I Award Ceiling:** $305,000.00 (ceiling, not an expected award)
- **Windows:** 2026-11-04 (passed), **2027-03-04 (next)**, 2027-07-07 — 5:00 PM local time
- **Process:** an invited Project Pitch is required before a full proposal.
- **Zero-Token pgvector Fit Score:** `0.5738`
- **Primary Proof-of-Capability Asset:** **Agent Smith (Claw MCP Server v0.2.0 — Rust Hybrid Multi-Agent Orchestrator)** (`agent_smith`)
- **Applicant Entity:** Digital Nomad LLC
- **Refreshed:** `2026-11-05` (T-032) — mirrors the Project Pitch text in the submission package.

> Source note: solicitation number, ceiling, windows and the 500-words-per-section limit were supplied by the applicant from seedfund.nsf.gov. Biography, eligibility and registration facts are applicant-attested and not independently verified.

---

## 1. Technology Innovation (≤500 words)

Small businesses want AI agents that act on their systems, but current agent stacks run tools with broad host access, depend entirely on cloud models, and keep no durable shared state across agents. **Digital Nomad LLC** is building a secure orchestration layer that addresses all three:

- **Deterministic sandboxed tool execution (Rust):** every tool call passes path canonicalization and a command allowlist before it runs, so an agent cannot read or write outside declared roots.
- **Hybrid inference routing:** a local model (Ollama, `qwen2.5-coder:1.5b`) handles low-cost work and cloud foundation models (Gemini Flash family) handle the rest, with automatic fallback.
- **Collective vector memory:** PostgreSQL 16 + `pgvector` 768-dimension semantic memory shared by the Rust control plane and a Python worker plane under a fixed embedding contract.
- **Decentralized state synchronization:** the `nexus-state-bus` architecture synchronizes state between independent agents through a shared ledger, with polling, sanitized state packets and automatic recovery.

The working base is **Agent Smith** (`claw-mcp-server`, a Rust Model Context Protocol server exposing 17 tools), operated internally in production together with `nexus-state-bus`. The innovation to fund is what is not yet built or proven: measurable security guarantees, a cost-aware routing policy, and multi-agent coordination over shared memory and state.

## 2. Technical Objectives and Challenges (≤500 words)

1. **Sandbox assurance:** define a tool-call policy model and test it with adversarial prompt-injection and path-traversal suites. Target: zero boundary violations across the test corpus.
2. **Cost-aware routing:** learn a routing policy between local and cloud models. Target: a defined reduction in cloud-token spend at equal task success on a benchmark task set.
3. **Multi-agent memory and state coordination:** concurrent agents reading and writing shared `pgvector` memory and bus state without stale or conflicting results. Target: measured recall precision, synchronization latency and failure-recovery time under concurrent load.

Principal technical risks: prompt-injection that survives allowlisting, quality loss from routing to small local models, and state divergence between agents under partial failure. Phase I exists to quantify each.

## 3. Market Opportunity (≤500 words)

The target market is autonomous agent orchestration and DevOps-style automation: businesses deploying multi-agent systems to automate pipelines, operations and routine engineering work. Buyers are small and mid-sized businesses, and the integrators who serve them, that need agent automation but cannot accept unrestricted tool access or unbounded per-token cloud costs.

Published forecasts show both adjacent markets growing quickly. Market research firm MarketsandMarkets projects the global AI agents market will grow from about $7.84 billion in 2025 to $52.62 billion by 2030, a 46.3% CAGR [1]; BCC Research independently forecasts about $8.0 billion in 2025 to $48.3 billion by 2030, a 43.3% CAGR [2]. The surrounding DevOps market, where automated delivery pipelines are moving from rule-based to AI-augmented operation, is estimated by Grand View Research at about $16.2 billion in 2025, reaching $74.1 billion by 2033 at a 21.9% CAGR [3]. These are broad, vendor-defined total-market figures; the company's initial serviceable market is the small-business and integrator segment within them, which we have not yet sized.

The value proposition is efficiency in automated pipeline routing (sending each task to the cheapest model that succeeds) and reliable decentralized state synchronization between agents. Initial go-to-market is fixed-scope integration engagements, moving to a licensed or hosted orchestration product.

*Evidence base:* the applicant's own production use of Agent Smith and `nexus-state-bus` demonstrates multi-agent inter-process communication and failure recovery under live operating constraints. These are internal deployments, not external design partners. *Any named external pilot customers will be added when available.*


**Sources** *(verified via HTTP 200 primary press releases / publisher report pages on 2026-11-05, T-032):*
1. MarketsandMarkets (PR Newswire release), *"AI Agents Market worth \$52.62 billion by 2030 — Exclusive Report by MarketsandMarkets"*: \$7.84B (2025) to \$52.62B (2030), CAGR 46.3% — https://www.prnewswire.com/news-releases/ai-agents-market-worth-52-62-billion-by-2030---exclusive-report-by-marketsandmarkets-302435486.html
2. BCC Research (Official Pressroom), *"AI Agents Market to Grow 43.3% Annually"*: \$8.0B (2025) to \$48.3B (2030), CAGR 43.3% — https://www.bccresearch.com/pressroom/ait/ai-agents-market-to-grow-433-annually
3. Grand View Research, *"DevOps Market Size, Share & Trends Analysis Report (2026–2033)"*: \$16.2B (2025), \$18.5B (2026) to \$74.1B (2033), CAGR 21.9% — https://www.grandviewresearch.com/industry-analysis/development-to-operations-devops-market

## 4. Company and Team (≤500 words)

**Digital Nomad LLC** is a U.S.-owned and operated small business that the applicant states meets SBA size standards and domestic performance requirements for Phase I.

**William Jasper Land** (principal / principal investigator) holds an M.S. in Computer Science from UNCC. He has a production software engineering background, formerly at Amazon, and specializes in systems architecture. He designed and operates Agent Smith, `nexus-state-bus`, and the supporting Phronarch and Shelter Source codebases.

Registrations: the applicant states that Digital Nomad LLC is active and verified on SAM.gov, Grants.gov and seedfund.nsf.gov.

---

## 5. Specific Aims & Verification Plan

| Aim | TRL | Objective | Phase I KPI | Prior Art |
| :--- | :--- | :--- | :--- | :--- |
| **1. Verified Sandbox Policy** | `TRL-4 -> TRL-6` | Formal tool-call policy plus adversarial harness | 0 boundary violations on the adversarial corpus | `claw-mcp-server` (`resolve_safe_path`, command allowlist) |
| **2. Cost-Aware Hybrid Routing** | `TRL-4 -> TRL-6` | Routing policy between local and cloud models | Cloud-token reduction at equal task success | Ollama + Gemini fallback router |
| **3. Multi-Agent Memory & State** | `TRL-4 -> TRL-6` | Concurrency-safe shared memory and state sync | Recall precision, sync latency, recovery time under N agents | `store_memory` / `recall_memory`, `nexus-state-bus` |

## 6. Submission Notes

- Form guidance (applicant): use the broadest category that fits agentic orchestration and automated workflows; NSF 26-510 is stated to evaluate deep-tech innovation holistically. Check the topic-code field on the live form.
- Each pitch section above is within the 500-word limit stated by the applicant.
- Final submission is a manual step through the official NSF portal.
