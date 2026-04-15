Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/CSSBI/pages/1800574729/Cascade+to+Alpaca+-+Push+api+Job+-+Work+in+Progress

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Cascade to Alpaca - API/Push

## Overview

### Job Identity

| Property | Value |
|----------|-------|
| Databricks Job Name | hp-ai-opt-outs-to-alpaca |
| Internal Code Name | hp_ai_opt_outs_to_alpaca |
| Purpose | Sync daily B2C email marketing consent changes to HP's Alpaca platform |
| Schedule | Daily — runs after all upstream consent_sync upsert jobs complete |
| Typical Runtime | 10–20 minutes |
| Owner | Cascade / P2-Retargeting Team |
| Failure Alerts | dataos-p2-retargetting-itt@external.groups.hp.com |

### What It Does

Every day, this job:

- Reads the consent_sync Unity Catalog table (master consent repository)
- Identifies records where a customer opted out of B2C email marketing today
- Identifies records where a customer changed their email today
- Constructs JSON payloads for each consent change
- POSTs each payload to the Alpaca Consent Write API
- Saves success/failure results back to Unity Catalog tables

### Pipeline Position

This is Job 6 — the final step in the 6-job consent_sync pipeline:

```
Job 1: stage_consent_sync_job         → Create staging table from 24+ sources
Job 2: upsert_based_on_stratus_id     → Merge by STRATUS_ID
Job 3: upsert_based_on_customer_id    → Merge by CUSTOMER_ID
Job 4: upsert_based_on_email          → Merge by email
Job 5: b2c_wrapper_and_printer_flag   → Apply business rules & finalize
Job 6: hp_ai_opt_outs_to_alpaca  ★    → Push changes to Alpaca API
```

### Key Facts

- Purpose ID: advertising.email.marketing.b2c (hardcoded)
- Parallelism: 20 concurrent API workers via Python multiprocessing.Pool
- Two credential sets: One for records with timestamp, one for records without
- Invalid emails filtered: ANON, GDPR_ERASURE
- Supports reprocessing: Set run_type = "reprocess" and provide backfill_date

## Alpaca Platform Integration

### What is Alpaca?

Alpaca is HP's centralized Consent Management Platform. It stores and manages customer consent preferences across all HP products and services. When a customer opts out of email marketing in any HP system (HPSmart, Instant Ink, HP One, etc.), that decision must be propagated to Alpaca so all downstream marketing systems respect it.

### How This Job Fits In

This job is the outbound sync leg of the consent pipeline:

```
┌──────────────────┐     ┌──────────────────┐     ┌─────────────────┐     ┌──────────────────────────┐
│   HP Systems     │     │  consent_sync    │     │   THIS JOB      │     │   Alpaca Platform        │
│  - HPSmart       │────►│  Unity Catalog   │────►│  hp_ai_opt_outs │────►│  Consent Management      │
│  - Instant Ink   │     │  (24+ sources    │     │  _to_alpaca     │     │                          │
│  - HP One        │     │   consolidated)  │     │                 │     │          ▼               │
│  - Print AuthX   │     │                  │     │                 │     │  All HP Marketing Systems│
│  - HPC Campaign  │     │                  │     │                 │     │  respect the consent     │
└──────────────────┘     └──────────────────┘     └─────────────────┘     └──────────────────────────┘
```

The consent_sync table is assembled by 5 upstream jobs that consolidate consent data from 24+ sources. This job (Job 6 in the pipeline) reads the finalized consent_sync table and pushes today's changes to Alpaca.

### Two Alpaca APIs

| # | API | HTTP Method | Purpose | Used By |
|---|-----|-------------|---------|----------|
| 1 | Consent Write API | POST | Write opt-out/opt-in consent records for a customer | This job (hp_ai_opt_outs_to_alpaca) |
| 2 | Purpose Info API | GET | Fetch locale/version mapping for a given purpose ID | Separate job (alpaca_purpose_locale_version_mapping) |

Note: This job only calls the Consent Write API directly. The Purpose Info API is called by a separate upstream job that populates the alpaca_purpose_locale_version_mapping table, which this job reads.

### Organizations Synced

Only consent changes from these HP organizations are sent to Alpaca:

| Organization URN | System |
|------------------|--------|
| urn:instantink:salesforce | Instant Ink |
| urn:hpsmart:salesforce | HP Smart |
| urn:hho:salesforce | HP Home Office |
| urn:print:authx | Print AuthX |
| urn:hpc:campaign | HPC Campaign |
| NULL | Organization unknown |

## Data Flow

### End-to-End Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────────────────────────┐
│                            hp_ai_opt_outs_to_alpaca Job                                 │
│                                                                                         │
│  ┌──────────────────────┐                                                               │
│  │ consent_sync (UC)    │─┐                                                             │
│  │ (Unity Catalog Table)│ │   ┌───────────────────────────────────┐                     │
│  └──────────────────────┘ ├──►│  1. Filter Opt-Outs for Today    │                      │
│                           │   │  2. Get Email Update Records      │                      │
│                           │   │  3. Get Simultaneous Updates      │                      │
│                           │   └──────────────┬────────────────────┘                      │
│                           │                  │                                           │
│                           │                  ▼                                           │
│  ┌──────────────────────┐ │   ┌───────────────────────────────────┐                     │
│  │ alpaca_purpose_       │ │   │  4. Union All 3 Record Sets      │                     │
│  │ locale_version_       │─┘   │  5. Add purposeId column         │                     │
│  │ mapping (UC)          │     │  6. Join locale/version mapping   │                     │
│  └──────────────────────┘     │  7. Remove invalid emails         │                     │
│                               └──────────────┬────────────────────┘                     │
│                                              │                                          │
│                                   ┌──────────┴──────────┐                               │
│                                   ▼                     ▼                                │
│                          ┌───────────────┐    ┌──────────────────┐                      │
│                          │  WITH timestamp│   │ WITHOUT timestamp │                      │
│                          │  (primary creds)│  │ (non_ts creds)   │                      │
│                          └───────┬───────┘   └────────┬─────────┘                       │
│                                  │                    │                                  │
│                                  ▼                    ▼                                  │
│                          ┌───────────────┐   ┌──────────────────┐                       │
│  ┌────────────────────┐  │  Convert to    │   │  Convert to      │                      │
│  │ AWS Secrets Manager│──│  JSON + POST   │   │  JSON + POST     │                      │
│  │ (OAuth creds)      │  │  to Alpaca API │   │  to Alpaca API   │                      │
│  └────────────────────┘  │  (20 parallel) │   │  (20 parallel)   │                      │
│                          └───────┬───────┘   └────────┬─────────┘                       │
│                                  │                    │                                  │
│                                  ▼                    ▼                                  │
│                          ┌──────────────────────────────────────┐                       │
│                          │  Union Results, Add date_added       │                       │
│                          └──────────────┬───────────────────────┘                       │
│                                   ┌─────┴─────┐                                        │
│                                   ▼           ▼                                         │
│                 ┌──────────────────────┐  ┌──────────────────────────┐                  │
│                 │ alpaca_consent_api_   │  │ alpaca_consent_api_       │                 │
│                 │ successful_requests   │  │ unsuccessful_requests     │                 │
│                 │ (UC Table, Parquet)   │  │ (UC Table, Parquet)       │                 │
│                 └──────────────────────┘  └──────────────────────────┘                  │
└─────────────────────────────────────────────────────────────────────────────────────────┘
```

### Step-by-Step Sequence

| Step | Action | Input | Output |
|------|--------|-------|--------|
| 1 | Determine processing date | run_type widget | date_str (today or backfill) |
| 2 | Load consent_sync table | Unity Catalog | consent_sync_df (DataFrame) |
| 3 | Filter direct opt-outs | consent_sync_df | latest_opt_out_df |
| 4 | Get email update records | consent_sync_df | email_update_records_df |
| 5 | Get simultaneous updates | consent_sync_df | simultaneous_email_newsletter_notify_update_df |
| 6 | Union all records | 3 DataFrames | latest_updates_df with purposeId column |
| 7 | Load locale/version mapping | Unity Catalog | locale_version_mapping_df |
| 8 | Join verbiage (locale→version) | latest_updates_df + mapping | latest_updates_with_verbiage_df |
| 9 | Remove invalid emails | verbiage DataFrame | latest_updates_with_valid_emails_df |
| 10 | Split by timestamp | valid emails DataFrame | Two lists: with_timestamp, without_timestamp |
| 11 | Convert to JSON | DataFrames | json_list_with_timestamp, json_list_without_timestamp |
| 12 | Get OAuth credentials | AWS Secrets Manager | alpaca_oauth_credentials dict |
| 13 | Send WITH timestamp | JSON list + primary creds | success/failure DataFrames |
| 14 | Swap to non-ts credentials | credentials dict | Updated credentials |
| 15 | Send WITHOUT timestamp | JSON list + non-ts creds | success/failure DataFrames |
| 16 | Union results | 4 DataFrames | successful_requests_df, unsuccessful_requests_df |
| 17 | Save to Unity Catalog | Result DataFrames | Parquet (append, partitioned by date_added) |

### Mermaid Diagram

```
flowchart TD
    CS[(consent_sync)] --> Date{Determine<br/>Processing Date}
    
    Date -->|daily| Today[date_str = today]
    Date -->|reprocess| Backfill[date_str = backfill_date]
    
    Today --> Extract
    Backfill --> Extract
    
    Extract[Extract Daily Changes] --> OptOut[Get Opt-Outs on date_str<br/>Filter by organization]
    Extract --> EmailUpdate[Get Email Updates on date_str]
    Extract --> Simultaneous[Get Simultaneous Email+NN Updates]
    
    OptOut --> Union[Union All Updates]
    EmailUpdate --> Union
    Simultaneous --> Union
    
    Union --> Locale[Join Locale Version Mapping<br/>Create verbiage column]
    
    Locale --> Validate[Remove Invalid Emails]
    
    Validate --> Split{Has<br/>Timestamp?}
    
    Split -->|Yes| WithTS[Records with Timestamp]
    Split -->|No| NoTS[Records without Timestamp]
    
    WithTS --> JSON1[Convert to JSON Payloads]
    NoTS --> JSON2[Convert to JSON Payloads]
    
    JSON1 --> Auth1[Get OAuth Token<br/>Primary credentials]
    JSON2 --> Auth2[Get OAuth Token<br/>Non-timestamp credentials]
    
    Auth1 --> API1[POST to Alpaca API<br/>20 parallel workers]
    Auth2 --> API2[POST to Alpaca API<br/>20 parallel workers]
    
    API1 --> Success1[Successful Requests]
    API1 --> Failure1[Unsuccessful Requests]
    API2 --> Success2[Successful Requests]
    API2 --> Failure2[Unsuccessful Requests]
    
    Success1 --> UnionSuccess[Union Successful]
    Success2 --> UnionSuccess
    Failure1 --> UnionFailure[Union Unsuccessful]
    Failure2 --> UnionFailure
    
    UnionSuccess --> SaveSuccess[(Save to UC<br/>successful_requests)]
    UnionFailure --> SaveFailure[(Save to UC<br/>unsuccessful_requests)]
    
    SaveSuccess --> End([Job Complete])
    SaveFailure --> End
```

## APIs Used

### API 1: Alpaca Consent Write API (POST)

This is the primary API used by this job. Every consent change is sent as an individual POST request.

Endpoint Details

| Property | Value |
|----------|-------|
| URL | Stored in AWS Secrets Manager as alpaca_write_consent_url |
| Method | POST |
| Authentication | OAuth 2.0 Bearer Token |
| Content-Type | Stored in secrets (typically application/json) |

Request Headers

```
POST {alpaca_write_consent_url}
Authorization: Bearer {oauth_access_token}
Content-Type: {Content-Type from secrets}
```

Request Body — Variant A (Without mdmPersonIds)

Used for direct opt-outs where no HPID customer identifier exists:

```json
{
  "action": "opt-out",
  "purposeId": "advertising.email.marketing.b2c",
  "organization": "urn:instantink:salesforce",
  "timestamp": "2025-01-28T14:30:00Z",
  "verbiage": {
    "locale": "en-US",
    "version": "3"
  },
  "dataSubject": {
    "email": "customer@example.com"
  }
}
```

Request Body — Variant B (With mdmPersonIds)

Used for email-update records where HPID exists:

```json
{
  "action": "opt-out",
  "purposeId": "advertising.email.marketing.b2c",
  "organization": "urn:css:hpid",
  "timestamp": "2025-01-28T14:30:00Z",
  "verbiage": {
    "locale": "en-US",
    "version": "3"
  },
  "dataSubject": {
    "email": "old-email@example.com",
    "mdmPersonIds": ["HPIDcustomer123"]
  }
}
```

Response — Success (HTTP 200)

```json
[
  {
    "transactionId": "abc-123-def",
    "transactionDate": "2025-01-28T14:30:05Z",
    "dataSubject": {
      "email": "customer@example.com"
    }
  }
]
```

Response — Error (HTTP 4xx/5xx)

```json
{
  "errors": [
    {
      "statusCode": "400",
      "message": "Invalid request body"
    }
  ]
}
```

Field Reference

| Field | Value | Description |
|-------|-------|-------------|
| purposeId | advertising.email.marketing.b2c | Hardcoded — all records are B2C email marketing |
| action | opt-out or opt-in | The consent action to record |
| organization | URN string | Source system (e.g. urn:instantink:salesforce) |
| timestamp | ISO 8601 (yyyy-MM-dd'T'HH:mm:ss'Z') | When consent changed. Can be NULL |
| verbiage.locale | e.g. en-US, de-DE, und | Locale of consent verbiage. Falls back to und |
| verbiage.version | e.g. "3" | Version of consent language for that locale |
| dataSubject.email | Email address | Customer email |
| dataSubject.mdmPersonIds | Array of strings | Optional — HPID prefixed with "HPID" |

Code Location

```python
# Post function: hp_ai_opt_outs_to_alpaca_utils.py
def post_request_to_alpaca_consent_api(json, alpaca_oauth_credentials):
    url = alpaca_oauth_credentials["alpaca_write_consent_url"]
    headers = {
        "Authorization": f"Bearer {alpaca_oauth_credentials['oauth_access_token']}",
        "Content-Type": alpaca_oauth_credentials["Content-Type"],
    }
    response = requests.post(url, data=body, headers=headers)
    return response
```

### API 2: Alpaca Purpose Info API (GET)

Used by the separate alpaca_purpose_locale_version_mapping job to refresh the locale/version mapping table. This job does NOT call this API directly — it consumes the resulting Unity Catalog table.

Endpoint Details

| Property | Value |
|----------|-------|
| URL | {purpose_info_url}{purpose_id} (from secrets) |
| Method | GET |
| Authentication | OAuth 2.0 Bearer Token |

Request

```
GET {purpose_info_url}advertising.email.marketing.b2c
Authorization: Bearer {oauth_access_token}
```

Response (HTTP 200)

```json
{
  "records": [
    {
      "verbiage": {
        "locale": "en-US",
        "version": "3"
      }
    },
    {
      "verbiage": {
        "locale": "de-DE",
        "version": "2"
      }
    }
  ]
}
```

This response is parsed into the alpaca_purpose_locale_version_mapping Unity Catalog table:

| Column | Example |
|--------|----------|
| purposeId | advertising.email.marketing.b2c |
| locale | en-US |
| version | 3 |

Code Location

```python
# alpaca_purpose_locale_version_mapping_utils.py
def get_request_to_alpaca_purpose_api(alpaca_oauth_credentials, purpose_id):
    url = f"{alpaca_oauth_credentials['purpose_info_url']}{purpose_id}"
    headers = {
        "Authorization": f"Bearer {alpaca_oauth_credentials['oauth_access_token']}",
    }
    response = requests.get(url, headers=headers)
    return response
```

## Authentication and Secrets

### OAuth 2.0 Client Credentials Flow

All Alpaca API calls use OAuth 2.0 Client Credentials Grant. This is a server-to-server flow — no user interaction required.

Token Request

```
POST {oauth_token_url}
Content-Type: application/x-www-form-urlencoded
Authorization: Basic base64(client_id:secret)

grant_type=client_credentials&scope={scope}
```

Token Response

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIs...",
  "token_type": "Bearer",
  "expires_in": 3600
}
```

Code Implementation

```python
# util_commons.py (line 1409)
def get_oauth_access_token(alpaca_oauth_credentials):
    body = {
        "grant_type": alpaca_oauth_credentials["grant_type"],
        "scope": alpaca_oauth_credentials["scope"],
    }
    response = requests.post(
        alpaca_oauth_credentials["oauth_token_url"],
        data=body,
        auth=(
            alpaca_oauth_credentials["client_id"],
            alpaca_oauth_credentials["secret"],
        ),
    )
    access_token = response.json()["access_token"]
    return access_token
```

### AWS Secrets Manager

| Property | Value |
|----------|-------|
| Secret Name (prod) | prod/team-cascade/alpaca-api-secrets |
| Retrieval Method | get_secret(alpaca_oauth_secret_name) (Databricks utility) |
| Region | Same as Databricks workspace |

### Two Credential Sets

The secret contains two sets of OAuth credentials. Alpaca requires different credentials for requests depending on whether a timestamp is present:

Secret Key Map

| Key | Used For | Description |
|-----|----------|-------------|
| client_id | Records WITH timestamp | Primary OAuth client ID |
| secret | Records WITH timestamp | Primary OAuth client secret |
| scope | Records WITH timestamp | Primary OAuth scope |
| non_timestamp_client_id | Records WITHOUT timestamp | Alternate OAuth client ID |
| non_timestamp_secret | Records WITHOUT timestamp | Alternate OAuth client secret |
| non_timestamp_scope | Records WITHOUT timestamp | Alternate OAuth scope |
| oauth_token_url | Both | Token endpoint URL |
| alpaca_write_consent_url | Both | Consent Write API URL |
| purpose_info_url | Locale mapping job | Purpose Info API base URL |
| Content-Type | Both | Request content type |
| grant_type | Both | Always client_credentials |

Why Two Credential Sets?

- With timestamp: Direct opt-outs have an explicit opt_out_date. These use the primary credentials.
- Without timestamp: Email-update records for old emails may not have an opt-out date. These use separate credentials that don't require the timestamp field.

Credential Swap in Code

```python
# Main notebook — after sending with-timestamp records, swap credentials:
alpaca_oauth_credentials["client_id"] = alpaca_oauth_credentials["non_timestamp_client_id"]
alpaca_oauth_credentials["secret"]    = alpaca_oauth_credentials["non_timestamp_secret"]
alpaca_oauth_credentials["scope"]     = alpaca_oauth_credentials["non_timestamp_scope"]

# Then send without-timestamp records
send_consents_to_alpaca(json_list_without_timestamp, alpaca_oauth_credentials)
```

Token Lifecycle

```
Job Start
  │
  ├── Get OAuth token (primary credentials)
  │     └── Token cached in credentials dict
  │
  ├── Send WITH-timestamp records (20 parallel workers)
  │     ├── If 401 → Refresh token → Retry once
  │     └── If 5xx → Sleep 5 min → Retry once
  │
  ├── Swap to non-timestamp credentials
  │
  ├── Get OAuth token (non-timestamp credentials)
  │     └── New token cached
  │
  └── Send WITHOUT-timestamp records (20 parallel workers)
        ├── If 401 → Refresh token → Retry once
        └── If 5xx → Sleep 5 min → Retry once
```

## Record Types and Business Logic

The job processes three categories of consent changes from the consent_sync table each day.

### Type 1: Direct Opt-Outs

Function: get_opt_out_on_given_date(consent_sync_df, date_str)

Logic

- Filter where newsletter_notify = "opt-out" AND newsletter_notify_updated_date = today
- Exclude records where (css_flag = True OR imt_flag = True) AND newsletter_notify == alpaca_b2c_consent
  - These are already synced to Alpaca — no need to send again
- Filter to specific organizations:

| Organization URN | System |
|------------------|--------|
| urn:instantink:salesforce | Instant Ink |
| urn:hpsmart:salesforce | HP Smart |
| urn:hho:salesforce | HP Home Office |
| urn:print:authx | Print AuthX |
| urn:hpc:campaign | HPC Campaign |
| NULL | Organization unknown |

Output Columns

| Column | Value |
|--------|-------|
| organization | From consent_sync |
| timestamp | From opt_out_date |
| locale | From consent_sync |
| email | From consent_sync |
| action | "opt-out" (hardcoded) |
| customer_id | NULL (set to null — Alpaca doesn't need it for direct opt-outs) |

### Type 2: Email Update Records (from Opt-In Users)

Function: get_email_update_records_from_opt_in(consent_sync_df, date_str)

Scenario

A customer who is opted-in to marketing changes their email address from old@hp.com to new@hp.com.

What Happens

| Record | Email | Action | Organization | customer_id |
|--------|-------|--------|--------------|-------------|
| Old email opt-out | old@hp.com | opt-out | urn:css:hpid | HPIDcustomer123 |
| New email opt-in | new@hp.com | opt-in | urn:css:hpid | HPIDcustomer123 |

Logic

- Filter where email_updated_date = today AND newsletter_notify = "opt-in"
- Remove current email from possible_emails array
- For each old email in possible_emails:
  - Generate opt-out record
  - Set organization to urn:css:hpid
  - Set timestamp from email_updated_date
- For new (current) email:
  - Generate opt-in record
  - Set organization to urn:css:hpid
  - Set timestamp from email_updated_date
- Prefix customer_id with "HPID" → becomes mdmPersonIds in API request

Why This Matters

If a user changes their email while opted-in, marketing to the old email must stop (opt-out), and the new email must be registered as opted-in. Without this, the old email could continue receiving marketing.

### Type 3: Simultaneous Email + Newsletter Notify Updates

Function: get_simultaneous_email_newsletter_notify_update_records(consent_sync_df, date_str)

Scenario

A customer changes their email AND opts out of marketing on the same day.

What Happens

| Record | Email | Action | Timestamp Source |
|--------|-------|--------|------------------|
| Old email(s) | Each from possible_emails | opt-out | email_updated_date |
| New email | Current email | opt-out | opt_out_date |

Logic

- Filter where newsletter_notify_updated_date = today AND email_updated_date = today AND newsletter_notify = "opt-out"
- Remove current email from possible_emails
- Old email(s) → opt-out records
- New email → opt-out record (uses opt_out_date as timestamp, not email_updated_date)

Why This Matters

Edge case: Both old and new emails must be opted out. The timestamps differ because the opt-out and email change may have occurred at different times within the same day.

### After Getting All Three Types

Union All Records

```python
latest_updates_df = (
    latest_opt_out_df
    .unionByName(email_update_records_df)
    .unionByName(simultaneous_email_newsletter_notify_update_df)
    .withColumn("purposeId", F.lit("advertising.email.marketing.b2c"))
)
```

Verbiage Enrichment

Function: create_verbiage_column(all_records_df, locale_version_mapping_df)

- Left join with alpaca_purpose_locale_version_mapping on (purposeId, locale)
- If locale is NULL or no version found → fallback locale to "und" (undefined)
- Re-join with mapping to get the und version
- Create verbiage struct: {locale, version}
- Format timestamp to ISO 8601: yyyy-MM-dd'T'HH:mm:ss'Z'

Invalid Email Filtering

Function: remove_invalid_emails(df)

Removes records where email is in the blocklist:

| Blocked Email | Reason |
|---------------|--------|
| ANON | Anonymous/placeholder email |
| GDPR_ERASURE | Email erased under GDPR right-to-erasure |

Split by Timestamp

```python
with_timestamp    = df.filter(F.col('timestamp').isNotNull())
without_timestamp = df.filter(F.col('timestamp').isNull())
```

Each group uses different OAuth credentials when calling the Alpaca API.

## API Request Response Schemas

### JSON Payload Construction

The function get_request_json_list(df) converts a DataFrame into a list of JSON strings for the API.

Records Without customer_id

```python
# dataSubject contains only email
{
  "dataSubject": {"email": "customer@example.com"},
  "action": "opt-out",
  "purposeId": "advertising.email.marketing.b2c",
  "organization": "urn:instantink:salesforce",
  "timestamp": "2025-01-28T14:30:00Z",
  "verbiage": {"locale": "en-US", "version": "3"}
}
```

Records With customer_id

```python
# dataSubject contains email + mdmPersonIds array
{
  "dataSubject": {
    "email": "customer@example.com",
    "mdmPersonIds": ["HPIDcustomer123"]
  },
  "action": "opt-out",
  "purposeId": "advertising.email.marketing.b2c",
  "organization": "urn:css:hpid",
  "timestamp": "2025-01-28T14:30:00Z",
  "verbiage": {"locale": "en-US", "version": "3"}
}
```

### Successful Response Schema

Saved to: team_cascade_prod.engineering_room.alpaca_consent_api_successful_requests

PySpark Schema Definition

```python
SUCCESSFUL_REQUESTS_SCHEMA = StructType([
    StructField("transactionId", StringType(), True),
    StructField("transactionDate", StringType(), True),
    StructField("dataSubject", StructType([
        StructField("email", StringType(), True)
    ]), True)
])
```

Columns Written to Table

| Column | Type | Source | Description |
|--------|------|--------|-------------|
| transactionId | STRING | API response | Alpaca transaction identifier |
| transactionDate | STRING | API response | When Alpaca processed the request |
| email | STRING | dataSubject.email from response | Customer email |
| date_added | DATE | current_date() | Partition column |

### Unsuccessful Response Schema

Saved to: team_cascade_prod.engineering_room.alpaca_consent_api_unsuccessful_requests

PySpark Schema Definition

```python
UNSUCCESSFUL_REQUESTS_SCHEMA = StructType([
    StructField("Response Status Code", StringType(), True),
    StructField("Response Error", StructType([
        StructField("statusCode", StringType(), True),
        StructField("message", StringType(), True),
    ]), True),
    StructField("Request opt_out_json", StringType(), True),
])
```

Columns Written to Table

| Column | Type | Source | Description |
|--------|------|--------|-------------|
| Response Status Code | STRING | HTTP status code | NULL if exception occurred |
| Response Error.statusCode | STRING | Error response body | Alpaca error code |
| Response Error.message | STRING | Error response body | Alpaca error message |
| Request opt_out_json | STRING | Original request | The JSON payload that failed |
| date_added | DATE | current_date() | Partition column |

### Storage Details

| Property | Value |
|----------|-------|
| Format | Parquet |
| Write Mode | Append |
| Partition Column | date_added |
| Location | Unity Catalog managed tables |