# Product Category Name Translation — Data Cleaning Report

## 1. Overview

**Project:** Olist E-Commerce Business Performance Analysis  
**Dataset:** `product_category_name_translation.csv`  
**Tool:** Microsoft Power BI — Power Query Editor  
**Purpose:** Clean and validate product category translations to support accurate product categorization and English-language reporting.

The dataset provides a mapping between Portuguese product category names and their English equivalents.

During the investigation, two categories used in the Products dataset were found to be missing from the translation table. These mappings were added to improve category coverage.

## 2. Initial Dataset Assessment

| Metric | Initial Result |
|---|---|
| Original records | 71 |
| Columns | 2 |
| Distinct Portuguese category names | 71 |
| Unique Portuguese category names | 71 |
| Missing values | 0 |
| Errors | 0 |

**Columns examined:**

- `product_category_name` — Portuguese category name.
- `product_category_name_english` — English translation.

The original translation table contained no missing values, duplicate category names, or detected errors.

## 3. Data Cleaning Transformations

The following transformations were performed using the Power Query interface:

**Promoted Headers**

Converted the first row of the imported CSV into the appropriate column headers.

**Trimmed Text**

Applied the Trim transformation to both columns to remove unnecessary leading and trailing whitespace.

**Cleaned Text**

Applied the Clean transformation to both columns to remove non-printable characters.

**Standardized Letter Case**

Converted both columns to lowercase to ensure consistent formatting and improve the reliability of category matching.

These transformations were recorded as reproducible Applied Steps in Power Query.

## 4. Missing Category Translation Investigation

To assess translation coverage, the Products dataset was compared against the translation table.

A temporary investigation query was created from the Products dataset. Empty category names were excluded, and duplicate category names were removed.

This produced **73 distinct non-empty product categories**.

A Left Anti Join was then performed using `product_category_name` as the matching column.

The Left Anti Join returned two categories that existed in the Products dataset but were absent from the translation table.

| Missing Portuguese Category | Added English Translation |
|---|---|
| `pc_gamer` | `gaming_pc` |
| `portateis_cozinha_e_preparadores_de_alimentos` | `portable_kitchen_appliances_and_food_preparation_equipment` |

The English labels were added as descriptive mappings to support reporting. They are project-defined translations rather than values supplied in the original translation CSV.

## 5. Adding the Missing Translations

The two missing records were inserted using Power Query's Advanced Editor and the M function `Table.InsertRows()`.

This was the only cleaning transformation that required directly editing M code. The preceding transformations were completed through the Power Query user interface.

The insertion used the existing cleaned table as its input and appended two new category mappings.

This approach preserved the original CSV file while making the additions reproducible whenever the Power Query transformations are refreshed.

## 6. Final Data Quality Validation

| Metric | Final Result |
|---|---|
| Total records | 73 |
| Total columns | 2 |
| Distinct Portuguese category names | 73 |
| Unique Portuguese category names | 73 |
| Distinct English category names | 73 |
| Unique English category names | 73 |
| Valid values | 100% in both columns |
| Errors | 0 |
| Empty values | 0 |
| Missing mappings identified during investigation | 2 |
| Missing mappings added | 2 |

The final table was reviewed using Power Query column profiling, configured to use the entire dataset.

The two additional records were visible as rows 72 and 73.

## 7. Quality Assurance and Save Verification

After cleaning:

1. The transformations were applied using **Close & Apply**.
2. The Power BI project was saved.
3. The PBIX file was closed and reopened.
4. The translation query was checked to confirm that the 73 records and added translation step persisted.

The save-and-reopen verification was completed successfully.

Temporary investigation queries were removed after their purpose was fulfilled.

## 8. Final Outcome

The Product Category Name Translation dataset now contains **73 unique Portuguese-to-English category mappings**, compared with the original 71.

The cleaning process improved text consistency, addressed two missing mappings, and preserved a reproducible transformation history.

The resulting table is ready to support product category relationships and English-language reporting in the Olist E-Commerce Business Performance Analysis project.

**Status:** Cleaning completed, validated, saved, and verified.