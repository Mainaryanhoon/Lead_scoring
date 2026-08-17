Lead Scoring System — Architecture, Production Implementation & Operations

Scope: This document combines the conceptual lead-scoring architecture with the actual Royal Enfield (RE) production implementation by Tatvic.

Important distinction: The conceptual sections describe recommended/general architecture. The Royal Enfield sections document the observed production behavior, including known limitations, bugs, workarounds, and UAT differences. Where the two differ, the Royal Enfield production section is authoritative for RE.

Table of Contents

Overview

What Is Lead Scoring?

Why Lead Scoring Is Needed

Conceptual Architecture

Conceptual Pipeline Layers

Identity Resolution

Feature Engineering

Data Leakage Prevention

Model Inference

Prediction Output

Scheduling and Dependencies

CRM Delivery

Backfills and Incremental Processing

Idempotency, Processing State, and Watermarks

Error Handling

Monitoring and Alerting

Debugging and Data Lineage

Audit Fields and Model Versioning

Training vs. Inference

System Responsibilities

Royal Enfield Production Implementation

Royal Enfield Data Sources and Architecture

Royal Enfield Identity Resolution

Royal Enfield GA4 Constraints

Royal Enfield Feature Engineering

Royal Enfield Model and Threshold Logic

Royal Enfield Output and Delivery

Royal Enfield Leakage Gates

Royal Enfield Known Bugs and Fixes

Royal Enfield Reconciliation Sweeper

Royal Enfield Monitoring and Alerting

Royal Enfield Dashboard and Reporting

Royal Enfield UAT Environment

Royal Enfield Glossary

Key Takeaways

1. Overview

The Lead Scoring System is a machine-learning-driven pipeline that identifies and prioritizes leads according to their likelihood of completing a defined business outcome.

Instead of treating every incoming lead equally, the system combines available customer, CRM, website, and behavioral information to generate a prediction or score representing the likelihood of conversion.

The conceptual lifecycle is:

Lead Sources
    ↓
Raw Lead Data
    ↓
Data Preparation
    ↓
Historical / Behavioral Enrichment
    ↓
Feature Preparation
    ↓
ML Inference
    ↓
Prediction / Lead Score
    ↓
Downstream CRM / Business Systems

The objective is:

Capture leads → enrich them with relevant historical and behavioral information → generate predictions → make scored leads available to downstream business systems.

2. What Is Lead Scoring?

Lead scoring assigns a numerical score, probability, or category to a lead based on the likelihood that the lead will complete a target business action.

For example:

Lead

Predicted Conversion Probability

Priority

Lead A

0.92

High

Lead B

0.71

Medium

Lead C

0.23

Low

A higher score can be used to prioritize sales follow-up or marketing treatment.

The exact meaning of "conversion" depends on the model's training objective and business definition. Possible outcomes include:

Purchase

Booking

Test drive

Form completion

Qualified lead

Sales conversion

Another explicitly defined business outcome

The core ML question is:

Given only the information available at prediction time, what is the probability that this lead will convert?

3. Why Lead Scoring Is Needed

Without systematic lead scoring:

Sales teams may spend time on low-intent leads.

High-intent leads may not receive timely attention.

Prioritization can depend heavily on manual judgment.

Marketing spend may be less efficient.

High-potential customers are harder to identify consistently.

A scoring model uses historical patterns to identify signals associated with the target outcome and helps the business prioritize leads more systematically.

4. Conceptual Architecture

The following architecture is generic and is not a representation of the exact Royal Enfield production implementation.

                    +----------------------+
                    |     Lead Sources     |
                    | CRM / Web / Forms   |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |    Raw Lead Data     |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |  Data Preparation    |
                    | Cleaning / Validation|
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Historical /         |
                    | Behavioral Enrichment|
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |   Feature Dataset    |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |    ML Inference      |
                    |   Trained Model      |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    |  Prediction Output   |
                    | Score / Probability  |
                    +----------+-----------+
                               |
                               v
                    +----------------------+
                    | Downstream CRM /     |
                    | Business Systems     |
                    +----------------------+

A conceptual end-to-end flow is:

Lead Sources
    ↓
Raw Lead Data
    ↓
Base Data
    ↓
Historical Aggregation
    ↓
Stitched / Model-Ready Data
    ↓
ML Model
    ↓
Prediction
    ↓
Scored Output
    ↓
CRM / Business System

5. Conceptual Pipeline Layers

5.1 Raw Lead Data

The raw layer stores incoming lead information.

Possible sources include:

Web CRM

Lead forms

Facebook Instant Lead Forms

Other CRM or campaign sources

Website behavioral data such as GA4

A conceptual lead record may contain:

Lead ID
Name
Phone Number
Email
Pincode
Campaign Information
Lead Source
Timestamp
CRM Attributes
Other Business Attributes

The exact schema depends on the source system.

The raw layer should provide enough information to:

Identify the lead.

Join it with additional data where required.

Generate model features.

Track the resulting prediction.

5.2 Base Data

The base-data layer prepares a reliable dataset for downstream processing.

Typical responsibilities:

Select required columns.

Clean raw data.

Standardize fields.

Apply source filters.

Remove invalid records.

Create consistent identifiers.

Prepare data for joins.

Raw Lead
    ↓
Cleaning
    ↓
Standardization
    ↓
Validation
    ↓
Base Lead Dataset

The base layer should primarily prepare data rather than contain model-specific business logic.

5.3 Historical and Behavioral Enrichment

Current lead information is often insufficient to capture intent.

Historical or behavioral signals may include:

Previous website activity
Previous sessions
Historical engagement
Past interactions
Campaign history
Customer behavior

These signals can be aggregated into model features such as:

Previous Sessions
Previous Website Events
Previous Form Visits
Historical Engagement

5.4 Stitched / Model-Ready Data

The stitched layer combines information from multiple sources:

Lead Data
    +
Historical Data
    +
CRM Data
    +
Behavioral Data
    ↓
Stitched Dataset

A model-ready dataset should contain:

Current lead information.

Historical behavioral information.

Required derived features.

Consistent identifiers.

Model-compatible columns.

6. Identity Resolution

Identity resolution connects records belonging to the same user across systems.

A user may exist across:

GA4
CRM
Lead Forms
Web CRM

Possible identifiers include:

client_id
custom_client_id
ga_session_id

Conceptually:

Website User
    |
    | Identity Identifier
    v
GA4 Behavior
    |
    v
Lead Record
    |
    v
ML Features

Incorrect identity resolution can cause:

Duplicate records.

Missing historical behavior.

Incorrect features.

Incorrect predictions.

Important: The identifiers and join rules used by Royal Enfield are documented separately in the production section because they are more specific than this generic architecture.

7. Feature Engineering

Machine-learning models require structured features rather than raw business records.

Common feature categories include:

7.1 Demographic / Geographic Features

Location
Pincode
Region
City

7.2 Acquisition Features

Campaign
Source
Medium
Channel

7.3 Behavioral Features

Number of sessions
Website engagement
Events performed
Previous visits
Interaction history

7.4 Historical Features

Past user activity
Aggregated interactions
Previous engagement
Historical conversion signals

These features form the input to the trained model.

8. Data Leakage Prevention

Data leakage occurs when the model receives information that would not have been available at the time the prediction was supposed to be made.

Example:

Lead created at 10:00 AM

Feature calculation includes:
Website activity at 3:00 PM

That future activity should not influence the 10:00 AM prediction.

The correct principle is:

Historical features must only use information available before the prediction time.

Past Data
    |
    v
Prediction Time
    |
    X
Future Data
must not be used

9. Model Inference

Once the model-ready feature dataset is available, it is passed to the trained ML model.

A generic inference flow is:

Scheduler
    ↓
Python Application / Script
    ↓
Feature Dataset
    ↓
Trained ML Model
    ↓
Prediction
    ↓
Output Storage

The inference process typically:

Loads or accesses the trained model.

Retrieves the prepared lead dataset.

Selects required features.

Applies required preprocessing.

Generates predictions.

Stores or sends the results downstream.

The production Royal Enfield implementation uses a CatBoost classifier; its exact artifact and threshold behavior are documented later.

10. Prediction Output

For each eligible lead, the model generates a prediction.

Lead
  ↓
Feature Dataset
  ↓
ML Model
  ↓
Prediction Score

Example:

Lead ID: 12345
Prediction Probability: 0.87

A business implementation may map scores into categories such as:

High
Medium
Low

Example thresholds:

0.80 - 1.00 → High
0.50 - 0.79 → Medium
0.00 - 0.49 → Low

These thresholds are illustrative only. They must not be assumed to be the Royal Enfield production thresholds. The actual RE threshold logic is documented in Section 26.

11. Scheduling and Dependencies

A production pipeline should execute in a controlled sequence:

Task A
  ↓
Task B
  ↓
Task C
  ↓
Task D

Relying solely on fixed execution times is less reliable:

Query A → 10:00
Query B → 10:10
Query C → 10:20

If Query A takes longer than expected, Query B may start before its dependency is ready.

Explicit dependencies are therefore preferable where the orchestration platform supports them.

The Royal Enfield production implementation differs from this recommendation: its major services are independently scheduled and communicate through shared BigQuery tables.

12. CRM Delivery

After inference:

Raw Lead
    ↓
Feature Processing
    ↓
Prediction
    ↓
Scored Lead
    ↓
Delivery / Routing Layer
    ↓
CRM / DMS

Downstream systems can use scores for:

Lead prioritization.

Sales assignment.

Campaign targeting.

Follow-up prioritization.

Reporting.

Business analysis.

For Royal Enfield, the delivery path contains both genuinely scored leads and fallback/unscored records. Therefore, "scored" and "delivered" are not equivalent concepts in the RE implementation.

13. Backfills and Incremental Processing

13.1 Backfills

A backfill is required when historical leads were not processed normally.

Example:

Expected Leads: 100
Processed:       52
Missing:         48

A generic backfill process is:

Historical Source Data
        ↓
Identify Missing Leads
        ↓
Validate Required Fields
        ↓
Insert into Processing Layer
        ↓
Run Normal Processing
        ↓
Generate Predictions

The preferred principle is:

A backfilled lead should enter the same logical processing path as a normal production lead whenever possible.

This keeps historical and live processing behavior consistent.

Backfill validation

Before inserting historical data, validate:

Column names
Column data types
Timestamp formats
Lead identifiers
Required fields

Duplicate prevention

Lead already exists?
       |
   +---+---+
   |       |
  Yes      No
   |       |
 Skip    Insert

Duplicates can cause:

Duplicate predictions.

Multiple CRM updates.

Incorrect reporting.

13.2 T-1 Incremental Processing

A common daily pattern is:

T = Today
T-1 = Yesterday

Process T-1

For example, on August 10, the pipeline may process August 9.

T-1 processing gives upstream systems time to complete ingestion before downstream scoring.

However, T-1-only processing can miss late-arriving historical records.

A more robust approach is to process eligible records that remain unprocessed:

WHERE prediction IS NULL
  AND record_is_eligible = TRUE

rather than relying exclusively on:

WHERE date = T-1

The exact approach depends on the production pipeline's processing state and idempotency design.

14. Idempotency, Processing State, and Watermarks

14.1 Idempotency

A production pipeline should ideally be idempotent:

Running the same processing operation multiple times should not create unintended duplicate results.

Desired behavior:

Run 1:
Lead 123 → Score Generated

Run 2:
Lead 123 → Same record updated or skipped

Undesired behavior:

Run 1 → Score
Run 2 → Duplicate Score
Run 3 → Another Duplicate Score

Idempotency is particularly important for:

Retries.

Backfills.

Pipeline failures.

Manual reruns.

14.2 Processing State

A robust design can track the state of each lead:

Lead ID

Status

123

RAW

123

FEATURE_READY

123

SCORED

123

SENT_TO_CRM

123

FAILED

Typical flow:

RAW
 ↓
FEATURE_READY
 ↓
SCORED
 ↓
SENT_TO_CRM

Failure example:

RAW
 ↓
FEATURE_READY
 ↓
FAILED

This makes recovery and debugging easier.

14.3 Watermarks

A watermark records the last successfully processed point in time.

Example:

Last successful watermark:
2026-08-10 10:00:00

The next run can process records after that point:

Last Successful Watermark
          ↓
Fetch New Records
          ↓
Process Successfully
          ↓
Update Watermark

Watermarks can help handle:

Late-arriving data.

Pipeline delays.

API delays.

High-frequency ingestion.

Recovery after failures.

15. Error Handling

A production pipeline should not silently fail.

Common failure classes include:

Data Ingestion

Source unavailable
Missing data
Schema changes

BigQuery

Permission denied
Missing table
Invalid query
Quota issues

Model

Model unavailable
Feature mismatch
Invalid input
Prediction error

CRM / DMS Delivery

API failure
Authentication issue
Network failure
Invalid payload

Each stage should produce meaningful logs and, where appropriate, retries and alerts.

A generic failure flow is:

Lead Processing
      │
      ▼
Data Validation
      │
      ├── Invalid → Log Failure
      │
      ▼
Feature Generation
      │
      ├── Failure → Log Failure
      │
      ▼
Model Prediction
      │
      ├── Failure → Log Failure
      │
      ▼
CRM / DMS Update
      │
      ├── Failure → Retry / Alert
      │
      ▼
Success

16. Monitoring and Alerting

Monitoring should cover both technical health and business outcomes.

16.1 Technical Monitoring

Monitor:

Pipeline execution status
Query failures
API failures
Model inference failures
Latency

16.2 Business Monitoring

Monitor:

Number of leads received
Number of eligible leads
Number of leads scored
Number of leads delivered
Number of failed leads
Number of missing predictions

A useful reconciliation is:

Incoming Leads
      ↓
Eligible Leads
      ↓
Scored Leads
      ↓
Successfully Updated / Delivered Leads

Example:

Stage

Count

Incoming Leads

1,000

Eligible Leads

950

Scored Leads

940

CRM Updated

938

Business-level validation is important because a job can technically succeed while processing zero records.

Useful alerts include:

Scored lead count below an expected threshold.

Large gap between incoming and scored leads.

CRM/DMS update failures.

Pipeline execution failure.

Sudden increase in invalid records.

No predictions generated.

17. Debugging and Data Lineage

When a lead is missing, trace the complete lineage rather than debugging individual components randomly.

1. Does the lead exist in the source?
        ↓
2. Does it exist in the raw / ingestion layer?
        ↓
3. Does it exist in the base dataset?
        ↓
4. Does it exist in the historical / stitched dataset?
        ↓
5. Did it reach model inference?
        ↓
6. Was a prediction generated?
        ↓
7. Was the result written to the downstream system?

The core principle is:

Start at the point where the failure is visible and trace the data backward through the pipeline.

A generic lineage is:

SOURCE SYSTEMS
      ↓
RAW DATA
      ↓
BASE DATA
      ↓
HISTORICAL FEATURES
      ↓
STITCHED DATA
      ↓
MODEL INPUT
      ↓
PREDICTION
      ↓
CRM / DMS OUTPUT

For any scored lead, the system should ideally answer:

Where did this lead come from?
Which pipeline processed it?
Which features were used?
Which model version produced the prediction?
What prediction was generated?
Was it successfully delivered?

18. Audit Fields and Model Versioning

A robust scored dataset should ideally maintain:

lead_id
source
lead_created_time
processing_time
prediction_time
prediction_score
prediction_label
model_version
pipeline_run_id
processing_status
crm_update_status

Model Versioning

Predictions should ideally be traceable to the model that produced them.

Example:

Lead ID

Score

Model Version

123

0.82

v1

456

0.91

v2

This answers:

Which model generated this prediction?

Royal Enfield production gap

The current RE production scoring output does not store a model_version field in audience_l2_lead_score_* or dms_raw_data. Therefore, historical RE scores cannot currently be traced to a model version through those tables.

19. Training vs. Inference

Training and production scoring are different processes.

Model Training

Historical Data
      ↓
Feature Engineering
      ↓
Train Model
      ↓
Evaluate
      ↓
Deploy Model

Training happens periodically.

Model Inference

New / Eligible Lead
      ↓
Generate Features
      ↓
Load Existing Model
      ↓
Predict
      ↓
Store Score

The daily lead-scoring pipeline is primarily performing:

Inference, not model training.

20. System Responsibilities

A generic lead-scoring system is responsible for:

Data Ingestion

Collect incoming leads.

Data Preparation

Clean and standardize lead information.

Feature Engineering

Create model-compatible features.

Historical Enrichment

Add relevant behavioral and CRM history.

Prediction

Run the trained model.

Output Delivery

Store and deliver predictions to downstream systems.

Monitoring

Verify that expected records are processed.

Recovery

Support retries, reconciliation, and backfills.

21. Royal Enfield Production Implementation

21.1 Production Architecture

The Royal Enfield system documented here is not a single unified orchestrated pipeline.

It consists of three major scheduled services:

CRM ingestion

GA4 join + candidate selection + model scoring

DMS delivery

The services are independently triggered by Cloud Scheduler and communicate through shared BigQuery tables.

CRM INGESTION
daily-btr-data-export-to-bq-v2
        │
        │ every ~15 min
        ▼
RE_web_crm_data_*
        │
        ▼
GA4 JOIN + CANDIDATE SELECTION + SCORING
New_30day_logic_real_time
        │
        ▼
CatBoost Model
        │
        ▼
audience_l2_lead_score_*
        │
        ▼
DMS DELIVERY
scored-leads-backto-web-crm
        │
        ▼
dms_raw_data
        │
        ▼
DMS API

Critical architectural distinction

dms_raw_data is not the raw ingestion table in the RE production system.

The actual RE ingestion table is:

re-platform-model-dl.web_crm_data.RE_web_crm_data_*

dms_raw_data is a pre-DMS delivery/routing staging table containing a mixture of genuinely scored and fallback records.

22. Royal Enfield Data Sources and Architecture

Concept

Actual RE Production Table / Service

Raw CRM lead ingestion

re-platform-model-dl.web_crm_data.RE_web_crm_data_*

GA4 ↔ CRM identity resolution / candidate selection

re-platform-model-dl.sept_test_ls.New_30day_logic_real_time

Genuine model scoring output

re-platform-model-dl.ga4_ls_model_dataset.audience_l2_lead_score_*

DMS routing / delivery staging

re-platform-model-dl.dms_logs.dms_raw_data

Dashboard snapshot

re-platform-model-dl.tvc_reports.leads_count_t-1

Dashboard tracking table

re-platform-model-dl.dms_logs.lead_traking_dashboard

The production architecture depends on scheduled services and query filters correctly identifying eligible records at each stage.

There is no single production orchestrator connecting all three services.

23. Royal Enfield Identity Resolution

RE identity resolution connects CRM and GA4 using a client identifier.

CRM side

clientId is captured on the RE website when a lead form is submitted. It represents the GA4 browser client_id value passed through the form.

GA4 side

custom_client_id is extracted from GA4 user properties:

(
  SELECT up.value.string_value
  FROM UNNEST(user_properties) up
  WHERE up.key = 'custom_client_id'
)

The join condition is:

ON a.client_id = b.clientid_crm
AND a.event_date = b.date_crm

Critical production limitation

If clientId is blank at the CRM level, there is no fallback identity-resolution mechanism in this stage.

The lead is excluded from scoring candidacy.

Production debugging found this to be a major cause of missing leads; across audited days, blank client IDs accounted for roughly 40–85% of dropped leads on a given day.

This behavior is specific to the documented RE implementation and should not be generalized to lead-scoring systems in general.

24. Royal Enfield GA4 Constraints

The RE GA4 join has two hard production constraints.

24.1 45-Minute Intraday Window

The join reads only from GA4 intraday data and restricts events to the preceding 45 minutes:

FROM `re-enterprise-dl.analytics_253617568.events_intraday_*`
WHERE TIMESTAMP_DIFF(
  CURRENT_TIMESTAMP(),
  event_timestamp,
  MINUTE
) <= 45

There is no fallback to finalized GA4 daily export tables.

Therefore, once the 45-minute matching window has passed without a match, that opportunity is permanently missed by this scoring path.

24.2 Hostname Whitelist

Only these exact hostnames are eligible:

^www.royalenfield.com$
|^finance.royalenfield.com$
|^accessories.royalenfield.com$
|^makeityours.royalenfield.com$

GA4 activity on other RE-owned hostnames is excluded from this join.

25. Royal Enfield Feature Engineering

The RE scoring script's pre_processing() function applies hardcoded categorical whitelists that were frozen at model training time.

The model artifact references:

gcs_model_path = "sept_model"

Current documented caps include:

Feature

Known Values

Fallback

source

8 known ad sources

other sources

medium

8 known mediums

other_mediums

browser

7 known browsers

other_browsers

bikemodel

~19 named models frozen at training

other_bk_models

city

Fixed A-1/A/B-1/B-2 tier lists

(Others)

Production risk

A bike model introduced after the model's training period can still be scored, but if it is absent from the hardcoded list it is silently categorized as:

other_bk_models

This can reduce prediction quality without generating an explicit model or data-quality alert.

26. Royal Enfield Model and Threshold Logic

26.1 Model Artifact

The documented production model is:

Artifact: ls_sept_model_23.pkl
Type: CatBoost classifier
GCS bucket: kubeflow-ls-model

26.2 Lead Priority Buckets

The production bucket logic is:

lead_score >= 0.57
    → Bucket 1 / Hot
    → leadpriority = 163650001

lower_thresh <= lead_score < 0.57
    → Bucket 2 / Warm
    → leadpriority = 163650002

lead_score < lower_thresh
    → Bucket 3 / Cold
    → leadpriority = 163650003

Unless explicit thresholds are supplied, lower_thresh and higher_thresh are recalculated for each run from that batch's score quantiles using:

0.3 quantile
0.7 quantile

Model-version traceability gap

Although the conceptual architecture recommends storing model_version, the current RE output tables do not contain a model-version field.

Therefore:

A historical RE prediction cannot currently be traced to a model version through audience_l2_lead_score_* or dms_raw_data.

27. Royal Enfield Output and Delivery

The RE DMS delivery query, ga_leadscoring_data, contains three distinct branches.

27.1 pred_data

Genuinely model-scored leads:

CRM joined with audience_l2_lead_score_*

Prediction timestamp within the relevant ~16-minute window

27.2 remaining_leads_data

Fallback path for leads that:

Have a valid clientId

Miss genuine scoring

Fall within the documented creation-time window

These records are delivered without a genuine model score and use:

pred_time = ''

27.3 fb_instant_lead_form

Facebook Instant Lead Form fast path:

Uses a 20-minute window.

Bypasses the clientId requirement.

This is the only documented delivery branch that bypasses the normal client-ID requirement.

Critical reporting distinction

In RE:

Delivered to DMS does not necessarily mean genuinely model-scored.

For accurate reporting, distinguish:

pred_time != ''
    → genuinely scored

pred_time = ''
    → fallback / sweep / unscored delivery

28. Royal Enfield Leakage Gates

The following production gates were confirmed during debugging:

Gate

Stage

Effect

Blank clientId

CRM → GA4 join

Excludes the lead from normal scoring; Facebook fast path is the exception

No GA4 session within 45-minute window

GA4 join

Excludes the lead from scoring candidacy

GA4 session on non-whitelisted hostname

GA4 join

Excludes the lead

TIME()-only comparison around midnight

Multiple queries

Can silently exclude records around 23:45–00:15 IST because of date-rollover logic

remaining_leads_data 24-minute window overlapping a ~15-minute cadence

DMS delivery

Can cause duplicate delivery

Pincode

Pincode was tested and confirmed not to be a leakage gate.

It affects lead-priority sub-bucketing but does not determine basic scoring eligibility.

29. Royal Enfield Known Bugs and Fixes

Period

Issue

Root Cause

Status

Aug 2

45-minute scoring gap / false "no leads scored" investigation

Two consecutive scoring cycles (05:32 and 05:47) did not execute despite healthy upstream inputs

Root-caused; owning scheduler still needs identification

Aug 1–9

Duplicate rows in dms_raw_data, up to 586/day observed

remaining_leads_data 24-minute window overlaps a ~15-minute cadence; no deduplication against existing rows

Fix drafted (NOT IN exclusion), not yet deployed

Aug 6

leads_count_t-1 showed stale count after backfill

Daily snapshot had no automatic re-sync mechanism

Fixed using a MERGE-based update

Ongoing

email_alert_for_dms() crash (NameError: url)

url / headers variables were commented out while requests.post() remained active

Fixed

30. Royal Enfield Reconciliation Sweeper

To mitigate missing leads without waiting for every upstream scoring-path issue to be fixed, a dedicated reconciliation Cloud Function was built:

daily-leads-reconciliation

It runs approximately once daily:

~01:00 AM IST
      │
      ▼
Scan T-1 and T-0 CRM partitions
      │
      ▼
Find leadIds absent from dms_raw_data
      │
      ▼
Push the entire batch to DMS API in one request
      │
      ▼
Log successful pushes into dms_raw_data
(pred_time = '')

Important: The Sweeper Does Not Score Leads

The sweeper does not re-enter the normal scoring pipeline.

It delivers directly to DMS with:

leadpriority = 163650002
leadinsights = "Pitch for GMA"
pred_time = ''

Therefore:

Swept leads are delivered but do not receive genuine model-driven scores.

This is an intentional recovery strategy: deliver something rather than deliver nothing.

Deduplication

Production verification for Aug 7–9 found zero duplicate leadId overlap across sweep days, confirming that the sweeper's own deduplication logic works independently of the separate remaining_leads_data duplication issue.

31. Royal Enfield Monitoring and Alerting

Three documented alert mechanisms exist.

Alert

Trigger

Known Behavior

CRM ingestion alert

0 rows in RE_web_crm_data_* during the last 60 minutes

Can generate false positives during naturally low-traffic periods

DMS scoring alert

0 rows in dms_raw_data with pred_time != '' during the last 16/30 minutes

Correctly detected the genuine Aug 2 scoring gap

Both alerts

TIME()-only comparisons

Can misfire around midnight because of the date-rollover issue

The DMS scoring alert is therefore more directly aligned with genuine model-scoring activity, while the CRM ingestion alert must be interpreted in the context of expected traffic.

32. Royal Enfield Dashboard and Reporting

The live RE dashboard:

[Royal Enfield] Lead Scoring Dashboard - GA4

is powered by daily BigQuery snapshots, rather than querying dms_raw_data live.

Therefore:

Dashboard numbers represent the state captured by the most recent snapshot.

Backfills or sweeps performed after a snapshot may not appear immediately.

Snapshot tables may require a re-sync to reflect subsequent changes.

Reporting requirement

When reporting from dms_raw_data, use:

COUNT(DISTINCT leadId)

rather than:

COUNT(*)

because duplicate rows have been observed in production.

This prevents the duplication issue from inflating reported lead counts.

33. Royal Enfield UAT Environment

The RE UAT script is a structurally separate test system, not a mirror of the production ML pipeline.

It targets:

api-uat2.royalenfield.com

The UAT process:

Fetches leads through a direct fetch-btr-leads GET API.

Does not read from RE_web_crm_data_*.

Uses hardcoded, rule-based priority logic based on the last character of leadId.

Does not use the production ML model.

Has no connection to audience_l2_lead_score_*.

Has no connection to dms_raw_data.

Has no connection to the reconciliation sweeper.

Stores test leads in a separate 30-day rolling-retention BigQuery table:

uat_leads_hardcoded_priority

Important

Do not interpret UAT priority results as production ML scores.

UAT validates test-lead behavior using hardcoded rules; it does not replicate production model inference.

34. Royal Enfield Glossary

Term

Meaning

RE_web_crm_data_*

Daily-sharded raw CRM lead-ingestion table

New_30day_logic_real_time

GA4 ↔ CRM identity-resolution and candidate-selection view

audience_l2_lead_score_*

Daily-sharded genuine model-scoring output

dms_raw_data

Pre-DMS delivery staging table containing genuinely scored and fallback records

daily-btr-data-export-to-bq-v2

CRM ingestion Cloud Run service

scored-leads-backto-web-crm

DMS-push Cloud Run service

crm_lead_scoring_alert_v1

Alerting Cloud Function

daily-leads-reconciliation

Reconciliation / sweeper Cloud Function

lead_traking_dashboard

Dashboard-facing tracking snapshot table

leads_count_t-1

Dashboard-facing daily snapshot table

pred_time != ''

Convention indicating genuine model scoring

pred_time = ''

Convention indicating fallback / sweep / unscored delivery

35. Key Takeaways

Conceptual System

The generic lifecycle is:

Lead Generated
      ↓
Lead Stored
      ↓
Data Cleaned
      ↓
Historical / Behavioral Data Added
      ↓
Features Created
      ↓
ML Model Runs
      ↓
Conversion Probability Generated
      ↓
Lead Prioritized
      ↓
Score Delivered to CRM / Business System

The most important engineering principles are:

Maintain traceable source-to-output data lineage.

Prevent duplicate lead processing.

Maintain schema consistency during backfills.

Use timestamps consistently.

Prevent data leakage.

Prefer explicit dependencies over arbitrary scheduling assumptions.

Make processing idempotent.

Track processing state.

Monitor business outcomes, not only job success.

Support reliable reconciliation and recovery.

Track model versions where possible.

Make each prediction traceable to its source, processing run, and model.

Royal Enfield Production System

The documented RE implementation has several important characteristics that must not be confused with the generic architecture:

It uses three independently scheduled services, not one unified orchestrator.

RE_web_crm_data_* is the raw CRM ingestion layer.

dms_raw_data is a DMS delivery staging layer, not the raw ingestion layer.

Normal scoring depends on CRM clientId ↔ GA4 custom_client_id identity resolution.

The GA4 scoring path uses a strict 45-minute intraday window.

Only four specified RE hostnames are accepted by the GA4 join.

Blank clientId is a major source of missing scored leads.

The production model is a CatBoost classifier using ls_sept_model_23.pkl.

lead_score >= 0.57 maps to the Hot bucket.

DMS delivery includes genuine scoring, fallback delivery, and a Facebook-specific fast path.

A lead being delivered to DMS does not necessarily mean it was model-scored.

The reconciliation sweeper delivers missing leads directly to DMS but does not score them.

Duplicate records have been observed in dms_raw_data; reporting should use COUNT(DISTINCT leadId).

The current production scoring output does not store a model version.

The UAT system is separate from production and uses hardcoded rules rather than the ML model.

One-Line Summary

The Lead Scoring System transforms incoming leads and behavioral/CRM information into actionable predictions, while the Royal Enfield production implementation uses independently scheduled ingestion, GA4-based candidate selection, CatBoost scoring, DMS delivery, and a separate reconciliation mechanism to recover leads that miss the normal scoring path.