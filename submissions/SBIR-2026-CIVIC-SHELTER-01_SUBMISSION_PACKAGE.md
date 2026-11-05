# 📦 Staged SBIR Phase I Submission Package: `SBIR-2026-CIVIC-SHELTER-01`

- **Opportunity ID:** `sbir_civic_rfp:SBIR-2026-CIVIC-SHELTER-01`
- **Solicitation Number:** **`HHS-2027-ACL-NIDILRR-BISA-0308`** (Grants.gov Opportunity ID `363966`)
- **Lane:** `HIGH_TICKET_GRANT_RFP`
- **Listing Type:** `live_solicitation` (`forecasted` on Grants.gov — placeholder tag removed in `T-034`)
- **Title:** HHS ACL/NIDILRR SBIR Phase I: Accessible Emergency Shelter & Community Living Resource Navigation Platform
- **Agency:** U.S. Department of Health and Human Services — Administration for Community Living (`HHS-ACL`) / NIDILRR
- **Source / Portal URL:** https://www.grants.gov/search-results-detail/363966 (Program info: https://www.acl.gov/grants/applying-grants)
- **Agency Contact:** Brian Bard — `brian.bard@acl.hhs.gov` | `(202) 795-7298`
- **Matched Capability Anchor:** `shelter_source` (`fit_score = 0.6444`, `raw_cosine = 0.8044`)
- **Grant Award Ceiling:** `\$100,000.00` (Award Floor: `\$95,000.00`; 6-Month Phase I Execution; 11 Expected Awards; No Cost Sharing)
- **Close Date:** 12/16/2026 (Estimated Application Response Date; Estimated Synopsis Posting Date: 10/16/2026)
- **Full Technical Dossier:** [`dossiers/HIGH_TICKET_GRANT_RFP/SBIR-2026-CIVIC-SHELTER-01.md`](../dossiers/HIGH_TICKET_GRANT_RFP/SBIR-2026-CIVIC-SHELTER-01.md)
- **Package Status:** `STAGED_LIVE_SOLICITATION_T034`

> [!NOTE]
> All solicitation metadata (`HHS-2027-ACL-NIDILRR-BISA-0308`, Grants.gov ID `363966`, `$100,000` ceiling, `$95,000` floor, 6-month period, 11 awards, contact Brian Bard, and `12/16/2026` estimated response date) was verified directly from the live Grants.gov `fetchOpportunity` API on `2026-11-05` (`T-034`). Because this notice is currently in `forecasted` status on Grants.gov, confirm the final posted NOFO package and exact submission deadline on Grants.gov once the synopsis transitions from `forecasted` to `posted`.

---

## 1. Executive Project Pitch & Technical Abstract (250-Word SBIR Portal Format)

**Technical Abstract:**  
During natural disasters, power outages, and housing crises, people with mobility, sensory, cognitive, and electricity-dependent medical disabilities face disproportionate barriers to safe evacuation and community living due to fragmented, unverified information on shelter accessibility and supportive resources—compounded by degraded cellular connectivity. **Digital Nomad LLC** proposes a **6-month HHS ACL/NIDILRR SBIR Phase I project (`$100,000`)** in response to **`HHS-2027-ACL-NIDILRR-BISA-0308`** to advance our existing **TRL-6 cross-platform civic resource navigation platform (`Shelter Source` & `Nomad Nexus`)** to **TRL-7 accessibility and field readiness**. Built on a unified **Flutter/Dart** codebase for iOS, Android, and Web backed by **Firebase Auth, Cloud Firestore, and local offline-first persistence**, the Phase I R&D program targets three technical innovations aligned with NIDILRR's Community Living and Health & Function outcome domains: (1) a standardized **Disability & Accessibility Geospatial Schema** (wheelchair/ADA access, durable medical equipment [DME] backup power, sensory-quiet spaces, ASL/communication support, and accessible transit) paired with a WCAG 2.1 AA screen-reader and low-cognitive-load interface ($<2.0\text{s}$ query latency across 100+ nodes); (2) deterministic **offline-first local storage (SQLite/Hive)** with conflict-free reconnection sync achieving $\ge 95\%$ integrity during blackouts; and (3) a **Center for Independent Living (CIL) and peer verification workflow** enforced via Firestore security rules ($\ge 80\%$ automated verification throughput). Phase I concludes with a 15-stakeholder accessibility usability evaluation ($\ge 4.0/5.0$ satisfaction), positioning the platform for Phase II (`HHS-2027-ACL-NIDILRR-BISB-0309`) and SaaS deployment across municipal OEMs and CIL/ADRC networks.

---

## 2. Specific Aims Summary & Prior Art Verification Matrix

| Specific Aim | Target TRL Progression | Core Technical Objective | Quantitative Phase I KPI | Existing Digital Nomad LLC TRL-6 Prior Art |
| :--- | :--- | :--- | :--- | :--- |
| **Aim 1: Accessible Geospatial Core & WCAG 2.1 AA UI** | `TRL-6 -> TRL-7` | Unified iOS/Android/Web Flutter mapping with NIDILRR disability accessibility schema (ADA, DME power, sensory/ASL) & TalkBack/VoiceOver support | Map load $< 2.0\text{s}$ for 100 active nodes; 15+ disability attributes; 100% automated WCAG 2.1 AA pass | `/home/psycry/Projects/shelter_source` unified Flutter/Dart geospatial navigation engine |
| **Aim 2: Offline-First Blackout Resilience & Sync** | `TRL-6 -> TRL-7` | Local SQLite/Hive offline cache, offline accessibility filtering, and CRDT/timestamp conflict resolution on reconnect | $\ge 95\%$ sync success across intermittent/offline transitions; zero loss of accessibility flags | `shelter_source` offline-first field persistence & `nomad_nexus` distributed state modules |
| **Aim 3: CIL & Community Accessibility Verification** | `TRL-6 -> TRL-7` | Real-time crowd/CIL reporting of shelter accessibility & backup power, reputation-weighted peer verification, and moderator queue | $\ge 80\%$ verification throughput; 1,000 concurrent simulated users; 15-user disability usability pilot ($\ge 4.0/5.0$ CSAT) | Firestore composite indexes & role-based security rules proven in `shelter_source` |

---

## 3. Phase I Budget Allocation Blueprint (`\$100,000.00` — 6 Months)

| Budget Category | Allocation (USD) | Share (%) | Scope & Justification |
| :--- | :--- | :--- | :--- |
| **Direct Senior Engineering & Principal Investigator Labor** | `\$60,000.00` | `60.0%` | Principal Investigator / Lead Systems Architect (Flutter/Dart WCAG 2.1 AA accessibility core, offline sync engine, security rules, pilot orchestration across Months 1–6) |
| **Accessibility UX & Disability Community Pilot Coordination** | `\$16,000.00` | `16.0%` | Center for Independent Living (CIL) stakeholder co-design (Months 1–2), assistive-technology usability testing, and 15-participant evaluation (Month 6) |
| **Cloud Infrastructure, Geospatial Tile APIs & Assistive Test Devices** | `\$8,000.00` | `8.0%` | Firebase Blaze concurrency testing, offline vector map packs, screen-reader/accessibility test harness |
| **Indirect / F&A & Compliance Overhead** | `\$9,000.00` | `9.0%` | Accounting, SAM/Grants.gov/ACL reporting compliance, and legal/IP administration |
| **SBIR Statutory Fee / Profit (7%)** | `\$7,000.00` | `7.0%` | Statutory SBIR small business fee (within standard 7% guideline) |
| **Total Phase I Request** | **`\$100,000.00`** | **`100.0%`** | **6-Month TRL-6 $\rightarrow$ TRL-7 Execution (`HHS-2027-ACL-NIDILRR-BISA-0308` Ceiling)** |

---

## 4. Pre-Submission Portal Checklist

- [x] **Official Solicitation & Ceiling Verified on Grants.gov:** `HHS-2027-ACL-NIDILRR-BISA-0308` (`opportunityId=363966`, `$100,000` ceiling, `$95,000` floor, 6-month project period — `T-034`).
- [x] **Specific Aims & 6-Month Work Plan Aligned to NIDILRR Disability Mandate:** [`dossiers/HIGH_TICKET_GRANT_RFP/SBIR-2026-CIVIC-SHELTER-01.md`](../dossiers/HIGH_TICKET_GRANT_RFP/SBIR-2026-CIVIC-SHELTER-01.md).
- [x] **TRL-6 Prior Art Repository Present Locally:** `/home/psycry/Projects/shelter_source` & `/home/psycry/Projects/nomad_nexus`.
- [x] **Entity Registration Alignment:** Digital Nomad LLC (SAM.gov & Grants.gov active per applicant).
- [ ] **Confirm Full NOFO Posting & Submit via Grants.gov Workspace:** Monitor `HHS-2027-ACL-NIDILRR-BISA-0308` transition from `forecasted` to `posted` (contact: Brian Bard, `brian.bard@acl.hhs.gov`) and submit prior to the `12/16/2026` response date.
