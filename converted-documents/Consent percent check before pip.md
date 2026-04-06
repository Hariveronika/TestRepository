Source tag : https://ascendionhub.sharepoint.com/sites/HPDeltaProject-Data/Shared%20Documents/Forms/AllItems.aspx?viewid=3cd53e91%2D9898%2D433d%2Da3a1%2D66258e9efa32&csf=1&CID=7e61e1e0%2D6cf7%2D4dc1%2Dab95%2Dcd473f470071&FolderCTID=0x012000D7020EF1377CB048877145427D9878D1&id=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage%2FConsent%20percent%20check%20before%20pip%2Epdf&parent=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Consent percent check before pipeline run

We check, whether they have loaded data for current month or not.

| Sr No | Region | Check % |
|-------|--------|---------|
| 1     | US     | 90      |

Drops below a threshold, then do not execute pipeline

Not Present for other region - TO-DO

Action Item:

- Connect with Sijina/BA to get the threshold value
- Prioritization
- Implementation

Source Validation: All the current prechecks on the source data list:

- Check the latest snapshot and verify if created on right date or not
- Check for duplicate records --verify
- Check both consented and non consent data is present or not in Overall Snapshot table country level.TO-DO

To-do: (Requested by Rohit, need more understanding)

Current vs Previous month total billing count coming into Snapshot,

Aggregate by: Country, retailer, program type.

Staples and Office depot ( they follow custom_calender )

List of prechecks stages:

Testing Type: Funnel Testing, to validate % Drop in each table.

Validation done: Trend is checked with Previous 2-3 month run

Bounty:

DF0 to DF1:

  Checks:

DF1 to DF2:

  Checks:

DF2 to DF3:

  Checks:

DF3 to DF4:

  Checks:

RevShare:

DF0 to DF1:

  Checks:

DF1 to DF2:

  Checks:

DF2 to DF3:

  Checks:

DF3 to DF6:

  Checks:

OUTPUT: Once Funnel testing is done, Summery is triggered.

Shashank runs the pipeline, prasath does the Funnel testing and if results are PASS.. Shashank will trigger the Summary jobs

Summery:

Validation is done in DELTA Tables:

Validation: DF6/DF4 --> Check whether match across Total Subscribers , Total Enrollees, Total RevShare , Total Bounty , Total Compensation summaries with respect to each Retailer , Country, Program Type.

Validation: In - we have IPaper Columns, ( IPaper Enrollees and IPaper Amount ) will be taken from DF4 and will be checked with ipaper Summary tables

Script:

After Validation pass, data gets loaded to RedShift table.

| RevShare (DF6) | Bounty (DF4) | Distributor (distributor-DF6) | IPaper | Summery | Description |
|----------------|---------------|-------------------------------|--------|---------|-------------|
| store_summary | store_summary | | | Total Bounty = (Total Bounty + Ipaper Bounty) | Region: Multiple retailers: Store --> Summery is generated based on Store Level to get "Total Bounty Amount" and "Total Enrollees" |
| Serial_summary (Only US Data \| Staples) | Serial_summary (Only US Data \| Staples) | | | Total Bounty = (Total Bounty + Ipaper Bounty) | Summary is generated based on "Printer_Serial_Number" only for US Staples |
| overall_summary | overall_summary | overall_summary | | Total Bounty = (Total Bounty + Ipaper Bounty) | Combine Final table of Bounty and RevShare along with CatchUp to generate the Summary |
| overall_summary_allpp | overall_summary_allpp | overall_summary_allpp | | Total Bounty = (Total Bounty + Ipaper Bounty) | same as overall summary with all Page plans |

INDEX Table Update:

Validation: No Validation Needed.

Retailer which are common in overall_summary and Rules table( ) will be updated.

List of all the validations that needs to be performed:

validation.xlsx