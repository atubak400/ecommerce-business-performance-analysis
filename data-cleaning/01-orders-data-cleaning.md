# Olist E-Commerce Analysis — Orders Dataset Cleaning and Quality Report

**Project:** Olist E-Commerce Analysis  
**Working Power BI file:** `Olist E-Commerce Analysis.pbix`  
**Source:** Brazilian Olist E-Commerce public dataset  
**Dataset covered:** `olist_orders_dataset.csv`  
**Power Query query name:** `olist_orders_dataset`  
**Review date:** 11 October 2026  
**Stage:** Data cleaning and quality assessment — first of nine source tables  
**Assessment:** Ready to continue to the next source table, with a documented date-sequence anomaly and some checks deferred to modelling/analysis.

## 1. Business context and project objective

The Olist project aims to help management understand sales, customers, delivery performance, and customer satisfaction. The intended report areas are:

1. Sales Overview
2. Customer Analysis
3. Delivery Performance
4. Customer Satisfaction

The project uses nine related CSV files: orders, order items, customers, products, payments, reviews, sellers, geolocation, and product category translation. This report covers **only the Orders dataset**, not the whole project's cleaning stage.

## 2. Why the Orders dataset was chosen first

The choice was driven by the **business process and table relationships**, not by an assumption that Orders must be the final fact table.

- **Order lifecycle:** Orders records when an order was purchased, approved, passed to a carrier, delivered, and expected to arrive. These fields establish the business process behind delivery analysis.
- **Connecting identifiers:** `order_id` connects to order items, payments, and reviews; `customer_id` connects to customers. Understanding these keys first helps prevent incorrect joins later.
- **Clear grain:** The table is intended to have **one row per order**. This is different from order items, where one order can have multiple lines. Confirming this distinction helps avoid double-counting.
- **Direct business value:** Order statuses and timestamps support counts of placed, delivered, and cancelled orders, as well as delivery timeliness and duration measures.
- **Manageable first cleaning exercise:** Eight columns allowed systematic practice with headers, data types, profiling, missing values, duplicate keys, and date validation before moving to larger or more complex tables.

**Important:** Choosing Orders first does **not** determine the final semantic-model design. The fact/dimension structure will be decided after reviewing all relevant tables. The Orders table also lacks item prices, so revenue analysis will require other datasets.

## 3. Source table structure and meaning

The source Orders dataset contains **99,441 rows and eight original columns**.

| Column | Business meaning | Reviewed data type |
|---|---|---|
| `order_id` | Identifier for an order | Text |
| `customer_id` | Customer record identifier associated with the order | Text |
| `order_status` | Current recorded order status | Text |
| `order_purchase_timestamp` | When the order was placed | Date/Time |
| `order_approved_at` | When the order was approved | Date/Time |
| `order_delivered_carrier_date` | When the order was handed to the carrier | Date/Time |
| `order_delivered_customer_date` | When the order was delivered to the customer | Date/Time |
| `order_estimated_delivery_date` | Estimated customer delivery date | Date/Time |

The carrier date means **handover to the carrier**, not delivery by the carrier to the customer. `customer_id` should not automatically be interpreted as a unique person across all orders; that distinction will be investigated using the Customers table and `customer_unique_id`.

## 4. Work completed in Power Query, in order

### 4.1 Import and header correction

- Imported `olist_orders_dataset.csv` into Power Query Editor.
- Used **Transform → Use First Row as Headers** to promote the first CSV row to column names.
- Confirmed that the original dataset has eight meaningful columns.

**Why:** Without correct headers, subsequent data-type changes and modelling would be confusing and error-prone.

### 4.2 Date/time type correction with locale

- Inspected the five timestamp/date columns.
- Converted all five to **Date/Time** using **Change Type → Using Locale → Date/Time → English (United Kingdom)**.
- Retained the type-conversion steps in Applied Steps.

**Why:** The CSV dates were displayed in a day/month/year format; locale-aware parsing avoids incorrectly interpreting day and month. Date/Time (rather than Date only) preserves hours, minutes, and seconds needed for duration calculations.

### 4.3 Whole-dataset profiling

- Enabled Power Query column profiling/quality/distribution features.
- Changed the profiling basis from the first 1,000 rows to the **entire dataset**.
- Examined validity, errors, empty values, distinct counts, and unique counts.

**Why:** Profiling only the first 1,000 rows can miss rare problems. The full dataset was needed to detect anomalies affecting very few records.

### 4.4 Order-status inspection

Identified eight distinct status labels:

- `delivered`
- `shipped`
- `canceled`
- `unavailable`
- `invoiced`
- `processing`
- `created`
- `approved`

Observed that delivered orders dominate. Exact frequencies were not recorded in this exercise. The learner initially reported no `created` or `approved` orders, but those labels appeared in the eight distinct values; this may mean they are rare, so **do not claim zero records without checking their counts**.

**Spelling note:** `canceled` is standard US English. It is not a source-data error; retain the original status, and optionally display `Cancelled` in a UK-facing report later.

### 4.5 Missing customer-delivery-date investigation

- Filtered `order_delivered_customer_date` to `(null)`.
- Examined `order_status` for the filtered records.
- Found the following statuses among records with no customer-delivery timestamp: `invoiced`, `shipped`, `processing`, `unavailable`, and `canceled`.
- Observed that `shipped` was the most common among the filtered records; **the exact count was not recorded**.
- Found **no `delivered` orders** with a missing customer-delivery timestamp.
- Removed the temporary filter after the investigation.

**Interpretation:** These missing dates are consistent with orders that have not reached the delivered stage. They are not sufficient evidence of defective records. They were **left as null**, rather than filled with an invented date or deleted.

**Scope:** This exercise investigated missing **customer delivery dates** by status. It did not complete a status-by-status investigation of every other nullable date column.

### 4.6 Duplicate-order-ID investigation

- Selected `order_id` with profiling based on the full dataset.
- Observed **99,441 distinct values** and **99,441 unique values**.
- Concluded that no `order_id` repeats in this Orders table.
- Did **not** use Remove Duplicates.

**Why:** One row should represent one order. The result supports `order_id` as a unique key in this table. Duplicate `order_id` values may be expected in the separate Order Items table, because a single order can contain multiple items.

### 4.7 Purchase-to-customer-delivery chronology check

Created a **temporary** Power Query custom column `Delivery_Date_Check`:

```powerquery
if [order_delivered_customer_date] = null then "Not Delivered"
else if [order_delivered_customer_date] < [order_purchase_timestamp] then "Invalid"
else "Valid"
```

- Changed the custom column's data type from Any to **Text**, so value distribution could be calculated.
- Profiled the full dataset.
- Found **0 records labelled `Invalid`**.
- Removed the temporary custom column after reviewing it.

**Interpretation:** No recorded customer delivery occurred before its purchase timestamp, according to this check. Rows without customer-delivery dates were classified separately as `Not Delivered`.

### 4.8 Carrier-to-customer chronology check

Created a second **temporary** custom column `Carrier_Date_Check`:

```powerquery
if [order_delivered_carrier_date] = null
   or [order_delivered_customer_date] = null
then "Cannot Compare"
else if [order_delivered_customer_date] < [order_delivered_carrier_date]
then "Invalid"
else "Valid"
```

- Changed the column type to **Text**.
- Used whole-dataset value distribution.
- Filtered the results to `Invalid`.
- Found **23 flagged orders**, all showing status `delivered` in the filtered screenshot.
- Confirmed the filtered view showed 23 rows.
- Removed the temporary `Filtered Rows`, `Changed Type`, and `Added Custom` steps, restoring the original eight columns.

**Rate:** 23 / 99,441 ≈ **0.0231%** of orders.

**What was detected:** The recorded customer-delivery timestamp was **earlier** than the recorded carrier-handover timestamp. This is a date-sequence anomaly, not proof that the physical delivery happened in the wrong order.

**Illustrative observed record:** A screenshot showed an order purchased on 20 July 2017, with customer delivery on 25 July 2017 and carrier handover on 28 July 2017. This is internally inconsistent for a normal handover-to-delivery sequence.

**Possible explanations, not established facts:** late carrier-event updates, incorrectly entered timestamps, inconsistent event definitions, or source-system synchronization problems. No cause has been verified.

### 4.9 Identifier type and completeness confirmation

- Confirmed `order_id` is **Text**.
- Confirmed `customer_id` is **Text**.
- Found **no nulls or errors** in either identifier column.

**Why:** IDs are labels, not quantities; they should not be treated as numbers. Missing IDs would jeopardize joins and counts.

## 5. Summary of data-quality findings

| Check | Finding | Decision |
|---|---|---|
| Headers | Correctly promoted | Keep |
| Five date/time fields | Parsed as Date/Time using UK locale | Keep conversions |
| Full-dataset profiling | 99,441 orders profiled | Use full-data profiling for checks |
| Order statuses | Eight distinct labels | Preserve original labels |
| Missing customer delivery date | Present only in non-delivered statuses found in investigation | Keep nulls |
| `order_id` duplicates | 0; 99,441 distinct and 99,441 unique | No deduplication |
| Delivery before purchase | 0 flagged | No correction |
| Customer delivery before carrier handover | **23 flagged** | Retain; document; treat selectively in analysis |
| Identifier data types | Both Text | Keep |
| Identifier nulls/errors | None observed | No correction |

The original date columns had approximate missing-value rates in Power Query of **~1% for approval**, **~2% for carrier handover**, and **~3% for customer delivery**. These are rounded display percentages, not audited exact counts.

## 6. Formal data-quality issue log

### DQ-ORD-001 — Customer delivery timestamp precedes carrier handover

| Field | Entry |
|---|---|
| Issue ID | `DQ-ORD-001` |
| Table | `olist_orders_dataset` |
| Rule | Where both timestamps exist, `order_delivered_customer_date` should normally be on or after `order_delivered_carrier_date` |
| Detection | Temporary `Carrier_Date_Check` custom column in Power Query, filtered to `Invalid` |
| Count | **23** |
| Denominator | **99,441 orders** |
| Proportion | **~0.0231%** |
| Status | Identified and documented; cause unresolved |
| Source data action | **No rows deleted; no timestamps overwritten** |
| Risk | Negative or misleading carrier-to-customer transit durations; potentially distorted duration averages |
| Proposed analysis handling | Exclude these orders **only from metrics that require a valid carrier-to-customer interval**; retain them for unaffected metrics |
| Evidence | Power Query screenshots showing the full distribution and the filtered 23 invalid rows |
| Follow-up | Recreate the condition or a reusable quality flag when building delivery-performance measures; confirm metric-specific exclusions and document them |

#### Evidence and reproducibility

The screenshot showing the filtered 23 rows is valuable supporting evidence, but **the screenshot alone is not a complete audit trail**. This written issue log records the rule, the observed count, the interpretation, and the decision.

If the screenshots are saved alongside this Markdown report, use these evidence filenames:

- `Screenshot 2026-10-11 023244.png` — custom column and full distribution (including a small `Invalid` category).
- `Screenshot 2026-10-11 023314.png` — filtered `Invalid` records; 23 rows visible in the count.

To reproduce the finding later, add the `Carrier_Date_Check` formula above as a temporary custom column, filter to `Invalid`, and confirm the count. For a stronger audit trail, a future quality report could store the affected `order_id` values, but that export **has not been performed** and is not claimed here.

#### Recommended metric logic later

For a carrier-to-customer duration, include only records where:

1. `order_delivered_carrier_date` is not null;
2. `order_delivered_customer_date` is not null; and
3. `order_delivered_customer_date >= order_delivered_carrier_date`.

This removes invalid intervals **from that calculation**, without deleting the order from the underlying model. For total orders, revenue, and other unaffected measures, keep the orders unless a separate business rule requires otherwise.

## 7. What was changed versus what was only investigated

**Permanent changes retained in the query:** promoted headers and appropriate column data types (including UK-locale Date/Time conversions).

**Temporary investigation steps removed:** filter on null customer-delivery dates; `Delivery_Date_Check`; `Carrier_Date_Check`; filter on `Invalid`; temporary type-change steps for those custom columns.

**Source records retained:** all 99,441 orders, including the 23 flagged date-sequence records and orders with legitimate missing delivery dates.

This distinction matters: **documenting a data-quality problem does not mean deleting the data or permanently adding a flag**. The source table currently has no persistent column marking the 23 records.

## 8. Limitations and checks still to consider

The Orders table is ready **for the next cleaning task**, not certified as free from every conceivable issue.

- The exact frequencies of all eight order statuses were not recorded.
- The exact counts of null approval and carrier-handover dates were not recorded, and their relationship to statuses was not investigated in detail.
- The purchase → approval → carrier sequence and estimated-date logic were not exhaustively checked.
- Business definitions for revenue, cancellation, delivery lead time, and late delivery still need to be agreed during analysis.
- Relationships to other datasets and `customer_id` versus `customer_unique_id` will be evaluated when the Customers and other tables are loaded.
- The source is a historical snapshot; a `shipped` status does not prove a parcel is *currently* in transit.

These are documented scope limits, not reasons to discard the work completed.


## 10. Next step in the project

Move to **`olist_order_items_dataset.csv`** in Power Query:

1. Home → New Source → Text/CSV.
2. Select `olist_order_items_dataset.csv`.
3. Choose **Transform Data** if prompted.
4. Confirm the new query appears alongside `olist_orders_dataset`.
5. **Stop and inspect the raw structure before making changes.**

Do not assume duplicate `order_id` values in Order Items are errors: one order can contain several line items. We will first establish that table's grain and expected key(s).

**Current project checkpoint:** Orders table reviewed and restored to eight columns; 23 carrier/date anomalies documented here; no records deleted; proceed to Order Items inspection. Do not click **Close & Apply** until the planned source-table cleaning work is complete.
