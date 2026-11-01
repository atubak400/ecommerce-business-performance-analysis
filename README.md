# E-Commerce Business Performance Analysis

**Business Intelligence | Power BI | Power Query | DAX | Data Modelling | Business Analytics**

**Project Status:** In Progress  
**Industry:** E-Commerce / Online Retail  
**Data Source:** Brazilian E-Commerce Public Dataset by Olist

---

## 1. Executive Overview

This project involves developing an end-to-end business intelligence solution for an e-commerce business to evaluate commercial performance, customer purchasing behaviour, delivery operations and customer satisfaction.

The objective is to transform raw transactional data into reliable, actionable business insights that support management decision-making.

The engagement covers business requirements, data preparation, quality assurance, data modelling, KPI development, interactive reporting and business recommendations.

## 2. Business Problem

An e-commerce business requires better visibility into its sales performance, customer activity, order fulfilment and service quality.

Management needs a reporting solution that can help answer questions such as:

- How is sales revenue performing over time?
- Which products and categories contribute most to revenue?
- Where are customers located, and how does purchasing behaviour vary?
- How efficiently are customer orders fulfilled?
- How frequently are deliveries completed later than expected?
- What relationships exist between delivery performance and customer satisfaction?

## 3. Project Objectives

The project aims to:

1. Prepare and validate nine related e-commerce datasets.
2. Develop a reliable Power BI semantic model.
3. Define meaningful business KPIs using DAX.
4. Build interactive dashboards for commercial and operational analysis.
5. Identify trends, performance gaps and data-quality risks.
6. Deliver evidence-based findings and practical business recommendations.

## 4. Scope of Work

| **Workstream** | **Expected Deliverable** | **Status** |
| --- | --- | --- |
| Data preparation | Cleaned and validated source tables | Completed |
| Data quality | Documented checks, anomalies and treatment decisions | Completed — source-table assessments |
| Data modelling | Power BI semantic model and relationships | Planned |
| KPI development | Documented DAX measures | Planned |
| Reporting | Interactive Power BI dashboards | Planned |
| Business analysis | Findings and recommendations | Planned |
| Quality assurance | Validation of calculations and report behaviour | Planned |

## 5. Data Sources

The analysis uses nine related CSV datasets from the public Olist Brazilian E-Commerce dataset:

| **Dataset** | **Business Purpose** |
| --- | --- |
| Orders | Order lifecycle, status and delivery timestamps |
| Order Items | Purchased products, item prices and freight costs |
| Customers | Customer identifiers and geographical information |
| Products | Product attributes and category identifiers |
| Payments | Payment methods and transaction values |
| Reviews | Customer feedback and review scores |
| Sellers | Seller identifiers and locations |
| Geolocation | Postal-code-level geographical information |
| Product Category Translation | English translations of product categories |

## 6. Tools and Technologies

| **Tool** | **Application** |
| --- | --- |
| Power BI Desktop | Data modelling, analysis and interactive reporting |
| Power Query | Data cleaning, transformation and validation |
| DAX | Business metrics and analytical calculations |
| GitHub | Version control, documentation and project delivery |

## 7. Planned Dashboard Deliverables

### Sales Overview

Revenue trends, order volumes, product performance and commercial KPIs.

### Customer Analysis

Customer distribution, purchasing patterns and geographical performance.

### Delivery Performance

Order fulfilment, delivery duration, delays and operational efficiency.

### Customer Satisfaction

Review scores, customer feedback and relationships with delivery outcomes.

## 8. Data Quality Assessment

Data quality is assessed before developing analytical measures and reports.

Checks include column completeness, data types, identifier uniqueness, missing values, business-rule validation, timestamp consistency and cross-table investigations.

**All nine source datasets have been assessed, cleaned where appropriate, documented and verified after saving.** Unusual records have been retained where there is insufficient evidence to justify changing or removing them. Cross-table integrity and analytical suitability will receive further validation during semantic modelling.

### Orders Dataset — Completed Assessment

The Orders dataset contains **99,441 orders**.

Key findings:

- Order IDs are unique, with no missing or erroneous identifiers.
- Customer identifiers contain no missing values or errors.
- Missing customer delivery timestamps occur in orders not marked as delivered.
- No recorded customer delivery date precedes its purchase timestamp.
- **23 orders** have a customer delivery timestamp earlier than the recorded carrier handover timestamp.

The 23 flagged orders have been retained without modifying their original dates. They will be excluded from calculations that require a valid carrier-to-customer delivery interval, while remaining available for other relevant analyses.

**Detailed documentation:** [Orders Data Cleaning and Quality Assessment](https://github.com/atubak400/ecommerce-business-performance-analysis/blob/main/data-cleaning/01-orders-data-cleaning.md)

### Order Items Dataset — Completed Assessment

The Order Items dataset contains **112,650 records**.

Key findings:

- All seven columns passed completeness and validity checks.
- The combination of `order_id` and `order_item_id` is unique.
- Item prices are positive, with a minimum recorded value of **R$0.85**.
- No negative freight charges were identified.
- **383 records**, representing **339 orders**, have zero freight charges.
- Shipping-limit dates include unusually late values requiring consideration during analysis.

Zero-freight records were retained because they may represent legitimate free-shipping arrangements; their precise cause has not been established.

### Customers Dataset — Completed Assessment

The Customers dataset contains **99,441 records** across five columns.

Key findings:

- All five columns passed the completed data-quality checks.
- `customer_id` is unique.
- No null values or errors were found in the assessed columns.
- Customer postcode prefixes were standardised as five-character text values to preserve leading zeros and support geographical matching.

The Customers table has been retained for subsequent relationship and customer-level analysis.

### Order Payments Dataset — Completed Assessment

The Order Payments dataset contains **103,886 payment records** associated with **99,440 distinct orders**.

Key findings:

- All five columns contain valid values, with no nulls or errors detected.
- Five payment types were identified: `credit_card`, `boleto`, `voucher`, `debit_card` and `not_defined`.
- Credit cards account for **76,795 payment records**.
- **Three** payment records use the category `not_defined`; their business meaning requires caution.
- **Two** credit-card records have zero instalments despite positive payment values.
- **Six** voucher payment records have zero payment value, across five orders.
- The combination of `order_id` and `payment_sequential` is unique.
- **80 orders** have irregular payment sequence numbering; **78** are marked delivered, **one** unavailable and **one** processing.

The anomalous payment records were retained. Missing sequence numbers were not fabricated, and zero amounts were not replaced with assumed values.

**Detailed documentation:** [Order Payments Data Cleaning and Quality Assessment](https://github.com/atubak400/ecommerce-business-performance-analysis/blob/main/data-cleaning/04-order-payments-data-cleaning.md)

### Products Dataset — Completed Assessment

The Products dataset contains **32,951 product records** across nine columns.

Key findings:

- All **32,951 product identifiers** are unique, with no missing values or errors.
- **610 products** have no category name or accompanying name-length, description-length and photo-quantity metadata.
- All 610 products with missing category names were matched to Order Items records, confirming that they appear in recorded purchases.
- **Two products** have missing weight and dimensional measurements.
- **Four products** have a recorded weight of zero; these belong to the `cama_mesa_banho` category and have non-zero dimensions.
- The highest recorded product weight is **40,425 g**, while **143 products** have a recorded weight of **30,000 g**.
- Product weights of at least **29,000 g** occur across several categories, indicating that high weights are not confined to a single product category.
- Product length ranges from **7 to 105 cm**, height from **2 to 105 cm**, and width from **6 to 118 cm**.

Missing metadata, zero weights and unusually high weights were retained because the source data does not provide sufficient evidence to determine corrected values.

These findings will be considered when analysing product categories, freight costs and delivery performance. Products with missing categories will not be assigned invented category labels.

**Detailed documentation:** [Products Data Cleaning and Quality Assessment](https://github.com/atubak400/ecommerce-business-performance-analysis/blob/main/data-cleaning/05-products-data-cleaning.md)

### Order Reviews Dataset — Completed Assessment

The Reviews dataset contains **99,224 records**. Its source-table assessment and cleaning are complete. Review data will support customer satisfaction analysis, including review scores and their relationship with delivery performance. Review-level relationships and the appropriate analytical grain will be validated during semantic modelling.

**Detailed documentation:** [Reviews Data Cleaning Report](data-cleaning/06-reviews-data-cleaning.md)

### Sellers Dataset — Completed Assessment

The Sellers dataset contains **3,095 records** across four columns.

Key findings and actions:

- All **3,095 seller identifiers** are distinct and unique.
- Seller ZIP code prefixes were stored as five-character text to preserve leading zeros.
- City names were standardised using Trim, Clean and Lowercase.
- Investigated city–state inconsistencies using ZIP code evidence and cross-checks against customer records.
- Corrected **three** seller state values to `RJ` for ZIP prefixes `21210`, `22783` and `21320`.
- Final columns passed validity and completeness checks.

**Detailed documentation:** [Sellers Data Cleaning Report](data-cleaning/07-sellers-data-cleaning.md)

### Geolocation Dataset — Completed Assessment

The Geolocation dataset initially contained **1,000,163 records** across five columns.

Key findings and actions:

- Standardised ZIP code prefixes as five-character text, preserving leading zeros.
- Cleaned city names and verified all **27** Brazilian state and Federal District codes.
- Investigated latitude and longitude ranges and identified **31 potential coordinate outlier records** using combined screening conditions. These were retained rather than corrected without sufficient supporting evidence.
- Identified **128,174 groups** of repeated identical records, containing **390,005 records** in total.
- Removed **261,831 excess exact duplicates** across all five columns, preserving different coordinate combinations.
- Final table contains **738,332 distinct records**.

The geographical dataset requires particular care during modelling because ZIP code prefixes are not necessarily unique and a direct relationship could introduce ambiguous matches or duplicate aggregations.

**Detailed documentation:** [Geolocation Data Cleaning Report](data-cleaning/08-geolocation-data-cleaning.md)

### Product Category Translation Dataset — Completed Assessment

The Product Category Translation dataset originally contained **71** Portuguese-to-English category mappings.

Key findings and actions:

- Promoted the first row to column headers.
- Applied Trim, Clean and Lowercase to both category columns using the Power Query interface.
- Compared **73 distinct nonblank product categories** with the translation table using a Left Anti Join.
- Identified and added two missing mappings using Power Query M (`Table.InsertRows`):
  - `pc_gamer` → `gaming_pc`
  - `portateis_cozinha_e_preparadores_de_alimentos` → `portable_kitchen_appliances_and_food_preparation_equipment`
- Final table contains **73 rows**, with **73 distinct and unique values** in each column, no errors and no empty values.

The added English descriptions are project-defined mappings, not values from the original CSV. **610 products** still have no source category; these have not been assigned invented categories.

**Detailed documentation:** [Product Category Translation Data Cleaning Report](data-cleaning/09-product-category-translation-data-cleaning.md)

### Remaining Cross-Table Validation

Source-table cleaning is complete. The next phase will validate relationship keys, cardinality, filter direction, category translation coverage in the model, payment-to-order reconciliation, and the implications of repeated ZIP prefixes and review/payment grains. These modelling checks should not be described as completed yet.

## 9. Analytical Approach

The project follows a structured workflow:

1. Understand the business requirements and source data.
2. Profile and prepare individual datasets.
3. Investigate and document data-quality issues.
4. Design and validate the semantic model.
5. Develop and test business KPIs.
6. Build management-focused Power BI reports.
7. Validate analytical results and report functionality.
8. Present findings, recommendations and limitations.

## 10. Project Progress

**Completed — Data Preparation and Quality Assessment**

- Defined the business objectives, analytical questions and reporting scope.
- Imported, assessed and cleaned all **nine** Olist source datasets.
- Investigated missing values, exact duplicates, timestamp inconsistencies, payment anomalies, product attributes, seller locations and coordinate outliers.
- Standardised relevant ZIP code prefixes and text columns.
- Corrected three supported seller state assignments.
- Removed 261,831 exact duplicate geolocation records while retaining 31 potential coordinate outliers for cautious downstream use.
- Added two missing product category translations, increasing coverage from 71 to 73 nonblank categories.
- Prepared and committed individual data-cleaning documentation for all nine datasets.
- Saved and reopened the Power BI project to verify the transformations persisted.

**Current Phase — Semantic Modelling**

- Inspect existing model relationships.
- Define table grains, keys, cardinalities and filter directions.
- Validate cross-table joins and design a reliable reporting model.

**Upcoming**

- Develop and test DAX measures and KPIs.
- Build and validate management dashboards.
- Produce business insights, limitations and recommendations.

## 11. Data Source

**Dataset:** [Brazilian E-Commerce Public Dataset by Olist — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)
