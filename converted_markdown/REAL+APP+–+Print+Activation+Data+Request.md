Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/CSSBI/pages/1770630560/REAL+APP+%E2%80%93+Print+Activation+Data+Request

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# REAL APP – Print Activation Data Request

Jira Epic: [DATAPROD-1298] Data Product access for Printer models activated in a country (GA) periodically - HP-Jira

1. Purpose

2. Key Problem

3. What Was Discussed
  - Data Needs
  - Data Volume (Indonesia)
  - Compliance
  - Technical Notes

4. Agreed Path Forward

5. Business Impact
  - If Data Not Provided:
  - If Data Provided:

6. Action Items

## 1. Purpose

REAL APP (SEA Retail Management, Greater Asia) requires printer activation data with serial numbers to validate and reconcile partner sell-out reporting. This supports real-time visibility, audit accuracy, and reliable sell-out capture.

## 2. Key Problem

Manual partner sell-out reporting is incomplete and inaccurate. Existing GA activation dataset lacks serial numbers, which are mandatory for REAL APP's safety‑net validation.

## 3. What Was Discussed

### Data Needs

- Print Activation Data with serial numbers for Greater Asia, starting with Indonesia.
- Data to be delivered into SEA's AWS-based MySQL DB.
- Ben's existing activation table will be enhanced to include missing serial numbers.

### Data Volume (Indonesia)

- Historical records ≈ 33K, but due to privacy/compliance:
  - Only Nov 2025 onwards data will be shared initially.
  - Full historical data + weekly refresh after approvals and data product update.
  - team_cascade_prod.engineering_room.stg_printer_pet_printers where PRINTER_COUNTRY ="Indonesia" ;  --33152

### Compliance

- Privacy review (Terence's team) required before serial-number-level data sharing.
- GA country list needs verification.

### Technical Notes

- SEA platform is on AWS Asia region (not Databricks).
- Initial data transfer likely via SFTP.

## 4. Agreed Path Forward

- Privacy Review – Validate sharing of serial numbers.
- Enhance Data Product – Add serial numbers to Ben's activation table.
- Share Sample Data – Provide Indonesia activation data from Nov 2025 onward.
- Full Rollout – Deliver historical + weekly‑refreshed data post‑prod.
- Confirm GA Country List – Validate mapping using country table.

## 5. Business Impact

### If Data Not Provided:

- Inaccurate partner performance tracking
- Loss of revenue and ROI
- Compliance gaps and audit risk
- Delays to REAL APP launch (1 Nov 2025)

### If Data Provided:

- Accurate sell‑out capture
- Strong compliance and governance
- Real-time PSI visibility
- Improved decision-making and operational efficiency

## 6. Action Items

| Owner | Action | Status |
|-------|--------|--------|
| Sujith | Send meeting notes | Completed |
| Terence's Team | Privacy review for serial numbers | Pending |
| Ben's Team | Add serial numbers to data product | Pending |
| Phani / Sujith | Provide Nov 2025+ Indonesia sample | Upcoming |
| Nes / SEA Retail | Confirm MySQL schema | Pending |
| Data Engineering | Validate GA country list | Pending |