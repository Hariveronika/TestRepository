Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/CSSBI/pages/1800574729/Cascade+to+Alpaca+-+Push+api+Job+-+Work+in+Progress

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Cascade to Alpaca - Push/api Job - Work in Progress

Comprehensive platform-level documentation for the Alpaca Consent Sync job

## What This Job Does

Every day, the hp_ai_opt_outs_to_alpaca job reads the consent_sync Unity Catalog table, identifies customers who opted out of B2C email marketing (or changed their email), and POSTs each consent change to HP's Alpaca Consent Management Platform via REST API.

| Property | Value |
|----------|-------|
| Job Name | hp-ai-opt-outs-to-alpaca |
| Internal Name | hp_ai_opt_outs_to_alpaca |
| Schedule | Daily (after upstream consent_sync jobs) |
| Runtime | 10–20 minutes |
| Owner | Cascade / P2-Retargeting Team |
| Failure Alerts | dataos-p2-retargetting-itt@external.groups.hp.com |

## Wiki Pages

### Architecture & Design

| Page | Description |
|------|-------------|
| Overview | Job purpose, schedule, and pipeline context |
| Alpaca Platform Integration | What is Alpaca, how this job fits in the pipeline |
| Data Flow | End-to-end data flow diagram |

### APIs & Authentication

| Page | Description |
|------|-------------|
| APIs Used | Consent Write API (POST) and Purpose Info API (GET) — full request/response examples |
| Authentication and Secrets | OAuth 2.0 flow, AWS Secrets Manager, two credential sets |

### Business Logic

| Page | Description |
|------|-------------|
| Record Types and Business Logic | Three record types: direct opt-outs, email updates, simultaneous changes |
| API Request Response Schemas | Success/failure schemas saved to Unity Catalog |

### Data & Configuration

| Page | Description |
|------|-------------|
| Unity Catalog Tables | Input/output tables and key columns |
| Job Configuration | Widget parameters, config files, deployment |
| Cluster and Infrastructure | Compute, Spark config, libraries, retry policy |

### Operations

| Page | Description |
|------|-------------|
| Error Handling and Retry Logic | HTTP response handling, token refresh, retry strategy |
| Parallelism and Performance | Multiprocessing pool, throughput, sequential steps |
| Code Reference Map | File-by-file source code map |
| Operational Runbook | Health checks, reprocessing, troubleshooting |

## Quick Links

- Main Notebook: hpone-hpx-processor/notebooks/main/hp_ai_opt_outs_to_alpaca/hp_ai_opt_outs_to_alpaca_job.py
- Utils: hpone-hpx-processor/notebooks/main/hp_ai_opt_outs_to_alpaca/hp_ai_opt_outs_to_alpaca_utils.py
- Prod Config: hpone-hpx-config/jobs/hp-ai-opt-outs-to-alpaca/prod.yml
- Secret: prod/team-cascade/alpaca-api-secrets (AWS Secrets Manager)
- Purpose ID: advertising.email.marketing.b2c

## Pipeline Position

```
[24+ Sources] → [Stage] → [Upsert by ID] → [Upsert by CID] → [Upsert by Email] → [Finalize] → [Alpaca Sync ★]
     ↓             ↓           ↓                ↓                   ↓               ↓              ↓
   Job 1         Job 1       Job 2            Job 3               Job 4           Job 5          Job 6
```

This is Job 6 — the final step in the consent_sync pipeline.

Document Version: 1.0 | Last Updated: March 2026 | Source: Codebase analysis of hpone-hpx-processor and hpone-hpx-config