# Part 1: Business Data Cleaning, Validation & Excel Reporting

**Jahnavi Gurram | bitsom_ba_2511092**

---

## Problem Summary

A retail company exported order-level sales data from multiple internal systems. The raw dataset had several data quality problems — inconsistent text formatting, mixed date formats, duplicate records, missing values, invalid discounts, and calculation mismatches. The task was to clean this data, validate it against business rules, and produce analysis-ready summaries for business review.

---

## Dataset Description

| Property | Value |
|---|---|
| File | raw_orders.xlsx |
| Rows | 932 (raw) → 912 (after deduplication) |
| Columns | 21 |
| Key fields | order_id, order_date, ship_date, customer_name, segment, region, category, sub_category, quantity, unit_price, discount, sales, cost, profit, payment_status, order_status |

---

## Tools Used

- Microsoft Excel / LibreOffice Calc
- Python (pandas, openpyxl) for processing and report generation

---

## Cleaning Steps Performed

1. **Preserved raw data** — original file kept untouched as `data/raw_orders.xlsx`
2. **Cleaned text fields** — trimmed spaces, standardized casing to Title Case, collapsed double spaces across 10 text columns (102 cells corrected)
3. **Standardized dates** — parsed 4 different date formats into a consistent YYYY-MM-DD format
4. **Handled duplicates** — removed 20 exact duplicate rows; flagged 12 conflicting duplicate order_ids for review
5. **Filled missing values** — region (25 → Unknown), ship_mode (21 → Unknown), discount (18 → 0 where valid)
6. **Cleaned discounts** — converted 8 percentage-format values (70%, 85%) to decimals; flagged 15 negative and 15 high (>50%) discounts
7. **Created calculated columns** — cleaned_discount, calculated_sales, calculated_profit, profit_margin, shipping_delay_days, order_month, order_year, data_quality_flag
8. **Applied data quality flags** — Clean (676), Warning (202), Invalid (34)

---

## Business Rules Applied

| Rule | Action |
|---|---|
| Missing region | Filled as "Unknown", flagged in quality report |
| Missing ship_mode | Filled as "Unknown", flagged in quality report |
| Missing discount | Set to 0 only when quantity, unit_price, and sales were valid |
| Negative discount | Flagged as Invalid |
| Discount > 50% | Flagged as Warning |
| Cancelled orders | Excluded from completed sales summaries |
| Failed payments | Excluded from completed sales summaries |
| Refunded orders | Summarized separately |
| Ship date before order date | Flagged as Invalid |

---

## Summary of Data Quality Issues

| Issue | Count |
|---|---|
| Text formatting problems | 102 cells |
| Missing region values | 25 |
| Missing ship_mode values | 21 |
| Missing discount values | 18 |
| Negative discounts | 15 |
| High discounts (>50%) | 15 |
| Percentage-format discounts | 8 |
| Ship date before order date | 21 |
| Exact duplicate rows (removed) | 20 |
| Conflicting duplicate order_ids (flagged) | 12 |
| Sales calculation mismatches (>5%) | 47 |

---

## Summary of Pivot Reports

| Report | Key Finding |
|---|---|
| Sales by Region | South and East regions lead in total sales; "Unknown" region has minimal contribution |
| Sales by Category | Technology generates highest sales; Furniture has the widest sub-category spread |
| Orders by Ship Mode | Standard Class is the most used shipping method |
| Profit Margin by Segment | Segments show varying margin efficiency; useful for pricing strategy |
| Problem Orders by Region | Cancellations and returns distributed across all regions |
| Monthly Trend | Sales show seasonal patterns with peaks in certain months |

---

## Key Business Insights

1. Roughly 15.8% of orders are cancelled and 7.8% of payments are refunded — the operations team should investigate root causes
2. 21 orders have ship dates earlier than order dates, indicating either data entry errors or system timestamp issues in the logistics pipeline
3. Some orders have discounts exceeding 50% (and 8 were entered as "70%" or "85%" in percentage format instead of decimals), suggesting inconsistent discount entry practices across systems
4. There are 47 records where the reported sales doesn't match the calculated value (qty × price × (1-discount)) — this points to potential pricing overrides or manual adjustments that aren't captured in the standard fields
5. Standard Class shipping dominates order count, but profitability analysis by ship mode would help decide whether faster options are worth promoting

---

## Assumptions and Limitations

**Assumptions:**
- First occurrence of exact duplicates is the valid record
- Ambiguous date "06/08/2024" is treated as MM/DD/YYYY based on surrounding data context
- "Unknown" is a reasonable placeholder for missing region/ship_mode

**Limitations:**
- Cannot verify whether original sales or recalculated sales is correct without source billing system access
- Conflicting duplicate order_ids need manual business review to resolve
- Date format inference may be wrong for a small number of ambiguous dates
- "Unknown" does not recover actual missing region or shipping data

---

## Screenshots

| File | Shows |
|---|---|
| screenshots/raw_data_preview.png | Raw dataset before any cleaning |
| screenshots/cleaned_data_preview.png | Cleaned dataset with all calculated columns and quality flags |
| screenshots/pivot_summary_1.png | Sales and profit by region |
| screenshots/pivot_summary_2.png | Monthly sales trend |
