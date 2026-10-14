# 03 — Customers Data Cleaning and Quality Assessment

## 1. Purpose

This report documents the profiling, validation and cleaning of the `olist_customers_dataset` table in the Olist E-Commerce Business Performance Analysis project. The objective is to prepare customer identifiers and geographical attributes for reliable analysis in Power BI.

## 2. Dataset overview

- **Source:** Olist Brazilian E-Commerce dataset (`olist_customers_dataset.csv`)
- **Records:** 99,441
- **Columns:** 5
- **Tool:** Power BI Power Query Editor
- **Profiling:** Based on the entire dataset, not just the preview

| Column | Final data type | Purpose |
|---|---|---|
| `customer_id` | Text | Identifies a customer record associated with an order |
| `customer_unique_id` | Text | Identifies customers across multiple customer records |
| `customer_zip_code_prefix` | Text | Five-character Brazilian postcode prefix |
| `customer_city` | Text | Customer city |
| `customer_state` | Text | Two-letter Brazilian state/Federal District code |

## 3. Initial quality assessment

All five columns showed **100% valid values**, **0% errors** and **0% empty values** in full-dataset column profiling. No null values were identified during the checks.

| Column | Distinct values | Values occurring exactly once | Notes |
|---|---:|---:|---|
| `customer_id` | 99,441 | 99,441 | Unique for every row |
| `customer_unique_id` | 96,096 | 93,099 | Repetition is expected |
| `customer_zip_code_prefix` | 14,994 | 3,012 | Postcode prefix, not a numeric measure |
| `customer_city` | 4,119 | 1,144 | Text normalisation performed |
| `customer_state` | 27 | 0 | All expected Brazilian state/Federal District codes |

**Interpretation note:** In Power Query, *distinct* means the number of different values, while *unique* means the number of values that occur exactly once.

## 4. Customer identifier validation

### `customer_id`

- Distinct count: **99,441**
- Unique count: **99,441**
- Nulls/errors: **0**

**Decision:** Retain as the row-level identifier and candidate relationship key to the Orders table. Cross-table relationship integrity will be tested during modelling.

### `customer_unique_id`

- Distinct count: **96,096**
- Values appearing exactly once: **93,099**
- Identifiers appearing more than once: **2,997** (`96,096 − 93,099`)

**Decision:** Retain repeated values. This identifier is intended to help analyse customers across multiple records; removing duplicates would discard valid customer/order associations.

## 5. Repeat-customer frequency investigation

A temporary **reference query**, `Customer_Repeat_Check`, grouped records by `customer_unique_id` and counted rows into `Order_Record_Count`.

| Measure | Result |
|---|---:|
| Customer groups | 96,096 |
| Minimum records per customer identifier | 1 |
| Maximum records per customer identifier | 17 |
| Average records per customer identifier | Approximately 1.0348 |
| Distinct frequency values | 9 |
| Top three observed frequencies | 17, 9 and 7 |
| Nulls/errors in grouped count | 0 |

**Interpretation:** About **3.12%** of distinct customer identifiers (`2,997 / 96,096`) occur in more than one customer record. These represent **potential repeat customers**, not yet confirmed repeat *completed purchases*. Order status and relationships will be evaluated later.

**Decision:** No deduplication of the original Customers table.

## 6. Postcode prefix cleaning

The `customer_zip_code_prefix` field was initially inferred as **Whole Number**. Because Brazilian postcode prefixes are identifiers with five characters, numeric typing could remove leading zeros.

**Actions:**

1. Changed `customer_zip_code_prefix` to **Text**.
2. Added a custom column using:

   ```powerquery
   Text.PadStart([customer_zip_code_prefix], 5, "0")
   ```

3. Verified that shorter prefixes were left-padded to five characters while already padded values were preserved.
4. Removed the original postcode column.
5. Renamed the corrected column to `customer_zip_code_prefix`.
6. Explicitly set its final data type to **Text**.

**Example:** `1003` → `01003`; `09790` remains `09790`.

**Validation:** The corrected column showed **100% valid**, **0% errors**, **0% empty**, and **14,994 distinct values**. Its final header displayed the Text (`ABC`) type.

**Decision:** Retain postcode prefixes as five-character Text values for later geographical matching, including potential joins to Geolocation.

## 7. City cleaning

Initial `customer_city` profiling showed:

- **4,119 distinct city names**
- **1,144 city names appearing exactly once**
- **0 nulls, empty strings or errors**

**Actions:** Applied **Transform → Format → Trim** to remove leading/trailing spaces, followed by **Transform → Format → Clean** to remove non-printable characters.

**Validation:** The distinct count remained **4,119** after each operation.

**Interpretation:** Neither operation merged any previously distinct city names. This does not independently establish that all spellings or naming conventions are standardised.

**Decision:** Keep existing lowercase city names. Further city/geographical consistency checks may be performed during modelling and mapping.

## 8. State code validation

Initial profiling of `customer_state` found **27 distinct codes**, with no nulls, empty strings or errors. A temporary reference query, `Customer_State_Check`, removed duplicates on `customer_state` and sorted the results alphabetically. All 27 codes were checked against the expected Brazilian state/Federal District abbreviations:

`AC`, `AL`, `AM`, `AP`, `BA`, `CE`, `DF`, `ES`, `GO`, `MA`, `MG`, `MS`, `MT`, `PA`, `PB`, `PE`, `PI`, `PR`, `RJ`, `RN`, `RO`, `RR`, `RS`, `SC`, `SE`, `SP`, `TO`.

**Validation:** **27/27 expected codes** present, with no unexpected codes observed. The most frequent customer-record state was `SP`, followed by `RJ` and `MG`.

**Decision:** Retain the original two-letter state codes; no correction required.

## 9. Temporary investigation queries

The following reference queries were used for analysis and then deleted after findings were recorded:

- `Customer_Repeat_Check`
- `Customer_State_Check`

The source query `olist_customers_dataset` was retained, along with the existing Orders and Order Items source queries.

## 10. Outstanding checks and limitations

These checks are **deferred** until relevant tables are available in the semantic model:

- Validate the relationship between Customers and Orders using `customer_id`.
- Determine repeat *completed* purchases using Orders statuses and `customer_unique_id`.
- Check geographical joins and postcode-prefix coverage against the Geolocation dataset.
- Investigate any city naming inconsistencies or city/state mismatches identified by cross-table checks.

The current findings describe customer **records**; they should not automatically be interpreted as distinct people, delivered orders, or completed purchases.

## 11. Conclusion

The Customers table passed the completed column-level quality checks. Its identifiers were validated, repeat identifiers were appropriately retained, postcode prefixes were standardised to five-character Text, city text was trimmed and cleaned, and all 27 Brazilian state/Federal District codes were verified. The table is ready to proceed to the next project stage, subject to the deferred cross-table and modelling checks.

---
