# Lead Scoring System — Complete Architecture, Implementation, Operations, and Royal Enfield Production Documentation

> This document consolidates the complete lead-scoring documentation into a single Markdown knowledge base. It includes:
>
> 1. Lead scoring fundamentals and models
> 2. Generic end-to-end data architecture and machine learning concepts
> 3. Pipeline processing, feature engineering, inference, delivery, monitoring, and recovery
> 4. Backfills, idempotency, watermarks, debugging, data lineage, and model versioning
> 5. Royal Enfield's actual production implementation, real tables, services, constraints, bugs, fixes, reconciliation, alerting, reporting, and UAT environment
>
> **Important:** Where the Royal Enfield implementation differs from the generic/conceptual architecture, the Royal Enfield section documents the actual production behavior.

---

## Table of Contents

1. Introduction and Fundamentals
2. Lead Scoring Models
3. Data Pillars and Feature Engineering
4. Generic System Architecture and Data Flow
5. Pipeline Processing Layers
6. Identity Resolution
7. Data Leakage Prevention
8. Model Inference and Prediction Output
9. Scheduling and Dependency Management
10. CRM Delivery
11. Backfills and Incremental Processing
12. Idempotency, Processing State, and Watermarks
13. Error Handling and Failure Scenarios
14. Monitoring and Alerting
15. Debugging and Data Lineage
16. Audit Fields and Model Versioning
17. Training vs. Inference
18. System Responsibilities and Key Takeaways
19. Royal Enfield Production Implementation
20. Royal Enfield Architecture and Data Sources
21. Royal Enfield Identity Resolution and GA4 Constraints
22. Royal Enfield Feature Engineering and Model Logic
23. Royal Enfield Output and Delivery Paths
24. Leakage Gates, Bugs, and Fixes
25. Reconciliation and Backfill Sweeper
26. Monitoring, Dashboard, Reporting, and UAT
27. Royal Enfield Glossary

---


# Lead Scoring System
## Functional, Data Flow, and Technical Documentation
---

## 1. Overview
The ****Lead Scoring System**** is a machine learning–driven pipeline designed to identify and prioritize leads based on their likelihood of conversion.

Instead of treating every incoming lead equally, the system analyzes available customer, CRM, website, and behavioral information and assigns a score or prediction indicating the likelihood that a lead will perform the target business action.

The overall objective is:

\> ****Capture leads → enrich them with historical and behavioral information → generate predictions → make the scored leads available to downstream CRM/business systems.****

This helps the business prioritize high-value leads and focus sales or marketing efforts on users with a higher probability of conversion.

---

# 2. What is Lead Scoring?
Lead scoring is the process of assigning a numerical score, probability, or category to a lead based on how likely that lead is to convert.

For example:

| Lead   | Predicted Conversion Probability | Priority |
| ------ | -------------------------------: | -------- |
| Lead A |                             0.92 | High     |
| Lead B |                             0.71 | Medium   |
| Lead C |                             0.23 | Low      |

A lead with a higher score can be prioritized for faster sales follow-up or different marketing treatment.

The exact business meaning of a conversion depends on how the model was trained.

Examples include:

* Purchase
* Booking
* Test drive
* Form completion
* Qualified lead
* Sales conversion
* Any other defined business outcome

---

# 3. Why Do We Need Lead Scoring?
Without lead scoring, sales teams may treat all leads similarly.

This creates several problems:

* Sales teams spend time on low-intent leads.
* High-intent leads may not receive immediate attention.
* Lead prioritization depends heavily on manual judgment.
* Marketing budgets may be spent inefficiently.
* It becomes difficult to systematically identify high-potential customers.

The lead scoring model solves this by using historical data to identify patterns associated with conversion.

The goal is not simply:

\> "Which lead is good?"

Instead, the machine learning problem is:

\> ****"Based on the information available for this lead, what is the probability that this lead will convert?"****

---

# 4. High-Level Architecture
The overall flow of the system can be represented as:

```text
                 +-------------------+
                 |   Lead Sources    |
                 | CRM / Web / Forms |
                 +---------+---------+
                           |
                           v
                 +-------------------+
                 |   Raw Lead Data   |
                 |   dms_raw_data    |
                 +---------+---------+
                           |
                           v
              +------------------------+
              | Data Processing /      |
              | Feature Preparation    |
              +-----------+------------+
                          |
                          v
              +------------------------+
              | Historical Aggregation |
              | Behavioral Features    |
              +-----------+------------+
                          |
                          v
                 +-------------------+
                 | Feature Dataset   |
                 | Stitched Data     |
                 +---------+---------+
                           |
                           v
                 +-------------------+
                 | ML Inference      |
                 | main.py / Model   |
                 +---------+---------+
                           |
                           v
                 +-------------------+
                 | Prediction Output |
                 | Lead Score        |
                 +---------+---------+
                           |
                           v
                 +-------------------+
                 | CRM / Web CRM /   |
                 | Business System   |
                 +-------------------+
```

---

# 5. Major Components of the Pipeline
The pipeline consists of multiple stages.

## 5.1 Raw Lead Data Layer
The raw data layer stores incoming lead information.

One of the key tables involved is:

```text
re-platform-model-dl.dms_logs.dms_raw_data
```

This table acts as a central raw or intermediate source for leads entering the scoring pipeline.

Lead information may originate from systems such as:

* Web CRM
* Lead forms
* Facebook Instant Lead Forms
* Other CRM or campaign sources

The purpose of this layer is to create a structured dataset that downstream pipeline components can process.

---

# 6. Lead Data Sources
The system can ingest lead data from multiple sources.

Examples discussed in the existing pipeline include:

```text
Web CRM
Facebook Instant Lead Forms
CRM Data
GA4 / Website Behavioral Data
```

A conceptual flow looks like:

```text
                    +-------------+
                    |   Web CRM   |
                    +-------------+
                           |
                           |
                    +-------------+
                    | Facebook    |
                    | Lead Forms  |
                    +-------------+
                           |
                           |
                    +-------------+
                    | Other CRM   |
                    | Lead Data   |
                    +------+------+ 
                           |
                           v
                  +------------------+
                  |  dms_raw_data    |
                  +------------------+
```

---

# 7. Raw Lead Storage
The raw lead table acts as the starting point for scoring.

Conceptually, a lead record may contain information such as:

```text
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
```

The exact schema can vary depending on the source.

The key requirement is that the lead should contain enough information to:

1. Identify the lead.
2. Join it with additional data where required.
3. Generate the features required by the model.
4. Track the prediction generated for that lead.

---

# 8. Data Processing Pipeline
The processing pipeline can be understood in the following stages:

```text
Raw Data
   ↓
Base Data
   ↓
Historical Aggregated Data
   ↓
Stitched Data
   ↓
Model Inference
   ↓
Prediction Output
```

Each layer has a specific responsibility.

---

# 9. Base Data
The ****Base Data**** layer prepares the fundamental dataset required for further processing.

Typical responsibilities include:

* Selecting required columns.
* Cleaning raw data.
* Standardizing fields.
* Applying source filters.
* Removing invalid records.
* Creating consistent identifiers.
* Preparing data for joins.

Conceptually:

```text
Raw Lead
   ↓
Cleaning
   ↓
Standardization
   ↓
Validation
   ↓
Base Lead Dataset
```

This layer should not contain unnecessary model logic.

Its main responsibility is to create a reliable starting dataset.

---

# 10. Historical Aggregated Data
Lead scoring often requires more than just the current lead information.

Historical data can provide additional signals.

For example:

```text
Previous website activity
Previous sessions
Historical engagement
Past interactions
Campaign history
Customer behavior
```

These signals are aggregated to create historical features.

Example:

```text
User A

Previous Sessions:        5
Previous Website Events:  32
Previous Form Visits:     2
Previous Engagement:      High
```

These historical characteristics may help the model distinguish between:

```text
High Intent Lead
```

and

```text
Low Intent Lead
```

---

# 11. Stitched Data
The ****Stitched Data**** layer combines information from different sources.

Conceptually:

```text
Lead Data
    +
Historical Data
    +
CRM Data
    +
Behavioral Data
    ↓
Stitched Dataset
```

This dataset is closer to the final model input.

The stitched dataset should ideally contain:

* Current lead information.
* Historical behavioral information.
* Required derived features.
* Consistent identifiers.
* Model-ready columns.

---

# 12. Identity Resolution
A critical part of the pipeline is identifying and connecting information belonging to the same user.

For example, a user may exist across:

```text
GA4
CRM
Lead Forms
Web CRM
```

Different systems may use different identifiers.

Examples discussed include:

```text
client_id
custom_client_id
ga_session_id
```

The pipeline may use identifiers from GA4 user properties and event parameters to connect behavioral activity with lead records.

Conceptually:

```text
Website User
     |
     | client_id / custom_client_id
     v
GA4 Behavior
     |
     +-------------------+
                         |
                         v
                    Lead Record
                         |
                         v
                    ML Features
```

Proper identity resolution is important because incorrect joins can cause:

* Duplicate records.
* Missing historical behavior.
* Incorrect model features.
* Wrong predictions.

---

# 13. Feature Engineering
Machine learning models do not directly understand raw business data.

The pipeline converts raw information into ****features****.

Examples of possible feature categories include:

## Demographic Features
```text
Location
Pincode
Region
```

## Acquisition Features
```text
Campaign
Source
Medium
Channel
```

## Behavioral Features
```text
Number of sessions
Website engagement
Events performed
Previous visits
Interaction history
```

## Historical Features
```text
Past user activity
Aggregated interactions
Previous engagement
Historical conversion signals
```

These features become the input to the machine learning model.

---

# 14. Preventing Data Leakage
One important engineering principle in the pipeline is avoiding ****data leakage****.

Data leakage happens when the model receives information that would not realistically be available at the time of prediction.

For example:

```text
Lead created at 10:00 AM

Feature calculation accidentally includes:
Website activity at 3:00 PM
```

This would create an unrealistic model.

The model would appear more accurate during training but fail in real production usage.

Therefore:

\> Historical features should only use information available before the prediction time.

Conceptually:

```text
Past Data
    |
    v
Prediction Time
    |
    X
    |
Future Data must not be used
```

---

# 15. Model Inference
Once the final feature dataset is ready, it is passed to the trained machine learning model.

The inference process is executed through the scoring application.

The pipeline discussed includes execution through components such as:

```text
Cron Job / Scheduler
      ↓
Python Script / Notebook
      ↓
main.py
      ↓
ML Model
      ↓
Prediction
```

The main inference process performs tasks such as:

1. Load or access the model.
2. Retrieve the prepared lead dataset.
3. Select required features.
4. Apply required preprocessing.
5. Run model prediction.
6. Generate the prediction score.
7. Store or send the output downstream.

---

# 16. Prediction Output
The model produces a prediction for each eligible lead.

Conceptually:

```text
Lead
  ↓
Feature Dataset
  ↓
ML Model
  ↓
Prediction Score
```

Example:

```text
Lead ID: 12345

Prediction Probability: 0.87
```

Depending on the business implementation, this may be converted into categories such as:

```text
High
Medium
Low
```

For example:

```text
0.80 - 1.00 → High Priority
0.50 - 0.79 → Medium Priority
0.00 - 0.49 → Low Priority
```

The actual thresholds should be determined based on the production model and business requirements.

---

# 17. Scheduled Pipeline Execution
The lead scoring pipeline is designed to run automatically.

A conceptual schedule is:

```text
New Leads Arrive
       ↓
Scheduled Data Processing
       ↓
Feature Preparation
       ↓
Model Inference
       ↓
Prediction Generation
       ↓
Write Results
```

The existing pipeline includes scheduled query and cron-based execution.

One important engineering consideration is dependency management.

A pipeline should ideally follow:

```text
Task A
  ↓
Task B
  ↓
Task C
  ↓
Task D
```

rather than relying only on arbitrary execution times such as:

```text
Query A → 10:00
Query B → 10:10
Query C → 10:20
```

Time-based assumptions can fail when an upstream query takes longer than expected.

Explicit dependencies are more reliable.

---

# 18. End-to-End Data Flow
The complete flow can be summarized as:

```text
                 LEAD SOURCES
                      │
          ┌───────────┼────────────┐
          │           │            │
          ▼           ▼            ▼
       Web CRM      CRM       Lead Forms
          │           │            │
          └───────────┼────────────┘
                      │
                      ▼
              RAW LEAD DATA
               dms_raw_data
                      │
                      ▼
                BASE DATA
                      │
                      ▼
          HISTORICAL AGGREGATION
                      │
                      ▼
               STITCHED DATA
                      │
                      ▼
             FEATURE DATASET
                      │
                      ▼
                ML MODEL
                      │
                      ▼
             LEAD PREDICTION
                      │
                      ▼
              SCORED OUTPUT
                      │
                      ▼
               WEB CRM / CRM
```

---

# 19. Writing Scores Back to CRM
After inference, the scored output needs to be made available to downstream systems.

The pipeline can therefore follow:

```text
Raw Lead
   ↓
Feature Processing
   ↓
Prediction
   ↓
Scored Lead
   ↓
Web CRM
```

The downstream CRM can then use the score for:

* Lead prioritization.
* Sales assignment.
* Campaign targeting.
* Follow-up prioritization.
* Reporting.
* Business analysis.

---

# 20. Backfilling Leads
A backfill is required when historical leads were not processed by the pipeline.

For example:

```text
Expected Leads: 100

Leads Processed: 52

Missing Leads: 48
```

Those missing leads can be inserted back into the raw processing table.

The general process is:

```text
Historical Source Data
        ↓
Identify Missing Leads
        ↓
Validate Required Fields
        ↓
Insert into dms_raw_data
        ↓
Run Normal Pipeline
        ↓
Generate Predictions
```

The key principle is:

\> A backfilled lead should ideally enter the pipeline through the same logical path as a normal production lead.

This ensures consistency between historical and live processing.

---

# 21. Backfill Example
Conceptually:

```sql
INSERT INTO dms_raw_data

SELECT *
FROM historical_lead_source
WHERE lead_date BETWEEN start_date AND end_date
```

In the existing implementation, leads were backfilled from Web CRM source tables into:

```text
re-platform-model-dl.dms_logs.dms_raw_data
```

After insertion, the expectation is that these leads become available for downstream processing and scoring, provided they meet all pipeline conditions.

---

# 22. Important Backfill Considerations
Before backfilling, the following should be validated.

## Schema Consistency
The inserted data should match the expected schema.

Important checks include:

```text
Column names
Column data types
Timestamp formats
Lead identifiers
Required fields
```

---

## Timestamp Consistency
Timestamp fields such as:

```text
pred_time
lead creation time
event time
processing time
```

should follow the format expected by downstream queries and applications.

Incorrect timestamp formats can cause:

* Join failures.
* Filtering issues.
* Duplicate processing.
* Incorrect historical aggregation.
* Pipeline failures.

---

## Duplicate Prevention
Before inserting historical data, check whether the lead already exists.

Conceptually:

```text
Lead ID already exists?
        |
   ┌────┴────┐
   │         │
  Yes        No
   │         │
Skip      Insert
```

This is important because duplicate records may result in:

* Duplicate predictions.
* Multiple CRM updates.
* Incorrect reporting.

---

# 23. Daily Incremental Processing
The pipeline may process data using a daily schedule.

For example:

```text
Today = August 10

Pipeline processes:

August 9
```

This is commonly referred to as:

```text
T-1 Processing
```

The pattern is:

```text
Current Day
     ↓
Process Previous Day's Data
```

Example:

```text
T-1 = Yesterday

T = Today
```

This allows upstream systems time to complete data ingestion before scoring begins.

---

# 24. Important Consideration for T-1 Processing
If the pipeline only processes:

```text
Yesterday's records
```

then historical or late-arriving records may not automatically be picked up.

For example:

```text
Lead Date: August 1

Current Processing Date: August 10

Pipeline Filter:

WHERE date = August 9
```

The August 1 lead will not be processed unless:

1. A backfill process is executed.
2. The pipeline has a lookback window.
3. The query supports unprocessed historical records.

A more robust approach can be:

```text
Process records where:

prediction IS NULL
AND
record is eligible for scoring
```

rather than relying exclusively on:

```text
WHERE date = T-1
```

However, this depends on the production pipeline's idempotency and business logic.

---

# 25. Idempotency
A production scoring pipeline should ideally be ****idempotent****.

This means:

\> Running the same pipeline multiple times should not create unintended duplicate results.

For example:

```text
Run 1:
Lead 123 → Score Generated

Run 2:
Lead 123 → Same record updated or skipped
```

Instead of:

```text
Run 1:
Lead 123 → Score Generated

Run 2:
Lead 123 → Another duplicate score

Run 3:
Lead 123 → Another duplicate score
```

Idempotency is especially important for:

* Retries.
* Backfills.
* Pipeline failures.
* Manual reruns.

---

# 26. Recommended Processing State
A robust system should track the state of every lead.

For example:

| Lead ID | Status        |
| ------- | ------------- |
| 123     | RAW           |
| 123     | FEATURE_READY |
| 123     | SCORED        |
| 123     | SENT_TO_CRM   |
| 123     | FAILED        |

Conceptually:

```text
RAW
 ↓
FEATURE_READY
 ↓
SCORED
 ↓
SENT_TO_CRM
```

If a failure occurs:

```text
RAW
 ↓
FEATURE_READY
 ↓
FAILED
```

This makes debugging and recovery easier.

---

# 27. Watermark-Based Processing
Instead of relying only on fixed dates, a robust pipeline can maintain a processing watermark.

Example:

```text
Last Successfully Processed Time:

2026-08-10 10:00:00
```

The next run processes:

```text
Records created after:

2026-08-10 10:00:00
```

Conceptually:

```text
Last Successful Watermark
          ↓
Fetch New Records
          ↓
Process Successfully
          ↓
Update Watermark
```

This helps with:

* Late-arriving data.
* Pipeline delays.
* API delays.
* High-frequency ingestion.
* Reliable recovery after failures.

---

# 28. Error Handling
A production pipeline should not silently fail.

Failures can occur at multiple stages.

## Data Ingestion Failure
```text
Source unavailable
Missing data
Schema changes
```

## BigQuery Failure
```text
Permission denied
Missing table
Invalid query
Quota issues
```

## Model Failure
```text
Model unavailable
Feature mismatch
Invalid input
Prediction error
```

## CRM Write Failure
```text
API failure
Authentication issue
Network failure
Invalid payload
```

Each stage should generate meaningful logs.

---

# 29. Example Failure Flow
```text
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
CRM Update
      │
      ├── Failure → Retry / Alert
      │
      ▼
Success
```

---

# 30. Monitoring
The system should monitor both technical and business outcomes.

Technical monitoring:

```text
Pipeline execution status
Query failures
API failures
Model inference failures
Latency
```

Business monitoring:

```text
Number of leads received
Number of leads scored
Number of leads sent to CRM
Number of failed leads
Number of missing predictions
```

A useful reconciliation check is:

```text
Incoming Leads

vs

Eligible Leads

vs

Scored Leads

vs

Successfully Updated CRM Leads
```

Example:

| Stage          | Count |
| -------------- | ----: |
| Incoming Leads | 1,000 |
| Eligible Leads |   950 |
| Scored Leads   |   940 |
| CRM Updated    |   938 |

This makes it easier to identify where records are being lost.

---

# 31. Recommended Alerting
Alerts should focus on business impact.

For example:

```text
Expected Leads: 1,000
Scored Leads: 10
```

This should trigger an alert.

Useful alerts include:

* Scored lead count below threshold.
* Large difference between incoming and scored leads.
* CRM update failures.
* Pipeline execution failure.
* Sudden increase in invalid records.
* No predictions generated.

Business-level validation is often more useful than simply checking:

```text
"Did the job run?"
```

A job may technically succeed but still process zero records.

---

# 32. Common Failure Scenarios
## Scenario 1: Lead Exists in Source but Not in Scoring
Possible causes:

```text
Lead not inserted into raw table
Date filter excluded the lead
Missing required fields
Join failure
Duplicate filtering
Pipeline did not run
```

---

## Scenario 2: Lead Exists in Raw Data but Is Not Scored
Possible causes:

```text
Feature generation failure
Historical data missing
Lead not eligible
Model input mismatch
Filtering condition
```

---

## Scenario 3: Lead Is Scored but Not Updated in CRM
Possible causes:

```text
CRM API failure
Incorrect payload
Authentication issue
Incorrect identifier
Downstream query filter
```

---

# 33. Debugging Strategy
When a lead is missing, debugging should follow the complete data lineage.

```text
Step 1:
Does the lead exist in the source?

        ↓

Step 2:
Does it exist in dms_raw_data?

        ↓

Step 3:
Does it exist in the base dataset?

        ↓

Step 4:
Does it exist in the historical/stiched dataset?

        ↓

Step 5:
Did it reach model inference?

        ↓

Step 6:
Was a prediction generated?

        ↓

Step 7:
Was the prediction written to CRM?
```

This approach is more reliable than debugging randomly.

The principle is:

\> Start from where the failure is visible and trace the data backward through the pipeline.

---

# 34. Data Lineage
The data lineage for the system can be represented as:

```text
SOURCE SYSTEMS
      │
      ▼
RAW DATA
      │
      ▼
BASE DATA
      │
      ▼
HISTORICAL FEATURES
      │
      ▼
STITCHED DATA
      │
      ▼
MODEL INPUT
      │
      ▼
PREDICTION
      │
      ▼
CRM OUTPUT
```

Every layer should ideally be traceable.

For any lead, it should be possible to answer:

```text
Where did this lead come from?

Which pipeline processed it?

Which features were used?

Which model version scored it?

What prediction was generated?

Was it successfully delivered downstream?
```

---

# 35. Recommended Lead-Level Audit Fields
A scored dataset should ideally maintain fields such as:

```text
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
```

These fields significantly improve debugging and reproducibility.

---

# 36. Model Versioning
The model used for scoring should be traceable.

For example:

```text
Model Version: v1
```

Later:

```text
Model Version: v2
```

The scoring output should ideally store the model version.

Example:

| Lead ID | Score | Model Version |
| ------- | ----: | ------------- |
| 123     |  0.82 | v1            |
| 456     |  0.91 | v2            |

This helps answer:

\> Which model generated this prediction?

---

# 37. Retraining vs Inference
It is important to distinguish between model training and daily scoring.

## Model Training
```text
Historical Data
      ↓
Feature Engineering
      ↓
Train Model
      ↓
Evaluate
      ↓
Deploy Model
```

This generally happens periodically.

---

## Model Inference
```text
New Lead
      ↓
Generate Features
      ↓
Load Existing Model
      ↓
Predict
      ↓
Store Score
```

This happens regularly as part of the production pipeline.

The daily lead scoring pipeline is primarily performing:

\> ****Inference, not model training.****

---

# 38. Responsibilities of the Lead Scoring System
The system is responsible for:

### Data Ingestion
Collect incoming leads.

### Data Preparation
Clean and standardize lead information.

### Feature Engineering
Create model-compatible features.

### Historical Enrichment
Add previous behavioral and CRM information.

### Prediction
Run the trained model.

### Output Delivery
Store and send the prediction to downstream systems.

### Monitoring
Ensure expected leads are processed.

### Recovery
Support retries and backfills.

---

# 39. Complete Conceptual Architecture
```text
                        ┌─────────────────┐
                        │   LEAD SOURCES  │
                        │                 │
                        │ Web CRM         │
                        │ CRM             │
                        │ Lead Forms      │
                        └────────┬────────┘
                                 │
                                 ▼
                    ┌────────────────────────┐
                    │     RAW DATA LAYER     │
                    │                        │
                    │     dms_raw_data       │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │     BASE DATA          │
                    │                        │
                    │ Cleaning               │
                    │ Validation             │
                    │ Standardization        │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │ HISTORICAL AGGREGATION │
                    │                        │
                    │ Behavioral Features    │
                    │ Historical Activity    │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │    STITCHED DATA       │
                    │                        │
                    │ Lead + History         │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │     ML INFERENCE       │
                    │                        │
                    │ Python / main.py       │
                    │ Trained ML Model       │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │    PREDICTION OUTPUT   │
                    │                        │
                    │ Score / Probability    │
                    │ Priority               │
                    └───────────┬────────────┘
                                │
                                ▼
                    ┌────────────────────────┐
                    │     DOWNSTREAM CRM     │
                    │                        │
                    │ Sales Prioritization   │
                    │ Marketing Actions      │
                    └────────────────────────┘
```

---

# 40. Key Takeaways
The Lead Scoring System is an end-to-end machine learning inference pipeline that transforms raw lead information into actionable business intelligence.

The complete lifecycle is:

```text
Lead Generated
      ↓
Lead Stored
      ↓
Data Cleaned
      ↓
Historical Data Added
      ↓
Features Created
      ↓
ML Model Runs
      ↓
Conversion Probability Generated
      ↓
Lead Prioritized
      ↓
Score Sent to CRM
```

The most important engineering principles for maintaining the system are:

1. ****Ensure source-to-output data lineage is traceable.****
2. ****Prevent duplicate lead processing.****
3. ****Maintain schema consistency during backfills.****
4. ****Use timestamps consistently.****
5. ****Prevent data leakage in historical features.****
6. ****Prefer explicit dependencies over time-based CRON assumptions.****
7. ****Make the pipeline idempotent.****
8. ****Track processing state for each lead.****
9. ****Monitor business outcomes, not just job success.****
10. ****Support reliable backfills and recovery.****
11. ****Track model versions used for predictions.****
12. ****Trace every scored lead back to its source and pipeline execution.****

---

# 41. One-Line Summary
\> ****The Lead Scoring System collects leads from multiple sources, enriches them with historical and behavioral information, prepares machine-learning features, uses a trained model to predict conversion likelihood, and sends the resulting score back to downstream business systems for lead prioritization and action.****

# Part II — Royal Enfield Production Implementation — By Tatvic

(This section documents exactly how the concepts in Part 1 are implemented for Royal Enfield's live production system, including real table names, known limitations, and fixes applied.)

42. RE Lead Scoring — Real Architecture Overview

Unlike the conceptual pipeline in Part 1, the RE system runs across three separate scheduled services, each independently triggered by Cloud Scheduler, connected only through shared BigQuery tables — not a single unified pipeline.

```text
CRM INGESTION                GA4 JOIN + SCORING           DMS DELIVERY
(daily-btr-data-             (New_30day_logic_           (scored-leads-backto-
 export-to-bq-v2)             real_time → model)           web-crm)
      │                             │                             │
      ▼                             ▼                             ▼
RE_web_crm_data_*  ────►  audience_l2_lead_score_*  ────►  dms_raw_data  ────►  DMS API
(every \~15 min)            (scoring, every \~15 min)         (push, every \~15 min)

Each stage reads from the previous stage's output table. There is no single orchestrator — reliability depends entirely on each service running on schedule and each query's filters correctly catching all eligible leads.

```
43. RE Data Sources — Real Table Names
Concept (Part 1 term)   Actual RE Table
Raw Lead Data     re-platform-model-dl.web_crm_data.RE_web_crm_data_* (date-sharded, one table per day)
Feature Dataset / Stitched Data     re-platform-model-dl.sept_test_ls.New_30day_logic_real_time (GA4 ↔ CRM join view)
Prediction Output re-platform-model-dl.ga4_ls_model_dataset.audience_l2_lead_score_* (date-sharded)
Routing/Delivery staging table      re-platform-model-dl.dms_logs.dms_raw_data
Dashboard/reporting snapshot  re-platform-model-dl.tvc_reports.leads_count_t-1 and re-platform-model-dl.dms_logs.lead_traking_dashboard

Key correction to Part 1's diagram: dms_raw_data is NOT the raw ingestion table for RE — that role belongs to RE_web_crm_data_*. dms_raw_data is actually the delivery/routing staging table, sitting just before the DMS API push. Worth keeping this straight, since the generic doc's diagram places dms_raw_data at "Raw Lead Data," which does not match RE's real architecture.

44. RE Identity Resolution — clientId & GA4 client_id

RE's identity resolution joins two systems on a single field:

CRM side: clientId — captured on the RE website at the moment a lead form is submitted (this is the GA4 browser client_id cookie value, passed through the form).
GA4 side: custom_client_id — a user property extracted from GA4 event data via:
```sql
(SELECT up.value.string_value FROM UNNEST(user_properties) up WHERE up.key = 'custom_client_id')

The join condition is:

```sql
ON a.client_id = b.clientid_crm AND a.event_date = b.date_crm

Critical limitation not covered in Part 1's generic identity resolution section: if clientId is blank at the CRM level (which happens routinely — confirmed across multiple production days, roughly 40–85% of any given day's dropped leads), there is no fallback identity resolution mechanism. The lead is unconditionally excluded from scoring at this stage — this is the single most common cause of "missing" leads in the RE pipeline.

45. RE GA4 Behavioral Data Window & Hostname Restriction

Unlike Part 1's generic "use available historical data" guidance, RE's GA4 join has two hard, specific constraints:

1. A strict 45-minute rolling window, reading only from ephemeral intraday data:

```sql
FROM `re-enterprise-dl.analytics_253617568.events_intraday_*`
WHERE TIMESTAMP_DIFF(CURRENT_TIMESTAMP(), event_timestamp, MINUTE) <= 45

There is no fallback to GA4's finalized (non-intraday) daily export tables. Once this 45-minute window passes without a match, that specific opportunity is gone — permanently, with no retry.

```
2. An exact-match hostname whitelist:

```sql
REGEXP_CONTAINS(device.web_info.hostname, "^www.royalenfield.com$|^finance.royalenfield.com$|^accessories.royalenfield.com$|^makeityours.royalenfield.com$")

Only these four subdomains are eligible. GA4 activity on any other RE-owned hostname is invisible to this join, regardless of lead validity.

```
46. RE Feature Engineering — Hardcoded Capping (Live Implementation)

RE's pre_processing() scoring script implements Part 1's "Feature Engineering" concept via hardcoded categorical whitelists, frozen at model training time (gcs_model_path="sept_model"). Any value outside these lists collapses into a generic bucket:

Feature     Capped to   Fallback bucket
source      8 known ad sources      'other sources'
medium      8 known mediums   'other_mediums'
browser     7 known browsers  'other_browsers'
bikemodel   \~19 named models (frozen at Sept)   'other_bk_models'
city  Fixed A-1/A/B-1/B-2 tier lists      '(Others)'

RE-specific risk not present in Part 1's generic guidance: any bike model launched after the model's training date (e.g., newer models like Bear 650 observed in raw CRM data) still gets scored — but silently miscategorized into 'other_bk_models', reducing prediction accuracy without any error or alert.

47. RE Model — Version & Threshold Logic
Model artifact: ls_sept_model_23.pkl, a CatBoost classifier, stored in GCS bucket kubeflow-ls-model.
Bucket assignment (RE's real implementation of Part 1's "High/Medium/Low" example):
```text
lead_score >= 0.57                          → bucket 1 (Hot, leadpriority 163650001)
lower_thresh <= lead_score < 0.57            → bucket 2 (Warm, leadpriority 163650002)
lead_score < lower_thresh                    → bucket 3 (Cold, leadpriority 163650003)

where lower_thresh/higher_thresh are recalculated per run from that batch's own score quantiles (0.3/0.7), unless explicit thresholds are passed.

Model versioning gap (relative to Part 1 Section 36's recommendation): the current pipeline does not store a model_version field anywhere in audience_l2_lead_score_* or dms_raw_data. There is no way, today, to trace which model version produced a given historical score — this is a real gap against Part 1's recommended audit fields.

```
48. RE Output & Delivery — Multiple Parallel Paths

Unlike Part 1's single "Routing & Delivery Layer," RE's DMS-push query (ga_leadscoring_data) has three separate branches feeding into dms_raw_data, only one of which reflects genuine model scoring:

pred_data — genuinely scored leads (crm INNER JOIN audience_l2_lead_score_*, pred_time < 16 min window).
remaining_leads_data — fallback for leads with a valid clientId that missed genuine scoring (pred_time = '', createdontime diff between 1–25 min).
fb_instant_lead_form — a Facebook-specific fast path, 20-minute window, bypasses the clientId requirement entirely (the only branch that does).

This means "scored" and "delivered to DMS" are not the same thing in RE's system. Any dashboard or report must distinguish pred_time != '' (genuinely scored) from pred_time = '' (fallback/unscored placeholder) to be accurate.

49. RE's Complete List of Leakage Gates (Confirmed via Production Debugging)
Gate  Stage Effect if failed
Blank clientId    CRM → GA4 join    Unconditional exclusion, every branch except Facebook fast-path
No matching GA4 session in 45-min window  GA4 join    Excluded from scoring candidacy entirely
GA4 session on non-whitelisted hostname   GA4 join    Excluded, even with valid clientId + timing
TIME()-only comparisons near midnight (23:45–00:15 IST)     Multiple queries  Silent exclusion from both scoring and fallback delivery due to date-rollover math error
remaining_leads_data's 24-min window vs. 15-min run cadence DMS delivery      Duplicate delivery (opposite failure — same lead pushed twice), confirmed present on every day audited (Aug 1–9)

Pincode was tested and confirmed NOT a leakage gate — it only affects which leadpriority sub-bucket a lead receives, never eligibility.

50. RE Known Bugs & Fixes Applied (Live Incident History)
Date found  Bug   Root cause  Status
Aug 2 45-min scoring gap → false "no leads scored" alert investigation  Two consecutive scoring cycles (05:32, 05:47) silently didn't execute, despite healthy upstream inputs      Root-caused; owning scheduler still needs identification
Aug 1–9 (ongoing) Duplicate rows in dms_raw_data (up to 586/day observed)     remaining_leads_data's 24-min window overlapping a 15-min run cadence, no dedup against existing dms_raw_data rows      Fix drafted (NOT IN exclusion clause), not yet deployed to production query
Aug 6 Dashboard (leads_count_t-1) showing stale count after backfill    Table is a one-time daily snapshot with no re-sync mechanism      Fixed via MERGE-based update query
Ongoing     email_alert_for_dms() crash (NameError: url)    url/headers variables commented out but requests.post() call left active      Fixed
51. RE's Recon/Backfill System — "The Sweeper"

To address the leakage gates in Section 49 without waiting for root-cause fixes to every upstream query, a dedicated daily reconciliation Cloud Function (daily-leads-reconciliation) was built:

```text
Runs once daily (\~01:00 AM IST)
      │
      ▼
Scans T-1 and T-0 CRM partitions
      │
      ▼
Finds leadIds NOT present in dms_raw_data (universal sweep, no filter conditions)
      │
      ▼
Pushes entire batch to DMS API in ONE request (matches production's proven batching pattern)
      │
      ▼
Logs successful pushes into dms_raw_data (pred_time = '', so they remain honestly flagged as unscored)

Important distinction from Part 1's Section 20–22 backfill guidance: this sweeper does not re-enter the normal scoring pipeline — it delivers directly to DMS with a generic leadpriority = 163650002 / leadinsights = 'Pitch for GMA', bypassing scoring entirely. This is a deliberate design choice (deliver-something over deliver-nothing), not a limitation to fix — but it means swept leads never receive genuine model-driven scores.

Verified via production data (Aug 7–9): zero duplicate leadId overlap across sweep days, confirming the sweeper's own dedup logic works correctly, independent of the separate remaining_leads_data bug in Section 50.

```
52. RE Monitoring & Alerting — Live Implementation

Three separate alert mechanisms exist, corresponding to Part 1 Section 31's guidance, each with its own known behavior:

Alert Trigger condition Known issue
CRM ingestion alert     0 rows in RE_web_crm_data_* in last 60 min      Prone to false positives during naturally low-traffic hours (confirmed early-morning near-misses)
DMS scoring alert 0 rows in dms_raw_data with pred_time != '' in last 16/30 min     Correctly caught the genuine Aug 2 scoring gap (true positive, root-caused)
Both alerts TIME()-only comparisons Same midnight-rollover bug as Section 49 — can misfire near 12:00 AM IST regardless of actual pipeline health
53. RE Dashboard & Reporting (Looker Studio + BigQuery Snapshots)

RE's live dashboard ([Royal Enfield] Lead Scoring Dashboard - GA4) is powered by daily-snapshot BigQuery tables, not live queries against dms_raw_data directly. This means:

Dashboard numbers reflect the state of dms_raw_data at the moment the snapshot query last ran, not real-time.
Any backfill or sweep that happens after that day's snapshot query executed will not be reflected until the snapshot table is manually or automatically re-synced.
Always use COUNT(DISTINCT leadId) in any reporting query touching dms_raw_data — confirmed necessary due to the Section 50 duplication bug; plain COUNT(*) will overstate numbers.
54. RE UAT Environment — Structurally Separate System

RE maintains a separate, unrelated UAT script (targeting api-uat2.royalenfield.com) for test-lead validation. This is worth documenting clearly because it is easy to mistake for a UAT mirror of the production pipeline — it is not:

Fetches leads via a direct fetch-btr-leads GET API call, not from RE_web_crm_data_*.
Uses hardcoded, rule-based priority logic (last character of leadId mapped to fixed ranges) — not the real ML model.
Has no connection to audience_l2_lead_score_*, dms_raw_data, or the sweeper.
Now also stores leads to a dedicated 30-day rolling-retention BigQuery table (uat_leads_hardcoded_priority), separate from production reporting tables, named explicitly to prevent confusion with genuinely-scored data.
55. RE Glossary — Internal Table & Service Names
Term  Meaning
RE_web_crm_data_* Daily-sharded raw CRM lead ingestion table
New_30day_logic_real_time     GA4 ↔ CRM identity-resolution and candidate-selection view
audience_l2_lead_score_*      Daily-sharded genuine model scoring output
dms_raw_data      Pre-DMS-delivery staging table (mix of genuinely scored + fallback leads)
daily-btr-data-export-to-bq-v2      CRM ingestion Cloud Run service
scored-leads-backto-web-crm   DMS-push Cloud Run service
crm_lead_scoring_alert_v1     Alerting Cloud Function
daily-leads-reconciliation    The sweeper/backfill Cloud Function
lead_traking_dashboard / leads_count_t-1  Dashboard-facing snapshot tables
pred_time = ''    Convention indicating a lead reached DMS via fallback/sweep, not genuine scoring
