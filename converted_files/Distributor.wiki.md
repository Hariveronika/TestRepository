Source tag : https://ascendionhub.sharepoint.com/sites/HPDeltaProject-Data/Shared%20Documents/Forms/AllItems.aspx?viewid=3cd53e91%2D9898%2D433d%2Da3a1%2D66258e9efa32&csf=1&CID=7e61e1e0%2D6cf7%2D4dc1%2Dab95%2Dcd473f470071&FolderCTID=0x012000D7020EF1377CB048877145427D9878D1&id=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage%2FDistributor%2Ewiki%2Epdf&parent=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Distributor

Distributor architecture.pdf

New Changes:

| Change serial number | Change description | Jira | PR |
|---------------------|-------------------|------|----|
| 1. | Changes made on 28th oct 2024. | | |

1. Adding new parameters table names, feature name and instead of trail run parameter its renamed to refresh type which take inputs as audit, standard and dev.

2. distributor_mix_round_off column is added at df1 level which was not present before.

3. Email notification feature is added after every pipeline line run.

2. Changes in plan page buckets for SMB Toner Mono and SMB Toner Color - 

- 400 is assigned to 200 page plan
- 800,1500,3000 is assigned to 700 page plan

3. Changes made on 30 Jan 2025 - Pay as you print

- 0pp added to same bucket as 10 and 25
- New columns from source: plan_overage_block_price_cents, billing_cycle_charged_overage_blocks
- Added new column to represent 0pp billing plans seperately - billing_cycle_0pp_block
- Added new global rule to filter out all the 0 enrollment plan subscribers
- Master plan price table to have additional column for 0pp and we use it for calculation
- Created total_price_overage_blocks to represent total price of overage blocks subscribers have used
- Modified final_revshare to include 0pp billing_cycle_plan_pages
- Modified amount calculation to adjust conditions of 0pp

DISTRIBUTOR REVSHARE

Pipeline data flow structure -

1) Start -

2) Source Data –

- Data from retailer_compensation.cross_country_compensation and bi_fact.compensation_retailers_monthly_snapshot
- Read data from redshift using service account credentials.
- Cross country table is joined with the source table, and the country_id will be changed only for those subscription_ids in cross country. Based on this country_id_coalesced is created.
- Write data to redshift using mode-overwrite.

3) Define Parameters –

We will pass 6 parameters i.e

- InputDict – Used to mention all the countries and retailers.
- StartDateList – Used for custom catch up.If not mentioned default is 3months.
- EndDateList - Used for custom catch up.If not mentioned default is last date of the reporting month.
- Program_typeList – InstantInk,Instant Toner ,SMB Toner Mono,SMB Toner colour.
- Reporting_month – The month for which pipeline runs.
- MetaData – Used to mention reason of running pipeline.
- FreezeMix – For yes, last months mix will be taken and no mix will be calculated.For No, new mix will be calculated.
- Trialrun – If trail run is full, data in check points will be saved in S3.And if triage, no data will be saved.

- Required libraries should be imported.
- Adding delta storage paths and add configuration parameters for better performance.

4) Data–

Here we divide the source data as data based on enrolled_on_date.The subscribers who come after july 2022 comes under Data.

5) Data pre privacy–

Here we divide the source data as data based on enrolled_on_date.The subscribers who come before july 2022 comes under Data pre privacy.

6) Global Rules flag created –

- Applying global rules on revshare data. Customers who pass these rules will have further rules applied.
- Added another condition: comp_retailers of billing_cycle_free_month_type is null for free_months_deferred global rule

Global rules-

- Payment_successful_not_suspended when comp_retailers of suspended_entire_cycle_flag is False
- Payment_successful_successful_charged when comp_retailers of charge_complete_flag is True
- Billingcyclestatus_non_forgiven when comp_retailers of billing_cycle_status_code is not equal to FORGIVEN
- Billingcyclestatus_non_prepaid when comp_retailers of billing_cycle_status_code is not equal to PEGASUS-SUCCESS
- Flip_enrollment_all when comp_retailers of flip_type is equal Flip or comp_retailers of flip_type is equal Hybrid Flip & comp_retailers of flip_compensation_eligible_flag is True
- Legacy_rule_kitless_with_promocode when comp_retailers of kitless_revshare_eligibility_legacy is True & comp_retailers of flip_type is not equal to Flip & comp_retailers of flip_type not equal to Hybrid Flip
- Kitless_21_days_non_legacy when comp_retailers of kitless_revshare_eligibility_non_legacy is True& comp_retailers of flip_type not equal to Flip & comp_retailers of flip_type not equal to Hybrid Flip
- Kit_all_eligible when comp_retailers of enrollment_type is not equal KITLESS & comp_retailers of flip_type is not equal Flip & comp_retailers of flip_type is not equal to Hybrid Flip
- Free_months_deferred when comp_retailers of billing_cycle_free_month_type is Null or comp_retailers of billing_cycle_free_month_type is equal to full or comp_retailers of Billing_cycle_free_month_type is in RemainingCustomerSatMonth, RemainingPrinterReplacementMonth, RemainingReferAFriendMonth.
- Lups_no_revshare when, inverse of enrolled_on_date < 2020-11-01 & enrollment_plan is equal to 15 & billing_cycle_plan_pages is equal to 15.
- zero_page_plan_enrollments, flag is created to filter out 0 enrollment plan as they are not eligible for compensation.

7) Global_rules_exceptions_combined is created–

Creates global_rules_exceptions_result column and mark it pass if that row satisfies all program specific global rules conditions and expetions.

Here we have Instant Ink, Instant Toner and SMB.

- Instant Ink : If conditions are passed then we print II.For conditions refer code in databricks.
- Instant Toner : If conditions are passed then we print IT.For conditions refer code in databricks.
- SMB : If either of the conditions are passed its SMB.

8) Check point -0 –

This end check point 0, all these data come into df0.

- Source data added.
- Global rules flag added.

9) Distributor Rules columns are added–

Tables- retailer_compensation.new_distributor_history_mapping

retailer_compensation.mapping_new_distributor_rules

retailer_compensation.new_distributor_compensation

- Get only the approved rules for the current reporting_month and get the complete retailer_compensation_rules i.e from retailer_rules = load_retailer_rules_complete()
- Join with index table to get retailer rules only for the reporting_month i.e retailer_rules = index.join(retailer_rules, index.retailer_rules_index == retailer_rules.id)
- Then load_retailer_rules and few transformations are done as per business requirements.
- Adjustments To ensure unconsented logic applies to details and source columns as well
- Privacy and pre -privacy data is sorted and cleaned as required.

Phase 2 Allocation Process Algorithm

- Read current months data : comp_retailers = readDataFromSpectrum(query_all).
- Read pre-July data : comp_retailers_pre_privacy = readDataFromSpectrum(query_pre_privacy).
- Create cohorts:
  - Consented Enrollments: comp_retailers_consented.
  - Non-Consented Enrollments Pre-July 22: comp_retailers_non_consented_pre_jul_22.
  - Non-Consented Enrollments Post-July 22: comp_retailers_non_consented_post_jul_22.
- Perform Pre-July Allocation: Join comp_retailers_non_consented_pre_jul_22_passed_global with mix_combined_pre_jul to obtain comp_retailers_non_consented_pre_jul_allocated_passed_global.
- Perform Post-July Allocation: Join comp_retailers_non_consented_post_jul_22_passed_global with mix_combined_post_jul to obtain comp_retailers_non_consented_post_jul_allocated_passed_global.
- Final Allocation: Combine (Union) comp_retailers_consented_passed_global, comp_retailers_non_consented_pre_jul_allocated_passed_global, and comp_retailers_non_consented_post_jul_allocated_passed_global to obtain the final allocated dataset.
- Filter for consented i.e comp_retailers_consented = comp_retailers.filter of compensation_retailer_name != 'Not Attributed'
- Filter for non-consented i.e comp_retailers_non_consented = comp_retailers.filter of compensation_retailer_name= 'Not Attributed'
- Filter for Passed Global Rules.

10) Distributor_mix calculations Pre-July–

- Calculate subcriber count in each country - country-enrolee_count and calculate subcriber count of retailers for each country - retailer_ subcriber_count_in_county
- Calculate distributor mix by dividing country- subcriber_count with retailer_ subcriber_count_in_county

Logic: If(freezeMix == 'No')

if mix is no,then calculate the mix.If mix is yes, then take the mix from previous run.

11) Distributor_mix calculations Post-July–

- Calculate Mix for Post-July 22 subcribers using comp_retailers_consented_post_jul_22_passed_global, resulting in mix_combined_post_jul.
- Mix for Post July 22 Enrollees :
- Calculate subcriber count in each country - country- subcriber_count and calculate subcriber count of retailers for each country - retailer_ subcriber_count_in_county
- Calculate distributor mix by deviding country-enrolee_count with retailer_ subcriber _count_in_county

Logic: if(freezeMix == 'No'):

if mix is no,then calculate the mix.If mix is yes, then take the mix from previous run.

12) Final Allocation–

- Pre july joined with mix_combined_pre_jul on country_id_coalesced, allocation-comp_retailers_non_consented_pre_jul_22_passed_global.is enroll_fy, plan_pages_proxy, program_type,reporting_month
- Post july allocation-comp_retailers_non_consented_post_jul_22_passed_global is joined with mix_combined_post_jul on country_id_coalesced, relevant_month, enroll_fy, plan_pages_proxy, program_type, reporting_month
- Final allocation- final_eligible_allocated using comp_retailers_consented_passed_global and comp_retailers_non_consented_pre_jul_allocated_passed_global is joined by comp_retailers_non_consented_post_jul_allocated_passed_global
- Now filter for only country-retailer combinations decided and create Temp View first
- query_filter_relevant_eligible_allocated using final_eligible_allocated_sql
- Then data cleaning is done and stored in -stored in comp_retailers_sql

13) Check point-1 –

This end check point 1, all these data come into df1.

Global rules are passed.

14) Join distributor rules with result of allocated data–

1. Joining data post allocation with final retailer rules data on country , allocated_retailer,allocated_details,allocated_source,reporting_plan to get data from rules data table, rule_date_eligibility is generated by checking if enrolled on date is between rule start date and rule end date.

15) Create Rule date eligibility–

Checks if rule date fall between last_charge_attempt/billing_cycle_end_time of subscription_id

16) Plan price implementation–

Plan_price_data – S3://dataos-core-prod-team-iibi-iipa/data_products/master_plan_price/
Tax data- instantink vat_percentage

- Load Data from Source (CR):Include the plan_price_in_cents column while loading data from the source (CR).
- Load Master Plan Pricing Data:Load the master plan pricing table created using the dim plan table bi_dim.iink_plan.
- Join Pricing Data to Handle Cross-Country Pricing:Join the master plan pricing data with the loaded CR data using the following conditions:
  - Match country codes,match billing cycle plan pages.match program type and ensure the join occurs within the valid date range.
- If cross-country pricing is applicable:Check if the country IDs differ.
- And if billing_cycle_plan is not 0, replace plan_price_in_cents with plan_price_amount from the master plan price table. If billing_cycle_plan is 0 then replace plan_overage_block_price_cents with plan_price_amount.
- Join with Tax Rate Data (VAT Percentage):
- Join the overall data with the country tax rate data using join_country as the key.
- Create the join_country column based on allocated_details becacuse Tax table has differnt county codes for Spain(ES) and Spain canary(CI):If allocated_details contains "CANARY", set join_country to "CI". Its not applicable here for distributor.
- Otherwise, set join_country to country_id_coalesced.
- Adjust Plan Price Considering VAT:Calculate the planprice by adjusting plan_price_in_cents and plan_overage_block_price_cents based on the VAT percentage.
- Handle Null Plan Prices:If both planprice and plan_price_in_cents are null, set planprice to 0,otherwise, keep planprice as it is.
- Handle Null Plan Prices 0pp:If both planprice_0pp andplan_overage_block_price_cents are null, set planprice_0pp to 0,otherwise, keep planprice_0pp as it is.
- Mathematical Formula: planprice = round(100×plan_price_in_cents/(100 + vat_percentage), 0)

17) Check point -2–

This end check point 2, all these data come into df2.

- Distributor rules are added.
- Rule date eligibility flag added.
- Plan price columns added

18) Rule date eligibility is True? –

1. Filtering cases only where rule data eligibility is true.

19) Remove duplicaterules and Take latest rule–

Removing duplicates and We filter the most recent rule.

21) Create compensation_type –

1. Now we create Compensation type and categorize data if its Kit revshare, Kitless revshare or Not Applicable. It is decided by the following conditions:

- Kit revshare: When enrollment_type is not kitless and enrollement date falls within kit contract dates.
- Kitless revshare: When enrollment_type is kitless and enrollement date falls within kitless contract dates. If the enrollment date is before November 1, 2016, and falls within the Kit contract dates, the compensation type is Kitless Revshare.
- Not Applicable: If it doesn't fall under both of the cases above.
- We create json rules criteria is created as pass.

22) Create contract_eligibility_criteria –

Table -overall_joined_ruledate_filtered

1. Contract_eligibility_criteria is created as 'Pass' for all compensation_type except 'Not Applicable', for which it is fail in the table 'overall_joined_ruledate_filtered'.

23) Identify legacy customers –

Subscribers who enrolled before November 1,2016.

24) Create revshare_eligibility –

1. Here we decide the revshare eligibility if it is given, not given or eligible but no revshare:

- Eligible but no revshare: When compensation type is Kit revshare and kit revshare is 0 or when compensation type is kitless revhare and kitless revshare is 0 and json rules criteria is pass and contract eligibility criteria is pass.
- Revshare given: json rules criteria is pass and contract eligibility criteria is also pass.
- No Revshare: When both the above conditions are not satisfied.

25) Get frequency of reports –

Table – retailer_compensation.mapping_rules_index

1. Join with mapping_rules_index to get frequency related information.

26) Check point -3–

This end check point 3, all these data come into df3.

- Distributor rule date eligibility is true.
- Revshare eligibility flag is created.
- Contract eligibility criteria.

27) Revshare_eligibility is revshare given? –

1. Filtering for only revshare_eligibility i.e Revshare given.

28) Decimal round off columns created–

total_price_overage_blocks, Final_revshare and Amount columns are created.

- total_price_overage_blocks is created by billing_cycle_charged_overage_blocks * planprice_0pp when billing_cycle_plan_pages = 0 and billing_cycle_charged_overage_blocks is not null
- final revshare is created by (revshare_rate planprice_0pp) / 10000 if planprice)/ 10000 if billing_cycle_plan_pages is not 0 and (revshare_rate billing_cycle_plan_pages is 0.
- We create amount column based on these conditions:
  - When country id is US and billing_cycle_plan_pages is not 0 then (revshare_rate * total_price_overage_blocks)/10000.
  - When country id is US and billing_cycle_plan_pages is 0 then final_revshare.
  - When country id is not equal to US and billing_cycle_plan_pages is 0 then ((revshare_rate total_price_overage_blocks)/10000) retailer_mix_round_off
  - Otherwise, retailer mix round off * final_revshare.
  - Otherwise, retailer mix round off * kit revshare.

29) USD Conversion –

Table – App_BM_Instant_Ink_Ops.base_hist_curr_val_v2

1. We define dollar conversion table:

Table : App_BM_Instant_Ink_Ops.base_hist_curr_val_v2

Logic : We join dollar conversion table to convert the local currency amount to USD.

30) Gross up for US –

Table – bi_fact.printer_consents_country_agg_CR

1. We define the gross up table for revshsare,

Table : bi_fact.printer_consents_country_agg_CR

We take the latest gross up value by partioning according to country_id, program type,reporting_month,order by created_at.

Logic : We create gross up column from amount by dividing the amount with telimetry_option_and_blank_percent and muliply with 100.

This is done because US comes under phase one logic. We gross up the amount with the percentage present in this table which is updated every month.

2. 1st we convert all amount to USD then gross up only for US and convert the amount gross up column to USD.

31) Check point -4 –

This end check point 4, all these data come into df6.

Final revshare given to subscribers.

32) End –