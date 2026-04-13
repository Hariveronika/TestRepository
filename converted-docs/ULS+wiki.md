Source tag : https://rndwiki.inc.hpicorp.net/confluence/spaces/CSSBI/pages/1770397393/ULS+wiki

Note : This extracted file is generated from SharePoint or Wiki content. If the source document contains images, charts, or graphical elements, they may not be fully converted into Markdown format. For complete clarity, please refer to the original source using the source tag provided above.

# User Listing Service (ULS) – Architecture Evolution and Reference Guide

User Listing Service (ULS) is the system responsible for aggregating HP's customer account data and consent preferences for marketing communications. This document provides a comprehensive reference of ULS's architecture: from its legacy design to the re-architected solution in multiple phases (Phase 1–5). It details how ULS evolved, the integration points with systems like HPID (HP Identity), Alpaca (marketing platform), and DataOS (analytics data lake), the data pipelines and tables involved, known issues and data quality gaps, and the actions taken to address them. Core information is presented first, followed by in-depth technical details for each phase and component.

## ULS Legacy vs. Re-Architecture

Legacy ULS (pre-2026) pulled new account data and email consent directly from HPID into a Redshift data warehouse table, but had siloed data and missed profile updates. The re-architected ULS (2025–26) uses existing telemetry and consent feeds to unify all customer profiles and consents into one source, eliminating gaps and ensuring compliance.

## Phased Implementation

The ULS overhaul was delivered in 5 phases: Phase 1 unified opt-out data sources; Phase 2 brought in new accounts and initial consent logic; Phase 3 handled profile updates (especially email changes); Phase 4 cleaned and merged legacy vs. new data (one-time); Phase 5 deployed the daily sync and Alpaca updates. This phased approach ensured a smooth transition without disrupting ongoing marketing operations.

## Single Source of Truth for Consents

All customer email marketing consents are consolidated in a single Consent Sync table after re-architecture. This table merges legacy ULS data with current telemetry, giving each customer one canonical record with up-to-date profile info and an accurate "opt-in/opt-out/unknown" status. Downstream campaign tools now query this unified source, preventing duplicate or contradictory data.

## Privacy Compliance by Design

The new ULS workflow closes critical compliance gaps. For example, if a user updates their email address or unsubscribes via any channel, ULS will update their consent status and notify Alpaca within days. The system marks users who didn't explicitly opt in as "unknown" (treated as no-consent) and honors all sources of opt-outs, ensuring no marketing messages go to users without permission.

## Historical Overview: Legacy ULS Architecture and Challenges

Original Purpose and Design: ULS was originally created to funnel new HPID account registrations and their marketing email consent choices into HP's marketing databases in near-real-time. Under the legacy design, when a customer created an HP account:

- The HPID service (user identity system) would capture their profile (name, country, language, email) and whether they checked the "marketing emails" opt-in box. ULS acted as an intermediary service that pulled this data.
- ULS would drop the customer's data into a data warehouse (Redshift). A table (often referred to as the "Dim Customer" or Dim Vertica table) in Redshift stored the new user's profile along with the initial email consent flag (opted-in or not). This typically occurred within ~60 minutes of account creation, ensuring marketing systems knew about new opt-ins quickly.
- In parallel, HP's enterprise marketing platform Alpaca would also receive the opt-in event from HPID. (HPID forwards the consent to Alpaca at account creation as a courtesy, though not managing updates later.) However, this Alpaca feed could be delayed by up to several days in some cases.

Integration with Other Systems: The legacy ULS pipeline was tightly coupled with multiple systems:

- HPID – provided the source data via an API. HPID was authoritative for account data and initial consent capture, but did not handle post-signup consent changes (e.g. if a user later changed their email or marketing preferences).
- Enterprise Data Warehouse – stored ULS outputs (e.g. the Dim Vertica customer table in Redshift) as the official daily record of new accounts and their first consent status.
- Alpaca (Marketing Database) – stored the master list of opted-in contacts. HPID's initial opt-in was sent to Alpaca, but Alpaca would not get updates if a user's email changed or if they later opted out via HP's systems (that gap became a major issue).
- DataOS – HP's analytics lake (on Azure) receiving telemetry events. Notably, a "User Account" telemetry event stream was introduced later to ingest all HPID accounts (for the HP One program) into DataOS. This created an HP One Customers table in DataOS that had all user profiles (including those created outside ULS's lifetime) and ongoing updates, but without reliable email consent info. Essentially, HP had two parallel customer databases: one in Redshift with consent but stale profiles, and one in DataOS with current profiles but missing consent data.

Legacy Challenges: By 2025, several issues with this architecture had become clear:

- Two Sources of Truth: There were two disconnected pipelines for customer data:
  - ULS → Redshift (with email consent and profile at signup) and
  - User Account Telemetry → DataOS (with profile updates for all accounts). This meant no single unified customer view – a user might exist in both systems with different data. For example, the Redshift table captured their opt-in but might have an outdated email or name, whereas DataOS had the latest email but no indication if the user opted into marketing.
- Missed Profile Updates: The legacy HPID updater service that was supposed to update the Redshift customer table for profile changes broke and was never fixed. As a result, if a user changed their personal information (e.g. last name, country) or even their email address after registration, those changes appeared in DataOS (via the telemetry pipeline) but not in the Redshift ULS table. The Redshift data became increasingly stale.
- Consent Synchronization Gap: HPID would not manage ongoing consent status beyond the initial sign-up. If a user created an account and later updated their preferences or changed their email, HPID did not send any update to Alpaca or ULS for those changes. For example, if a user changed their email address on their HP account, HPID left the original email's opt-in in Alpaca as-is (tied to the old email) and did not inform Alpaca of the new email at all. Approximately 50,000 email changes were happening per week, creating "orphaned" consents and new emails without consent status in Alpaca. This was a serious compliance gap – other systems might continue emailing a user's old address or not know the user opted out on the new address.
- Delayed Opt-In Data: In the legacy flow, Alpaca's knowledge of a new opt-in could lag. ULS got new accounts within an hour, but Alpaca Mirror (DataOS copy of Alpaca data) only updated once daily and sometimes HPID took 3–4 days to forward the opt-in to Alpaca. ULS had no visibility into a confirmed opt-in until it appeared in Alpaca Mirror up to a day (or several) later, which complicated deciding whether a "no opt-in seen" meant a true opt-out or just a delay.
- Multiple Opt-Out Sources Untracked: Customers could opt out of marketing through various channels (HP Instant Ink portals, HP Smart app, customer support "do not contact" list, etc.), but in the legacy setup those opt-outs were not centrally compiled. The Redshift table only reflected the initial signup choice. Several independent systems were collecting unsubscribe requests, and there was a risk that a user who opted out via one product might still receive emails if that opt-out wasn't applied everywhere. This fragmentation needed addressing to honor all suppression requests across the company.

By late 2025, it was clear that the ULS service on Redshift would be deprecated (planned shutdown by early 2026) and that a new solution was needed. The goal was to preserve the functionality of ULS (aggregating user data and consents) using modern pipelines and to resolve the above gaps. The re-architecture effort focused on leveraging the existing DataOS telemetry pipeline and Alpaca data feeds, rather than building another monolithic service, and on unifying the multiple data sources into one robust process.

## Re-Architecture Strategy (2025–2026)

Approach: The ULS re-architecture was implemented in five structured phases. Each phase addressed a specific set of problems in a logical sequence, allowing incremental improvements without interrupting ongoing operations. The overarching strategy was to combine the strengths of the two pipelines (the DataOS "HP One Customers" feed and the Alpaca consent feed) into one and eliminate the legacy dependencies. The new design also introduced a Consent Sync process to reconcile and update consents continuously. Below is a timeline of these phases and their objectives:

### Phase 1 (Late 2025): Opt-Out Data Consolidation

Goal: Create a unified view of all marketing opt-outs. Actions: Combined multiple unsubscribe/opt-out lists (Instant Ink, HP Smart CRM, "Do Not Contact" lists, etc.) into one Master Opt-Out dataset. Cleaned up legacy opt-out handling code for efficiency. This ensured that any known opt-out for a customer (whether identified by email or account ID) is centrally recorded and can be applied to their profile.

### Phase 2 (Dec 2025): New Account Onboarding Logic

Goal: Replace ULS's function for new registrations using DataOS and Alpaca feeds. Actions: Built a job to ingest new customer profiles from the User Account telemetry (DataOS) and determine their initial email consent by checking for an Alpaca opt-in record up to 7 days post-signup. New profiles are written to the Consent Sync table as soon as they appear, marked "Opt-In" if a consent is found (with timestamp) or "Unknown" if not.

### Phase 3 (Jan 2026): Profile Update Synchronization

Goal: Ensure ongoing changes to customer data propagate to marketing consent records. Actions: Developed logic to handle updates from the HP One Customers feed (e.g. name changes, country, or email updates). When a user's email changes, the system updates their record and triggers appropriate consent changes: e.g. send an opt-out for the old email and carry over the user's opt-in status to the new email if applicable. All profile changes are merged into the Consent Sync table daily so it stays current.

### Phase 4 (Jan 2026): Data Reconciliation & Cleanup

Goal: Merge legacy and modern data into one clean dataset. Actions: Performed a one-time rebuild of the consent database by reconciling the legacy ULS (Redshift) records with the new HP One Customers data. Mapped old 64-bit customer IDs to new 32-bit IDs where needed, eliminated invalid or duplicate records, and aggregated each customer's historical emails and consent statuses into a single record. Also pulled the latest official profile info from HPID for accuracy. The result was a clean consolidated Consent Sync table (~200M records) ready for use going forward.

### Phase 5 (Feb 2026): Full Deployment and API Integration

Goal: Put the new ULS pipeline into daily production and integrate with Alpaca. Actions: Activated the combined daily job (Phases 2+3 logic using the cleaned data). The Consent Sync job now runs every day: ingesting new accounts, updating profiles, and applying any new opt-outs from the Master Opt-Out list. It also interfaces with Alpaca via API – for example, sending a notification if a user's status flips from opt-in to opt-out (or vice versa). At this point the legacy ULS warehouse table can be retired, and all consumers (campaign platforms, etc.) rely on the new Consent Sync data.

Each phase built upon the previous ones. By the end of Phase 5, ULS's functionality was fully replicated and improved: all customer records (past and present) were unified in one system, and every relevant consent change (opt-ins at signup, opt-outs from any source, email/address changes) would be captured and communicated to the marketing database. Below, we dive into the technical details of each phase and the resulting architecture components.

## Phase 1: Consolidating Opt-Out Sources (Master Opt-Out Table)

Problem Addressed: In the legacy setup, multiple teams and tools maintained separate lists of customers who opted out of communications. These included Instant Ink subscribers who unchecked marketing, HP Smart app users opting out, global "do not email" suppressions, etc. Without consolidation, a customer could slip through (e.g. opt out in one system but still be emailed by another). Phase 1 aimed to eliminate this risk by aggregating all these inputs.

Implementation: Phase 1 introduced a Master Opt-Out Table, which is essentially a union of all relevant opt-out and unsubscribe sources:

- It ingests email-based opt-outs (where an email address is the identifier) such as unsubscribe lists from marketing campaigns or the Instant Ink portal.
- It also ingests customer-ID–based opt-outs, such as flags from HP's CRM systems (Salesforce) tied to the user's HPID or another internal ID. (For example, some internal systems might mark a user account ID as "do not contact".)
- These various feeds are normalized and combined. In cases of overlap, the result ensures that if either a user's email or account ID is present on any opt-out list, the Master Opt-Out table records that user as opted-out.

The Master Opt-Out table stores, for each unique user (by customer ID and/or email):

- an indicator of "Newsletter/Marketing Opt-Out = Yes", and
- possibly metadata like the source or date of the latest opt-out (to help in conflict resolution). All opt-outs are essentially treated the same in terms of outcome (do not send marketing emails).

By Phase 1 completion, ULS's processes could reference one authoritative source to check if a user should never receive marketing emails. This table is used in Phase 2–5 to automatically suppress sending to those users. It replaced what was previously "a combination of two jobs" in the old system with "one unified job" aggregating all opt-out sources. The consolidation not only improved accuracy but also simplified code – the team removed redundant logic and scripts that had individually handled these sources, reducing complexity and maintenance overhead.

Example: How Master Opt-Out is Applied Suppose user Alice is in HP's database. She initially opted in at signup. Later, she unsubscribes via an Instant Ink email link (which goes to an Instant Ink-specific list). Separately, she also checks "do not contact" in the HP Smart app settings. In the legacy system, Alice's opt-out might be recorded in two places and not propagated everywhere. In the new system, both events feed into the Master Opt-Out table (identified by Alice's email and HPID). Alice will be marked as opted-out in that central table. When the Consent Sync process runs, it finds Alice in Master Opt-Out and ensures her consolidated profile is set to opt-out (Newsletter_Notify = "O"). Any marketing audience queries will now automatically exclude her.

Responsible: Phase 1 was implemented by gonnabathula, mohan (Lead Engineer), as noted in team discussions. By the end of 2025, the Master Opt-Out table was live and being used in testing the downstream phases.

## Phase 2: New Account Ingestion and Initial Consent Determination

Problem Addressed: Replace the legacy ULS new-registration processing with a modern solution that uses DataOS and Alpaca, and ensure we correctly capture the initial marketing consent status of new users. In the old flow, ULS got new user info via HPID push; in the new flow, we have to pull new user data from the User Account telemetry and marry it with Alpaca's consent info.

Data Sources: The Phase 2 job draws on two main sources:

- HP One Customers (User Account Table) – This table (in DataOS) receives a daily batch of events from HPID via Stratus, including new account creations. After Phase 4, all legacy users were also backfilled into this feed, so it truly contains every customer. Each new account event provides the user's core profile: unique customer ID (HPID GUID), name, email, country, language, and account creation timestamp.
- Alpaca Mirror (Consent Feed) – This is a reflection of the Alpaca marketing database, updated daily in DataOS. We query it for evidence of an opt-in consent record for each new user's email address. At account creation, if the user opted in to marketing, HPID sends an event to Alpaca (the primary system); that eventually appears in Alpaca Mirror (usually by the next day).

Logic: Each day after the new accounts are ingested into HP One Customers, the Phase 2 process performs the following for each new customer (one that did not exist in the consent table previously):

- Insert Profile: It creates a new entry in the Consent Sync table with the user's HPID (customer ID) as the key and all their profile fields (name, email, locale, etc.) from HP One Customers. Initially, a placeholder consent status is set (to be determined).
- Consent Lookup Window: For up to 7 days after the account creation, the system checks Alpaca Mirror to see if a marketing opt-in appears for that user's email. This seven-day window was chosen based on analysis of HPID's delays – normally, HPID posts the opt-in to Alpaca within minutes, but in rare cases it took a few days, so 7 days was a safe buffer. (HP's legal requirement is to honor consents within 10 days, so 7 days leaves margin.)
- Set Initial Consent Status:
  - If an opt-in record is found in Alpaca Mirror during that period, the user is marked "Opted-In" (Newsletter_Notify = "I"). The record stores the opt-in consent date (usually the account creation date).
  - If no opt-in is found within 7 days, the user is considered not opted-in. Importantly, instead of immediately labeling them "Opt-Out," the system marks them as "Unknown" (Newsletter_Notify = "U"). "Unknown" means the user did not explicitly consent – effectively treated as an opt-out for marketing purposes, but tracked separately to distinguish from an affirmative opt-out. (Per legal guidance, not ticking the opt-in box is an implicit refusal, not an explicit opt-out.) In practice, Unknown is handled the same as Opt-Out when selecting email audiences – i.e. only explicit Opt-Ins will be emailed.
- Update Consent Sync Entry: The new customer's record in Consent Sync is updated with the determined status (I or U) and the date. If it's "Unknown," we record that as their status going forward until changed (for example, if they later opt in via some other product, which would be a separate event).
- Master Opt-Out check: Although unlikely for a brand new user, we also cross-check the Master Opt-Out table. (It's possible, for example, the user had another HPID in the past or the email was previously on a suppression list.) If the new user's email or ID appears on the opt-out list, we would override their status to Opt-Out ("O") regardless of opt-in – but typically this is not the case for true new accounts.

The net effect is that within a day of sign-up (or at most a week in edge cases), the Consent Sync table reflects either "Yes, user opted in" or "No consent given (treat as opt-out)" for every new customer. During that short window, campaigns simply wouldn't target the new user until the status is resolved. Previously, ULS would get a provisional opt-in within an hour, but if HPID never forwarded it, marketing might have unknowingly continued emailing – now we wait to be sure.

Why 7 Days? The team analyzed historical data and found that while most initial opt-ins show up in Alpaca within 24 hours, in some cases HPID's post or Alpaca's processing lagged. They noticed rare instances of 3–4 day delays. By 7 days, essentially all genuine opt-ins have been received. After 7 days, if no consent is in Alpaca, it's nearly certain the user did not opt in (HP treats that as an implicit opt-out). Thus, 7 days balances being cautious with not delaying too long. (This falls well within the 10-day legal requirement for updating subscriptions.)

Example: User Bob creates an account and does not check the marketing box. On Day 1, Bob's profile is added to Consent Sync as Unknown. We query Alpaca daily – as expected, no opt-in is ever found. After Day 7, Bob remains Unknown (which for practical purposes means he will not be sent emails). User Carol creates an account and opts in. Her profile is added as Unknown initially, but by the next day Alpaca Mirror shows her consent. The job updates Carol to Opt-In with the timestamp (essentially backdating to her signup). Campaigns can now include Carol safely.

Result: By the end of Phase 2, the system could fully onboard new users. The new Consent Sync table entries contain all the fields formerly in ULS (customer ID, email, name, etc.) plus a Newsletter_Notify status that is set accurately. This accomplished ULS's primary task – capturing new users and their consent – using the new architecture. Additionally, because we insert the full profile from HP One Customers, the marketing database now gets fields (like last name, etc.) that were previously in Redshift, with no loss of detail.

Responsible: Phase 2 logic was implemented primarily by iska, mohana vamsi with oversight from Piilani, Benjamin (CW). Testing was done in late 2025 to verify that new opt-ins were picked up within a day and non-opt-ins were correctly marked unknown after a week.

## Phase 3: Syncing Profile Updates and Email Changes

Problem Addressed: Existing customers can change their information or marketing preferences over time. In the old system, such changes were often missed or not reflected in the marketing list (e.g. Redshift didn't get name/email updates, Alpaca didn't know if an email address changed). Phase 3 ensures that any update to a user's profile or consent is applied to the consolidated data and propagated to Alpaca. This is critical for maintaining data quality and honoring opt-outs as users' situations change.

Data Sources: The main source for updates is again the HP One Customers table, which is rebuilt daily from the User Account telemetry:

- This table not only lists new accounts but also includes entries for accounts that were updated or deleted (with appropriate flags or timestamps). For instance, if a user updates their profile, the "user account" event for that day will reflect changes.
- Additionally, Phase 3 logic uses the Master Opt-Out table continuously: each day, it checks if any customer (new or existing) appears on the opt-out list who wasn't previously marked opted-out, and vice versa.
- It also references the Consent Sync table itself for the latest stored state of each customer, to compare and identify changes.

Update Types Handled:

- Email Address Changes: This is the most crucial and complex update. When a user changes the email on their HP account (for example, from old@example.com to new@example.com):
  - Profile Update: The pipeline finds that the HP One Customers feed for that day has a record of the user's new email (and likely the same HPID). In the Consent Sync table, the user's entry is updated to set the primary email to the new address. We also add the old email to a list of "previous emails" associated with that user (to retain history).
  - Consent Carry-Over: We then carry over the user's consent status to the new email appropriately:
    - If the user was opted-in before, we assume they still want to be opted-in on the new email. The system will mark the new email as opted-in in our table and – importantly – inform Alpaca of this change. It does so by calling Alpaca's API to add an opt-in for new@example.com. Simultaneously, it will send an opt-out for the old email (old@example.com), to ensure no emails go to the old address anymore.
    - If the user was opted-out (or unknown) before, we assume they continue to be non-opted-in. The Consent Sync entry remains Opt-Out for the new email. The system notifies Alpaca that new@example.com should be marked opted-out as well. (If Alpaca had a record for the old email as opted-out, that doesn't automatically transfer, so we explicitly update the new address in Alpaca to avoid any confusion).
  - Rationale: As Piilani, Benjamin explained, without this, HPID's gap would cause Alpaca to continue holding the old email's opt-in with no knowledge of the new email, leading to compliance issues and missed marketing opportunities.