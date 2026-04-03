Source tag : https://ascendionhub.sharepoint.com/sites/HPDeltaProject-Data/Shared%20Documents/Forms/AllItems.aspx?viewid=3cd53e91%2D9898%2D433d%2Da3a1%2D66258e9efa32&csf=1&CID=7e61e1e0%2D6cf7%2D4dc1%2Dab95%2Dcd473f470071&FolderCTID=0x012000D7020EF1377CB048877145427D9878D1&id=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage%2F05%20SNStore%20reporting%20automation%20documentation%2Epdf&parent=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# SN/Store reporting automation documentation

Introduction

The Serial Number Compensation Reporting program has been set up to support key strategic partners who need hardware level compensation tracking. That is who wants to track channel compensation at a hardware serial number level.

This reporting is intended to allow these partners to better allocate revenue from HP to stores and individuals to support their internal bonus and sales programs or required P&L allocation.

To support this effort, HP must comply with Privacy guidelines as set by our Legal & Privacy teams. This process is intent on supporting partner needs through a compliant process.

Partners available

As of January 2025, the partners who are involved in this reporting program and receive reporting include the below.

- Expert (DE)
- Electronic Partner (DE)
- Eures (DE)
- Excellent IT (BE)
- Media Markt (BE)
- Media Markt (NL)
- Quantore (NL)

Print analytics performs a series of steps to ensure channel compensation for this reporting is available at the right date and time as per the flowchart illustrated on page 2.

On occasions, IIPA does receive incorrect data that basically includes issues with data that has been shared by the partner for example – mostly schema mismatches. (refer next set of bullets for what constitutes schema mismatches).

In case IIPA encounters any of the same issues, IIPA will share it back with the partners to correct and reshare.

IIPA has observed on many occasions that serial numbers with more than 10 characters often appear to be the cause of mismatches. In those cases, IIPA always need to look at the first 10 characters of the serial number from left.

Please note that all the validations IIPA does are strictly from a technical perspective and not from a data perspective.

If there are any issues with the rows of data - for example, incorrect serial numbers, incorrect store ID's etc, they are not captured as part of validations. Only for examples such as, the column name "serial number" is not mentioned correctly or not present at all, we highlight it to the partners stating schema mismatch. Also, if the file format is incorrect(not csv or xlsx but some other format), or naming convention is incorrect, we highlight it to the partner.

As part of the step for data validation, only the below are verified.

- If the file is in the correct format.
- Data quality checks such as the number of columns, column names and count of records.
- Any duplicate records

Below is a flowchart that explains the steps involved in the process and the ownership details for every task.

Process & timeliness

If the file doesn't exist as per deadlines, IIPA will send an automated email notification to CO requesting the file.

It is the responsibility of the CO to liaise with the respective CM's to get the files in case of any inconsistencies or delays and Print analytics will not have a role to play in this.

The scope of work for print analytics will begin only after the files have been shared with them by the CO.

CO will notify the IIPA team if the inbound file is uploaded after 20th day of the month, post which IIPA team will run the pipeline to generate the outbound report.

In case the file is not shared by the CO even after an automated reminder email, IIPA will not be checking the S3 folder between the 20th and the date of the job run which is the 26th. In case the file is not shared even by the 26th of the month, the automated job will run without that partner's file.

Only under exceptional cases, can the CO request IIPA for manually generating the missing partner's file and IIPA will fulfil it.

If IIPA encounters any issues during the validation of the data shared by partners, IIPA will send out notifications and the CO's will have to communicate with the partners regarding the same.

After they need to inform the CO and the CO can request IIPA for manually generating the missing partner's file so that IIPA can fulfil it.

Maintaining historical file

Inbound file

As per the naming conventions, the file that the partner provides should have YYYYMM as the shared format.

The name of the file should have the month in which the file is loaded and should not be basis the data inside. Therefore, for the month of June'2025, we should have 202506 as the file name even the data we have in the file is until 202505.

Overall, we expect partners to follow the below naming format for inbound files.

CountryCode-RetailerName-YYYYMM

For example - DE-Expert-202403.csv or DE-Expert-202403.xlsx

Outbound file

Print analytics will create the file with a date and timestamp.

All files to be provided in the dataos-core-prod-iibi-vendor-in is the S3 bucket for inbound files. Inside this bucket, we have folders for all partners, and they upload their files to their respective folders.

Similarly, after generating the outbound file, we place it in their folder in the dataos-core-prod-iibi-vendor-out bucket.

The partners must copy the files from the folder and ensure they don't move it because IIPA does not have any backup for these files.

In case the partners face any issues with accessing the files, all relevant actions including but not limited to creating folder structures, providing access etc needs to be fulfilled by IIDE. IIPA has no role to play in this. CO needs to drive conversations between the partners and IIDE to fulfill the requirements without any involvement or support required from IIPA.

Please reach out to parthiban.manavalan@hp.com and GTM - andrea.ferrer.cucurella@hp.com for any questions or issues with access.