Data Cleaning Summary
This repository contains cleaned and standardized datasets prepared for exploratory and analytical use. The original CSV files presented a number of data quality issues that were addressed using Power Query transformations.

---
## 📁 Datasets Reviewed

- `USER_TAKEHOME.csv`
- `TRANSACTION_TAKEHOME.csv`
- `PRODUCTS_TAKEHOME.csv`
---

## 1. User Data (`USER_TAKEHOME.csv`)
### Issues Identified:
- Missing or blank values in `STATE`, `LANGUAGE`, and `GENDER`.
- Incomplete `BIRTH_DATE` values, which impact age-based analysis.
- Inconsistent formatting due to leading/trailing whitespaces.
### Actions Taken:
- Replaced blank entries with `"Unknown"` for categorical fields.
- Removed rows with missing `BIRTH_DATE`.
- Trimmed whitespace from all text columns to improve join consistency.
---

## 2. Transaction Data (`TRANSACTION_TAKEHOME.csv`)
### Issues Identified:
- Invalid or missing `BARCODE` values (e.g., nulls, `-1`).
- Occasional error values in `FINAL_QUANTITY`.
- Whitespace in `RECEIPT_ID` and `STORE_NAME`.
### Actions Taken:
- Filtered out transactions with invalid or null `BARCODE`s.
- Replaced error values in `FINAL_QUANTITY` with `0`.
- Trimmed whitespace from key identifier fields.
---

## 3. Product Data (`PRODUCTS_TAKEHOME.csv`)
### Issues Identified:
- Incomplete records with missing `CATEGORY_1` or `CATEGORY_2`.
- Blank strings in `CATEGORY_3` and `CATEGORY_4`.
- Unclean values in `BRAND` and `MANUFACTURER`.
### Actions Taken:
- Removed rows with missing primary category fields.
- Substituted blanks in lower-level category fields with `"Unspecified"`.
- Standardized formatting by trimming whitespace across all text columns.
---
##  Outcome
All three datasets were cleaned with a focus on:
- Preserving key business logic.
- Preparing for reliable joins across `USER_ID` and `BARCODE`.
- Ensuring minimal information loss while improving data integrity.

The cleaned outputs are ready for downstream analysis and modeling.
