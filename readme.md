The Ultimate Guide to Lead Scoring: Models, Architecture, and Implementation
[PAGE 1] Introduction & Fundamentals of Lead Scoring
The Core Problem
In the modern business landscape, marketing teams are exceptionally good at generating leads. However, not all leads are created equal. When marketing hands over a massive, unfiltered list of leads to the sales team, it creates friction. Sales representatives waste valuable time chasing "window shoppers," leading to frustration, missed quotas, and a lack of trust between the marketing and sales departments.

What is Lead Scoring?
Lead scoring is a systematic, data-driven methodology used to rank prospects against a scale that represents the perceived value each lead represents to the organization. By assigning numerical values (points) to various data points and user behaviors, a lead scoring system objectively determines a lead's readiness to buy.

Primary Objectives
Prioritization: Ensure sales teams focus 100% of their effort on high-intent, sales-ready prospects.

Efficiency: Reduce the time spent on unqualified leads.

Alignment: Establish a quantifiable agreement between marketing and sales on what constitutes a "good" lead.

Increased Conversion: Engage prospects at the exact right time in their buying journey.

The Lead Lifecycle Stages
A lead scoring model is built to move prospects smoothly through a defined funnel:

Subscriber/Cold Lead: Individuals who have opted in to hear from you but have a low score. They require automated marketing nurturing.

Marketing Qualified Lead (MQL): Leads who have crossed a minimum engagement threshold. They fit the target buyer persona but are not yet ready for a sales pitch.

Sales Qualified Lead (SQL): Leads who have crossed the high-intent threshold (e.g., requested a demo, or scored above 80/100). These are immediately routed to a human sales representative.

[PAGE 2] Types of Lead Scoring Models
To build a robust system, organizations typically choose between two primary models, or a hybrid of both, depending on their data maturity.

1. Traditional / Rule-Based Scoring (Heuristic)
This is the foundational approach where marketing and sales teams collaborate to manually assign point values to specific actions and demographic traits based on their business experience.

Mechanism:

A lead receives +15 points for a "VP of Sales" job title.

A lead receives +10 points for downloading a pricing guide.

A lead receives -20 points for listing their industry as "Student."

Pros of Rule-Based Scoring:

Highly transparent; it is easy to explain why a lead scored an 85.

Simple to set up in traditional CRM or Marketing Automation platforms.

Requires no advanced data science or Machine Learning capabilities.

Cons of Rule-Based Scoring:

Relies heavily on human intuition, which can be subjective or biased.

Static in nature; requires continuous manual tweaking as market behaviors change.

Struggles to process thousands of variables simultaneously.

2. Predictive Lead Scoring (Machine Learning)
Predictive scoring removes human guesswork. It utilizes historical data and Machine Learning (ML) algorithms to automatically identify hidden patterns that lead to conversions.

Mechanism:

The ML model ingests historical CRM data, analyzing thousands of "Closed-Won" and "Closed-Lost" deals.

It trains algorithms (such as Logistic Regression, Random Forests, or XGBoost) to assign a conversion probability score (0% to 100%) to every new lead in real-time.

Pros of Predictive Scoring:

Completely objective and data-driven.

Continuously learns and adapts to new market trends without manual intervention.

Capable of handling complex, non-linear relationships between hundreds of data points.

Cons of Predictive Scoring:

Requires a massive volume of clean, historical data to train the model accurately.

Can sometimes act as a "black box," making it hard for sales teams to understand why a lead scored highly.

Requires technical expertise to deploy and maintain.

[PAGE 3] Data Pillars & Feature Engineering
A scoring model is only as good as the data fed into it. Feature engineering involves selecting the right data points across three primary pillars to build a comprehensive profile of the prospect.

Pillar 1: Explicit Data (Demographics & Firmographics)
This is the information the lead provides directly, or data enriched via third-party tools. It answers the question: Is this person a good fit for our product?

Job Title / Seniority: Decision-makers score higher than interns.

Company Size & Revenue: Enterprises score higher for high-ticket B2B products.

Industry / Vertical: Alignment with your ideal customer profile (ICP).

Geographic Location: Relevance to your serviceable areas.

Pillar 2: Implicit Data (Behavioral Engagement)
This data is gathered by tracking the prospect's digital body language. It answers the question: Is this person showing active buying intent?

Website Activity: High scores for visiting the "Pricing" or "Book a Demo" pages.

Content Engagement: Points for downloading whitepapers, case studies, or attending webinars.

Email Interaction: Points for high open rates and click-through rates.

Recency: Engagement from the last 24 hours scores exponentially higher than engagement from 6 months ago.

Pillar 3: Negative Scoring & Degradation
Equally important is identifying who not to sell to. Negative scoring subtracts points to prevent sales from wasting time.

Disqualification Cues: Visiting the "Careers" page or using a disposable/student email address.

Score Degradation (Time Decay): If a lead was highly active 90 days ago but has since gone completely silent, their score must automatically degrade over time to reflect a loss of interest.

[PAGE 4] System Architecture & Data Pipeline Flow
To automate lead scoring at scale, a robust Data Engineering architecture must be implemented. This ensures leads are scored and routed to sales in near real-time.

The End-to-End Pipeline
Data Ingestion Layer: Leads enter the ecosystem via various sources (Website Forms, CRM APIs, Social Media Lead Ads, or Manual Uploads).

Enrichment Layer: The raw lead data is passed through enrichment APIs (e.g., Clearbit, ZoomInfo) to automatically fill in missing firmographic details (like company revenue or industry).

Processing & Scoring Engine: The enriched payload triggers a cloud function or microservice. The rules engine or ML model processes the data and calculates the final Lead Score and Priority Category.

Routing & Delivery Layer: High-scoring leads (SQLs) are immediately pushed via API to the CRM or Dealer Management System (DMS) for instant sales action. Low-scoring leads are routed to an email nurturing tool.

Audit & Event Logging (Data Warehouse): Every single transaction is recorded in a Data Warehouse (e.g., Google BigQuery, Snowflake) using an Event Sourcing model.

The Importance of Pipeline Auditing
A best-in-class pipeline tracks the exact state of every lead. Instead of overwriting data, the system appends records with exact timestamps and delivery statuses.

Incoming Logs: Records the exact time a lead was fetched or received.

Delivery Status: Logs a simple "Yes/No" or "Success/Failed" tag indicating if the lead successfully reached the sales system.

This architecture allows data engineers to instantly query system health, identify API failures, and track end-to-end latency without disturbing the live production database.

[PAGE 5] Implementation Strategy & Business Impact
How to Launch a Lead Scoring System
Implementing lead scoring should not be an overnight switch; it requires a phased approach.

Start Simple: Begin with a basic rule-based model focusing on 5 to 10 highly predictive data points.

Define the SLA (Service Level Agreement): Marketing and Sales must agree on the exact threshold that defines an MQL and an SQL. Sales must commit to contacting SQLs within a specific timeframe (e.g., under 2 hours).

Test and Validate: Run the scoring model in the background against historical data to see if it accurately predicts past wins and losses.

Iterate and Evolve: Once the rule-based model proves successful and data matures, begin training a predictive Machine Learning model to take over the heavy lifting.

The Feedback Loop
A scoring model is never "finished." It requires a continuous feedback loop from the sales team. If Sales is consistently rejecting leads that scored 90+, the model's parameters must be adjusted. Similarly, if Closed-Won deals are emerging from leads scoring 40, the model is missing key indicators.

Key Performance Indicators (KPIs) for Success
To measure the ROI of a lead scoring implementation, track the following metrics:

Sales Acceptance Rate: The percentage of marketing leads that sales accepts and works on.

Time-to-Close: A successful model should significantly reduce the length of the sales cycle.

Win Rate: The percentage of SQLs that convert into paying customers.

Cost of Customer Acquisition (CAC): Targeted outreach should lower the overall cost of acquiring a new customer.

Conclusion
Lead scoring transforms a reactive sales floor into a proactive, data-driven revenue engine. By strategically utilizing explicit demographics, implicit behaviors, and a robust engineering pipeline, organizations can ensure that their most valuable resource—sales time—is spent exclusively on prospects ready to buy.

Bhai, yeh document copy-paste ke liye ekdum ready hai! Isme business logic se lekar technical architecture aur KPIs tak sab kuch smoothly cover ho gaya hai. Aap isko as a standard template apne kisi bhi presentation ya documentation mein confidently use kar sakte hain.




# Lead Scoring System

## Functional, Data Flow, and Technical Documentation

---

## 1. Overview

The **Lead Scoring System** is a machine learning–driven pipeline designed to identify and prioritize leads based on their likelihood of conversion.

Instead of treating every incoming lead equally, the system analyzes available customer, CRM, website, and behavioral information and assigns a score or prediction indicating the likelihood that a lead will perform the target business action.

The overall objective is:

> **Capture leads → enrich them with historical and behavioral information → generate predictions → make the scored leads available to downstream CRM/business systems.**

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

> "Which lead is good?"

Instead, the machine learning problem is:

> **"Based on the information available for this lead, what is the probability that this lead will convert?"**

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

The **Base Data** layer prepares the fundamental dataset required for further processing.

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

The **Stitched Data** layer combines information from different sources.

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

The pipeline converts raw information into **features**.

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

One important engineering principle in the pipeline is avoiding **data leakage**.

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

> Historical features should only use information available before the prediction time.

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

> A backfilled lead should ideally enter the pipeline through the same logical path as a normal production lead.

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

A production scoring pipeline should ideally be **idempotent**.

This means:

> Running the same pipeline multiple times should not create unintended duplicate results.

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

> Start from where the failure is visible and trace the data backward through the pipeline.

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

> Which model generated this prediction?

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

> **Inference, not model training.**

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

1. **Ensure source-to-output data lineage is traceable.**
2. **Prevent duplicate lead processing.**
3. **Maintain schema consistency during backfills.**
4. **Use timestamps consistently.**
5. **Prevent data leakage in historical features.**
6. **Prefer explicit dependencies over time-based CRON assumptions.**
7. **Make the pipeline idempotent.**
8. **Track processing state for each lead.**
9. **Monitor business outcomes, not just job success.**
10. **Support reliable backfills and recovery.**
11. **Track model versions used for predictions.**
12. **Trace every scored lead back to its source and pipeline execution.**

---

# 41. One-Line Summary

> **The Lead Scoring System collects leads from multiple sources, enriches them with historical and behavioral information, prepares machine-learning features, uses a trained model to predict conversion likelihood, and sends the resulting score back to downstream business systems for lead prioritization and action.**
