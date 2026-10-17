# 04 — Order Payments Data Cleaning and Quality Assessment

## Project
**Olist E-Commerce Business Performance Analysis**  
**Dataset:** `olist_order_payments_dataset`  
**Tool:** Microsoft Power BI — Power Query Editor  
**Assessment status:** Completed

## 1. Objective
Assess the quality and analytical suitability of the order payments data before modelling. Check field types, completeness, payment methods, numerical anomalies, payment sequence consistency, and potential duplicates. Preserve original records unless there is defensible evidence for correction or exclusion.

## 2. Dataset overview

| Metric | Finding |
|---|---:|
| Payment records | 103,886 |
| Distinct `order_id` values | 99,440 |
| Columns | 5 |
| Payment types | 5 |
| Null or error values | None observed across the five columns during full-dataset profiling |

The dataset is at **payment-record grain**, not order grain: one order can have multiple payment rows. Therefore, `order_id` is not expected to be unique by itself.

### Field definitions and data types

| Column | Power Query type | Analytical purpose |
|---|---|---|
| `order_id` | Text | Join payments to orders |
| `payment_sequential` | Whole number | Payment entry sequence within an order |
| `payment_type` | Text | Method of payment |
| `payment_installments` | Whole number | Number of instalments recorded |
| `payment_value` | Decimal number | Monetary amount of the payment record |

## 3. Payment method distribution

| Payment type | Records | Share of payment records |
|---|---:|---:|
| `credit_card` | 76,795 | 73.92% |
| `boleto` | 19,784 | 19.04% |
| `voucher` | 5,775 | 5.56% |
| `debit_card` | 1,529 | 1.47% |
| `not_defined` | 3 | <0.01% |
| **Total** | **103,886** | **100%** |

These figures count payment **records**, not distinct orders or customers. A single order may contribute more than one payment record.

## 4. Data-quality checks and investigations

### 4.1 Missing values, errors and types
Full-dataset Power Query column profiling showed valid values across all five columns, with no nulls or conversion errors observed. The identifiers and payment type were stored as text, sequence and instalment counts as whole numbers, and payment amounts as decimal numbers.

**Decision:** Retain the selected data types and all source records.

### 4.2 Undefined payment method with zero amount
Three payment records had `payment_type = not_defined` and `payment_value = 0`. A related order lookup identified a cancelled order among the investigated records. This is consistent with the possibility that payment was never completed, but the dataset does not establish a definitive cause for every record.

**Decision:** Retain the three records and document the undefined payment method and zero amount. Avoid assigning an invented payment type or value.

### 4.3 Zero-value voucher entries
Six payment records had `payment_type = voucher` and `payment_value = 0`. Inspection of the corresponding orders showed other payment records with positive amounts, including voucher and, in some cases, credit-card entries. Thus, a zero-value voucher row does not necessarily mean the entire order was unpaid.

**Decision:** Retain the six records. Treat the zero values as an investigated anomaly with an unconfirmed business explanation; do not automatically delete them or treat them as positive revenue.

### 4.4 Zero payment instalments
Two credit-card payment records had `payment_installments = 0`, with positive `payment_value` values of **58.69** and **129.94**. The instalment field contains 24 distinct values overall, from 0 to 24.

**Decision:** Retain the two records and flag zero instalments as unusual. Do not convert zero to one without a documented business rule.

### 4.5 Payment sequence range and unusual starting numbers
`payment_sequential` ranged from **1 to 29**. An order with sequence 29 was examined and had 29 payment entries, with sequence numbers 1 through 29, consistent with a multi-payment order rather than an error.

A grouped order-level check compared payment count, minimum sequence, and maximum sequence. **80 orders** were flagged for further investigation because their recorded payment sequence started at **2**, not 1. The order-status breakdown was:

| Order status | Flagged orders | Share |
|---|---:|---:|
| Delivered | 78 | 97.5% |
| Unavailable | 1 | 1.25% |
| Processing | 1 | 1.25% |
| **Total** | **80** | **100%** |

All 80 flagged order IDs matched a record in the Orders table during the lookup. The fact that 78 were delivered shows that unusual sequence numbering is not confined to unsuccessful orders. The reason sequence 1 is absent is **not established**.

**Decision:** Retain these payment records and document the numbering anomaly. Do not renumber them, infer missing amounts, or classify them as failed payments solely on this basis.

### 4.6 Duplicate payment sequence combinations
A reference query grouped the source payments by the combination of `order_id` and `payment_sequential` and counted rows per group. Filtering `Record_Count > 1` returned **no rows**.

**Result:** No duplicate `order_id` + `payment_sequential` combinations were found across the dataset. This is a **composite uniqueness check**; it does not mean `order_id` alone is unique.

**Decision:** No duplicate removal was required.

## 5. Summary of decisions

| Finding | Action | Reason |
|---|---|---|
| No nulls or conversion errors observed | Keep | Fields passed profiling checks |
| Three undefined, zero-value payments | Retain and document | Cause not fully confirmed |
| Six zero-value voucher payments | Retain and document | Orders can have other positive payment entries |
| Two zero-instalment credit-card payments | Retain and document | No authoritative correction rule |
| Sequence values up to 29 | Retain | A 29-entry order had a complete 1–29 sequence |
| 80 orders with sequence starting at 2 | Retain and flag | Missing initial sequence unexplained; most orders delivered |
| Duplicate composite payment keys | None found | Grouped count greater than one returned zero rows |

## 6. Modelling and reporting considerations

- Use `order_id` to relate Payments to Orders, respecting that **multiple payment records can belong to one order**.
- For monetary analysis, aggregate `payment_value` at the appropriate grain; do not equate payment-row count with order count.
- Avoid joining Payments directly to Order Items for revenue calculations without accounting for the potential many-to-many multiplication of rows.
- Distinguish payment amounts from product sales amounts (`price`) and freight charges; they measure different concepts.
- Consider separately reporting zero-value payment records and anomalous sequence starts as data-quality indicators.
- Any treatment of cancelled, unavailable or processing orders should follow an explicit reporting definition rather than an undocumented data-cleaning assumption.

## 7. Reproducibility and checkpoint

The investigations were performed with temporary **reference queries** in Power Query, including grouping, filtering, order lookups, and a composite duplicate check. These temporary investigation queries were deleted after findings were recorded. The original four loaded Olist tables remained intact at this checkpoint. Changes were applied using **Close & Apply**, and the Power BI file `Olist E-Commerce Analysis.pbix` was saved.

## 8. Final assessment

The Order Payments dataset is suitable for the next stage of modelling **with documented caveats**. The observed exceptions are small relative to the dataset and do not, on the available evidence, justify deleting or modifying source records. Maintaining the original values while clearly recording anomalies supports reproducibility and defensible business analysis.

---

*Evidence basis: Power Query screenshots and investigation results reviewed during the guided assessment. No claims about the underlying business causes of anomalies are treated as verified.*
