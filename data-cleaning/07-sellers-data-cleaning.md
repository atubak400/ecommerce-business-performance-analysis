# 07 — Sellers Dataset: Data Cleaning and Validation

## Project
**Olist E-Commerce Business Performance Analysis**  
**Source table:** `olist_sellers_dataset`  
**Tool:** Power BI Desktop — Power Query Editor  
**Status:** Completed, applied, saved, and verified after reopening the PBIX.

## 1. Dataset overview
The Sellers dataset describes sellers participating in the Olist marketplace and supports seller-level analysis and geographic reporting.

| Field | Final data type | Purpose |
|---|---|---|
| `seller_id` | Text | Seller identifier |
| `seller_zip_code_prefix` | Text | Five-digit Brazilian ZIP code prefix |
| `seller_city` | Text | Seller city |
| `seller_state` | Text | Two-letter Brazilian state code |

**Records:** 3,095  
**Columns:** 4  
**Seller IDs:** 3,095 distinct and 3,095 unique  
**Column quality:** 100% valid; 0% error; 0% empty across all four fields, based on profiling of the entire dataset.

## 2. Cleaning operations

### ZIP code prefix preserved as text
Changed `seller_zip_code_prefix` from the automatically inferred whole-number type to **Text**, preserving leading zeros (for example, `04195` rather than `4195`). Confirmed that ZIP code prefixes contain five characters.

### City text standardized
Applied **Trim**, **Clean**, and **Lowercase** to `seller_city` to remove surrounding whitespace, remove nonprinting characters, and standardize capitalization for consistent grouping and comparisons.

### State codes reviewed
Examined all **23 distinct state codes** and found no missing or invalid-format values in the inspected list. The final state field remains text.

## 3. City–state consistency investigation
Created temporary reference queries to examine distinct `(seller_city, seller_state)` combinations. Removing duplicate pairs made it possible to count how many different states appeared for each city name, without allowing multiple sellers from the same city–state pair to inflate the count.

**Findings:**
- Some city names appeared with more than one state code.
- `rio de janeiro` appeared with **three** state codes in the uncorrected seller data: `RJ`, `RN`, and `SP`.
- A city name appearing in multiple states was treated as a **signal for investigation**, not automatic proof of an error; names can be reused in different locations.

The original seller rows were not removed during these checks.

## 4. ZIP-based cross-check and targeted corrections
To investigate the `rio de janeiro` mismatches, examined seller ZIP prefixes and compared the corresponding ZIP prefixes with the previously validated Customers dataset. The customer records for the inspected prefixes identified `rio de janeiro`, `RJ`.

| Seller ZIP prefix | Observed incorrect seller state | Corrected state | Basis |
|---|---|---|---|
| `21210` | `RN` | `RJ` | Seller city and customer ZIP cross-check |
| `22783` | `SP` | `RJ` | Seller city and customer ZIP cross-check |
| `21320` | `SP` | `RJ` | Seller city and customer ZIP cross-check |

**Implementation:** Used a Power Query **Conditional Column** named `Corrected_Seller_State`:

```powerquery
if [seller_zip_code_prefix] = "21210" then "RJ"
else if [seller_zip_code_prefix] = "22783" then "RJ"
else if [seller_zip_code_prefix] = "21320" then "RJ"
else [seller_state]
```

This targeted rule updates the identified ZIP prefixes while preserving existing state values for all other sellers. The original `seller_state` column was removed, and `Corrected_Seller_State` was renamed to `seller_state`.

**Scope:** Three identified seller-state mismatches were corrected. This is **not** a claim that every possible geographic inconsistency in the dataset has been independently verified against an authoritative postal-code directory.

## 5. Final validation
After the corrections:
- The Sellers table still contains **3,095 records and 4 columns**.
- `seller_id` remains **3,095 distinct / 3,095 unique**.
- `seller_zip_code_prefix` remains **Text**, retaining five-digit prefixes and leading zeros.
- All four columns show **100% valid**, **0% error**, and **0% empty** in full-dataset column profiling.
- Temporary investigation queries were removed; only the seven original source queries remained at this checkpoint.
- Changes were **Close & Apply**-ed, the PBIX was saved, then closed and reopened to verify that the Sellers transformations persisted.

## 6. Modelling considerations
- `seller_id` is the candidate key for the Sellers dimension and can link to `seller_id` in Order Items.
- Use the corrected `seller_state` for seller-state reporting.
- Geographic fields reflect source data plus the three documented targeted corrections. Avoid assuming every city name uniquely identifies a state.
- Keep ZIP prefixes as text in any future merges, relationships, or exports.

## 7. Outcome
The Sellers dataset is ready for the next modelling stage, subject to subsequent relationship and cross-table validation. The work preserved all 3,095 sellers and documented the evidence and logic behind the three targeted state corrections.
