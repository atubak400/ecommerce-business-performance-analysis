# E-Commerce Business Performance Analysis

**Business Intelligence | Power BI | Power Query | DAX | Data Modelling | Business Analytics**

**Project Status:** In Progress  
**Industry:** E-Commerce / Online Retail  
**Data Source:** Brazilian E-Commerce Public Dataset by Olist

---

## 1. Executive Overview

This project involves developing an end-to-end business intelligence solution for an e-commerce business to evaluate commercial performance, customer purchasing behaviour, delivery operations and customer satisfaction.

The objective is to transform raw transactional data into reliable, actionable business insights that support management decision-making.

The engagement follows a client-style analytical workflow, covering business requirements, data preparation, quality assurance, data modelling, KPI development, interactive reporting and business recommendations.

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

| Workstream | Expected Deliverable | Status |
|---|---|---|
| Data preparation | Cleaned and validated source tables | In progress |
| Data quality | Documented checks, anomalies and treatment decisions | In progress |
| Data modelling | Power BI semantic model and relationships | Planned |
| KPI development | Documented DAX measures | Planned |
| Reporting | Interactive Power BI dashboards | Planned |
| Business analysis | Findings and recommendations | Planned |
| Quality assurance | Validation of calculations and report behaviour | Planned |

## 5. Data Sources

The analysis uses nine related CSV datasets from the public Olist Brazilian E-Commerce dataset:

| Dataset | Business Purpose |
|---|---|
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

| Tool | Application |
|---|---|
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

Checks include column completeness, data types, identifier uniqueness, missing values, business-rule validation and timestamp consistency.

### Orders Dataset — Completed Assessment

The Orders dataset contains **99,441 orders**.

Key findings:

- Order IDs are unique, with no missing or erroneous identifiers.
- Customer identifiers contain no missing values or errors.
- Missing customer delivery timestamps occur in orders not marked as delivered.
- No recorded customer delivery date precedes its purchase timestamp.
- **23 orders** have a customer delivery timestamp earlier than the recorded carrier handover timestamp.

The 23 flagged orders have been retained without modifying their original dates. They will be excluded from calculations that require a valid carrier-to-customer delivery interval, while remaining available for other relevant analyses.

**Detailed documentation:** [Orders Data Cleaning and Quality Assessment](data-cleaning/01-orders-data-cleaning.md)

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

**Completed**
- Defined the initial business objectives and reporting scope.
- Imported and assessed the Orders dataset.
- Investigated missing values, duplicates and delivery-date inconsistencies.
- Published detailed Orders data-cleaning documentation.

**In Progress**
- Cleaning and validating the remaining source datasets.

**Upcoming**
- Semantic modelling, DAX measures, dashboard development and business recommendations.

## 11. Data Source and Project Context

This is an independently executed, client-style business intelligence project using the publicly available Brazilian E-Commerce dataset by Olist.

The project is designed to demonstrate professional analytical delivery, including documented data-quality decisions, reproducible methods, reliable reporting and business-focused recommendations.

**Dataset:** [Brazilian E-Commerce Public Dataset by Olist — Kaggle](https://www.kaggle.com/datasets/olistbr/brazilian-ecommerce)

---

*This repository will be updated as additional project deliverables are completed.*
