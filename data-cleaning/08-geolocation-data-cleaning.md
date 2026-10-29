# 08 — Geolocation Data Cleaning and Validation

**Project:** Olist E-Commerce Business Performance Analysis  
**Source:** `olist_geolocation_dataset.csv`  
**Tool:** Microsoft Power BI — Power Query Editor  
**Status:** Cleaning completed; changes saved and verified after reopening the PBIX.

## 1. Purpose

Prepare the Brazilian geolocation reference data for geographic analysis while retaining distinct location observations. Assess missing values, types, ZIP code prefixes, city and state formatting, geographic coordinate anomalies, and exact duplicate rows.

## 2. Dataset overview

| Metric | Result |
|---|---:|
| Original rows | 1,000,163 |
| Original columns | 5 |
| Distinct ZIP code prefixes observed | 19,015 |
| Distinct city values after standardization | 8,010 |
| Distinct state codes | 27 |
| Exact excess duplicate rows removed | 261,831 |
| Final rows | **738,332** |
| Rows removed (%) | **26.18%** |

**Columns retained:** `geolocation_zip_code_prefix`, `geolocation_lat`, `geolocation_lng`, `geolocation_city`, `geolocation_state`.

All five columns were reported by Power Query's entire-dataset profiling as 100% valid, with no errors or empty values. **Type validity does not guarantee geographic accuracy.**

## 3. Transformations applied

### 3.1 ZIP code prefix normalization

- Changed `geolocation_zip_code_prefix` from **Whole Number** to **Text**.
- Created a temporary custom column to restore leading zeros, using:

  ```powerquery
  Text.PadStart([geolocation_zip_code_prefix], 5, "0")
  ```

- Replaced the original ZIP prefix column with the padded result, retaining its original name.
- Examples: `1037` → `01037`; `1046` → `01046`.
- The observed number of distinct ZIP prefixes remained **19,015** after padding.

ZIP prefixes are identifiers rather than quantities; Text prevents loss of meaningful leading zeros. The padding operation assumes the source values represent five-character Brazilian ZIP prefixes.

### 3.2 City text standardization

Applied these operations to `geolocation_city`:

1. **Trim:** removes leading and trailing whitespace.
2. **Clean:** removes non-printable characters.
3. **Lowercase:** standardizes letter case.

The observed distinct city count decreased from **8,011 to 8,010**. This is evidence of at least one consolidated formatting variant; it is **not** proof that all city spellings are correct or standardized across accent variants.

### 3.3 State code inspection

- Inspected the distribution of `geolocation_state`.
- Found **27 distinct codes**, corresponding to Brazil's 26 states and Federal District.
- No missing or invalid *code values* were observed in the inspected distribution.
- No state values were changed.

This validates the set of abbreviations, **not** every city–state or ZIP–state pairing.

## 4. Coordinate quality investigation

The latitude and longitude columns were valid numeric values without nulls or conversion errors. However, the observed extrema indicated geographically implausible coordinates for records identified as Brazilian locations.

| Coordinate | Observed minimum (approx.) | Observed maximum (approx.) |
|---|---:|---:|
| Latitude | −36.605° | +45.066° |
| Longitude | −101.467° | +121.105° |

### 4.1 Preliminary boundary checks

- Latitude **greater than +6°**: **26 records**.
- Latitude **less than −34°**: **5 records**.
- Longitude **greater than −34°**: **33 records**, **not all errors**: this preliminary cutoff also includes legitimate Brazilian offshore locations such as Fernando de Noronha (approximately −32.4° longitude).
- Longitude **less than −74°**: **4 records**, overlapping with latitude outliers.

These are exploratory rectangular boundary checks, not an authoritative geographic polygon or a definitive classification of every coordinate.

### 4.2 Combined potential-outlier check

A temporary reference query, `Geolocation_Outlier_Investigation`, flagged records meeting **any** of these conditions:

```powerquery
if [geolocation_lat] > 6
    or [geolocation_lat] < -34
    or [geolocation_lng] < -74
    or [geolocation_lng] > -30
then "Potential Outlier"
else "Within Range"
```

The broader eastern longitude cutoff (`> -30°`) was chosen to avoid incorrectly flagging Fernando de Noronha. **31 records** were flagged by the combined rule, avoiding double-counting records with multiple abnormal coordinate values. This represents approximately **0.0031%** of the original dataset.

**Decision:** The investigation did **not** remove or correct coordinates. The flagged records were retained in the source query because their true coordinates were not independently verified. The temporary outlier investigation query was deleted after use. Future map and spatial analyses should apply a documented geographic validation rule, potentially checking coordinates against a Brazil boundary polygon and ZIP-prefix reference data.

**Important qualification:** The 31-record figure describes the **original, pre-deduplication investigation**. The number of potential outliers remaining among the 738,332 final distinct rows was **not separately measured**; do not assume it is still 31.

## 5. Exact duplicate investigation

Created a temporary reference query, `Geolocation_Duplicate_Investigation`, and grouped by **all five original columns**, using `Count Rows` as `Record_Count`. This identifies *exact* duplicate records, not merely records sharing the same ZIP prefix.

| Investigation measure | Count |
|---|---:|
| Groups with `Record_Count > 1` | 128,174 |
| Total original rows in those groups | 390,005 |
| Excess duplicate rows: 390,005 − 128,174 | **261,831** |
| Distinct five-column combinations | **738,332** |

Some identical combinations occurred many times (for example, one Itapema, SC combination appeared 314 times). Different coordinates for the same ZIP prefix were **not** treated as exact duplicates.

### 5.1 Decision and implementation

Selected all five columns in the original `olist_geolocation_dataset` query and used **Home → Remove Rows → Remove Duplicates**. Power Query retained one occurrence of each exact five-column combination.

- Before: **1,000,163** rows.
- Removed: **261,831** excess duplicate rows (**26.18%**).
- After: **738,332** rows.

A subsequent `Count Rows` check verified the final row count. That temporary count step was then removed so the query continued to output a table. The temporary duplicate investigation query was also deleted.

**Caveat:** Deduplication removes repetition in the geolocation reference file. If repeated observations were intended to encode frequency or sampling weight, that information is no longer present in the cleaned query. The decision is suitable for a distinct-location reference use case but should not be interpreted as preserving observation frequencies.

## 6. Final validation and checkpoint

- Five original column names retained.
- ZIP prefix stored as Text and padded to five characters.
- City text trimmed, cleaned and lowercased.
- State code set inspected: 27 expected Brazilian abbreviations.
- No missing values or Power Query column errors observed.
- Exact duplicates removed across **all five columns**.
- Final row count verified: **738,332**.
- Temporary investigation queries removed.
- Changes **Close & Applied**, PBIX saved, Power BI closed and reopened, and the saved `Removed duplicates` transformation verified.

## 7. Remaining limitations and recommendations

1. **Coordinate anomalies remain:** The 31 original potential outliers were retained pending reliable geographic verification. Reassess them before mapping or distance calculations.
2. **ZIP prefixes are not unique location keys:** Multiple valid latitude/longitude pairs can share one prefix. Do not directly join this granular table to orders/customers without checking for many-to-many multiplication; consider a separately validated, one-row-per-prefix geographic lookup for the reporting model.
3. **City–state relationships were not exhaustively validated:** Correct state abbreviations do not establish that every associated city is correct.
4. **Outlier count after deduplication is unknown:** Re-run the geographic test on the final cleaned table if an exact post-cleaning count is needed.
5. **Distinct-location interpretation:** Exact deduplication preserves distinct five-field combinations, not original repetition counts.

## 8. Outcome

The Geolocation dataset is cleaned and saved for the next modeling stage, with **738,332 distinct five-column records** and documented geographic quality caveats. It is ready for further reference-table design, but not yet certified for unrestricted spatial analysis.
