Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/CSSBI/pages/1799651922/HP+Personal+Data+Retention+Policy+-+Cascade+-+Work+in+progress

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# HP Personal Data Retention Policy - Cascade - Work in progress

Legal Compass Legal Compass > Legal Policies > HP Personal Data Retention Policy

HP Legal - HP Personal Data Retention Policy

HP Personal Data Retention Policy into a practical retention matrix that's consistent with the policy language and still supports your business use cases (analytics, fraud/abuse, partner audits, disputes).

Below I'll do three things:

Translate the HP policy into "rules you can apply" to your datasetsPoint out where your sample matrix conflicts with the policy and how to resolve it cleanly

Provide a recommended retention matrix template + an example filled-in version for your columns (Highly Sensitive, Unique IDs, Telemetry, Subscription, Survey, Billing, Notes), including triggers, actions, and DSR scope.

## 1) Practical rules derived from HP Retention Policy (the "decision logic")

First split everything into Records vs Non‑Records

From the policy:

- Records containing personal data → retention must follow HP Records Retention Schedule and can be destroyed only through the GRM Record Retention Review process.
- Non‑Records containing personal data → should not be kept longer than 2 years after discontinued use unless there's a valid business need or legal hold.

✅ Key implication:

If you want to keep personal data for 6 / 15 years, you must either:

- (1) justify it as a Record category that maps to an approved retention schedule item, or
- (2) anonymize it after the non‑record window (since anonymized data is out of scope), or
- (3) document a legal hold / contractual requirement.

Apply the policy's special non‑record guidance by data type

The policy gives tighter examples for certain non‑record categories (these are extremely useful for your matrix):

- User behavior data (browser history, sentiment) → delete ASAP within 1 year
- Product usage & device data for cloud services → delete ASAP within 2 years after end of customer relationship, or anonymize
- CRM/customer relationship data → delete ASAP within 2 years of last continual use
- Personal contact data outside HP master DB / transactional DB → delete within 2 years of collection
- Call/chat/screen/video recordings → delete ASAP within 1 year (transcripts up to 12 months after recording destruction)
- Drafts/working papers/reference materials → delete ASAP within 2 years of creation
- Biometric → destroy as soon as purpose complete (or within 1 year after not in use if part of service)
- Customer financial data (credit card) → follow PCI, minimize retention, opt‑in for retention beyond necessity (unless prohibited)

✅ Key implication:

"Telemetry" and "usage" data retained for 15 years in identifiable form will be hard to square with the "usage/device data 2 years after relationship end" guidance unless you anonymize and retain only anonymous aggregates.

Controller vs Processor matters

- HP as Processor: personal data should not be maintained longer than 1 year after end of relationship unless contract/law/legal hold says otherwise.

✅ If any II datasets are processed purely on behalf of another controller, your matrix needs a separate "Processor" retention row.

Documentation requirement is mandatory

The policy requires a record of deletion/destruction kept by the BU/Function, produced upon request (processor customers, audits, regulator requests).

✅ Your matrix should include a column like: "Deletion evidence/log required? Where stored?"

## 2) Where your current example matrix needs adjustment (and how to fix it)

Issue 1 — "15 years for analytics" vs "Non‑record max 2 years"

Your sample says:

- Telemetry / subscription / billing held 15 years for analytics
- Unique identifiers held 15 years to link datasets

That can be compliant only if you do one of these:

Option A (best practice): "Two‑tier retention"

- Keep identifiable raw data for the non‑record window (often ≤2 years, sometimes ≤1 year depending on type)
- Then convert to anonymized/aggregated datasets and retain those longer (6–15 years), since anonymized data is out of scope.

Option B: Classify as "Record" If legal says partner audit requires 15 years, that portion should be treated as a Record and mapped to the Records Retention Schedule category.

Issue 2 — Unique identifiers retained long-term

Keeping SN/SubID/EPC for 15 years as personal data is risky unless:

- it's a Record for audit OR
- you keep it in a separate, access-restricted linkage table with a shorter retention, while long-term analytics uses rotating hashed IDs or aggregated keys.

Common compliant pattern:

- Linkage table (SubID ↔ AnalyticsID) retained 2 years after relationship end
- Long-term analytics uses AnalyticsID that cannot be re-identified (true anonymization) or is not reasonably linkable.

Issue 3 — Billing "15 years"

Billing/invoices/payment records are often Records, not non-records.

So the right answer is usually: "Retain per Records Retention Schedule" (and local statutory requirements), not an ad hoc number—unless your legal team explicitly gave 15 years and it maps to an approved record class.

## 3) A retention matrix that fits HP policy + your business use cases

3.1 Recommended matrix columns (what to include)

To make this audit-proof, I recommend your matrix include:

- Data category (your columns)
- Examples
- Role (Controller / Processor)
- Record vs Non‑Record (and record class if record)
- Primary purpose(s) (analytics, fraud, audit, dispute)
- Retention trigger (creation, last use, relationship end, case closed, contract terminated)
- Retention period (identifiable)
- Post-retention action (delete / anonymize / aggregate)
- Retention period (anonymized) (optional)
- DSR scope (Yes while identifiable; No once anonymized)
- Deletion evidence (log/ticket/system report)

3.2 Example "filled-in" matrix aligned to HP policy (based on your columns)

Note: Where you currently have "15 years for analytics," I'm converting that into (a) identifiable retention within policy windows + (b) anonymized retention for long-term trend analytics. This is the cleanest way to meet both the policy and your business goals.

A) Recommended Retention Policy (Controller use case; II)

| Data type | Examples | Record? | Primary uses | Retention trigger | Identifiable retention (policy-aligned) | Post-retention action | Anonymized retention (optional) | DSR scope |
|-----------|----------|---------|--------------|-------------------|----------------------------------------|----------------------|--------------------------------|----------|
| Highly Sensitive | Name, email, address, phone, payment token | Often Record for disputes/fraud; otherwise Non‑Record | Customer dispute, fraud investigations | Relationship end / case close / last use | 18 months–3 years only for cases; otherwise ≤2 years after discontinued use | Delete from all II tables; retain only record artifacts if applicable | Aggregates only (no re-id) | Yes (while identifiable) |
| Uniquely identifiable | Serial Number, SubID, EPC ID | Record if required for partner audit; otherwise Non‑Record | Linking, fraud, partner audit | Relationship end; audit period end | ≤2 years after relationship end for linkage tables; 15 years only if record-classed for audit | Delete linkage; keep record artifacts per schedule | Use non-reversible anonymized analytics keys | Yes (while identifiable) |
| Telemetry / usage | Printer info, usage, supply type | Usually Non‑Record (usage/device guidance) | Usage & loyalty analytics, fraud | Relationship end | ≤2 years after relationship end | Anonymize & aggregate (or delete) | 6–15 years for macro trends (anonymous only) | Yes until anonymized |
| Subscription info | Plan, changes, promotions, country, enrollment method, partner comp | Mix: transactional records may be Record | Compensation reporting, retention analytics, fraud | Relationship end / last continual use | ≤2 years after last continual use unless record | Delete/anonymize | 6–15 years aggregated trend history | Yes until anonymized |
| Survey (linked) | TY/ad-hoc survey linked to SubID | Non‑Record unless incorporated into official record | Service improvements | Creation/collection | ≤2 years after collection (linked) | Delete linkage; keep anonymized survey results | 6 years anonymous benchmarking | Yes until anonymized |
| Billing | Invoices, payment records | Typically Record | Audit, disputes, statutory | Invoice date / fiscal close | Per Records Retention Schedule (not ad hoc) | Destroy via GRM process | Aggregate spend trends only | DSR depends on record obligations |
| Notes | "CC info not within II system", free text | High risk; should be minimized | Operational troubleshooting | Last use | ≤2 years after discontinued use (prefer much shorter) | Delete; prevent sensitive notes capture | None | Yes (if contains personal data) |

✅ This preserves your business need for long-term trends without keeping identifiable personal data beyond the non-record guidance.

3.3 Mapping your specific use cases to the policy (your rows)

1) II Business Analytics

Policy-friendly implementation:

- Keep identifiable data only within the non-record windows (1–2 years depending on type)
- Keep anonymized aggregates long-term (6–15 years)

Recommended:

- Highly Sensitive: Not needed for analytics (agree with you)
- Unique IDs: keep only short-lived linkage table (≤2 years after relationship end)
- Telemetry/Subscription: ≤2 years identifiable, then anonymize → keep 6–15 years aggregated
- Survey: ≤2 years identifiable, then anonymize → keep 6 years aggregated

2) II Fraud and Abuse

Your "3 years" is generally reasonable if the fraud use case is real and documented.

Recommended structure:

- Retention trigger = case closed (or last suspicious event + fixed window)
- Retain minimum fields needed (avoid extra PII)
- Keep a separate "fraud marker list" with strict access controls
- After 3 years → delete or keep only anonymized signals

3) Channel Partner Audit/Dispute (15 years)

This can work if it is a Record class with legal guidance.

Recommended:

- Treat as Record
- Retain only the specific elements required to evidence partner compensation (e.g., SubID, SN, enrollment/qualification attributes, partner ID, payout evidence)
- Align to the Records Retention Schedule classification and GRM process
- Don't keep telemetry for 15 years unless anonymized

4) Customer Dispute (18 months)

Also reasonable, but align the trigger:

- Trigger = cancellation date or dispute close date (pick one and standardize)
- Ensure it covers billing lag + chargebacks + complaint cycle
- If disputes are Records, route via record retention class.

## 4) What I suggest you change in your "Recommended Retention Policy" row

Your current proposed row:

- 3 / 15 / 15 / 15 / 6 / 15 (years)

Policy-aligned alternative (clean and defendable):

Recommended Retention Policy (two-tier)

- Identifiable layer (personal data):
  - Highly Sensitive: 3 years only for fraud/dispute cases; otherwise ≤2 years after discontinued use
  - Unique IDs: ≤2 years after relationship end (linkage table); 15 years only in record-classed partner audit dataset
  - Telemetry: ≤2 years after relationship end
  - Subscription: ≤2 years after last continual use
  - Survey linked: ≤2 years after collection
  - Billing: per Records Retention Schedule
  - Notes: ≤2 years (prefer shorter + controls)

- Anonymized/aggregated layer (non-personal):
  - Telemetry/subscription/usage trends: 6–15 years (only if truly anonymized and cannot be reidentified)
  - Survey results: 6 years anonymous benchmarking

This lets you keep macro historic business data for 15 years without keeping personal data for 15 years.

## 5) DSR scope recommendations (the row you left blank)

A simple, defensible way to fill "DSR scope":

- If identifiable or reasonably linkable (direct ID or stable pseudonymous ID) → In scope
- If anonymized such that it cannot be reidentified → Out of scope
- Records: still in scope, but deletion may be limited by legal obligations (you may restrict/retain while fulfilling request by other means)

So in your matrix:

- Highly Sensitive / Unique IDs / linked Survey / Billing (identifiable) → DSR: Yes
- Telemetry if linked to SubID/SN → DSR: Yes
- Telemetry aggregated anonymous → DSR: No
- Partner audit record data → DSR: Yes, but subject to record-keeping obligations

## 6) What I need from you to finalize your exact matrix

To tailor this precisely (and avoid misclassifying "Record vs Non-Record"), send me:

- For each column (Highly Sensitive, Unique IDs, Telemetry, etc.), what systems store it? (e.g., Snowflake, Cosmos, logs, CRM, ERP)
- Is the dataset part of a system of record for transactions/audit (i.e., would Finance/Legal rely on it)?
- Your intended retention trigger for each row: creation, last use, relationship end, cancellation date, case close, contract termination
- Whether any data is processed as HP Processor (on behalf of another controller) in II scenarios

Once you share that, I can produce a final matrix in the exact format you want (and we can preserve your use case rows: analytics, fraud/abuse, partner audit, dispute) with language that maps cleanly to HP policy.

Quick question (so I align to your deliverable)

Do you want the output as:

- A single consolidated matrix (data types as columns; use cases as rows, like your sample), or
- A policy-first matrix (one row per data type/use case combination with triggers/actions)?

If you paste your matrix in a more structured way (even just a CSV-like block), I can reformat it into a polished version you can drop into Legal Compass documentation or a design review deck.