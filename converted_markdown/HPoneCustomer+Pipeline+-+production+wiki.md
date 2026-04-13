Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/CSSBI/pages/1770633358/HPoneCustomer+Pipeline+-+production+wiki

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# HPoneCustomer Pipeline - production wiki

HPOne Customers – Databricks Job (hpone_customers_job)

Platform: AWS Databricks (Spark / PySpark)
Primary Output: hpone_customers (Unity Catalog – Delta)
Last Updated: January 27, 2026

Production Job → hpone-customers-prod - Databricks

Git url → hpone-hpx-processor/notebooks/main/hpone_customers at master · dataos-p2-retargeting/hpone-hpx-processor

TL;DR (For Developers)

- What this does: Builds a unified, enriched 360° customer profile across HP systems by joining account data, consent, subscriptions, org affiliation, EPC mapping, last seen, and device linkages.
- Why it matters: Single source of truth for downstream marketing (SFMC), analytics, and identity-centric use cases—privacy-compliant and delta-aware.
- How it runs: Full refresh with a delta checkpoint table to track changes across runs.
- Key outputs: 30+ attributes per customer, including CUSTOMER_ID (HPID), HASHED_CUSTOMER_ID, newsletter_notify, unified_account_oid, ink_subscribed, epc_uid, last_seen, has_linked_printer.
- Critical filters: Excludes unsubscribed emails and non-HPID customers.
- Security: Customer IDs hashed with SHA256 + salt (AWS Secrets Manager).

Business Context

Purpose: Build a unified, enriched customer dataset for HP One that consolidates identities and engagement across HPID, EPC, Stratus, UCDE, Origami, SIPS, and Printer PET.

Primary Use Cases

- Marketing enablement with consent and subscriber intelligence
- Segment building (tenants, linked devices, Instant Ink)
- Identity resolution (HPID ↔ EPC UID ↔ Cascade)
- Engagement analytics via last seen and device linking

Job Type: Full refresh with delta checkpoint to support change detection for downstream consumers.

High-Level Architecture

INPUTS (Metastore/Unity/Parquet/CSV)
   ├─ User Account Status (primary customers)
   ├─ Subscription Enrollment (events)
   ├─ UCDE Org Updates
   ├─ Consents Sync
   ├─ Unsubscribed Emails
   ├─ Origami (subscription accounts)
   ├─ Customer Profile (Cascade mapping)
   ├─ HPID ↔ EPC Mapping
   ├─ DIM SIPS Prod Users (logins)
   └─ Printer PET (device linkage)

        │
        ▼

PROCESSING (Phases 1–17)
   1) CUSTOMER_ID definition (HPID only)
   2) Standardize columns
   3) Security: hash customer IDs
   4) Locale standardization & US override for subscribers
   5) Remove unsubscribed emails
   6) Email history (legacy_email)
   7) Tenant enrichment (subscription/org)
   8) Consent normalization (I/O/Unknown)
   9) Cascade ID mapping
  10) Origami (unified accounts, Instant Ink)
  11) EPC UID mapping
  12) Last seen enrichment (HPID/EPC)
  13) Printer attachment enrichment
  14) Final cleanup & defaults

        │
        ▼

OUTPUTS (Unity Catalog – Delta)
   ├─ hpone_customers (overwrite)
   └─ hpone_customers_delta_checkpoint (previous state)

Some of Input tables:

- user_account_status.user_account_status
- team_cascade_prod.operations_room.origami
- team_cascade_prod.operations_room.customer_profile
- team_cascade_prod.engineering_room.dim_sips_prod_users
- team_cascade_prod.engineering_room.stg_printer_pet_printers

output table: team_cascade_prod.operations_room.hpone_customers

Data Sources (Inputs)

Note: All filters/dedup rules are enforced at load where possible to reduce shuffle and ensure consistent lineage.

- user_account_status.user_account_status (Metastore)
  Primary account attributes (name, email, locale, IDP fields, created/updated, validations).

- subscription_enrollment_source (Parquet)
  Used to: set LOCALE='en_US' for subscribed Stratus IDs, fill missing tenant IDs.

- ucde_org_update_source (Unity)
  Latest org event per user where tenantType='Personal' to enrich company_name and idpTenantId.

- consents_sync_source (Unity)
  Consent to newsletter; mapped to I/O/Unknown with update timestamp.

- unsubscribed_email_source (Parquet)
  Anti-join to exclude opted-out emails.

- hpone_customers_destination_path (Unity)
  Current/previous dataset for legacy_email tracking and checkpointing.

- team_cascade_prod.operations_room.origami (Metastore)
  Aggregated subscriptions → unified_account_oid and ink_subscribed where active & paid.

- team_cascade_prod.operations_room.customer_profile (Metastore)
  Map customer_id → cascade_id.

- static_hpid_to_epc_mapping_source (Unity)
  HPID → EPC UID mapping; only non-null EPC.

- team_cascade_prod.engineering_room.dim_sips_prod_users (Metastore)
  Login activity; derive HPID/EPC by length (32/64), compute last_seen per identifier.

- team_cascade_prod.engineering_room.stg_printer_pet_printers (Metastore)
  Printer attachment counts, with non-null attach date.

- hpone_customers_test_data_source (CSV)
  Test fixtures unioned in for QA (kept distinct).

Outputs

1) hpone_customers (Primary)

- Type: Unity Catalog Delta (overwrite each run)
- Rowset: All HP One customers excluding unsubscribed emails
- Grain: One row per HPID customer (with enrichments)

2) hpone_customers_delta_checkpoint

- Type: Unity Catalog Delta (overwrite each run, written before new output)
- Purpose: Holds previous hpone_customers state for downstream delta/change detection.

End-to-End Data Flow

Detailed phase breakdown aligned to functions/utilities in the codebase.

- Load & Initialize: Filters pushed to sources (UCDE tenantType='Personal', Origami active/paid, PET non-null attach date). Test CSV unioned.
- Define CUSTOMER_ID: HPID only when associated_idp_type='hpid'.
- Standardize Columns: Rename raw fields to canonical names.
- Hashing: HASHED_CUSTOMER_ID = SHA256(CUSTOMER_ID + salt) (salt via AWS Secrets Manager).
- Locale: Assign en_US for subscribed users; normalize variants (en-US → en_US); derive LANGUAGE/COUNTRY.
- GDPR Filter: Remove unsubscribed emails (left anti).
- Email History: Compare against previous table → legacy_email when changed.
- Tenant Enrichment: Fill idpTenantId from subscription events first, then UCDE. Add company_name.
- Consent: Normalize newsletter_notify (I/O/Unknown) with timestamp.
- Cascade Mapping: Join Customer Profile → cascade_id.
- Subscriptions: Origami aggregation → unified_account_oid, ink_subscribed.
- EPC UID: Static mapping → epc_uid.
- Last Seen: Parse SIPS IDs, aggregate max across HPID/EPC.
- Printer Linkage: Per-customer counts, flags; join by CUSTOMER_ID or epc_uid.
- Final Touches: Default consent timestamp using ACCOUNT_CREATED for I/Unknown.
- Checkpoint & Write: Save previous → write new.

Source → Target Transformations

High-level mapping (details mirror code functions).

- ID & Identity
  - CUSTOMER_ID := idp_id when associated_idp_type='hpid'
  - HASHED_CUSTOMER_ID := SHA256(CUSTOMER_ID + salt)
  - epc_uid via static HPID ↔ EPC mapping
  - cascade_id via Customer Profile

- Person & Account
  - Names, email, timestamps: direct renames
  - legacy_email: inferred via previous run comparison

- Locale
  - US override for subscribed Stratus IDs
  - Normalize "lang-COUNTRY" → "lang_COUNTRY", derive LANGUAGE, COUNTRY

- Consent
  - newsletter_notify: "opt-in" → "I", "opt-out" → "O", else Unknown
  - newsletter_notify_updated_date: defaults to ACCOUNT_CREATED for I/Unknown

- Subscriptions & Devices
  - unified_account_oid: collected set of account_id from Origami
  - ink_subscribed: True if any program is "Instant Ink"
  - total_printers_attached, has_linked_printer: PET aggregations

- Engagement
  - last_seen: max over matched HPID/EPC from SIPS

Schema (Core Columns)

Full schema is 30+ columns; below are the most queried.

- Identity & Contact: CUSTOMER_ID, HASHED_CUSTOMER_ID, STRATUS_ID, EMAIL, legacy_email, epc_uid, cascade_id
- IDP: associatedIdpType, idpId, idpType, idpTenantId
- Person: FIRST_NAME, LAST_NAME
- Locale: LOCALE, LANGUAGE, COUNTRY
- Timestamps: ACCOUNT_CREATED, ACCOUNT_UPDATED, newsletter_notify_updated_date, last_seen
- Consent: newsletter_notify (I/O/Unknown)
- Phone Verification: hashed_phone_number, phone_number_validated, email_validated
- Subscriptions: unified_account_oid (array), ink_subscribed (True/False)
- Devices: total_printers_attached, has_linked_printer (True/False)
- Org: company_name

Operational Runbook

Parameters (required)

- env, job_name, run_job, hashing_secret, catalog_name

Sources (paths/tables)

- hpone_customers_destination_path, hpone_customers_delta_checkpoint_source, ucde_org_update_source, hpone_customers_test_data_source, unsubscribed_email_source, subscription_enrollment_source, consents_sync_source, static_hpid_to_epc_mapping_source, dim_sips_prod_source

Write Pattern

- Checkpoint: Load current hpone_customers → write to hpone_customers_delta_checkpoint
- Final: Overwrite hpone_customers with enriched output

Failure Modes & Recovery

- Source load failure: Job fails fast with source context; re-run after upstream fix.
- Schema drift: Columns added upstream → ensure column pruning; unexpected type mismatch requires small code patch.
- Secrets failure: Missing salt/permission → rotate/restore IAM policy; re-run.
- Large shuffle: Verify broadcast hints & partitions; consider adaptive execution.

Performance Notes

- Filter Pushdown: Apply UCDE/Origami/PET conditions at read.
- Column Pruning: select early to avoid wide shuffles.
- Broadcast Joins: Apply for small dims (HPID↔EPC mapping, consents).
- Window Functions: Used for UCDE latest row & SIPS last_seen aggregation; ensure partitions are well distributed.
- Partitioning: Consider partitioning the Delta table on COUNTRY or idpTenantId only if query patterns justify it (avoid over-partitioning).

Security, Privacy & Compliance

- PII: EMAIL, FIRST_NAME, LAST_NAME, phone verification fields.
- Pseudonymization: HASHED_CUSTOMER_ID = SHA256(HPID + salt) (salt via AWS Secrets Manager).
- Consent: newsletter_notify enforced; unsubscribed emails anti-joined.
- Access Control: Unity Catalog permissions; grant read on need-to-know basis.
- Data Retention: Align with enterprise policy (TBD) → define retention for checkpoint table if needed.
- Secrets Handling: No secrets in code; rely on Databricks secret scope / AWS Secrets Manager integration.

Monitoring & Observability

- Lifecycle Logs: log_job_start(), log_job_done(), info(), error() with task names (e.g., TASK_LOAD_*, TASK_UPLOAD_*).
- Key Metrics to Track
  - Record counts per phase
  - Null rates: CUSTOMER_ID, EMAIL, idpTenantId
  - Enrichment coverage: % with epc_uid, % with last_seen, % with unified_account_oid, % with printers
  - Email change rate: % with legacy_email
  - Job duration & shuffle metrics
- Alerting: Add Databricks job alerts on failure & SLA breach (TBD).

FAQs & Troubleshooting

Q: Why is epc_uid null for some customers?
A: Not all HPIDs map to EPC UIDs in the static mapping table. Verify upstream mapping freshness; ensure no invalid/underscore IDs are filtered out prematurely.

Q: My last_seen is missing but the user is active.
A: SIPS may log against EPC or HPID; confirm both are present and normalized. Check ID parsing logic (32 vs 64 length). Confirm the SIPS record has no underscores or blank IDs.

Q: How do I test changes safely?
A: Use the test CSV injection and run in dev catalog. Validate DQ queries above; then promote.

Q: Where does legacy_email come from?
A: From the previous run's hpone_customers via delta checkpoint; we store the prior email when it changes.

Mermaid Data Flow

User Account Status
A 2
Define Customer ID
v
Rename Columns
v
Hash Customer ID
|
Subscription Enrollment
Process Locale
Unsubscribed Emails
N
~
Remove Unsubscribed
Previous Customers
~
Track Email History
N
Enrich Tenant ID
Consents Sync
N
~
Add Consent Data
UCDE Org Updates
Add Org Info
Customer Profile
N
~
Add Cascade ID
Origami
N
/
Add Subscription
HPID-EPC Mapping
Add EPC UID
DIM SIPS Prod
.
/
Add Last Seen
Printer PET
N
/
Add Printer Info
Final Output
hpone_customers Table
Checkpoint Table