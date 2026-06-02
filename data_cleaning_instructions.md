# Data Cleaning Instructions

## Overview

| File | Rows | Status |
|------|------|--------|
| `mart_customers.csv` | 1,800 | ⚠️ 3 issues |
| `mart_products.csv` | 13 | ⚠️ 1 issue |
| `mart_promotions.csv` | 6 | ⚠️ 1 issue |
| `mart_sales_master.csv` | 71,654 | ❌ 12 issues |
| `mart_branches.csv` | 9 | ⚠️ 1 issue |

---

## mart_customers.csv

### 1. Missing gender (117 rows)
Fill blank gender values with "Unknown".

### 2. Missing member_tier (291 rows)
These customers are likely non-members. Fill blanks with "Non-member".

### 3. Missing age (61 rows)
Fill blanks with the median age of all customers, or leave blank and exclude from any age-related analysis.

### 4. Unrealistic age values (32 rows)
32 rows have age = 150, which is clearly a data entry error. Blank these out and treat them the same as missing age.

### 5. member_since is stored as text
Convert the `member_since` column from plain text into a proper date format.

---

## mart_products.csv

### 1. Duplicate product_id — SNK001
`SNK001` appears twice with two different names: `Potato Chips BBQ` and `Potato Chips Barbeque`. They are the same product. Remove the duplicate row and keep only one — use `Potato Chips BBQ` to match the more common spelling.

---

## mart_promotions.csv

### 1. BADPROMO row
`BADPROMO` exists in this file as a row with description "Corrupted Promo Code". Decide whether to keep it as a known invalid code or remove it. If you remove it here, also replace any `BADPROMO` values in `mart_sales_master` with `NONE`.

---

## mart_branches.csv

### 1. Duplicate branch codes — PJ and KL
`PJ` appears twice (`Petaling Jaya` and `Petaling Jayaa`) and `KL` appears twice (`Kuala Lumpur` and `Kuala Lumper`). The misspelled rows are dirty data accidentally included in the reference file. Remove the two rows with incorrect names (`Petaling Jayaa` and `Kuala Lumper`) so only 7 clean rows remain.

---

## mart_sales_master.csv

### 1. Duplicate rows (1,141 rows)
Over 1,100 rows are exact duplicates. Drop all duplicates and keep only the first occurrence of each.

### 2. Missing invoice_id (1 row)
1 row has no invoice ID. Drop it — a transaction with no identifier is not usable.

### 3. Inconsistent date formats in txn_raw
Dates are stored in at least 4 different formats across rows — for example `19/11/2025 21:34`, `2025-08-13 18:19:00`, and `Apr 14 2025 18:34` all appear in the same column. Standardise everything into one format: `YYYY-MM-DD HH:MM:SS`.

> Be careful with ambiguous dates like `02/08/2025` — it could mean 2 August or February 8 depending on the format. Treat day-first (`DD/MM`) as the default for this dataset.

### 4. Missing transaction dates (559 rows)
After standardising the date format, rows that still have no date should be dropped — a sale record with no timestamp is not usable.

### 5. Branch name inconsistencies
The same branch is written in many different ways (e.g. `PJ`, `P.J.`, `PETALINGJYA`, `PETALING JAYAA` all refer to Petaling Jaya). Use the cleaned `mart_branches.csv` as the reference and standardise all branch values to match the 7 canonical names:

- Kuala Lumpur
- Petaling Jaya
- Subang Jaya
- Cheras
- Ipoh
- Penang
- Johor Bahru

### 6. Junk branch values — ERROR, UNKNOWN
A small number of rows have `ERROR` or `UNKNOWN` as the branch name. These cannot be mapped to any real branch — blank them out.

### 7. unit_price stored as text with "RM" prefix
About 900 rows store the price as `RM8.2` instead of `8.2`. Strip the `RM` prefix from all affected rows and convert the column to a number.

### 8. qty has text values
About 98 rows have text in the quantity column instead of a number:
- `"two"` (49 rows) — replace with the number `2`
- `"abc"` (49 rows) — meaningless, blank these out

Then convert the entire `qty` column to a number type.

### 9. product_name is inconsistent
The same product appears under different names (e.g. `BEV001` shows up as `Coke 1.5L`, `CocaCola1.5L`, and `Coca Cola 1.5L`). Replace all product names in this file with the canonical names from the cleaned `mart_products.csv`, matched by `product_id`.

### 10. Mystery product IDs — BEV999, PRD999, XXX001
These IDs don't exist in `mart_products.csv`, and the product names attached to them are random and inconsistent across rows. These are corrupt records — blank out the `product_id` and `product_name` for all affected rows, or drop them entirely.

### 11. Invalid promo code — BADPROMO (1 row)
1 row has `BADPROMO` as the promo code. Replace it with `NONE`.

### 12. Missing promo_code (1 row)
1 row has a null promo code. Fill it with `NONE`.

### 13. Missing customer_id (102 rows)
102 rows have no customer ID — these are likely guest/walk-in transactions. Fill blanks with `"Guest"` or leave as null depending on whether your analysis needs to track customers.

---

## Recommended Order

1. Clean `mart_branches.csv` first — remove the 2 dirty rows
2. Clean `mart_products.csv` — remove the duplicate SNK001 row
3. Decide on `mart_promotions.csv` — keep or remove BADPROMO row
4. Clean `mart_customers.csv` — nulls, age outliers, date format
5. Clean `mart_sales_master.csv` in this order:
   - Drop duplicate rows
   - Drop the row with missing invoice_id
   - Standardise all dates in `txn_raw`, then drop rows still missing a date
   - Standardise branch names using cleaned `mart_branches.csv`
   - Blank out `ERROR` / `UNKNOWN` branches
   - Strip `RM` from `unit_price` and convert to number
   - Fix text values in `qty` and convert to number
   - Replace product names using cleaned `mart_products.csv`
   - Blank out or drop rows with mystery product IDs
   - Replace `BADPROMO` with `NONE`, fill null promo codes with `NONE`
   - Handle missing `customer_id`
6. Save all cleaned files
