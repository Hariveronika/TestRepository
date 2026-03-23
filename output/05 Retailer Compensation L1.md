# 02 - Retailer Compensation L1 Approval Process

Background

Report L1 approval process is a routine activity to ensure the compensation data is reflecting correctly on the reports for various retailer country combinations. To perform L1 approval, below are the steps we need to take.

Report Approval Process

Below are the key processes, and the teams who are responsible for in the context of the overall report approval process. For the purpose of this document, we will focus on the L1 report approval process (step 4) that falls within the roles and responsibilities of print analytics team.

Step 1 Freeze all rules for the reporting month CO and PA

Step 2 Snapshot Generated for Compensation Retailer Table IIDE

Step 3 Standard Refresh Pipeline and Summary Table Refresh PA

Step 4 UI Report Refresh IIBI

Step 4 L1 Approval PA

Step 5 L2 Approval GTM

Step 6 Payment Processing Sales Ops

Timelines for L1 approval activities

Here is the detailed timeline for the activities owned by PA
L1 Approval and database refresh Schedule is shared with stakeholders by the 7 calendar day

Standard refresh snapshot is created on the 7 Calendar Day of the month

Standard Refresh Pipeline runs on the 8 Calendar Day of the month (3 am IST)

The scheduled refresh can be postponed under circumstances such as urgent rule modifications, issues in snapshot data etc. In case of delays the stakeholders are informed of the updated schedule
Staples Report need to be approved within the 10 working day of the month

Priority Partner Reports need to be approved within the 12 working day of the month

Below are some additional details related to the L1 approval process to consider.

The list of priority partners will be updated by CO as and when required and communicated to print analytics team
All other partner reports need to be approved one day before the last Friday of the month for timely payment processing
L1 approved reports are communicated through automated mails to the stakeholders and to the GTM team for L2 approvals. The report in the email summary also contains the L1 approval comment which is checked by the country manager for L2 approval.
Payment is processed on every Friday in the PST hours

Below is a list of all sequential steps analysts need to check to certify L1 approval as complete.

Partner data generation

Analysts need to check if all the partner reports have been generated and to identify the reason for any issues. Below are the sub-steps involved in this check.

Ensure that data for 100% of active partners is generated. Cross-verify this information using the WW sheet and the UI portal.
Below is a snapshot from the WW sheet.

As per the WW master sheet, the portal should display seven monthly reports. For the November reporting month, only monthly reports will be available, with no quarterly reports. Refer to the WW master sheet and UI portal for a clearer understanding.

When there is a name change/ partner merge, ensure the new name is reflecting across all the months of the reporting timeline.

Audit report checks

The Audit Report, produced monthly by the Print Analytics (PA) team, is a key deliverable. This report is essential for the Finance team to estimate revenue compensation for the upcoming month and for us to verify data flow at both the country and retailer levels.
We run the audit pipeline 1-2 days before the end of the month. As a result, the audit report may not reflect the most current data, as some enrolments and their billing information might be missed during this period.
As a pre check before standard refresh, we typically review the data at the country level in the audit report using the Triage dashboard.
Also, we need to ensure there are no significant changes in the data flow for the upcoming reporting month. If any major changes are detected, we will address them before the standard refresh date.

Data mismatch checks

Analysts to check for any data mismatch between the summaries and the backend. The process to do so is as below.

Go to UI portal: Retailer
Click on Reporting and select your country.

By comparing the data from the summary table with the UI portal, we can identify any discrepancies. This allows us to validate the data and, if inconsistencies are found, we can proceed with further analysis.

Ensure 100% match of partner report on UI across the summaries. (Page plan summary/ store summary, serial number summary).
Begin by reviewing the overall summary for Instant Ink for the latest month. Check the following details:

- Total bounty
- Percentage change in bounty
- Total revenue share
- Percentage change in revenue share

Cross-check this data with the Month Summary Report, Revenue Share Summary Report, and Bounty Summary Report.

Repeat the steps for toner as well by reviewing the overall summary for Instant Ink for the latest month. Check the following details:
- Total bounty
- Percentage change in bounty
- Total revenue share
- Percentage change in revenue share
Cross-check this data with the Month Summary Report, Revenue Share Summary Report, and Bounty Summary Report.

Data Lineage tracing

Analysts to validate the data flow across different tables in the backend making sure there are no potential data loss.

Check total billings trend over last 4 months.
Navigate to the "Funnel View" in the Bridge Dashboard and adjust the reporting month to analyze billing trends across different months. This allows you to track the monthly billing generation (MoM) and observe the pass percentage for each global rule.

Check drops off in billings for each global rule and investigate any abnormalities in the trends.
In this section, we will investigate the reasons for drop-offs. For example, if we observe significant changes in the number of suspended accounts, free months, or unsuccessful payments, we need to conduct a deeper analysis to identify the causes of these changes or flag them for further review. Same investigation needs to do for other program types.

We then move to the "ineligible billings trend" tab. In this section, we will monitor changes in ineligible billings. For example, if we notice significant variations in the number of ineligible payments, free months, or suspended accounts, we need to conduct a thorough analysis to identify the underlying causes or flag them for further investigation.

Navigate to "eligible billing trends". In this tab, the line graph allows us to easily track trends in paying subscribers and revenue share. By analysing the fluctuations and patterns, we can gain valuable insights into these metrics over time.

Data Validation

This refers to the validation of the final summary data.

Navigate to Raw Data in RC QA dashboard.
MOM change of 5% for the paying subscribers and revshare amount is within the acceptable range.

As observed, the acceptable range for change is 5%, but we are seeing a change of -6.17% change in paying subscribers and -6.15% change in Revshare w/o catchup. This necessitates further analysis to identify the reasons behind this deviation. It could be due to seasonality or other factors that need to be understood through detailed analysis. The same analysis should be conducted for other program types as well.

For detailed understanding of  Revshare, navigate to the Rev Share tab and examine metrics such as Revshare amount by plan mix and Revshare per subscriber. By analyzing these numbers, we can identify any changes in the Revshare amount.
For example, we can observe that the Revshare per subscriber decreased from 1.71 in October to 1.70 in November. Additionally, when examining the Revshare amount by plan mix, it is evident that the numbers have declined in November across all plan pages except for 25PP and 700PP. This indicates a need for further analysis to understand the reasons behind these changes.

MOM change of +/- 10% for bounty enrollees/ amount is acceptable.
In the snapshot below, we observe a change exceeding -10%. To understand this better, we should compare the bounty enrollment data from the previous month with the current month.

For a detailed understanding of bounty changes, navigate to the Bounty tab and review the bounty enrollment and amount, as well as the Kit/Kitless enrollee mix. This will enable us to analyze the changes effectively.

YOY change should always be positive for revshare amount and subscribers.
For detailed understanding of YoY % change subscribers, navigate to raw data tab and look for YoY% change subscribers.

For detailed understanding of YoY % change Revshare amount, navigate to raw data tab and look for YoY% change Revshare amount.

Catchup change should always be within 3% unless custom catchup of any sort is performed. We can check the detailed information about catchup data in catchup overall trend tab. We can see it’s as per the trend.

Navigate to plan mix threshold in RC QA dashboard and thoroughly checked for any anomalies.

Check trend of cross-country subscribers

Once all the checks are complete, the report is marked as L1 approved with appropriate comments. Below is a snapshot of the final approved report.

Rules Check

This is done through automated scripts that is run before the scheduled Audit Refresh every reporting month. The script keeps getting updated as and when new checks are identified and added through code changes.

This ends the L1 approval process. The overall approval of reports gets tracked every month through Jira tickets that are raised and tracked as part of every sprint. Each of the tickets are mapped to various analysts who review and update the ticket in case of any deviations.