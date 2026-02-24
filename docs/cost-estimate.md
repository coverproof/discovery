# CoverProof AI — Infrastructure Cost Estimate (WIP)
**Dev Kickoff vs Production Ready**

> **Currency note:** All prices are in **USD**. For AUD equivalents, apply a conversion rate of **1 USD = 1.50 AUD**. Domain prices (Squarespace) are originally quoted in AUD and have been converted to USD at the same rate for consistency. GCP prices are based on australia-southeast1 (Sydney) Tier 2 region.

---

## 1. GitHub Copilot Licences

**Recommendation: Pro+ plan — $39/user/month**
The Pro+ plan gives both engineers unlimited completions, unlimited agent mode, coding agent, PR reviews, and access to all major models including Claude Opus 4 at volume. The standard Pro plan ($10/user/month) is sufficient if Claude Opus 4 access is not required.

| Engineers | Plan | Monthly | Dev Period (26 wks / ~6.5 months) |
|-----------|------|---------|----------------------------------|
| 2 × AI Engineers | Pro+ @ $39/user | **$78/mo** | **~$507 total** |

> Note: Security Engineer is engaged for 4 weeks (Sprints 3–4) only. A single-month Pro+ licence for that period adds ~$39.

**Copilot Subtotal: ~$546 (full dev period)**

---

## 2. Domain Names (Squarespace)

Two environments = two domains. Recommended naming:
- **Prod:** `coverproof.ai`
- **Non-prod:** `dev.coverproof.ai` (subdomain, no separate registration needed — just DNS config on the same domain)

> **OR** if the bank wants fully isolated non-prod branding:
- **Non-prod:** `coverproof-dev.com` or `coverproof-staging.com`

| Domain | TLD | Est. Annual Cost | Notes |
|--------|-----|-----------------|-------|
| `coverproof.ai` | .ai | ~$85/yr | Prod — confirm availability via Squarespace search |
| `coverproof-dev.com` | .com | ~$13/yr | Non-prod / staging (if separate domain preferred) |

**Domain Subtotal: ~$98/year**

> ⚠️ We should search both names on Squarespace before committing — if `coverproof.ai` is taken as a premium domain, price could jump to $333+/yr. Have a backup name ready (e.g. `getcoverproof.ai`, `coverproofapp.ai`).

---

## 3. GCP Projects (Prod & Non-Prod)

Two separate GCP projects: `coverproof-prod` and `coverproof-nonprod`. Billing is per project. Costs below are estimated per environment.

GCP services in use: **Cloud Run, Cloud Storage (GCS), Firestore, Secret Manager, Cloud Build, Pub/Sub, Cloud Scheduler, Cloud Armor.**

---

### 3a. GCP — Non-Prod (Dev/Staging) Environment

Low traffic. Engineers only. Used for development, testing, sprint demos.

| Service | Usage Assumption | Est. Monthly Cost |
|---------|-----------------|------------------|
| **Cloud Run** (backend + frontend, 2 services) | Sporadic dev traffic only. Likely within free tier most months (180k vCPU-sec/mo free). Occasional load tests may exceed. | ~$5–10/mo |
| **Cloud Storage (GCS)** | Test COI PDFs, container images. ~5 GB stored. Standard class. ~$0.026/GB/mo (Sydney) | ~$1/mo |
| **Firestore** | Dev/test data. Low read/write volume. Free tier covers most usage (50k reads, 20k writes/day free) | ~$1–2/mo |
| **Cloud Build** | CI/CD pipeline. ~120 build-minutes/day across sprints. Free tier: 120 build-minutes/day | ~$0–5/mo |
| **Secret Manager** | ~10 secrets. $0.06/secret/month + access charges | ~$1/mo |
| **Pub/Sub** | Dev campaign dispatch testing. Free tier: 10 GB/mo | ~$0/mo |
| **Cloud Scheduler** | 3 jobs (retention purge, campaign trigger, delta check) | ~$0 (3 free jobs/project) |
| **Cloud Armor** | WAF rules — pay-as-you-go, minimal dev traffic | ~$5/mo (policy + low requests) |
| **Networking / Egress** | Dev traffic minimal | ~$1/mo |

**Non-Prod GCP Estimate: ~$15–25/month**
**Over 26-week dev period (~6.5 months): ~$100–165 total**

---

### 3b. GCP — Production Environment

Assumptions for a pilot/initial production launch:
- **Bank portfolio:** ~1,000–5,000 collaterals
- **Borrower uploads:** ~200–500 COI PDFs/month (ramp-up period)
- **Bank dashboard:** ~10–20 concurrent admin users
- **COI PDF size:** avg. 2–3 MB per file

| Service | Usage Assumption | Est. Monthly Cost |
|---------|-----------------|------------------|
| **Cloud Run — Backend** (FastAPI) | 500k–1M requests/month. ~400ms avg latency. 1 vCPU / 512 MiB. Request-based billing. Beyond free tier: ~$0.000024/vCPU-sec | ~$15–25/mo |
| **Cloud Run — Frontend** (React, served via nginx) | Lower request volume (static-heavy). | ~$5–10/mo |
| **Cloud Storage (GCS)** | 500 PDFs × 2.5 MB avg = ~1.25 GB/mo growing. Standard storage: ~$0.026/GB/mo (Sydney). Plus GET/PUT ops. | ~$5–10/mo |
| **Firestore** | 5,000 collateral docs, 500 COI uploads/mo, campaign/audit records. Moderate read/write. Beyond free tier. | ~$5–15/mo |
| **Cloud Build** | Ongoing CI/CD. ~30 prod deploys/month post-launch. | ~$0–5/mo |
| **Secret Manager** | ~15 secrets in prod | ~$2/mo |
| **Pub/Sub** | Campaign dispatch queue. 500 messages/month. Well within free tier (10 GB). | ~$0/mo |
| **Cloud Scheduler** | 3–5 jobs (purge, campaign, delta reconciliation) | ~$0 (free tier covers this) |
| **Cloud Armor** | WAF on prod ingress. $5/policy/mo + $0.75/million requests. | ~$8–12/mo |
| **Cloud Monitoring & Alerting** | Log ingestion. First 50 GB/month free. | ~$0–5/mo |
| **Networking / Egress** | Dashboard loads, PDF downloads (bank only). ~10–20 GB/mo. First 1 GB free, $0.12/GB after. | ~$3–5/mo |

**Production GCP Estimate: ~$43–89/month at launch**

> This will scale with portfolio size. At 10,000+ collaterals and 2,000+ uploads/month, expect $150–250/month.

---

## 4. Third-Party Services (Prod)

These aren't in the original scope items but are required by the architecture and need budgeting.

| Service | Plan / Assumption | Monthly Cost |
|---------|------------------|-------------|
| **SendGrid** (email) | Essentials plan — up to 50k emails/mo. For a 5,000-collateral portfolio, 2 emails per borrower per campaign cycle. | ~$20/mo |
| **Twilio** (SMS) | Pay-as-you-go. ~$0.08/SMS in Australia. 5,000 SMS/campaign cycle. | ~$400/cycle (not monthly — triggered per campaign) |
| **Google Document AI** | PDF processing. $1.50/1,000 pages. 500 PDFs × avg 3 pages = 1,500 pages/mo. | ~$2/mo |
| **OpenAI GPT-4o** (fallback OCR) | ~$10–15/1M tokens. Used only as fallback. Estimate 5–10% of COIs. | ~$2–5/mo |

> Note: Twilio SMS is a per-campaign cost, not a fixed monthly line. Each time the bank runs a campaign across 5,000 borrowers, that's ~$400. Budget this as a campaign-level cost, not infrastructure.

---

## 5. Full Summary

### Dev Kickoff (first month, getting started)

| Item | Cost |
|------|------|
| GitHub Copilot Pro+ — 2 engineers (first month) | $78 |
| Domain registration — `coverproof.ai` | ~$85 (annual, paid upfront) |
| Domain registration — `coverproof-dev.com` | ~$13 (annual, paid upfront) |
| GCP Non-Prod project (first month) | ~$15–25 |
| GCP Prod project (first month — minimal, just standing up infra) | ~$10–15 |

**Dev Kickoff Total: ~$201–216 (first month)**

---

### Ongoing Dev Period (per month, Sprints 1–11)

| Item | Monthly |
|------|---------|
| GitHub Copilot Pro+ — 2 engineers | $78 |
| GCP Non-Prod | ~$15–25 |
| GCP Prod (standby/staging use) | ~$10–20 |
| Domains (amortised monthly) | ~$8 |

**Monthly During Development: ~$111–131/month**
**Total Across Full 26-Week Dev Period (~6.5 months): ~$722–852**

---

### Production Ready (monthly recurring, post-go-live)

| Item | Monthly |
|------|---------|
| GitHub Copilot Pro+ — 2 engineers (ongoing) | $78 |
| GCP Prod | ~$43–89 |
| GCP Non-Prod (maintained for staging/testing) | ~$15–25 |
| Domains (amortised) | ~$8 |
| SendGrid (email) | ~$20 |
| Google Document AI | ~$2 |
| OpenAI fallback OCR | ~$2–5 |

**Monthly Production Recurring: ~$168–227/month**

> Twilio SMS is excluded from recurring — budget ~$400 per active campaign cycle separately.

---

## 6. One-Time Setup Costs to Note

| Item | Est. Cost | Notes |
|------|-----------|-------|
| GCP initial credit | **$0** | New GCP accounts receive $300 free credit — apply this to non-prod during dev |
| GitHub Copilot 30-day free trial | **$0** | Both engineers can trial Pro+ before committing |
| Squarespace domain first-year discount | Check at search | .com domains sometimes offered at discounted rates for year 1 on promotions |

---

## Quick Reference

| Stage | Monthly Spend |
|-------|--------------|
| Dev Kickoff (Month 1) | **~$201–216** |
| Ongoing Dev (Months 2–6) | **~$111–131/mo** |
| Full Dev Period Total (6.5 months) | **~$722–852** |
| Production Recurring (post-launch) | **~$168–227/mo** |

*Excludes: Twilio SMS (~$400/campaign cycle), staff costs, legal/privacy counsel, and any third-party security pen test contractor fees.*

---
*Prices sourced February 2026: GitHub Copilot (github.com/features/copilot/plans), Squarespace Domains (domains.squarespace.com), Google Cloud Run/Firestore/GCS (cloud.google.com pricing pages). GCP Sydney (australia-southeast1) Tier 2 rates applied.*