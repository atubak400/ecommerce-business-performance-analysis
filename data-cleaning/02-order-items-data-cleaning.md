# Olist E-Commerce Analysis — Order Items Data Cleaning and Quality Report

**Project:** Olist E-Commerce Analysis  
**Source:** Brazilian Olist E-Commerce public dataset  
**Dataset reviewed:** `olist_order_items_dataset.csv`  
**Power Query query:** `olist_order_items_dataset`  
**Review period:** October 2026  
**Stage:** Data cleaning and quality assessment — second of nine source tables  
**Assessment:** Initial column-level and composite-key checks completed; cross-table validation deferred to later stages.

## 1. Business context and objective

The Olist project aims to help management understand e-commerce sales, customer behaviour, delivery performance and customer satisfaction. The Order Items dataset provides **line-item detail** for orders, including the product sold, seller, price, freight charge and shipping deadline.

This table is important for calculating item-level sales, assessing shipping charges, and connecting orders to products and sellers. It is not equivalent to the Orders table: **one order can contain multiple order-item rows**.

## 2. Dataset structure and grain

The dataset contains **112,650 rows** and **seven columns**.

| Column | Data type | Purpose |
|---|---|---|
| `order_id` | Text | Identifies the order; links to the Orders dataset |
| `order_item_id` | Whole number | Item sequence number within an order |
| `product_id` | Text | Identifies the product; links to the Products dataset |
| `seller_id` | Text | Identifies the seller; links to the Sellers dataset |
| `shipping_limit_date` | Date/Time | Deadline for the seller to ship the item |
| `price` | Decimal number | Recorded item price |
| `freight_value` | Decimal number | Recorded freight charge for the item |

**Expected grain:** One row per combination of `order_id` and `order_item_id`. `order_id` by itself is not expected to be unique in this table.

## 3. Power Query preparation

1. Imported the Order Items CSV into Power Query.
2. Promoted the first row to column headers.
3. Checked and applied appropriate column data types, keeping identifiers as text, `order_item_id` as a whole number, `shipping_limit_date` as Date/Time, and `price` and `freight_value` as decimal numbers.
4. Enabled column-quality and column-profile views and assessed statistics based on the **entire dataset**, not only the default preview sample.
5. Used separate investigation queries for checks that required grouping or filtering, preserving the main Order Items query.

## 4. Column-level quality results

Across all seven columns, profiling showed **100% valid values, zero nulls and zero errors**. The key observations were:

| Column | Observed result | Interpretation |
|---|---|---|
| `order_id` | 98,666 distinct values | Multiple items can belong to the same order; repetition is expected |
| `order_item_id` | 21 distinct values; minimum 1, maximum 21 | Item sequence numbers repeat across different orders by design |
| `product_id` | 32,951 distinct values; no empty strings | Products can appear in multiple order items |
| `seller_id` | 3,095 distinct values; no empty strings | Sellers can fulfil multiple order items |
| `shipping_limit_date` | No nulls or errors | Dates are available for every line item |
| `price` | Minimum 0.85; maximum 6,735.00 | No negative or zero item prices observed |
| `freight_value` | Minimum 0; maximum 409.68 | No negative freight charges observed; zero values investigated separately |

**Note:** Power Query's *unique count* means values appearing **exactly once**, not the number of distinct values. Repeated identifiers are not automatically bad data.

## 5. Composite-key investigation

To test the expected grain, a separate query named `Order_Items_Key_Check` was created by referencing the Order Items query. It grouped the data by both `order_id` and `order_item_id` and counted the number of rows per combination.

**Result:** Every combination had a `Row_Count` of **1**, across the **112,650 source rows**.

**Decision:** The combination (`order_id`, `order_item_id`) is a valid unique row identifier for the observed dataset. Neither `order_id` nor `order_item_id` should be treated as a unique key on its own.

## 6. Price validation

The `price` column was checked for data type, missing values, errors and implausible negative amounts.

| Measure | Result |
|---|---:|
| Minimum price | 0.85 |
| Maximum price | 6,735.00 |
| Null values | 0 |
| Errors | 0 |

**Decision:** Retain the values. No negative or zero prices were found. The maximum is a high observed value, but a high price alone does not establish an error; no unsupported capping or replacement was performed.

## 7. Freight validation and zero-freight investigation

The `freight_value` column was checked for missing values, errors and negative amounts.

| Measure | Result |
|---|---:|
| Minimum freight | 0.00 |
| Maximum freight | 409.68 |
| Null values | 0 |
| Errors | 0 |
| Records with zero freight | **383** |

A separate reference query, `Freight_Value_inv`, filtered the records where `freight_value = 0`.

### Findings for the 383 zero-freight records

| Measure | Result |
|---|---:|
| Order-item records | 383 |
| Distinct `order_id` values | 339 |
| Distinct `product_id` values | 10 |
| Distinct `seller_id` values | 9 |
| Earliest `shipping_limit_date` | 31 August 2017, 02:24:30 |
| Latest `shipping_limit_date` | 30 August 2018, 21:04:01 |
| Average `shipping_limit_date` | 1 May 2018, 12:19:47 |
| Null or erroneous shipping deadlines | 0 |

The filtered records included **positive item prices**, showing that zero freight does not simply correspond to free products. Zero-freight items were concentrated among a limited number of products and sellers and had shipping deadlines spanning approximately one year.

**Interpretation:** Free shipping, seller-specific arrangements or promotions are possible explanations, but none is proven by this dataset alone. `shipping_limit_date` is a shipping deadline, **not necessarily the order purchase date**; the date range cannot establish the timing of a promotion.

**Decision:** **Retain all 383 zero-freight records.** Replacing zero with an average freight charge or removing these rows would introduce unsupported assumptions and could distort shipping-cost and revenue analyses.

## 8. Data-quality decisions and rationale

| Issue or check | Decision | Reason |
|---|---|---|
| Repeated `order_id` | Retain | An order may contain multiple items |
| Repeated `order_item_id` | Retain | Item numbering restarts for each order |
| Repeated `product_id` and `seller_id` | Retain | Products and sellers can appear across many order items |
| Composite (`order_id`, `order_item_id`) key | Accept as unique for this dataset | Grouped check returned a row count of 1 for each combination |
| Positive prices, including high values | Retain | No negative values or errors were observed; high values need not be invalid |
| Zero freight charges | Retain and document | 383 valid zero-valued records; business cause not established |
| Date/Time shipping deadlines | Retain | No nulls or type errors observed |

No records were deleted or altered solely because an identifier repeated, a price was high, or freight was zero.

## 9. Checks deferred to modelling and analysis

The current assessment validates this table **on its own**. The following checks remain for later stages:

- **Referential integrity:** Confirm that `order_id`, `product_id` and `seller_id` values match records in the Orders, Products and Sellers tables respectively.
- **Relationship design:** Confirm the appropriate one-to-many relationships and filter directions, without assuming Order Items has one row per order.
- **Business metrics:** Define sales and freight measures at the correct grain to avoid double-counting when joining tables.
- **Shipping chronology:** Compare shipping deadlines with relevant order dates after the related tables are available, using clear business rules.
- **Zero-freight explanation:** Treat promotions or seller policies as hypotheses unless additional evidence supports them.

## 10. Final assessment and next step

**Status: Ready to proceed to the next source table, with cross-table checks pending.**

The Order Items dataset contains 112,650 complete records with valid data types, no nulls or errors, a verified composite row key, non-negative prices and freight values, and a documented investigation of the 383 zero-freight rows. The original data has been preserved.

**Next step:** Continue inspecting the remaining Olist source tables. Revisit relationships, referential integrity and business rules during semantic modelling and analysis.

---

*This report documents the checks actually performed in Power Query. It is not a claim that the full nine-table model or final Power BI report has been validated.*
