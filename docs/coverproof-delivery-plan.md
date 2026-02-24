# CoverProof AI (Phase 2) — Delivery Plan (WIP)
**Stack:** Google Cloud Platform · Cloud Run · React · Python · Firestore · GCS
**Team:** 2 AI Engineers (full-stack across frontend, backend, AI) · 1 Security Engineer (Sprints 3–4 and Sprint 10)

---

## Sprint Plan

### Sprint 1–2 (Weeks 1–4) — Foundation & Infrastructure
**Engineers:** 2 AI Engineers
**Capacity:** 4 engineer-weeks

**Split of work:**
- **Eng 1:** GCP project setup (Cloud Run, GCS, Firestore, Secret Manager, Pub/Sub), CI/CD pipeline (Cloud Build), Docker containers for backend
- **Eng 2:** React app scaffold (Vite + TypeScript), basic auth UI, white-label domain setup (`yourbank.coverproof.ai`), backend FastAPI skeleton

**Deliverables:**
- GCP environment: dev / staging / prod with automated deploys on merge
- Python FastAPI backend containerised on Cloud Run
- React frontend containerised on Cloud Run
- JWT auth protecting all endpoints
- Shared Firestore schema v1 (collections: `collaterals`, `borrowers`, `campaigns`, `audit_logs`)

**Milestone ✅ — "Infrastructure Live"**
> Both containers deploy to dev via CI/CD. Auth works. Firestore read/write confirmed.

---

### Sprint 3–4 (Weeks 5–8) — Security Part 1 (Transport, Encryption & Access Control)
**Engineers:** 2 AI Engineers + 1 Security Engineer
**Capacity:** 6 engineer-weeks (Security Engineer joins full-time for both sprints)

> The Security Engineer leads and owns all security deliverables. AI Engineers implement under direction and integrate security patterns into existing code. Pen test and compliance sign-off are deferred to Sprint 10 once the frontend is complete.

**Sprint 3 — Transport, Auth & Encryption (Security Engineer leads):**
- HTTPS / TLS 1.3 enforced across all Cloud Run services
- JWT token lifecycle: rotation, revocation, one-time-use flag (Firestore TTL)
- Encrypted URL generation service (JWT-signed, configurable expiry, single-use enforcement)
- Cloud Armor WAF (Web Application Firewall) rules on Cloud Run ingress
- VPC Service Controls — GCP service perimeter
- Secret Manager: all credentials moved out of codebase
- GCS CMEK (Customer-Managed Encryption Keys) on all buckets

**Sprint 4 — Access Control (Security Engineer leads):**
- Firestore security rules — role-based read/write per collection
- RBAC implementation: Bank Admin / Risk Team (read-only) / Super Admin
- Consent record storage — immutable, timestamped write-once pattern

**Milestone ✅ — "Security Part 1 Complete"**
> Encryption verified at rest and in transit. RBAC confirmed. Security Engineer offboards until Sprint 10.

---

### Sprint 5 (Weeks 9–10) — Communication Engine POC
**Engineers:** 2 AI Engineers
**Capacity:** 4 engineer-weeks

> Deliberately narrow. Goal is to prove the handshake works, not to build production infrastructure.

**Eng 1 — Backend:**
- SendGrid integration: white-labeled email with OTP code
- Twilio integration: SMS with encrypted one-time link
- OTP generation + Firestore TTL storage
- Basic JWT-signed URL → validate → mark used

**Eng 2 — Frontend:**
- Borrower portal landing page: code entry screen → unlock → success state

**Explicitly out of scope for POC:**
- Campaign scheduling, kill switch, Pub/Sub queuing
- Real borrower data

**Milestone ✅ — "Comms POC Sign-Off"**
> Internal demo: test email received → SMS received → link clicked → code entered → consent captured in Firestore. Go/no-go decision to proceed to full build.

---

### Sprint 6 (Weeks 11–12) — Communication Engine Full Build
**Engineers:** 2 AI Engineers
**Capacity:** 4 engineer-weeks

**Eng 1 — Backend:**
- Pub/Sub queue for async campaign dispatch
- Campaign Controller: `Launch_Date +/- X days` scheduling, configurable retry logic
- Kill switch: cancel all pending sends per `Collateral_ID`
- Audit logging: send timestamps, delivery status, link open events

**Eng 2 — Frontend:**
- Full borrower portal flow: authentication → COI upload screen (PDF/image) → confirmation screen
- Borrower data confirmation screen (display AI-extracted fields for borrower to validate)

**Milestone ✅ — "Comms Engine Live"**
> Campaign scheduling, kill switch, and full borrower upload flow verified. Audit log complete.

---

### Sprint 7–8 (Weeks 13–16) — AI Verification Engine
**Engineers:** 2 AI Engineers
**Capacity:** 4 engineer-weeks per sprint (8 total)

**Sprint 7 — Document Parsing & Tests A + C:**

- **Eng 1:** Data extraction using Claude, structured JSON extractor using Pydantic approach, LLM Judge using Gemini. Parsed JSON stored in Firestore mapped to `Collateral_ID`.
- **Eng 2:**
  - **Test A — Accuracy:** Fuzzy address match (RapidFuzz), name match, interested party keyword detection. Each sub-test: Pass / Fail / Needs Review + confidence score.
  - **Test C — Adequacy:** Coverage ratio `Sum_Insured / MAX(Loan_Balance, Last_Valuation, Rebuild_Cost)`, configurable target, rebuild cost proxy (CoreLogic/Cordell or postcode model), flag ratio < 1.0.

**Sprint 8 — AI Policy Library & Test B + Status Engine:**

- **Eng 1:** PDS scraper (Scrapy/Playwright) for Top 10 Australian insurers. Manual PDS upload fallback (admin UI). PDSs indexed in vector store (Vertex AI Matching Engine or pgvector on Cloud Run).
- **Eng 2:**
  - **Test B — Appropriateness:** Bank risk appetite config (required perils per LGA/postcode), peril lookup against PDS vector store, Pass / Fail per peril.
  - **Insurance Status Engine:** Aggregate A + B + C → Verified / Partially Compliant / Non-Compliant / Pending. Status written to `Collateral_ID`. Manual admin override with document attachment. Full version history (timestamped).

**Milestone ✅ — "Verification Engine Live"**
> 20 real COI PDFs processed. Accuracy >85% against ground truth. Adequacy ratios correct. All results stored against Collateral IDs.

---

### Sprint 9 (Weeks 17–18) — Frontend Build (Bank Dashboard)
**Engineers:** 2 AI Engineers
**Capacity:** 4 engineer-weeks

**Eng 1 — Dashboard core views:**
- Portfolio Overview: bar chart (% Verified / Pending / Non-Compliant / Inactive), weekly trend
- Gap Alert Panel ("Red Zone"): filterable/sortable table — Collateral_ID, Address, Failure Reason, Coverage Ratio, Last Action Date

**Eng 2 — Export, audit & admin:**
- Export Module: CSV + PDF export (APRA APS 220 formatted)
- Audit Trail View: per-borrower timeline, filterable by date / outcome / Collateral_ID
- Admin Controls Panel: risk appetite config, adequacy target toggle, campaign controller UI, cohort upload history
- Backend: paginated Firestore query APIs, PDF report generation (WeasyPrint or Puppeteer sidecar)

**Milestone ✅ — "Frontend Complete"**
> Full UI walkthrough with stakeholders — dashboard, exports, audit trail, admin controls.

---

### Sprint 10 (Weeks 19–20) — Security Part 2 (Pen Test & Compliance)
**Engineers:** 2 AI Engineers + 1 Security Engineer
**Capacity:** 6 engineer-weeks (Security Engineer rejoins full-time)

> Pen test and compliance sign-off are performed now that the full application — including frontend — is complete. This ensures all attack surfaces are in scope for testing.

**Security Engineer leads:**
- Data retention scaffold: Cloud Scheduler + Cloud Function (configurable TTL purge)
- Privacy Policy and ToS pages (static, borrower portal)
- Internal pen test executed by Security Engineer across full application stack (frontend + backend + borrower portal)
- GDPR / Australian Privacy Act compliance checklist signed off
- Others — to be confirmed by Security Engineer

**AI Engineers — support & remediation:**
- Implement fixes for any vulnerabilities identified during pen test
- Integrate data retention and privacy pages into existing application

**Milestone ✅ — "Security Hardened"**
> Pen test passed. Full application stack tested. Compliance checklist signed off. Security Engineer offboards.

---

### Sprint 11 (Weeks 21–22) — Test Plan & Test Execution
**Engineers:** 2 AI Engineers
**Capacity:** 4 engineer-weeks

> Engineers write and execute tests against their own system. No separate QA resource.

**Eng 1 — Unit & integration tests:**
- Schema validation (valid / malformed / missing fields)
- Delta reconciliation: new / discharged / existing collateral
- OTP: generation, expiry, single-use enforcement
- JWT: valid / expired / tampered
- Fuzzy match accuracy against labelled test set
- Coverage ratio edge cases (zero values, missing fields)
- Purge/retention logic
- RBAC: confirm role boundaries on all endpoints

**Eng 2 — End-to-end test cases:**

| # | Scenario | Expected Outcome |
|---|----------|-----------------|
| E2E-01 | New collateral in CSV upload | Onboarding campaign triggered |
| E2E-02 | Collateral missing from new upload | Marked Inactive, comms killed |
| E2E-03 | Full borrower verification flow | COI parsed, all 3 tests run, status = Verified |
| E2E-04 | COI address doesn't match bank record | Test A fails, status = Non-Compliant |
| E2E-05 | Sum insured < loan balance | Test C fails, flagged in Gap Alert |
| E2E-06 | Required peril (Flood) absent from COI | Test B fails |
| E2E-07 | Expired SMS link clicked | Access denied |
| E2E-08 | SMS link clicked twice | Second attempt rejected |
| E2E-09 | Admin pauses borrower comms | No sends triggered |
| E2E-10 | Admin manually overrides status + attaches doc | Status updated, version history recorded |
| E2E-11 | Data retention purge triggered | PII scrubbed, Collateral_ID + results retained |
| E2E-12 | Risk team attempts admin-only action | 403 Forbidden |
| E2E-13 | 500 concurrent COI uploads | <2s p95 latency, no data loss |
| E2E-14 | Export climate disclosure report | Correct CSV/PDF generated |

**Milestone ✅ — "Test Plan Complete & P1/P2 Defects Resolved"**

---

### Sprint 12 (Weeks 23–24) — Data Ingestion *(Post-Contract Gate)*
**Engineers:** 2 AI Engineers
**Capacity:** 4 engineer-weeks

> ⚠️ **This sprint does not begin until the bank contract is signed.** If the contract is delayed, Sprint 13 (UAT) shifts accordingly. All prior sprints are fully unblocked — the system runs on synthetic data until this point.

**Eng 1 — Backend ingester:**
- CSV/XLSX ingester with full schema validation against agreed bank fields
- Pandas mapping with rejection report (bad rows returned to bank)
- Delta reconciliation engine wired to live Firestore collections

**Eng 2 — Admin UI + pipeline wiring:**
- Secure admin upload portal (drag-and-drop with validation feedback)
- Manual Pause toggle per borrower
- Swap all mock data references in campaign controller, dashboard, and verification engine to live ingestion pipeline
- Smoke test with synthetic data shaped to agreed bank schema

**Milestone ✅ — "Data Ingestion Live"**
> Bank uploads 500-row pilot CSV. New / discharged / existing collaterals processed correctly. Delta summary confirmed.

---

### Sprint 13 (Weeks 25–28) — UAT, Hardening & Go-Live
**Engineers:** 2 AI Engineers
**Capacity:** 8 engineer-weeks (extended to 4 weeks to absorb UAT defect fixing)

**Weeks 25–26 — UAT:**
- Bank UAT with real portfolio slice (min. 100 collaterals)
- Borrower UAT pilot (5–10 real borrowers)
- Load test: 500 concurrent uploads (k6 or Locust)
- Defect triage and resolution

**Weeks 27–28 — Hardening & Go-Live:**
- Blue/Green production deployment via Cloud Run traffic splitting
- Cloud Monitoring dashboards: uptime, error rates, latency, Pub/Sub queue depth
- Alerting: Cloud Alerting / PagerDuty for critical failures
- Runbook documentation + handover pack
- 2-week hypercare post-launch

**Milestone ✅ — "Production Go-Live"**

---

## Summary Timeline

| Sprint | Weeks | Engineers | Focus | Milestone |
|--------|-------|-----------|-------|-----------|
| 1–2 | 1–4 | 2 AI | Foundation & Infrastructure | Infrastructure Live |
| 3–4 | 5–8 | 2 AI + 1 Security | Security Part 1 — Transport, Encryption & Access Control | Security Part 1 Complete |
| 5 | 9–10 | 2 AI | Comms POC | POC Sign-Off |
| 6 | 11–12 | 2 AI | Comms Full Build | Comms Engine Live |
| 7–8 | 13–16 | 2 AI | AI Verification Engine | Verification Engine Live |
| 9 | 17–18 | 2 AI | Frontend Build | Frontend Complete |
| 10 | 19–20 | 2 AI + 1 Security | Security Part 2 — Pen Test & Compliance | Security Hardened |
| 11 | 21–22 | 2 AI | Test Plan & Execution | Tests Passed |
| 12 | 23–24 | 2 AI | Data Ingestion *(post-contract)* | Ingestion Live |
| 13 | 25–28 | 2 AI | UAT, Hardening & Go-Live | Production Go-Live |

**Total: ~28 weeks** (assuming contract signed by Week 22 at latest)

---

## Capacity Reality Check

| Sprint | Eng-Weeks Available | Load Assessment |
|--------|-------------------|-----------------|
| 1–2 | 4 | 🟢 Comfortable |
| 3–4 | 6 (+ Security Eng) | 🟢 Comfortable — Security Eng carrying the load |
| 5 | 4 | 🟡 Tight — scope deliberately limited to POC |
| 6 | 4 | 🟡 Tight — builds on POC foundation |
| 7–8 | 8 | 🔴 High risk — most complex sprint. Buffer for AI edge cases |
| 9 | 4 | 🟡 Tight — parallel dashboard + APIs |
| 10 | 6 (+ Security Eng) | 🟢 Comfortable — Security Eng leading pen test |
| 11 | 4 | 🟢 Comfortable — test writing, not net-new build |
| 12 | 4 | 🟢 Comfortable — schema agreed in advance |
| 13 | 8 | 🟡 UAT defect volume is the unknown |

---

## Open Questions (Pre-Sprint 1)

1. **Security Engineer:** Needs to be confirmed before Sprint 3. Note: Security Engineer is engaged twice — Sprints 3–4 and Sprint 10.
2. **Draft schema agreement:** Can the bank provide a draft field list pre-contract so Sprint 12 prep can happen in parallel?
3. **SMS compliance:** Does existing bank customer consent cover SMS from `verification@yourbank.coverproof.ai`?
4. **PDS scope:** Which insurers are in scope for the AI Policy Library?
5. **Data retention:** Full retention for APRA audit, or PII scrub post-acknowledgement?
6. **Data Processing:** GCP doesn't have any Gemini models with Australia Southeast endpoints — is that going to be a problem for the bank?