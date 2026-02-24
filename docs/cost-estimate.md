# CoverProof AI — Infrastructure Cost Estimate
**Dev Kickoff vs Production Ready**
*All prices in AUD. USD prices converted at 1 USD = 1.50 AUD. GCP prices based on australia-southeast1 (Sydney) Tier 2 region.*

> **Currency note:** All USD-denominated services (GCP, GitHub Copilot, SendGrid, Twilio, OpenAI, Google Document AI) have been converted to AUD at a fixed rate of **1 USD = 1.50 AUD**. Squarespace domain prices are sourced in AUD directly. Actual costs will vary with exchange rate fluctuations.

---

## 1. GitHub Copilot Licences

**Recommendation: Pro+ plan — $58.50 AUD/user/month**
The Pro+ plan gives both engineers unlimited completions, unlimited agent mode, coding agent, PR reviews, and access to all major models including Claude Opus 4 at volume. The standard Pro plan ($15/user/month AUD) is sufficient if Claude Opus 4 access is not required.

| Engineers | Plan | Monthly | Dev Period (26 wks / ~6.5 months) |
|-----------|------|---------|----------------------------------|
| 2 × AI Engineers | Pro+ @ $58.50 AUD/user | **$117/mo AUD** | **~$760 AUD total** |

> Note: Security Engineer is engaged for 4 weeks (Sprints 3–4) only. A single-month Pro+ licence for that period adds ~$58.50 AUD.

**Copilot Subtotal: ~$819 AUD (full dev period)**

---

## 2. Domain Names (Squarespace)

Two environments = two domains. Recommended naming:
- **Prod:** `coverproof.ai`
- **Non-prod:** `dev.coverproof.ai` (subdomain, no separate registration needed — just DNS config on the same domain)

> **OR** if the bank wants fully isolated non-prod branding:
- **Non-prod:** `coverproof-dev.com` or `coverproof-staging.com`

| Domain | TLD | Est. Annual Cost | Notes |
|--------|-----|-----------------|-------|
| `coverproof.ai` | .ai | ~$128/yr AUD | Prod — confirm availability via Squarespace search |
| `coverproof-dev.com` | .com | ~$19/yr AUD | Non-prod / staging (if separate domain preferred) |

**Domain Subtotal: ~$147/year AUD**

> ⚠️ We should search both names on Squarespace before committing — if `coverproof.ai` is taken as a premium domain, price could jump to $500+ AUD/yr. Have a backup name ready (e.g. `getcoverproof.ai`, `coverproofapp.ai`).

---

## 3. GCP Projects (Prod & Non-Prod)

Two separate GCP projects: `coverproof-prod` and `coverproof-nonprod`. Billing is per project. Costs below are estimated per environment.

GCP services in use: **Cloud Run, Cloud Storage (GCS), Firestore, Secret Manager, Cloud Build, Pub/Sub, Cloud Scheduler, Cloud Armor.**

---

### 3a. GCP — Non-Prod (Dev/Staging) Environment

Low traffic. Engineers only. Used for development, testing, sprint demos.

| Service | Usage Assumption | Est. Monthly Cost |
|---------|-----------------|------------------|
| **Cloud Run** (backend + frontend, 2 services) | Sporadic dev traffic only. Likely within free tier most months (180k vCPU-sec/mo free). Occasional load tests may exceed. | ~$8–15/mo AUD |
| **Cloud Storage (GCS)** | Test COI PDFs, container images. ~5 GB stored. Standard class. | ~$2/mo AUD |
| **Firestore** | Dev/test data. Low read/write volume. Free tier covers most usage (50k reads, 20k writes/day free) | ~$2–3/mo AUD |
| **Cloud Build** | CI/CD pipeline. ~120 build-minutes/day across sprints. Free tier: 120 build-minutes/day | ~$0–8/mo AUD |
| **Secret Manager** | ~10 secrets. $0.06/secret/month + access charges | ~$2/mo AUD |
| **Pub/Sub** | Dev campaign dispatch testing. Free tier: 10 GB/mo | ~$0/mo |
| **Cloud Scheduler** | 3 jobs (retention purge, campaign trigger, delta check) | ~$0 (3 free jobs/project) |
| **Cloud Armor** | WAF rules — pay-as-you-go, minimal dev traffic | ~$8/mo AUD |
| **Networking / Egress** | Dev traffic minimal | ~$2/mo AUD |

**Non-Prod GCP Estimate: ~$24–40/month AUD**
**Over 26-week dev period (~6.5 months): ~$155–260 AUD total**

---

### 3b. GCP — Production Environment

Assumptions for a pilot/initial production launch:
- **Bank portfolio:** ~1,000–5,000 collaterals
- **Borrower uploads:** ~200–500 COI PDFs/month (ramp-up period)
- **Bank dashboard:** ~10–20 concurrent admin users
- **COI PDF size:** avg. 2–3 MB per file

| Service | Usage Assumption | Est. Monthly Cost |
|---------|-----------------|------------------|
| **Cloud Run — Backend** (FastAPI) | 500k–1M requests/month. ~400ms avg latency. 1 vCPU / 512 MiB. | ~$23–38/mo AUD |
| **Cloud Run — Frontend** (React, served via nginx) | Lower request volume (static-heavy). | ~$8–15/mo AUD |
| **Cloud Storage (GCS)** | 500 PDFs × 2.5 MB avg = ~1.25 GB/mo growing. Plus GET/PUT ops. | ~$8–15/mo AUD |
| **Firestore** | 5,000 collateral docs, 500 COI uploads/mo, campaign/audit records. Moderate read/write. Beyond free tier. | ~$8–23/mo AUD |
| **Cloud Build** | Ongoing CI/CD. ~30 prod deploys/month post-launch. | ~$0–8/mo AUD |
| **Secret Manager** | ~15 secrets in prod | ~$3/mo AUD |
| **Pub/Sub** | Campaign dispatch queue. 500 messages/month. Well within free tier (10 GB). | ~$0/mo |
| **Cloud Scheduler** | 3–5 jobs (purge, campaign, delta reconciliation) | ~$0 (free tier covers this) |
| **Cloud Armor** | WAF on prod ingress. | ~$12–18/mo AUD |
| **Cloud Monitoring & Alerting** | Log ingestion. First 50 GB/month free. | ~$0–8/mo AUD |
| **Networking / Egress** | Dashboard loads, PDF downloads (bank only). ~10–20 GB/mo. | ~$5–8/mo AUD |

**Production GCP Estimate: ~$65–135/month AUD at launch**

> This will scale with portfolio size. At 10,000+ collaterals and 2,000+ uploads/month, expect $225–375/month AUD.

---

## 4. Third-Party Services (Prod)

These aren't in the original scope items but are required by the architecture and need budgeting.

| Service | Plan / Assumption | Monthly Cost |
|---------|------------------|-------------|
| **SendGrid** (email) | Essentials plan — up to 50k emails/mo. For a 5,000-collateral portfolio, 2 emails per borrower per campaign cycle. | ~$30/mo AUD |
| **Twilio** (SMS) | Pay-as-you-go. ~$0.12/SMS in Australia. 5,000 SMS/campaign cycle. | ~$600/cycle AUD (not monthly — triggered per campaign) |
| **Google Document AI** | PDF processing. 500 PDFs × avg 3 pages = 1,500 pages/mo. | ~$3/mo AUD |
| **OpenAI GPT-4o** (fallback OCR) | Used only as fallback. Estimate 5–10% of COIs. | ~$3–8/mo AUD |

> Note: Twilio SMS is a per-campaign cost, not a fixed monthly line. Each time the bank runs a campaign across 5,000 borrowers, that's ~$600 AUD. Budget this as a campaign-level cost, not infrastructure.

---

## 5. Full Summary

### Dev Kickoff (first month, getting started)

| Item | Cost |
|------|------|
| GitHub Copilot Pro+ — 2 engineers (first month) | $117 AUD |
| Domain registration — `coverproof.ai` | ~$128 AUD (annual, paid upfront) |
| Domain registration — `coverproof-dev.com` | ~$19 AUD (annual, paid upfront) |
| GCP Non-Prod project (first month) | ~$24–40 AUD |
| GCP Prod project (first month — minimal, just standing up infra) | ~$15–23 AUD |

**Dev Kickoff Total: ~$303–327 AUD (first month)**

---

### Ongoing Dev Period (per month, Sprints 1–11)

| Item | Monthly |
|------|---------|
| GitHub Copilot Pro+ — 2 engineers | $117 AUD |
| GCP Non-Prod | ~$24–40 AUD |
| GCP Prod (standby/staging use) | ~$15–30 AUD |
| Domains (amortised monthly) | ~$12 AUD |

**Monthly During Development: ~$168–199/month AUD**
**Total Across Full 26-Week Dev Period (~6.5 months): ~$1,090–1,295 AUD**

---

### Production Ready (monthly recurring, post-go-live)

| Item | Monthly |
|------|---------|
| GitHub Copilot Pro+ — 2 engineers (ongoing) | $117 AUD |
| GCP Prod | ~$65–135 AUD |
| GCP Non-Prod (maintained for staging/testing) | ~$24–40 AUD |
| Domains (amortised) | ~$12 AUD |
| SendGrid (email) | ~$30 AUD |
| Google Document AI | ~$3 AUD |
| OpenAI fallback OCR | ~$3–8 AUD |

**Monthly Production Recurring: ~$254–345/month AUD**

> Twilio SMS is excluded from recurring — budget ~$600 AUD per active campaign cycle separately.

---

## 6. One-Time Setup Costs to Note

| Item | Est. Cost | Notes |
|------|-----------|-------|
| GCP initial credit | **$0** | New GCP accounts receive $450 AUD free credit — apply this to non-prod during dev |
| GitHub Copilot 30-day free trial | **$0** | Both engineers can trial Pro+ before committing |
| Squarespace domain first-year discount | Check at search | .com domains sometimes offered at discounted rates for year 1 on promotions |

---

## Quick Reference

| Stage | Monthly Spend |
|-------|--------------|
| Dev Kickoff (Month 1) | **~$303–327 AUD** |
| Ongoing Dev (Months 2–6) | **~$168–199/mo AUD** |
| Full Dev Period Total (6.5 months) | **~$1,090–1,295 AUD** |
| Production Recurring (post-launch) | **~$254–345/mo AUD** |

*Excludes: Twilio SMS (~$600 AUD/campaign cycle), staff costs, legal/privacy counsel, and any third-party security pen test contractor fees.*

---
*Prices sourced February 2026: GitHub Copilot (github.com/features/copilot/plans), Squarespace Domains (domains.squarespace.com), Google Cloud Run/Firestore/GCS (cloud.google.com pricing pages). GCP Sydney (australia-southeast1) Tier 2 rates applied. USD prices converted to AUD at 1 USD = 1.50 AUD.*