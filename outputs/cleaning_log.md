# Cleaning Log — Part 1: Business Data Cleaning

## Issues Found

### 1. Text Formatting Issues (102 cells affected)
- Leading and trailing spaces in region, segment, category, ship_mode, payment_status, order_status
- Inconsistent casing: "NORTH", "north", "North" all meant the same region
- Double spaces: "Small  Business", "Office  Supplies", "Standard  Class"
- Similar values written differently across multiple columns

### 2. Date Format Inconsistencies
- Four different date formats found across order_date and ship_date:
  - `21 Jul 2024` (dd Mon yyyy)
  - `08/31/2024` (MM/DD/YYYY)
  - `28-11-2024` (DD-MM-YYYY)
  - `2024-05-24` (YYYY-MM-DD)
- All standardized to YYYY-MM-DD format

### 3. Ship Date Before Order Date (21 records)
- 21 records had ship_date earlier than order_date, which is logically impossible
- These were flagged as Invalid in the data_quality_flag column

### 4. Duplicate Records
- **Exact duplicates:** 20 rows were identical across all columns
- **Conflicting duplicate order_ids:** 12 unique order_ids appeared more than once with different data
- Exact duplicates were removed (kept first occurrence)
- Conflicting duplicates were kept but flagged as Warning for manual review

### 5. Missing Values
- **region:** 25 missing → filled as "Unknown"
- **ship_mode:** 21 missing → filled as "Unknown"
- **discount:** 18 missing → set to 0 where quantity, unit_price, and sales were valid

### 6. Discount Issues
- **Negative discounts:** 15 records had negative discount values → flagged as Invalid
- **Percentage format:** 8 records had values like "70%" or "85%" instead of decimals → converted to 0.70, 0.85
- **High discounts (>50%):** 15 records exceeded 50% discount → flagged as Warning

### 7. Sales/Profit Calculation Mismatches
- 47 records where calculated_sales (qty × unit_price × (1 - discount)) differed from the reported sales by more than 5%
- Both original and recalculated values retained for comparison

### 8. Order Status Issues
- Cancelled orders: 145
- Failed payments: 69
- Refunded orders: 71

---

## Cleaning Actions Performed

| Action | Details |
|---|---|
| Trimmed whitespace | TRIM applied to all text columns |
| Standardized case | Title Case applied across segment, region, category, etc. |
| Collapsed double spaces | "Small  Business" → "Small Business" |
| Parsed mixed date formats | All dates converted to YYYY-MM-DD |
| Removed exact duplicates | 20 rows removed, first occurrence kept |
| Flagged conflicting duplicates | 12 order_ids with conflicting data flagged as Warning |
| Filled missing region | 25 records → "Unknown" |
| Filled missing ship_mode | 21 records → "Unknown" |
| Filled missing discount | 18 records → 0 (where other fields valid) |
| Converted percentage discounts | "70%" → 0.70, "85%" → 0.85 |
| Created calculated columns | calculated_sales, calculated_profit, profit_margin, shipping_delay_days, order_month, order_year |
| Applied quality flag | Clean / Warning / Invalid assigned per business rules |

---

## Business Rules Applied

1. Missing region → filled as "Unknown" and documented in quality report
2. Missing ship_mode → filled as "Unknown" and documented in quality report
3. Missing discount → treated as 0 only when quantity, unit_price, and sales fields were valid
4. Negative discount → flagged as Invalid (not corrected, as the true value is unknown)
5. Discount > 50% → flagged as Warning (unusually high but possible during clearance)
6. Cancelled orders and failed payments → excluded from completed sales pivot summaries
7. Refunded orders → summarized separately in the pivot report
8. Ship date before order date → flagged as Invalid shipping record
9. Sales calculation mismatch → both values retained, discrepancy noted in quality report

---

## Assumptions Made

- First occurrence of exact duplicates is the correct record
- Conflicting duplicate order_ids may represent legitimate corrections or data entry errors — kept both and flagged
- Discount values between 0 and 0.50 are treated as valid without further verification
- Percentage-format discounts (70%, 85%) were intended as decimal values (0.70, 0.85)
- Date formats were inferred from context: values like "06/08/2024" treated as MM/DD/YYYY based on the surrounding data patterns
- "Unknown" for missing region/ship_mode is a placeholder — ideally these should be looked up from the source system

---

## Records Removed

- **20 exact duplicate rows** removed (identical across all 21 columns)
- No other rows deleted — all problematic records retained and flagged

---

## Records Flagged

- **676 Clean** — no issues detected
- **202 Warning** — minor issues: conflicting duplicates, high discounts, cancelled/failed status
- **34 Invalid** — critical issues: negative discounts, ship before order date

---

## Limitations

- Date parsing assumes MM/DD/YYYY for ambiguous formats like "06/08/2024" — some could be DD/MM/YYYY
- Cannot verify whether original `sales` or `calculated_sales` is correct without access to the source billing system
- "Unknown" region/ship_mode is a temporary fix — does not recover the actual missing data
- Conflicting duplicate order_ids were not resolved — they need manual business review
- Rounding in profit_margin calculations may cause minor discrepancies at the fourth decimal
