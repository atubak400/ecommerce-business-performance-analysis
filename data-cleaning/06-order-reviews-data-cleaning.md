# 06 — Order Reviews Data Cleaning

## 1. Dataset Overview

**Source table:** `olist_order_reviews_dataset`

**Project:** E-Commerce Business Performance Analysis

**Tool:** Power BI — Power Query Editor

**Original records:** 99,224  
**Final records:** 99,224  
**Total columns:** 7  
**Records removed:** 0

### Business Purpose

The Order Reviews dataset contains customer satisfaction ratings, optional written feedback, and review timestamps associated with customer orders.

This dataset supports the Customer Satisfaction analysis by enabling the business to:

- Monitor customer satisfaction through review scores.
- Analyse the distribution of positive and negative ratings.
- Investigate customer feedback patterns.
- Explore relationships between customer satisfaction, order fulfilment, and delivery performance.
- Identify potential service improvement opportunities.

## 2. Data Structure

| Column | Data Type | Description |
|---|---|---|
| `review_id` | Text | Identifier associated with a customer review |
| `order_id` | Text | Identifier of the associated customer order |
| `review_score` | Whole Number | Customer satisfaction rating from 1 to 5 |
| `review_comment_title` | Text | Optional title of the customer's review |
| `review_comment_message` | Text | Optional written customer feedback |
| `review_creation_date` | Date/Time | Date the review was created |
| `review_answer_timestamp` | Date/Time | Timestamp when the review was answered |

## 3. Data Quality Assessment

### 3.1 Review Identifier Validation

The dataset contains:

- **99,224 review records**
- **98,410 distinct review IDs**
- **789 review IDs appearing more than once**

Further investigation established that:

- 764 repeated review IDs were associated with two distinct orders each.
- 25 repeated review IDs were associated with three distinct orders each.

The repeated identifiers were investigated rather than automatically classified as duplicate records.

**Decision:** Retain all records because repeated review IDs were associated with different orders.

### 3.2 Repeated Review Attribute Consistency

The 789 repeated review IDs were assessed for inconsistencies across five attributes.

| Attribute | Review IDs with conflicting values |
|---|---:|
| `review_score` | 0 |
| `review_comment_title` | 0 |
| `review_comment_message` | 0 |
| `review_creation_date` | 0 |
| `review_answer_timestamp` | 0 |

All five attributes remained consistent within each repeated review ID.

**Finding:** The repeated identifiers carried consistent review information, despite being associated with multiple orders.

**Decision:** Retain the records and account for repeated identifiers when designing analytical measures.

### 3.3 Review–Order Combination Validation

The combination of `review_id` and `order_id` was examined using Power Query's Group By operation.

Each combination appeared exactly once.

| Validation | Result |
|---|---:|
| Duplicate review–order combinations | 0 |
| Unique review–order combinations | 99,224 |

**Finding:** The combination of `review_id` and `order_id` uniquely identifies each record in the dataset.

**Decision:** No duplicate combinations were removed.

### 3.4 Review Score Validation

The `review_score` column was assessed for invalid, missing, and out-of-range values.

| Check | Result |
|---|---|
| Minimum score | 1 |
| Maximum score | 5 |
| Distinct score values | 5 |
| Missing scores | 0 |
| Invalid scores | 0 |

All review scores fell within the expected 1–5 rating scale.

**Decision:** Retain all review scores without modification.

### 3.5 Missing Review Comments

Column profiling identified substantial missing values in the optional written-feedback fields.

| Column | Approximate missing percentage |
|---|---:|
| `review_comment_title` | 88% |
| `review_comment_message` | 59% |

These fields are optional, and customers can provide numerical ratings without submitting written comments.

The missing values therefore do not automatically indicate invalid review records.

**Decision:**

- Retain missing comment titles and messages.
- Do not generate artificial review text.
- Preserve the original information for subsequent analysis.

### 3.6 Review Timestamp Validation

The following columns were validated:

- `review_creation_date`
- `review_answer_timestamp`

A temporary Power Query validation column checked whether the answer timestamp occurred before the creation date.

**Validation rule:**

```powerquery
if [review_answer_timestamp] < [review_creation_date]
then "Invalid"
else "Valid"
```

| Validation | Result |
|---|---:|
| Records checked | 99,224 |
| Valid timestamp sequences | 99,224 |
| Invalid timestamp sequences | 0 |
| Missing creation dates | 0 |
| Missing answer timestamps | 0 |

**Finding:** Every review answer timestamp was equal to or later than its creation date.

**Decision:** No timestamp corrections were required.

### 3.7 Referential Integrity with Orders

The Reviews dataset was validated against `olist_orders_dataset` using `order_id`.

A **Left Anti Join** was performed to identify review records without a matching order.

| Validation | Result |
|---|---:|
| Review records checked | 99,224 |
| Records with matching orders | 99,224 |
| Records without matching orders | 0 |

**Finding:** All review records referenced existing orders in the Orders dataset.

**Decision:** Retain all records. No unmatched order references required correction.

## 4. Cleaning and Validation Approach

Data-quality investigations were conducted using temporary Power Query reference queries.

The process included:

1. Profiling the original dataset across all records.
2. Identifying repeated review IDs.
3. Investigating the number of distinct orders associated with repeated review IDs.
4. Checking consistency of review scores, comments, and timestamps.
5. Validating the uniqueness of review–order combinations.
6. Checking review scores against the expected rating scale.
7. Assessing missing optional comment fields.
8. Validating chronological consistency between review timestamps.
9. Verifying order references against the Orders dataset.
10. Removing temporary investigation queries after completing validation.

The original Reviews dataset was preserved throughout the investigation.

## 5. Final Data Cleaning Decisions

| Finding | Action Taken | Justification |
|---|---|---|
| Repeated review IDs | Retained | Associated with distinct orders |
| Consistent repeated review attributes | No modification | No conflicting values identified |
| Duplicate review–order combinations | None removed | No duplicate combinations found |
| Valid review scores | Retained | All scores within the expected range |
| Missing comment titles | Retained | Optional customer feedback |
| Missing comment messages | Retained | Optional customer feedback |
| Invalid timestamp sequences | None corrected | No invalid sequences identified |
| Unmatched order references | None removed | All reviews matched existing orders |

**Final dataset size: 99,224 records and 7 columns.**

No source records were removed or modified during the assessment.

## 6. Implications for Power BI Data Modelling

The findings have several implications for the Customer Satisfaction report.

### 6.1 Review Counts

Because `review_id` is not unique, counting rows and counting distinct review IDs will produce different results.

- **Review records:** 99,224
- **Distinct review IDs:** 98,410

Measures must clearly distinguish between review records, unique review identifiers, and reviewed orders.

### 6.2 Customer Satisfaction Metrics

Review scores can support analysis of:

- Average review score.
- Distribution of ratings from 1 to 5.
- Percentage of low-rated reviews.
- Satisfaction trends over time.
- Relationships between review scores and delivery performance.

The analytical grain must be considered when aggregating reviews, particularly where a review ID is associated with multiple orders.

### 6.3 Written Feedback Analysis

Missing written comments should be distinguished from numerical ratings.

For example, the business may analyse:

- Reviews containing written feedback.
- Reviews containing only numerical ratings.
- Common themes in positive and negative written feedback.

Missing comments should not automatically be interpreted as negative customer sentiment.

### 6.4 Data Model Relationships

All review order IDs match existing order IDs, supporting the creation of relationships between the Reviews and Orders tables.

However, the Orders dataset contains unique order IDs, while the Reviews dataset can contain multiple records associated with an order.

Relationship cardinality and filter direction should therefore be determined from the actual table structure during semantic model development.

## 7. Final Quality Assessment

The Reviews dataset passed the key validation checks performed during this assessment.

**Key findings:**

- All review scores were valid.
- No conflicting review attributes were identified among repeated review IDs.
- Every review–order combination was unique.
- No invalid review timestamp sequences were found.
- Every review record matched an existing order.
- Missing written feedback was retained because it is optional.
- No records required removal.

The assessment also identified an important modelling consideration: repeated review IDs must be handled carefully to prevent misleading review counts or customer satisfaction metrics.

## 8. Verification and Completion

After the investigations:

- All temporary validation queries were removed.
- The six original source queries were retained.
- Changes were applied using **Close & Apply**.
- The Power BI project was saved.
- The PBIX file was closed and reopened.
- The original Reviews dataset was verified with all seven columns intact.

**Final status: Data-quality assessment completed.**

The dataset is ready for the next stage of Power BI data modelling, subject to appropriate relationship design and measure definitions.