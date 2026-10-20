# Products Dataset — Data Cleaning and Quality Assessment

**Project:** E-Commerce Business Performance Analysis  
**Source table:** `olist_products_dataset`  
**Tool:** Power BI Desktop / Power Query  
**Assessment status:** Completed  
**Records:** 32,951  
**Columns:** 9

---

## 1. Purpose

Assess the completeness, validity and analytical suitability of the product master data before building the Power BI semantic model and reporting on product categories, sales and fulfilment. Preserve source values where no defensible correction is available.

## 2. Dataset Structure

| Column | Meaning | Data type |
| --- | --- | --- |
| `product_id` | Product identifier | Text |
| `product_category_name` | Original product category | Text |
| `product_name_lenght` | Length of product name | Whole number |
| `product_description_lenght` | Length of product description | Whole number |
| `product_photos_qty` | Number of product photos | Whole number |
| `product_weight_g` | Recorded product weight in grams | Whole number |
| `product_length_cm` | Recorded product length in centimetres | Whole number |
| `product_height_cm` | Recorded product height in centimetres | Whole number |
| `product_width_cm` | Recorded product width in centimetres | Whole number |

*Note: The source dataset spells `lenght` rather than `length` in the two text-length field names; the original names are reproduced here.*

## 3. Assessment Summary

| Field | Confirmed finding | Decision |
| --- | --- | --- |
| `product_id` | 32,951 distinct and 32,951 unique identifiers; no nulls or errors | Retain as product-level identifier |
| `product_category_name` | 610 blank values; 74 distinct values including blank | Retain blanks; do not invent categories |
| `product_name_lenght` | 610 nulls; observed non-null range 5–76 | Retain nulls |
| `product_description_lenght` | 610 nulls; observed non-null range 4–3,992 | Retain nulls |
| `product_photos_qty` | 610 nulls; observed non-null range 1–20 | Retain nulls |
| `product_weight_g` | 2 nulls; 4 zeros; maximum 40,425 g | Retain and document anomalies |
| `product_length_cm` | 2 nulls; no zeros or errors; range 7–105 cm | Retain nulls |
| `product_height_cm` | 2 nulls; no zeros or errors; range 2–105 cm | Retain nulls |
| `product_width_cm` | 2 nulls; no zeros or errors; range 6–118 cm | Retain nulls |

The assessments used Power Query column profiling based on the **entire dataset**, not only the first 1,000 rows.

## 4. Key Investigations

### 4.1 Missing product categories and descriptive attributes

The category field contains **610 blank values**. The same count of missing values appears in the product name-length, description-length and photo-quantity fields. An investigation isolated the 610 products without category information and linked their product IDs to the Order Items dataset. The matching order-item records had positive prices, demonstrating that missing product descriptions do not automatically make those products unusable for sales analysis.

**Treatment:** Preserve these products and their original blank descriptive attributes. They can contribute to overall sales calculations, while category-level reports should explicitly account for unclassified products rather than silently dropping them. The cause of missing descriptive information is not established.

### 4.2 Zero product weights

**Four products** have a recorded weight of **0 g**. All four appeared in the `cama_mesa_banho` category, with other product attributes populated.

**Treatment:** Preserve the zero values in the source data, flag them as unsuitable for analyses requiring a positive physical weight, and avoid substituting an estimated weight.

### 4.3 Missing product weights and dimensions

**Two products** have null weights. The length, height and width fields also each have **two null values**. The investigation of missing weights showed one product with missing descriptive attributes and another in the `bebes` category.

**Treatment:** Preserve null values. Exclude records with missing required measurements only from calculations that depend on those measurements; do not exclude them automatically from sales or order analysis.

### 4.4 Maximum recorded weight

The heaviest product has a recorded weight of **40,425 g (40.425 kg)** in the `cama_mesa_banho` category, with dimensions **13 × 65 × 28 cm**. Its recorded volume is **23,660 cm³ (23.66 litres)**. This is an unusual weight relative to the recorded dimensions and broad product category, but the available data cannot establish whether it is incorrect.

**Treatment:** Retain the original value and document it as a potential outlier. Do not replace it with an assumed weight.

### 4.5 Concentration at 30,000 g

**143 products** have a weight of exactly **30,000 g**, representing approximately **0.43% of all products**. These records span multiple categories.

A separate investigation found **160 products weighing at least 29,000 g**, of which **143 (89.4%)** were recorded at exactly 30,000 g. The filtered group contains **12 distinct weight values**, with a maximum of 40,425 g.

This concentration suggests that 30,000 g *may* be a standardised or default value, but the source system's rules have not been verified. The presence of a 40,425 g record means that 30,000 g is not an absolute maximum in this dataset.

**Treatment:** Retain all 143 values and document the concentration as a potential limitation for weight-related reporting.

## 5. Analytical Implications

- **Sales and order analysis:** Keep products with missing descriptive or physical attributes; do not discard their related transactions solely because product metadata is incomplete.
- **Category reporting:** Explicitly identify unclassified products so category totals can be reconciled with overall totals.
- **Shipping and size analysis:** Treat zero or null weights, and null dimensions, as unavailable or unsuitable measurements for calculations that require positive physical values.
- **Outlier interpretation:** Avoid assuming the 40,425 g record or the repeated 30,000 g values are errors without supporting evidence.
- **Modelling:** Use `product_id` as the unique identifier in the Products table and validate its relationship to Order Items during semantic modelling.

## 6. Data Cleaning Decisions

No unsupported corrections, category assignments, weight imputations or row deletions were made to the original Products table during this assessment. Temporary Power Query reference queries were used to investigate anomalies and removed after the checks were completed. The original Products query was retained and the Power BI file saved.

## 7. Outstanding Considerations

The investigation identifies potential data-quality issues, not their confirmed root causes. Before publishing final product-level conclusions, validate relevant model relationships and ensure measures handle unclassified products and unavailable physical measurements appropriately. If authoritative source documentation becomes available, revisit whether 30,000 g represents a system convention.

---

**Overall assessment:** The Products dataset is suitable for continued modelling and business analysis, subject to transparent handling of missing product metadata and caution when interpreting physical weight and dimension metrics.
