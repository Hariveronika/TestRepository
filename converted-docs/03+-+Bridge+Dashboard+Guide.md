# Bridge Dashboard Guide

The bridge dashboard helps to visualize the data transformations that happen across the Retailer and Distributor compensation Pipeline.

Currently, the dashboard contains the reporting month data, without catchup, from Jan'22.

ROAD MAP Of Changes

The above slider gives us an summarized view of the logic changes which took place.

Phase 1 logic did not have any allocation logic in place which compensates non-consented data; hence you would not see an increase in Privacy grossup when viewing Pre May 2023 data for any retailer.

The same goes for Phase 0 times. Pre Jan 2023

Overview:

This documentation covers the Retailer and Distributor compensation framework driven by global eligibility rules applied at the billing cycle ID level. Each tab presents a specific analytical view of eligibility and revenue share calculations.

Please Note: The Retailer tab includes a navigation button that redirects to the Distributor section. Similar cross-navigation options are available across relevant tabs to allow seamless movement between Retailer and Distributor views.

Funnel View Tab:

This tab provides a consolidated view of how billing cycles flow through the global eligibility framework used to determine subscriber eligibility for retailer payment.

Each stage in the funnel represents a global eligibility rule applied sequentially to billing cycles.

The progressive reduction across the funnel reflects the drop-off of billing cycles that fail to specific eligibility criteria.

These global rules collectively define whether a subscriber is eligible or ineligible for revshare payout.

The funnel enables quick identification of which eligibility rules contribute most to billing ineligibility.

All calculations are performed at the billing cycle ID level to ensure accurate eligibility tracking.

Catch-up billings are excluded from this analysis to avoid double counting.

The definitions and business logic for each eligibility criterion and global rule shown in this tab are documented in the Appendix tab.

Eligibility Split Tab

This tab provides a comparative view of eligible vs. ineligible billings at each stage of the global eligibility framework.

Each bar represents the total billing population at a specific eligibility checkpoint, split into:

- Eligible Billings (passed the rule)
- Ineligible Billings (failed the rule)

The eligibility checkpoints shown in this tab correspond to the same global rules used in the Funnel View, enabling a complementary analysis.

Unlike the funnel (which shows only remaining billings), this tab explicitly highlights the composition of pass vs. fail at every rule stage.

This view helps quantify the magnitude of ineligibility introduced by each eligibility criterion.

The split enables analysts to assess whether ineligibility is:

- Concentrated at early-stage rules (e.g., Kit/Kitless, Lups Freemium, Suspended accounts, free months, forgiven Accounts, prepaid, legacy zero pp, payment successful)
- Or driven by downstream validations (Privacy, Contract and rule eligibility)

Ineligible Billings Trend

The tab shows how total billings are distributed into eligible and ineligible billings at each eligibility checkpoint, helping identify which rules drive the largest eligibility drop-offs.

It also tracks global ineligible billings as a percentage of total billings across reporting months, enabling normalized trend comparison.

Under normal operating conditions, the ineligible billing percentage is expected to remain relatively stable.

A significant rise or drop in the ineligible billing percentage may indicate:

- A systemic issue in eligibility logic
- A pipeline or processing issue
- A source data quality issue

Any large deviation from the historical trend requires further investigation.

This tab is actively monitored when a new program, rule, or eligibility change is launched.

During new implementations, the trend is used to assess:

- Whether the implementation is behaving as expected
- Whether unintended eligibility leakage or over-filtering has occurred

Eligible Billings Tab

This tab focuses on tracking how eligible billings evolve over time after applying global eligibility rules and how this translates into revenue share compensation.

Billing cycles that pass all global eligibility rules are classified as eligible billings and form the base for revenue share calculation.

The left-side visual shows the monthly trend of eligible billings, further broken down by eligibility rule categories.

This helps understand:

- Overall stability or growth in eligible billings
- How different rule categories contribute to eligible volumes over time

Fluctuations in eligible billing trends may indicate:

- Seasonality effects
- Program mix changes
- Rule or eligibility logic updates

The right-side visual correlates eligible billing volume (paying subscribers) with revenue share (USD) over time.

This comparison helps validate whether:

- Growth or decline in eligible billings is reflected proportionally in revenue
- Any decoupling exists between subscriber eligibility and revenue outcomes

Sudden divergence between eligible billings and revenue share may indicate:

- Pricing changes
- Contract-level adjustments
- Revenue calculation or allocation issues

This tab is particularly useful during:

- New program launches
- Pricing or plan changes
- Contract or compensation model updates

Consistent alignment between eligible billing trends and revenue share trends indicates healthy and correct pipeline behavior.

FAQs

What is the difference between Active Enrollments and Total Billings?

Active Enrollments is the cumulative sum of all the enrollments, considering the cancellations, using the adoption fields (Adoption country, adoption retailer kit printer). Adoption fields are widely being used by country managers. Adoption fields in the source refer to the data as per the latest printer. However, the retailer compensation gets calculated on the first printer data.

Total Billings is the sum of all the billings generated in the billing source table for that month. There will usually be a difference between these 2 fields because:

- The fields under consideration are different (e.g. Latest printer data vs original printer data).
- Billings could be generated for a particular month even though there were cancellations.
- Previous months billings coming along with current billing month, because of multiple payments by customers.

The difference between these 2 fields should relatively be consistent across different months.

Can the order of the global rule failures change? (partial enrollments - freemium 15pp LUPS - payment suspended - free months- prepaid - payment successful).

No - Because of the hierarchy of the global rules applied on the billings. No billing ID exists between 2 different global rules failure criteria.

Why is the drop from Total Billings to Kit/ Kittles Eligibility higher than the drop due to other global rule failures?

That's because most of the ineligible partial enrollment billings get filtered out here.

For an example, revshare will be paid to only P1 kit & Kitless enrollments and P2 kit enrollments, Only Promo code enrollments prior to Nov 2016 are eligible for revshare, only eligible Flip subscribers who are full enrollments receive revshare.

Note that these may include payment unsuccessful billings too, but as this failure criteria is above the hierarchy of Global failures, most of the billings fall into this category.

Why is there not much difference between Prepaid Eligibility and Payment Eligibility?

This difference can be considered as a parameter to validate every month to understand if there is an irregularity in the payment successful billings for the month.

Why is there an increase from Payment Eligibility to Privacy Grossup?

This increase can only be identified when the Retailer filter is applied in the dashboard. Until payment eligibility, with the Retailer filter, the data is consented data only. Privacy grossup includes both Consented and Non-Consented data, mapped to the retailer post the phase 2 allocation process.

When the same data is visualized without the retailer filter, I and K will be the same, because we will be viewing the country level data, where until J, both Consented and Non-Consented data will be visible.

What is Privacy Grossup?

Due to the privacy changes, it's a customer's wish to provide consent to use his data for any internal processes in the company. Hence, when the customer doesn't give a consent, we won't be able to identify the retailer/ country that the customer belongs to. This data is very much essential for calculating the compensation to a particular retailer. This change is in effect since May'2023. To tackle this issue and to provide a fair compensation to the retailers, we have an allocation process in place where every billing of a non-consented customer will be divided across the country based on the retailer mix. Hence when the data is viewed from a country level, without a retailer filter, Global Rule Pass and Privacy Grossup s will be equal.

What does privacy grossup denote for US versus non-US retailers on bridge?

Privacy grossup handles non-consented customer data differently for US and non-US.

For US, we apply consent percentage adjustment directly (Phase 1 approach).

For non-US, non-attributed records are allocated to retailers based on retailer_mix percentage (Phase 2 approach).

Why is there a drop from Privacy Grossup to Contract Eligibility when the data is viewed from country level without retailer filter?

Privacy Grossup includes all data - Consented and Non-Consented, Contracted and Non-Contracted. But Retailer-Contract includes only Contracted data.

The drop from Privacy Grossup to Retailer-Contract when viewed by Country level- explains the number of partners who do not have an active contract with Hp for the retailer compensation.

What is the difference between (Retailer Contract Eligibility) and (Revenue Share Eligible)?

Retailer-Contract majorly tells us the number of billings who are associated with contracted live partners and Rev Share Eligible majorly filters the billings based on the contract dates of the partner. In other words, a billing will be Revshare Given only when the enrolled-on date of the subscriber is between the contract starts date and contract end dates of the retailer.

Data Lineage

Source Data Global Rules Allocation
= 1@ 3 = ee
c= | c= c=
Process Process Process

Table 1 Table 2 Table 3

All

Region

Country ID
All

Generic Retailer

All v

Program Type

Reporting Month

All

Eligible Billings Trend Revenue Share Trend

@Total Bill
14M

12M

10M

@Kit/Kitle... @ Freemiu... @Suspen... @ Free Mo... @ Forgive.

Jul 2024

Month

@ Prepaid ... @Legacy ...

54M

Jan 2024

@ Paying Subscriber @Rev Share USD.

Jul 2025

agtant
nk

Region Country ID Retailer Generic Retailer Program Type Reporting Month
All All v All v All All Vv All v
Eligible Billings @ Ineligible Billings
oabn
03bn
co | | 0.01bn J
| 0.01bn
0.2bn
[0.01 |
o.1bn
0.0bn
Total Billings —Kit/Kitless-Freemium 15pp Suspended _—_—Free Months Forgiven Prepaid Payment Privacy Gross Up Retailer Rev Share
'Accounts 'Accounts Successful Contract Eligible

2 Country ID
v All

Retailer

Generic Retailer

All Vv

All

Program Type

Reporting Month

All

Forgiven Accounts Ineligible

Prepaid ineligible @Legacy Zero PP Ineligible @Payment neigh

@etailer Il Contract ineligible

"3 g B £ EF BF B g§ § BF £ F F F F EF F BF F £ F BF E F EF FE FE EF ZB
2 ¢ ¢ #@ 2 | €  F F 2 8 @ 2 2 2:5 § FE F F F 2 § 2 2 2 EE
2 § = § &§ &€ § = 2 5 € § § = 5 = 2 5 € 8 & =
<3 © 5 8 2 & " $ © 58 g 8 & "$e © g gg 8

2003 20208 26
46.02%
44.94%
AAA6%
os 43.72% ance
45.19% "307% pase 43.38%
44.24% A2A6%
43.11%
ons ons 22.06% 42.25% ay
41979 4191 450%
40%
su 2024 Jan 20

jfgtent
ink .

Distributor Region
Compensation a a

Country ID

|| Program Type v

All Vv ial Vv ya

352.02M

199.62M

152.40M

173.52M

Reporting Month

Total Billings
kit/Kitless Eligiblity

Freemium 15pp Eligiblity
Suspended Accounts Eligiblty
Free Months Eligiblity
Forgiven Accounts Eligiblity
Prepaid Eligiblity

Legacy Zero PP Eligiblty
Payment Eligiblity

Privacy Gross Up

Retailer II Contract,

Rev Share Eligible

100%

49.39%

Oaion

03en

0.2en

o.tn

0.0en

0.35bn

-0.086n
-0.01bn

Total Kitless_Freemi
Billings Ineligible 15pp
Ineligible

—
coin

-0.02bn

0.00bn 0.00 04000n

-0.01bn

Suspen. Free Forgiven Prepaid Legacy
Accounts. Months Accounts Ineligible Zero PP
Ineligible Ineligible Ineligible Ineligible

Payment
Ineligible

0.2060

Total

No concept of te onanwad dat = ope To Proctsslybolouite the retailer mix
non-consented data y
Phase-1 ,
privacy consent ac onwante for each retailer.

Phase 1 Logic
Consent 'Yes' = Yes
Consent 'No'=No
Consent Null = Yes

True subscriber count is calculated
through allocation of non-consented
billings to all the retailers.

Phase 2 logic
Consent 'Yes' = Yes
Consent 'No' =No
Consent Null= No

Final revshare amount is grossed up
for all retailers in accordance with the
country level consent percent.

EMEA/AP) all region consent Percent

Phase 0 countries migratedto (egy eae
Until Jan Phase 2 EMEA = 65.3%
2023 All countries migrated to Except US which carried ——
Phase 1 with Phase 1 logic.
consent Percent
US = 97.76%
EMEA = 65.3%

Roadmap of Changes