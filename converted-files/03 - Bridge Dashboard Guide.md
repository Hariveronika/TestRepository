Source tag : Hp Delta - Instant Ink - Folder 1 ​MANDATORY TOOL USAGE:
You MUST call the DirectoryRead and FileReadTool with the user's question
DO NOT attempt to answer without calling the tool
DO NOT generate synthetic or assumed information
Tool calling is REQUIRED - no exceptions./n  - 03 - Bridge Dashboard Guide.pdf

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# 03 - Bridge Dashboard Guide

The bridge dashboard helps to visualize the data transformations that happen across the Retailer Compensation pipeline.

Currently, the dashboard contains the reporting month data, without catchup, from Jan'22.

FAQs

What is the difference between Active Enrollments and Total Billings?

Active Enrollments is the cumulative sum of all the enrollments, considering the cancellations, using the adoption fields (Adoption country, adoption retailer kit printer). Adoption fields are widely being used by country managers. Adoption fields in the source refer to the data as per the latest printer. However, the retailer compensation gets calculated on the first printer data.

Total Billings is the sum of all the billings generated in the billing source table for that month. There will usually be a difference between these 2 fields because:

- The fields under consideration are different (eg. Latest printer data vs original printer data).
- Billings could be generated for a particular month even though there were cancellations.
- Previous months billings coming along with current billing month, because of multiple payments by customers.

The difference between these 2 fields should relatively be consistent across different months.

Can the order of the global rule failures change? (partial enrollments - freemium 15pp LUPS - payment suspended - free months- prepaid - payment successful).

No - Because of the hierarchy of the global rules applied on the billings. No billing ID exists between 2 different global rules failure criteria.

Why is the drop from Total Billings to Kit/ Kitless Eligibility higher than the drop due to other global rule failures?

That's because most of the ineligible partial enrollment billings get filtered out here. For an example, revshare will be paid to only P1 kit & Kitless enrollments and P2 kit enrollments, Only Promo code enrollments prior to Nov 2016 are elligible for revshare, Only eligible Flip subscribers who are full enrollments receive revshare. Note that these may include payment unsuccessful billings too, but as this failure criteria is above the hierarchy of Global failures, most of the billings fall into this category.

Why is there not much difference between Prepaid Eligibility and Payment Eligibility?

This difference can be considered as a parameter to validate every month to understand if there is an irregularity in the payment successful billings for the month.

Why is there an increase from Payment Eligibility to Privacy Grossup?

This increase can only be identified when the Retailer filter is applied in the dashboard. Until payment eligibility, with the Retailer filter, the data is consented data only. Privacy grossup includes both Consented and Non-Consented data, mapped to the retailer post the phase 2 allocation process. When the same data is visualized without the retailer filter, I and K will be the same, because we will be viewing the country level data, where until J, both Consented and Non-Consented data will be visible.

What is Privacy Grossup?

Due to the privacy changes, it's a customer's wish to provide consent to use his data for any internal processes in the company. Hence, when the customer doesn't give a consent, we wont be able to identify the retailer/ country that the customer belongs to. This data is very much essential for calculating the compensation to a particular retailer. This change is in effect since May'2023. To tackle this issue and to provide a fair compensation to the retailers, we have an allocation process in place where every billing of a non-consented customer will be divided across the country based on the retailer mix. Hence when the data is viewed from a country level, without a retailer filter, Global Rule Pass and Privacy Grossup will be equal.

Why is there a drop from Privacy Grossup to Contract Eligibility when the data is viewed from country level without retailer filter?

Privacy Grossup includes all data - Consented and Non-Consented, Contracted and Non-Contracted. But Retailer-Contract includes only Contracted data. The drop from Privacy Grossup to Retailer-Contract when viewed by Country level- explains the number of partners who do not have an active contract with Hp for the retailer compensation.

What is the difference between (Retailer Contract Eligibility) and (Revenue Share Eligible)?

Retailer-Contract majorly tells us the number of billings who are associated with contracted live partners and Rev Share Eligible majorly filters the billings based on the contract dates of the partner. In other words, a billing will be Revshare Given only when the enrolled-on date of the subscriber is between the contract start date and contract end dates of the retailer.

ROAD MAP Of Changes

The above slider gives us an summarized view of the logic changes which took place.

Phase 1 logic did not have any allocation logic in place which compensates non-consented data, hence you would not see an increase in Privacy grossup when viewing Pre May 2023 data for any retailer. The same goes for Phase 0 times. Pre Jan 2023

Eligible Billings Tab

Filter Country = GB, Retailer = Dixons, Program Type = Instant Ink
This tab is used to observe the trend of eligible billings after passing through each Rule Category (Kit/Kitless, Lups Freemium, Suspended accounts, freemonths, prepaid, payment eligibility)
Privacy Grossup and Retailer || Contract trend is also present here.
In the right section of graph there is the trend line of Total Revenue Paid $ vs Total Revshare eligible subscribers.
These trend lines help in analyzing the data and gather quicker insights.
eg. There was Instant Ink price change which happened in the month of Feb, hence we see a huge spike in Revshare Amount Paid $.

Ineligible Billings Trend

Fliter Country = GB, Retailer = Dixons, Program Type = Instant Ink

This tab gives you the count of billings which failed in each rule category (Kit/Kitless, Lups Freemium, Suspended accounts, freemonths, prepaid, payment eligibility)
The chart is spread out month wise, so it is helpful in gaining insights looking and understanding the trend on where the most of the billings are getting failed.

Eligibility Split Tab

Fliter Country = GB, Retailer = Dixons, Program Type = Instant Ink

This tab is almost same as the funnel view but it is in a bar chart form.
Each bar represents the number of billings after passing through the subsequent Global Rule Category logic checks. (Kit/Kitless, Lups Freemium, Suspended accounts, freemonths, prepaid, payment eligibility)
The Red Bar represent the failure count of billings of that Global Rule Category.
The Blue Bar tells us the pass count of the billings of that Global Rule Category.