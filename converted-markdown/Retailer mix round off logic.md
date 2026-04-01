Source tag : Sharepoint - Hp Delta - Instant Ink - Folder 1

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Retailer mix round off logic

Retailer mix is the ratio of every non consented billing in each billing cycle that is allocated to every retailer in the country. This ratio is calculated based on the number of enrolees for every country – retailer – page plan mix. When we add up all the ratios for all combinations, the retailer mix must add up to 1. For each consented billings however, the retailer mix will always be 1.

Below is a snapshot table showing the relevant data for each retailer which includes details such as compensation retailer (which is reflecting as "not attributed" which essentially means, "non consented"), allocated retailer, the original plan price in cents, plan price after VAT deduction, retailer mix, revshare rate, and the final calculated amount.

The retailer mix is used to calculate the revshare or final amount using below steps for every row in the above snapshot.

Step 1 - The retailer mix will be rounded off to two decimal places, resulting in a new column named retailer_mix_round_off (refer snapshot above).

Step 2 - The pre plan mix amount is calculated as planprice(post VAT_deduction)*revshare_rate/100*100

Please note that the denominator is calculated by multiplying the converted plan price in cents to dollar and revshare rate to percent. The above pre_plan_mix_amount gets rounded off to two decimal places.

Step 3 - The final_amount is calculated as retailer_mix_round_off × pre_plan_mix_amount which is again rounded to two decimal places. This becomes the final revshare that is paid out to every allocated retailer for each row in the above snapshot.

Step 4 - All these rows of billing ids will finally be aggregated (summation) from a program type, country, retailer, page plan level and will be loaded into the summary tables. For determining the total paying subscribers, the retailer_mix_round_off column gets aggregated and for determining the Total revshare, the final_amount column gets aggregated. This aggregated data is loaded into the summary tables.

Step 5 - The data from the summary tables finally gets loaded into the QA dashboard and into the RC Portal.

Data gets loaded into the QA dashboard using Power BI data refresh.
Data gets loaded into the RC Portal through UI refresh performed by the UI team.

Please find the visualized decimal round off documentation ppt in below ticket where it explains why we introduced round off columns and how it fixed the issue,

IIPA-2690 BA: New Initiative: Creation of detailed documentation for decimal roundoff CLOSED