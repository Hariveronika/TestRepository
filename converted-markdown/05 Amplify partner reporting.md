Source tag : https://ascendionhub.sharepoint.com/sites/HPDeltaProject-Data/Shared%20Documents/Forms/AllItems.aspx?viewid=3cd53e91%2D9898%2D433d%2Da3a1%2D66258e9efa32&csf=1&CID=7e61e1e0%2D6cf7%2D4dc1%2Dab95%2Dcd473f470071&FolderCTID=0x012000D7020EF1377CB048877145427D9878D1&id=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage%2F05%20Amplify%20partner%20reporting%2Epdf&parent=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Amplify partner reporting

The Amplify retail program is a reporting initiative where retailers are onboarded as Amplify partners. Backend subscription data for these partners is used to populate metrics on a dedicated PowerBI Amplify KPI dashboard.

The IIPA Team loads revshare subscriber data every month between the 21st and 25th to update the metrics available in the Amplify KPI dashboard.

## Table Descriptions

The key tables used as part of Amplify reporting are as below.

- retailer_compensation.overall_summary: Contains essential information about retailers and subscription-related data.
- gold_css_dim.retailer_masterref: Retrieves the partner_hq_id for a given retailer and helps link retailers to their location_id.
- gold_css_dim.partner_profile: Provides the partner_location_id for each retailer and includes location-specific information.

Detailed steps to generate and validate the report are present at the below location.

Amplify Data Feed Process Document - CSS Business Analytics - HP R&D Wiki

## Output

The final output of the SQL code includes aggregated subscriber data for reporting months, countries, retailers, and partner locations. This aggregated data is referred to as Amplify data.

## Email Notifications

Email notifications are sent out depending on the success and failure in generating the partner reporting.

### Success Scenario

When Amplify reports are successfully generated, an email notification is sent as follows:

Subject: Amplify Data for Reporting Month: XXXXXXXX
Sender: pa_instantink@hp8.us

Message:
Hi Team,
Successfully generated Amplify data for reporting month: XXXXXXXX.

Please find the data at the following locations:

SFTP path: s3://dataos-core-prod-iibi-sftp-internal/hpsb-prod/amplify/HPI_IIB_RD_Amplify_InstantInk_Subscribers.csv
IIPA S3 path: s3://dataos-core-prod-team-iibi-iipa/data_products/Amplify_data/Amplify-May-24.csv

### Failure Scenario

In case of failures, an email notification is sent as follows, and the respective business stakeholders can reach out to the IIPA team:

Subject: Amplify Data for Reporting Month: XXXXXXXX
Sender: pa_instantink@hp8.us

Message:
Hi Team,
Could not process Amplify data for reporting month: XXXXXXXX due to data issues.

Please expect a delay.
Kindly reach out to XXXXXXXXX if you have any questions.

## Key Stakeholders and roles and responsibilities are as below

The following stakeholders and R&R are part of this reporting process:

- The overall summary table is maintained by instant ink print analytics.
- Other tables are maintained by IIDE team - Vinod P V(vijayan@hp.com)
- Any logic related issues need to be looked into by IIDE team.
- The reports are consumed by the below contacts:
  - brossa.dachs@hp.com
  - falgueras@hp.com
  - alier@hp.com