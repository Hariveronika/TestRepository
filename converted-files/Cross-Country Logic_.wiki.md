Source tag : https://ascendionhub.sharepoint.com/sites/HPDeltaProject-Data/Shared%20Documents/Forms/AllItems.aspx?viewid=3cd53e91%2D9898%2D433d%2Da3a1%2D66258e9efa32&csf=1&CID=7e61e1e0%2D6cf7%2D4dc1%2Dab95%2Dcd473f470071&FolderCTID=0x012000D7020EF1377CB048877145427D9878D1&id=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage%2FCross%2DCountry%20Logic%5F%2Ewiki%2Epdf&parent=%2Fsites%2FHPDeltaProject%2DData%2FShared%20Documents%2FHP%20Delta%20Project%20%2D%20Data%2FDelta%20Team%2Fprojects%2FInstant%20Ink%20Retailer%20Compensation%2FKB%2Fwiki%5Fpage

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# Cross-Country Logic

Understanding Cross Country

The main purpose of cross border compensation in our RC system is to attribute subscriptions across borders.

For example, an user purchases printer in Amazon DE but enrolls the printer to instant ink in UK. Should the instant ink compensation then go to Amazon DE or Amazon UK?

For such scenarios where same retailer and multiple countries are involved in the instant ink enrolment process, we attribute compensation using the table below:

| Scenario | Country of Enrolment | Country of origin for Pin/Promo/Purl code (Kit enrolment) | Country pf first printer purchase incase of (Kitless enrolment) | Country where Retailer has contract with HP | Compensation |
|----------|---------------------|-----------------------------------------------------------|------------------------------------------------------------------|---------------------------------------------|-------------|
| 1 | Country A | Country B | | Country A & B | Country A currency is used to pay country A retailer |
| 2 | Country A | Country B | | Country A & B | Country A currency is used to pay country A retailer |
| 3 | Country A | Country B | | Country B | Country B currency is used to pay Country B retailer if enrollee is P1 |
| 4 | Country A | Country B | | Country A | Country A currency is used to pay Country A retailer if enrollee is P1 |
| 5 | Country A | Country B | | Country B | Country B contracted currency is used to pay Country B retailer regardless P1/P2 |

If retailer has contract in both Country A & B, we compensate in the country where the enrollee resides (Country A for above scenarios).

PPP = Pin code, PURL, Promotional code

Logic that runs through RC flow

There are Three Sections to this logic.

Section 1: Cross-Country Enrollment Table

The first section of this logic is an important section that applies varies conditions and transformations to determine cross-country enrollment information. However, some exceptions are followed -

country id should not be equal to the printer retailer country and if the enrollment type is kitless then printer_retailer_country else if the enrollement type is kit then kit_retailer_country.
Enrollment type is identified from the first enrollment date and therefore the condition where replacement index is either null or 1 (B.replacement_index IS NULL OR B.replacement_index = 1) is used.
For Apple, in EMEA region only country IE should get mapped as a cross country and compensated unless if we have rules existing in any of the country of region EMEA.
Retailers in "('MCR','DMI','COMERCIAL DEL SUR DE PAPELERIA SL','General Office Products Iberia','Informatica Megasur','Depau Sistemas','IDIOMUND SL','EXADI BUSINESS SL')" country_id IN ('ES','PT') do not participate in the cross-country logic.

Country ID is the enrolled-on country, and cross country is the customer residing country (country where they have subscribed/purchased).

```sql
SELECT
    subscription_id,
    enrollment_type,
    CASE 
      WHEN country_id <> printer_retailer_country
           AND upper(printer_retailer_country) NOT LIKE 'NOT ATTR%'
           AND enrollment_type = 'Kitless' 
           AND printer_retailer_name IN ('MCR','DMI','COMERCIAL DEL SUR DE PAPELERIA SL','General Office Products Iberia','Informatica Megasur','Depau Sistemas','IDIOMUND SL','EXADI BUSINESS SL')
           AND country_id NOT IN ('ES','PT')
      THEN country_id 
      WHEN country_id <> printer_retailer_country
           AND upper(printer_retailer_country) NOT LIKE 'NOT ATTR%'
           and upper(concat(printer_retailer_name, region_id)) not in ('APPLEEMEA')
           AND enrollment_type = 'Kitless'
      THEN printer_retailer_country 
      WHEN country_id <> kit_retailer_country
           AND upper(kit_retailer_country) NOT LIKE 'NOT ATTR%'
           and upper(concat(kit_retailer_name, region_id)) not in ('APPLEEMEA')
           AND enrollment_type != 'Kitless'
      THEN kit_retailer_country
      WHEN printer_retailer_name IN ('Apple') --All Apple retailer map to country IE as cross country across EMEA region
           AND region_id in ('EMEA')
           AND enrollment_type = 'Kitless'  
      THEN 'IE'
       WHEN kit_retailer_name IN ('Apple') --All Apple retailer map to country IE as cross country across EMEA region
           AND region_id in ('EMEA')
           AND enrollment_type != 'Kitless'
      THEN 'IE'
      ELSE country_id
      
    END AS Cross_country_id,

CASE 
      WHEN country_id <> printer_retailer_country
           AND upper(printer_retailer_country) NOT LIKE 'NOT ATTR%'
           AND enrollment_type = 'Kitless'
           AND printer_retailer_name IN ('MCR','DMI','COMERCIAL DEL SUR DE PAPELERIA SL','General Office Products Iberia','Informatica Megasur','Depau Sistemas','IDIOMUND SL','EXADI BUSINESS SL')
           AND country_id NOT IN ('ES','PT')
      THEN 'No Cross Country'
      WHEN country_id <> printer_retailer_country
           AND upper(printer_retailer_country) NOT LIKE 'NOT ATTR%'
           and upper(concat(printer_retailer_name, region_id)) not in ('APPLEEMEA')
           AND enrollment_type = 'Kitless'
      THEN 'Kitless_cross_country' 
      WHEN country_id <> kit_retailer_country
           AND upper(kit_retailer_country) NOT LIKE 'NOT ATTR%'
           and upper(concat(kit_retailer_name, region_id)) not in ('APPLEEMEA')
           AND enrollment_type != 'Kitless'
      THEN 'Kit_cross_country'
      when enrollment_type = 'Kitless'
      and printer_retailer_name IN ('Apple') --All Apple retailer map to country IE as cross country across EMEA region
           AND region_id in ('EMEA')
      THEN 'Kitless_cross_country'
      when enrollment_type != 'Kitless'
      and Kit_retailer_name IN ('Apple') --All Apple retailer map to country IE as cross country across EMEA region
           AND region_id in ('EMEA')
      THEN 'Kit_cross_country'
      ELSE 'No Cross Country'
    END AS Cross_country_enroll,
    
    enrolled_on_date,
    DATE(TO_CHAR(DATE_TRUNC('month', enrolled_on_date), 'YYYY-MM-DD')) AS Enrolled_month,
    compensation_start_date,
    DATE(TO_CHAR(DATE_TRUNC('month', compensation_start_date), 'YYYY-MM-DD')) AS Compensation_start_month,
    country_id AS Enrollment_country,
    printer_retailer_country,
    kit_retailer_country,
    LOWER(printer_retailer_name) AS Printer_retailer_name,
    LOWER(original_printer_retailer_name) AS Original_printer_retailer_name, 
    LOWER(kit_retailer_name) AS Kit_retailer_name, 
    LOWER(retailer_kit_printer_org_name) AS Retailer_kit_printer_org_name, 
    program_type,
    region_id 
  FROM bi_fact.iink_sub_base_monthly_snapshot b
  WHERE (B.replacement_index IS NULL OR B.replacement_index = 1)
    AND snapshot_date = '20230929'
  ORDER BY enrolled_on_date;
```

Section 2: Master Contract Table

The purpose of introducing this Active Rules Logic Table is not to compensate a retailer with a cross-country map if we already have a contract with the enrolled-on country. Please keep this logic updated as per today's because this is crucial to identify eligible cross-country enrollees.

```sql
drop table master_contract_table;
create temp table  master_contract_table as(

  SELECT
    country,
    LOWER(retailer) AS retailer,
    retailer_id,
    program_type,
    LEAST(contract_start_date_kit, contract_start_date_kitless) AS master_contract_date,
    contract_status,
    status
  FROM (
    SELECT
      rcr.country AS country,
      rcr.retailer AS retailer,
      ri.retailer_id AS retailer_id,
      ri.program_type,
      contract_status,
      status,
      MIN(contract_start_date_kit) AS contract_start_date_kit,
      MIN(contract_start_date_kitless) AS contract_start_date_kitless
    FROM retailer_compensation.mapping_retailer_compensation_rule rcr
    INNER JOIN retailer_compensation.mapping_rules_index ri
    ON ri.retailer_rules_index = rcr.id
    WHERE status = 'Approved'
      AND contract_status = 'Live'
and ri.rules_start_month_year < ri.rules_end_month_year 
and ri.rules_end_month_year >= cast(replace(left(dateadd(month,2,getdate()), 7), '-', '') as int)
    GROUP BY 1, 2, 3, 4, 5, 6
  ) temp
  ORDER BY 1, 2, 3
);
```

Section 3: Cross-Country Compensation Table

In this section, 'Cross_country_enroll' not equal to 'No Cross Country' is filtered out, and finally, one more column, 'Cross_Country_Comp_flag,' is added to the logic to identify which retailers with cross-country IDs are eligible for compensation.

```sql
drop table cross_Country_Compensation;
create temp table cross_Country_Compensation AS 
  (SELECT
    a.*,
    cr.region_code,
    b.master_contract_date,
    b.country AS contract_country,
    b.retailer AS contract_retailer,
    CASE
      WHEN a.cross_country_enroll != 'No Cross Country' AND b.retailer IS NULL
      THEN 'Yes'
      ELSE 'No'
    END AS Cross_Country_Comp_flag
  FROM cross_country_enroll a 
  LEFT JOIN master_contract_table b 
  ON a.enrollment_country = b.country
    AND a.retailer_kit_printer_org_name = b.retailer
    AND a.program_type = b.program_type
  LEFT JOIN app_instant_ink_bi_dim.dim_country_currency cr
  ON a.cross_country_id = cr.country_name_iso_code_alpha2
  WHERE a.cross_country_enroll != 'No Cross Country'
  ORDER BY cross_country_id, retailer_kit_printer_org_name, subscription_id, enrolled_on_date
 );

 drop table final_table;
 create temp table final_table as 
(SELECT
  subscription_id,
  enrollment_type,
  cross_country_id,
  region_code AS cross_country_region,
  cross_country_enroll,
  enrolled_on_date,
  enrolled_month,
  compensation_start_date,
  compensation_start_month,
  enrollment_country,
  printer_retailer_country,
  kit_retailer_country,
  printer_retailer_name,
  original_printer_retailer_name,
  kit_retailer_name,
  retailer_kit_printer_org_name,
  program_type,
  master_contract_date,
  contract_country,
  contract_retailer,
  cross_country_comp_flag
FROM Cross_Country_Compensation);
```