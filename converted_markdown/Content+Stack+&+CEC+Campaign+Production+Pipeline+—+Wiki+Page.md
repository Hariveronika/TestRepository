Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/CSSBI/pages/1770633532/Content+Stack+CEC+Campaign+Production+Pipeline+%E2%80%94+Wiki+Page

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Content Stack & CEC Campaign Production Pipeline — Wiki Page

Overview

The Content Stack and CEC Campaign Pipeline is a multi‑stage Spark/Delta Lake workflow running in Databricks to process marketing content from:

- ContentStack CMS
- CEC (Customer Engagement Center)

Its purpose is to ingest, flatten, merge, normalize, and prepare campaign content for downstream analytics, targeting, and activation pipelines.

It processes 11 content types, produces historical and production-ready Delta tables, and applies change tracking, deduplication, and production state logic.

Architecture Summary

Core Pipeline Stages

| Stage | Job | Purpose |
|-------|-----|----------|
| 1 | content_stack_archive_job | Archive raw external JSON exports. |
| 2 | flatten_cec_campaign_content_stack_job | Flatten CEC JSON into Delta. |
| 3 | flatten_content_stack_job | Flatten ContentStack JSON across 11 content types. |
| 4 | combined_content_stack_and_cec_campaign_job | Merge CS + CEC datasets with schema alignment. |
| 5 | process_content_stack_and_cec_campaign_job | Extract jumpIds, track variation changes, dedupe, output final Delta. |

3. High-Level Data Flow

External S3 (CMS Export)
        │
        ▼
[Job 1] Archive JSON
        │
   ┌────┴────────┐
   ▼             ▼
[Job 2]       [Job 3]
Flatten CEC   Flatten ContentStack
   └───────┬────────┘
           ▼
       [Job 4]
        Combine
           ▼
       [Job 5]
       Process
           ▼
 Final Production Delta

(Derived from DataFlow.md diagram set)

4. Detailed Job Sequence

Execution order (from JobSequence.md):

- Job 1 – Archive
- Job 2 – Flatten CEC — runs parallel to Job 3
- Job 3 – Flatten ContentStack — runs parallel to Job 2
- Job 4 – Combine
- Job 5 – Process

Parallel execution is permitted only for Jobs 2 and 3. All others are serial dependencies.

(Reference: JobSequence.md)

5. Inputs & Outputs

Full inventory (from SourcesAndTargets.md):

Total Inputs: 9 unique S3 paths

Total Outputs: 6 unique S3 paths

No Unity Catalog tables are referenced — all I/O uses direct S3 paths.

Example (Job 3)

Inputs

- s3a://.../content_stack_staging/ (JSON)
- s3a://.../content_stack_archive/ (JSON)

Outputs

- s3a://.../content_stack_archived_table/ (Delta)
- s3a://.../exploded_jsons/ (Delta)

Every job follows similar S3-based patterns.

6. Content Types Processed

From HLD + LLD: 11 types

- Universal
- Email
- Email to ES
- HPX Action Tile
- HPX Interstitial
- HPX Next Steps Tile
- HPX Promotional CEC Tile
- HPX Welcome Banner
- Message Hub Events
- Message Hub Events to ES
- Supplies Tile

Each type has its own schema normalization utilities and alignment logic in Jobs 3, 4, and 5.

7. Key Processing Features

1. Incremental Processing

Jobs 2 & 3 only process new files by comparing archive vs staging.

2. Schema Normalization

95+ mandatory columns across all content types unified in Job 4.

3. JumpID Extraction

Job 5 extracts jumpIds across platforms:

- Android
- iOS
- Windows
- Mac
- For Message Hub: atlas/chat/crmflow/mobilepush/desktoppush

4. Deduplication

Key: uid, variation_id, campaign_name

Keeps latest record based on insert_date.

5. Change Tracking

Adds:

- variation_last_changed
- Production flags:
  - in_production
  - variation_in_production

8. Final Output Dataset

From Job 5 output:

Final Location (configurable):

content_stack_and_cec_campaign_destination (S3 path)

Contains:

- unified, deduped, schema‑standardized content
- 16 casted fields
- production flags
- jumpIds normalized across content types
- 11 content type structures unified

This dataset is used by downstream targeting, activation, and analytics pipelines.

9. Operational Instructions (Runbook Summary)

Before Each Run

- Validate external S3 source availability
- Validate IAM AssumeRole permissions
- Verify cluster configuration (4+ workers, 11.3 LTS)
- Ensure staging paths have no corruption

Normal Execution

Recommended: Databricks Workflow: content_stack_pipeline_prod

Auto‑runs all 5 jobs with dependency management.

Monitoring

- Check S3 for new archived files
- Check row counts after Jobs 2, 3, 4, and 5

Common Issues

- No new files → expected for incremental
- Schema mismatch → update schema_commons
- OOM → increase cluster size or repartition
- Delta corruption → use Delta Time Travel

10. Mermaid Diagrams

You already have full diagrams in DataFlow.md.

Here is the main execution sequence diagram, compatible with Confluence:

2. flatten_cec_campaign_content_stack_job

4. combined_content_stack_and_cec_campaign_job

1. content_stack_archive_job

5. process_content_stack_and_cec_campaign_job

3. flatten_content_stack_job

12. Summary

This wiki page consolidates the entire documentation suite into a single operational and architectural reference. It covers:

- End‑to‑end pipeline architecture
- Data lineage
- Job sequence
- Inputs/outputs
- Content models
- Operational guidance