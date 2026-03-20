1.  
2.  
3.  

1.  
2.  
3.  
4.  

Consent percent check before pipeline run
We check, whether they have loaded data for current month or not.

Sr No Region Check %

1 US 90

Drops below a threshold, then do not execute pipeline :

Not Present for other region - TO-DO

Action Item:

Connect with Sijina/BA to get the threshold value.
Prioritization
Implementation

Source Validation: All the current prechecks on the source data list:

Check the latest snapshot and verify if created on right date or not.
Check for duplicate records --verify
Check both consented and non consent data is present or not in Overall Snapshot table
    :  country level.TO-DO

To-do: (Requested by Rohit, need more understanding)

Current vs Previous month total billing count coming into Snapshot,

Aggregate by: Country, retailer, program type.

Staples and Office depot ( they follow " " )custom_calender

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

       Validation:  DF6/DF4 --> Check whether   match across Total Subscribers , Total Enrollees, Total RevShare , Total Bounty , Total Compensation summaries with respect to each Retailer , Country, Program Type.

       Validation: In - we have IPaper Columns, (  and ) will be taken from DF4 and will be checked with ipaper DF4 IPaper Enrollees IPaper Amount Summary tables

        Script:

                https://dataos-prod.cloud.databricks.com/?o=3337297011599809#notebook/1710040515398862/command/1710040515398863

      After Validation pass, data gets loaded to RedShift table.

RevShare (DF6) Bounty (DF4) Distributor (distributor-DF6)

IPaper Summery Description

store_summary   store_summary   Total Bounty = (Total Bounty + Ipaper Bounty)

Region:
     Multiple retailers:
                      : Store --> Summery is generated based on Store Level to get "Total Bounty Amount" and "Total Enrollees"

Serial_summary (Only US Data | Staples)

  Serial_summary (Only US Data | Staples)

  Total Bounty = (Total Bounty + Ipaper Bounty)

Summary is generated based on "Printer_Serial_Number" only for US Staples

overall_summary   overall_summary overall_summary Total Bounty = (Total Bounty + Ipaper Bounty)

Combine Final table of Bounty and RevShare along with CatchUp to generate the Summary

overall_summary_allpp   overall_summary_allpp overall_summary_allpp

Total Bounty = (Total Bounty + Ipaper Bounty)

same as overall summary with all Page plans

INDEX Table Update:

https://dataos-prod.cloud.databricks.com/?o=3337297011599809#notebook/1710040515398862/command/1710040515398863

Validation: No Validation Needed.

     Retailer which are common in overall_summary and Rules table(  ) will be updated.

List of all the validations that needs to be performed:

validation.xlsx

https://hp-my.sharepoint.com/:x:/r/personal/writtik_dey_hp_com/Documents/validation.xlsx?d=wf27e21d76315488681a6f95e0623c799&csf=1&web=1&e=gOJVMJ

Consent percent check before pipeline run