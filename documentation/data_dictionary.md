```markdown
# Data Dictionary  
## IT Project Clearance & Service Performance Analytics

This document defines the tables, fields, relationships, business rules, calculated measures, and analytical logic used in the Power BI project.

> **Data Note:** All records in this project are fully synthetic and fictional. No real organisational, government, applicant, financial, or operational data is included.

---

## 1. Data Model Overview

The Power BI semantic model contains:

- 2 fact tables
- 4 dimension tables
- 1 dedicated measures table

### Fact Tables

- `FactApplications`
- `FactProcessStages`

### Dimension Tables

- `DimOrganisation`
- `DimProjectType`
- `DimApplicationStatus`
- `DimDate`

### Measures Table

- `_Measures`

---

## 2. FactApplications

### Purpose

`FactApplications` is the primary application-level fact table.

### Grain

**One row per IT project application.**

### Row Count

**1,000 applications**

| Column | Data Type | Key | Description |
|---|---|---|---|
| `ApplicationID` | Text | Primary Key | Unique identifier for each project application. |
| `OrganisationID` | Text | Foreign Key | Links the application to `DimOrganisation`. |
| `ProjectTypeID` | Text | Foreign Key | Links the application to `DimProjectType`. |
| `SubmissionDate` | Date | Date FK | Date the application was submitted. |
| `ReviewStartDate` | Date | Date FK | Date formal review of the application began. |
| `CompletionDate` | Date / Blank | Date FK | Date processing was completed. Blank for open applications. |
| `StatusID` | Text | Foreign Key | Links the application to `DimApplicationStatus`. |
| `ProjectValue` | Whole Number | — | Synthetic monetary value of the proposed IT project. |
| `ResubmissionCount` | Whole Number | — | Number of times the application was resubmitted. |
| `ClarificationRequired` | Boolean | — | Indicates whether additional clarification was requested. |
| `Priority` | Text | — | Application priority classification: Low, Medium, or High. |
| `RiskRating` | Text | — | Risk classification: Low, Medium, or High. |

### Business Rules

#### Open Application

```text
CompletionDate is blank
```

#### Completed Application

```text
CompletionDate is populated
```

#### Resubmitted Application

```text
ResubmissionCount > 0
```

#### Multiple Resubmission

```text
ResubmissionCount >= 2
```

#### Clarification Case

```text
ClarificationRequired = TRUE
```

---

## 3. FactProcessStages

### Purpose

`FactProcessStages` stores workflow-stage records for each application and enables process-bottleneck analysis.

### Grain

**One row per application per workflow-stage occurrence.**

### Row Count

**4,523 stage records**

| Column | Data Type | Key | Description |
|---|---|---|---|
| `StageRecordID` | Text | Primary Key | Unique identifier for each process-stage record. |
| `ApplicationID` | Text | Foreign Key | Links the stage record to `FactApplications`. |
| `Stage` | Text | — | Name of the workflow stage. |
| `StageStartDate` | Date | — | Date the application entered the stage. |
| `StageEndDate` | Date / Blank | — | Date the application exited the stage. Blank if currently in the stage. |
| `StageSequence` | Whole Number | — | Numeric sequence representing workflow order. |

### Workflow Stages

| Sequence | Stage |
|---:|---|
| 1 | Initial Screening |
| 2 | Technical Review |
| 3 | Financial / Cost Review |
| 4 | Final Assessment |
| 5 | Decision |

### Business Rules

#### Completed Stage

```text
StageEndDate is populated
```

#### Open Stage

```text
StageEndDate is blank
```

#### Stage Duration

```text
Stage Duration = StageEndDate - StageStartDate
```

Only completed stage records are used for historical stage-duration calculations.

---

## 4. DimOrganisation

### Purpose

Contains descriptive information for the fictional organisations submitting applications.

### Grain

**One row per organisation.**

### Row Count

**40 organisations**

| Column | Data Type | Key | Description |
|---|---|---|---|
| `OrganisationID` | Text | Primary Key | Unique organisation identifier. |
| `OrganisationName` | Text | — | Fictional organisation name. |
| `OrganisationType` | Text | — | Organisation classification such as Ministry, Agency, Authority, Commission, or Department. |
| `Sector` | Text | — | Sector classification. |
| `Region` | Text | — | Geographic regional classification. |
| `PerformanceBand` | Text | — | Synthetic generation aid describing intended organisation performance characteristics. |
| `PerformanceFactor` | Decimal | — | Synthetic factor used when generating processing-time behaviour. |

### Hidden Fields

The following fields are retained in the model but hidden from report consumers:

```text
PerformanceBand
PerformanceFactor
```

These fields were used to introduce realistic patterns into the synthetic dataset and are not intended to directly reveal analytical conclusions.

---

## 5. DimProjectType

### Purpose

Contains project-type classifications.

### Grain

**One row per project type.**

### Row Count

**7 project types**

| Column | Data Type | Key | Description |
|---|---|---|---|
| `ProjectTypeID` | Text | Primary Key | Unique identifier for the project type. |
| `ProjectType` | Text | — | Descriptive project-type name. |
| `Category` | Text | — | Higher-level project category. |
| `ComplexityFactor` | Decimal | — | Synthetic data-generation factor representing relative project complexity. |
| `ValueFactor` | Decimal | — | Synthetic data-generation factor influencing project-value distribution. |

### Project Types

- Software Development
- Network Infrastructure
- Cloud Services
- Cybersecurity
- Data & Analytics
- Hardware Procurement
- Digital Transformation

### Hidden Fields

```text
ComplexityFactor
ValueFactor
```

These are retained only to document how the synthetic data was generated.

---

## 6. DimApplicationStatus

### Purpose

Provides descriptive application-status information.

### Grain

**One row per application status.**

### Row Count

**6 statuses**

| Column | Data Type | Key | Description |
|---|---|---|---|
| `StatusID` | Text | Primary Key | Unique status identifier. |
| `Status` | Text | — | Application status description. |
| `StatusGroup` | Text | — | Higher-level grouping used to distinguish open and completed cases. |

### Status Values

| Status ID | Status | Status Group |
|---|---|---|
| `ST-01` | Approved | Completed |
| `ST-02` | Rejected | Completed |
| `ST-03` | Pending Review | Open |
| `ST-04` | Clarification Required | Open |
| `ST-05` | Resubmission Required | Open |
| `ST-06` | Withdrawn | Closed |

---

## 7. DimDate

### Purpose

Dedicated calendar dimension used for time intelligence and chronological filtering.

### Grain

**One row per calendar date.**

### Calendar Range

```text
1 January 2025 – 31 December 2026
```

| Column | Data Type | Description |
|---|---|---|
| `Date` | Date | Unique calendar date and primary date key. |
| `Year` | Whole Number | Calendar year. |
| `Month Number` | Whole Number | Month number from 1 to 12. |
| `Month Name` | Text | Full month name. |
| `Month Short` | Text | Three-character month name. |
| `Year Month` | Text | Year-month display field, e.g. `2025-06`. |
| `Year Month Sort` | Whole Number | Numeric year-month sort key, e.g. `202506`. |
| `Quarter` | Text | Calendar quarter such as Q1. |
| `Day` | Whole Number | Day of month. |
| `Day Name` | Text | Name of weekday. |
| `Weekday Number` | Whole Number | Monday-to-Sunday numeric sort field. |

### Date Sorting Rules

```text
Month Name  → Month Number
Month Short → Month Number
Year Month  → Year Month Sort
Day Name    → Weekday Number
```

The technical sort fields can be hidden from report users.

---

## 8. Model Relationships

The semantic model uses predominantly one-to-many relationships.

| From | To | Cardinality | Active | Filter Direction |
|---|---|---|---|---|
| `DimOrganisation[OrganisationID]` | `FactApplications[OrganisationID]` | 1:* | Yes | Single |
| `DimProjectType[ProjectTypeID]` | `FactApplications[ProjectTypeID]` | 1:* | Yes | Single |
| `DimApplicationStatus[StatusID]` | `FactApplications[StatusID]` | 1:* | Yes | Single |
| `FactApplications[ApplicationID]` | `FactProcessStages[ApplicationID]` | 1:* | Yes | Single |
| `DimDate[Date]` | `FactApplications[SubmissionDate]` | 1:* | Yes | Single |
| `DimDate[Date]` | `FactApplications[ReviewStartDate]` | 1:* | No | Single |
| `DimDate[Date]` | `FactApplications[CompletionDate]` | 1:* | No | Single |

---

## 9. SLA Definitions

### SLA Target

The dashboard uses a:

```text
20-calendar-day SLA
```

### Completed Application Processing Time

```text
Processing Time = CompletionDate - SubmissionDate
```

An application is considered within SLA when:

```text
Processing Days <= 20
```

An application is considered over SLA when:

```text
Processing Days > 20
```

### Open Application Reporting Date

Open-case ageing is calculated using a fixed synthetic reporting date:

```text
31 December 2025
```

This prevents the synthetic dataset from becoming artificially older as the real current date changes.

### Open Application Age

```text
Application Age = Reporting Date - SubmissionDate
```

### Approaching SLA

An open application is classified as approaching SLA when:

```text
15 <= Application Age <= 20 days
```

### Backlog Thresholds

The model also tracks:

```text
Open Applications 30+ Days
Open Applications 45+ Days
```

---

## 10. Core Application Measures

### Total Applications

Counts all application records.

```DAX
Total Applications =
COUNTROWS(FactApplications)
```

### Approved Applications

Counts applications with Approved status.

### Rejected Applications

Counts applications with Rejected status.

### Open Applications

Counts applications belonging to the `Open` status group.

### Approval Rate

```text
Approved Applications
÷
Total Applications
```

---

## 11. Project Value Measures

### Total Project Value

Sum of `FactApplications[ProjectValue]`.

### Approved Project Value

Total value of applications with Approved status.

### Average Project Value

Average project value across the current filter context.

---

## 12. Processing Measures

### Completed Applications

Number of applications with a populated `CompletionDate`.

### Average Processing Days

Average calendar days between submission and completion for completed applications.

### Median Processing Days

Median calendar days between submission and completion.

### Average Review Start Delay

Average number of days between:

```text
SubmissionDate
and
ReviewStartDate
```

---

## 13. SLA Measures

### Applications Within SLA

Completed applications where:

```text
Processing Days <= 20
```

### Applications Over SLA

Completed applications where:

```text
Processing Days > 20
```

### SLA Compliance %

```text
Applications Within SLA
÷
Completed Applications
```

### Open Applications Within SLA

Open applications aged 20 days or less as of the reporting date.

### Open Applications Over SLA

Open applications aged more than 20 days.

### Average Open Application Age

Average age of all currently open applications.

### Applications Approaching SLA

Open applications aged between 15 and 20 days.

### Open Applications 30+ Days

Open applications aged at least 30 days.

### Open Applications 45+ Days

Open applications aged at least 45 days.

---

## 14. Rework & Quality Measures

### Applications Resubmitted

Applications where:

```text
ResubmissionCount > 0
```

### Resubmission Rate

```text
Applications Resubmitted
÷
Total Applications
```

### Applications Requiring Clarification

Applications where:

```text
ClarificationRequired = TRUE
```

### Clarification Rate

```text
Applications Requiring Clarification
÷
Total Applications
```

---

## 15. First-Time-Right Measures

An application is defined as **First-Time-Right** when:

```text
ResubmissionCount = 0
AND
ClarificationRequired = FALSE
```

### First-Time-Right Applications

Count of applications meeting the First-Time-Right rule.

### First-Time-Right Rate

```text
First-Time-Right Applications
÷
Total Applications
```

---

## 16. Resubmission Severity Measures

### Applications With Multiple Resubmissions

Applications where:

```text
ResubmissionCount >= 2
```

### Multiple Resubmission Rate

```text
Applications With Multiple Resubmissions
÷
Total Applications
```

### Average Resubmissions

Average `ResubmissionCount` across all applications.

### Average Resubmissions When Resubmitted

Average resubmission count calculated only among applications with one or more resubmissions.

---

## 17. Defect Analysis

Three process defect opportunities are defined for every application:

1. Resubmission
2. Clarification requirement
3. SLA breach

### Defective Application

An application is classified as defective when **at least one** of the three defect conditions occurs.

### Defective Applications

Count of applications experiencing one or more defect conditions.

### Defect Rate

```text
Defective Applications
÷
Total Applications
```

### Defect Occurrences

Unlike `Defective Applications`, defect occurrences allow a single application to contribute more than one defect.

Example:

An application that:

- required resubmission,
- required clarification,
- breached SLA

would represent:

```text
1 defective application
but
3 defect occurrences
```

### Resubmission Defect Occurrences

Count of applications experiencing a resubmission defect.

### Clarification Defect Occurrences

Count of applications experiencing a clarification defect.

### SLA Defect Occurrences

Count of applications experiencing an SLA breach.

### Total Defect Occurrences

```text
Resubmission Defects
+
Clarification Defects
+
SLA Defects
```

---

## 18. DPMO

### Total Defect Opportunities

Because each application has three defined opportunities:

```text
Total Defect Opportunities =
Total Applications × 3
```

For 1,000 applications:

```text
1,000 × 3 = 3,000 opportunities
```

### Defects Per Opportunity

```text
Total Defect Occurrences
÷
Total Defect Opportunities
```

### DPMO

```text
Defects Per Opportunity
×
1,000,000
```

The synthetic portfolio produces approximately:

```text
337,000 DPMO
```

---

## 19. Process Bottleneck Measures

### Average Stage Duration

Average duration of completed stage records.

```text
StageEndDate - StageStartDate
```

### Median Stage Duration

Median duration of completed stage records.

### Total Stage Days

Cumulative duration of completed stage records.

### Stage Share of Total Time %

```text
Stage Total Days
÷
Total Days Across All Stages
```

### Completed Stage Records

Count of stage records with a populated end date.

### Open Stage Records

Count of stage records without an end date.

### Current Applications in Stage

Count of currently open stage records.

### Average Current Stage Age

Average age of active stage records as of:

```text
31 December 2025
```

---

## 20. Bottleneck Ranking

### Bottleneck Rank

Ranks stages by:

```text
Average Stage Duration
```

in descending order.

```text
Rank 1 = longest average stage duration
```

### Stage Workload Rank

Ranks stages by:

```text
Total Stage Days
```

in descending order.

```text
Rank 1 = greatest cumulative processing workload
```

---

## 21. Time-Intelligence Measures

### Applications Submitted

Application count based on the active `SubmissionDate` relationship.

### Applications Submitted Previous Month

Uses the previous calendar month.

### Applications MoM Change

```text
Current Month Applications
-
Previous Month Applications
```

### Applications MoM Change %

```text
MoM Change
÷
Previous Month Applications
```

### Applications Submitted YTD

Cumulative submissions from the beginning of the year.

---

## 22. Completion-Date Time Intelligence

Because `CompletionDate` is connected to `DimDate` through an inactive relationship, completion-date analysis uses:

```DAX
USERELATIONSHIP(
    DimDate[Date],
    FactApplications[CompletionDate]
)
```

Measures include:

- Applications Completed by Completion Date
- Applications Completed Previous Month
- Completions MoM Change %
- Applications Completed YTD

---

## 23. Review-Start Time Intelligence

Review-start analysis uses the inactive relationship between:

```text
DimDate[Date]
and
FactApplications[ReviewStartDate]
```

The `Reviews Started` measure activates this relationship using `USERELATIONSHIP()`.

---

## 24. Primary Synthetic Dataset Findings

At the default reporting context:

| Metric | Value |
|---|---:|
| Total Applications | 1,000 |
| Approved Applications | 727 |
| Approval Rate | 72.7% |
| Completed Applications | 811 |
| Average Processing Days | 24.6 |
| Median Processing Days | 22 |
| Applications Within SLA | 336 |
| Applications Over SLA | 475 |
| SLA Compliance % | 41.4% |
| Applications Resubmitted | 192 |
| Resubmission Rate | 19.2% |
| Clarification Cases | 165 |
| Clarification Rate | 16.5% |
| First-Time-Right Rate | 68.1% |
| Defective Applications | 725 |
| Defect Rate | 72.5% |
| DPMO | ~337K |

---

## 25. Process Bottleneck Finding

The primary process bottleneck in the synthetic portfolio is:

### Technical Review

| Metric | Approximate Value |
|---|---:|
| Average Stage Duration | 8.6 days |
| Median Stage Duration | 7 days |
| Total Stage Days | 7,757 |
| Share of Total Stage Time | 40.7% |
| Current Applications in Stage | 54 |
| Bottleneck Rank | 1 |
| Stage Workload Rank | 1 |

The result is supported by multiple independent indicators rather than stage duration alone.

---

## 26. Reporting Filters

The report uses synchronized slicers for:

- Year
- Sector
- Project Type
- Risk Rating
- Priority

### Default Report State

```text
Year = 2025
Sector = All
Project Type = All
Risk Rating = All
Priority = All
```

---

## 27. Hidden Technical Fields

The following fields are retained for model functionality but hidden from normal report users where appropriate:

```text
FactApplications[OrganisationID]
FactApplications[ProjectTypeID]
FactApplications[StatusID]

FactProcessStages[StageRecordID]

DimOrganisation[PerformanceBand]
DimOrganisation[PerformanceFactor]

DimProjectType[ComplexityFactor]
DimProjectType[ValueFactor]

DimDate[Month Number]
DimDate[Year Month Sort]
DimDate[Weekday Number]
```

These fields support relationships, sorting, or synthetic-data generation but are not required for report consumption.

---

## 28. Data Limitations

This dataset is intentionally synthetic and is intended for:

- portfolio demonstration
- Power BI modelling practice
- process analytics
- SLA analysis
- DAX development
- dashboard-design practice

It should not be interpreted as representing the performance of any real organisation.

Additional limitations include:

- SLA is measured in calendar days rather than working days.
- No public-holiday calendar is currently implemented.
- Open-case age uses a fixed reporting date.
- Project values are fictional.
- Stage durations are synthetic.
- Organisation-performance patterns were intentionally introduced during data generation.

---

## 29. Future Data Model Enhancements

Potential extensions include:

- Business-day calendar
- Public-holiday dimension
- Reviewer/team dimension
- Application category dimension
- SLA target dimension
- Dynamic SLA parameter
- Dynamic reporting-date parameter
- Stage-owner information
- Rework reason codes
- Clarification reason codes
- Application event history
- Automated refresh metadata
- Row-level security mapping

---

## 30. Disclaimer

This data dictionary documents a portfolio and learning project built entirely with synthetic data.

All organisations, application records, financial values, workflow stages, performance patterns, and quality outcomes are fictional and are not intended to represent any real organisation or operational process.