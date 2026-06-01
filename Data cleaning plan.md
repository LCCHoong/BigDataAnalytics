# Data Cleaning Instructions

## Overview

| File | Rows | Status |
|------|------|--------|
| `mart_customers.csv` | 1,800 | ⚠️ 3 issues |
| `mart_products.csv` | 13 | ✅ Clean |
| `mart_promotions.csv` | 6 | ✅ Clean |
| `mart_sales_master.csv` | 71,654 | ❌ 10 issues |
| `mart_branches.csv` | 9 | ✅ Clean (use as reference) |

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

## mart_sales_master.csv

### 1. Inconsistent date formats in txn_raw
Dates are stored in at least 4 different formats across rows — for example `19/11/2025 21:34`, `2025-08-13 18:19:00`, and `Apr 14 2025 18:34` all appear in the same column. Standardise everything into one format: `YYYY-MM-DD HH:MM:SS`.

> Be careful with ambiguous dates like `02/08/2025` — it could mean 2 August or February 8 depending on the format. Treat day-first (`DD/MM`) as the default for this dataset.

### 2. Missing transaction dates (559 rows)
After standardising the date format, rows that still have no date should be dropped — a sale record with no timestamp is not usable.

### 3. Branch names are inconsistent
The same branch is written in many different ways (e.g. `PJ`, `P.J.`, `PETALINGJYA`, `PETALING JAYAA` all refer to Petaling Jaya). Use `mart_branches.csv` as the reference and standardise all branch values to match the 7 canonical names:

- Kuala Lumpur
- Petaling Jaya
- Subang Jaya
- Cheras
- Ipoh
- Penang
- Johor Bahru

> Note: `mart_branches.csv` itself contains 2 dirty rows (`Petaling Jayaa`, `Kuala Lumper`) — ignore those when using it as a reference.

### 4. Junk branch values — ERROR, UNKNOWN
A small number of rows have `ERROR` or `UNKNOWN` as the branch name. These cannot be mapped to any real branch — blank them out.

### 5. unit_price stored as text with "RM" prefix
About 900 rows store the price as `RM8.2` instead of `8.2`. Strip the `RM` prefix from all affected rows and convert the column to a number.

### 6. qty has text values
About 98 rows have text in the quantity column instead of a number:
- `"two"` (49 rows) — replace with the number `2`
- `"abc"` (49 rows) — meaningless, blank these out

Then convert the entire `qty` column to a number type.

### 7. product_name is inconsistent
The same product appears under different names (e.g. `BEV001` shows up as `Coke 1.5L`, `CocaCola1.5L`, and `Coca Cola 1.5L`). Replace all product names in this file with the canonical names from `mart_products.csv`, matched by `product_id`.

### 8. Mystery product IDs — BEV999, PRD999, XXX001
These IDs don't exist in `mart_products.csv`, and the product names attached to them are random and inconsistent (the same ID shows up with completely different product names across rows). These are corrupt records — blank out the `product_id` and `product_name` for all affected rows, or drop them entirely.

### 9. Invalid promo code — BADPROMO
1 row has `BADPROMO` as the promo code, which does not exist in `mart_promotions.csv`. Replace it with `NONE`.

### 10. Fully blank rows (1 row)
1 row has every key field blank (date, branch, product, qty, price). Drop it.

---

## mart_products.csv
No action needed. Use as a reference to fix product names in `mart_sales_master`.

---

## mart_promotions.csv
No action needed. Use as a reference to validate promo codes in `mart_sales_master`.

---

## mart_branches.csv
No action needed. Use as a reference to standardise branch names in `mart_sales_master`.

---

## Recommended Order

1. Open `mart_branches.csv` and `mart_products.csv` — keep them open as references
2. Clean `mart_customers.csv` — nulls, age outlier, date format
3. Clean `mart_sales_master.csv` in this order:
   - Drop the fully blank row
   - Standardise all dates in `txn_raw`, then drop rows still missing a date
   - Standardise branch names using `mart_branches.csv`
   - Blank out `ERROR` / `UNKNOWN` branches
   - Strip `RM` from `unit_price` and convert to number
   - Fix text values in `qty` and convert to number
   - Replace product names using `mart_products.csv`
   - Blank out or drop rows with mystery product IDs
   - Replace `BADPROMO` with `NONE`
4. Save all cleaned files
