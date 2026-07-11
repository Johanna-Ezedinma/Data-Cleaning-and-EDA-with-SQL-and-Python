# Data Cleaning and EDA (SQL + Python)
**E-commerce Transactions**

> Turning a raw, real-world transactional dataset (541,909 rows of UK online retail sales, Dec 2010 to Dec 2011) into a structured, analysis-ready dataset and uncovering patterns and trends through exploratory data analysis.

**Author:** Johanna Ezedinma  
**Date:** July 2026   

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/) [![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@johannaezedinma) [![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma)
[![← Back to Main Summary Repository](https://img.shields.io/badge/BACK_TO_MAIN_SUMMARY_REPO←-181717?style=for-the-badge&logo=README&logoColor=white)](../README.md) 



---
## Approach

**The main approach is SQL** (PostgreSQL, connected through Python via `jupysql`), used for the data understanding, cleaning, and validation. Python and `plotnine` were used to turn the SQL query results into charts.    
A second, fully pandas-based version of the same project sits alongside it as a supplementary reference, cleaned and analyzed entirely in Python with no database involved. Both were checked against each other to confirm they agree, see the note at the bottom.

## Aim

To clean, validate, and explore a raw dataset. Raw data rarely comes with a rulebook. Most of the real work here was applying judgement calls that don't have a textbook answer: deciding what counts as an error versus a real event, what should be fixed, what should be flagged, and what should be left alone. Every decision below is explained in enough detail that it can be applied again to a similar dataset.

## Dataset

Raw Dataset: [E-commerce Transactions Dataset (Kaggle)](https://www.kaggle.com/datasets/vijayuv/onlineretail)   

- 541,909 rows, 8 columns
- One row per product line, per invoice
- No single-column primary key. `invoice_no` and `stock_code` together identify one line item
- Columns: `invoice_no`, `stock_code`, `description`, `quantity`, `invoice_date`, `unit_price`, `customer_id`, `country`

## Missing values

`description` missing in 1,454 rows.    
`customer_id` missing in 135,080 rows, about 25% of the data.   
No other column had missing values.

**description.** Each `stock_code` represents one specific product, so every row with that code should carry the same description. The fix used a grouped lookup: for each `stock_code`, the most frequently recorded description elsewhere in the data was pulled in to fill the gap, using a subquery that ranks each code's descriptions by frequency and keeps the top one. A stock code is a fixed identifier, so it's a far safer key to fix from than the free text itself, which varies in spelling and capitalization even for the same product. Any stock code with no known description anywhere was labelled `UNKNOWN_ITEM`.

**customer_id.** A blank customer ID represents a guest checkout; a real, valid transaction, not an error. There's no reliable way to guess who a guest was, so rather than dropping a quarter of the data, these rows were filled with a standard placeholder, `'GUEST'`.

---

## Duplicates

Exact duplicate rows (same invoice, same product, same everything) were identified using `ROW_NUMBER()` partitioned across every column, then removed via a `ctid`-based delete. 5,268 duplicates were found and removed. These weren't repeat purchases, a genuine repeat purchase would show up as a separate invoice line, not an identical copy of an existing row. This pattern points to the same line item being recorded twice, most likely an export or scanning glitch, so it was safe to drop the repeat and keep one copy.

---

## Standardisation

Whitespace was trimmed and `country` was standardised into consistent Title Case using `INITCAP(TRIM(country))`, so the same country doesn't get split into several different-looking entries purely because of formatting (`"united kingdom"` versus `"United Kingdom"` versus `"UNITED KINGDOM "`).

## Validation and anomalies

Looking closely at `quantity` and `unit_price` turned up a few distinct issues that all needed different treatment.

**Cancelled orders.** Invoice numbers starting with `C` carry negative quantities. Which is not an error, it's the dataset's own way of marking a cancellation or return. These are real business events, so they were kept, not deleted, and flagged with `is_cancelled` so any analysis can choose to include or exclude them depending on what it's measuring.

**Rows with no legitimate explanation.** A separate set of rows also had negative quantity, but were not cancellations (`invoice_no` didn't start with `C`). Checking further, all of these had no customer ID and a unit price of exactly zero. On their own, none of these three details would justify removing a row. Together, they described a transaction that didn't behave like a cancellation, a guest checkout, or a normal sale. 1,336 such rows were found and removed, since they looked like internal stock write-offs rather than customer activity. 
*The lesson here: a row should only be removed once multiple independent signals agree, not from a single suspicious value.*

**Ledger entries mixed into sales data.** 3 rows used the stock code `B` for "Adjust bad debt," an accounting entry, not a product being sold (one with a large positive value and two with large negative values, the debt and its reversal). These were removed, since they don't represent a transaction at all.

**Non-product stock codes.** A first attempt at flagging these used a pattern rule, anything that wasn't "5 digits, optionally followed by letters," was flagged as not a real product. That rule turned out to be too broad. Checking every code the pattern caught, most were genuine non-products (`POST`, `DOT`, `M`, `C2`, `D`, `BANK CHARGES`, `S`, `AMAZONFEE`, `CRUK`, `PADS`), but a handful were real products that simply used an unusual code format, `DCGSSGIRL` ("Girls Party Bag"), `DCGSSBOY` ("Boys Party Bag"), the `GIFT_0001_*` gift card codes, and several `DCGS00xx` codes.   

Once verified, the flag was rebuilt using only the confirmed list of 10 genuine non-product codes, so real products with unconventional codes stay counted, and only fees, postage, and adjustments get excluded. These are still valid, complete rows (someone really was charged postage), so they're kept and flagged with `is_product = FALSE`, letting product-specific analysis exclude them without losing the row from the rest of the dataset.

**Zero-price rows.** Some rows have a real, positive quantity, but a unit price of exactly zero, most likely a free or promotional item. The transaction happened, it just generated no revenue. These were kept and flagged with `is_zero_price`.

The pattern across all of these: flag instead of delete, unless a row is confirmed not to represent a real event, and verify what a rule actually catches before trusting it, especially pattern-based rules, which can misclassify real data that just happens to look unusual.

## Data Cleaning Summary

| Issue Found | Action Taken |
| :--- | :--- |
| **Missing Values** | `description` filled using the most frequent description already recorded for that `stock_code`; unmatched codes labelled `'UNKNOWN_ITEM'`. `customer_id` filled with `'GUEST'` for guest checkouts. |
| **Duplicates** | Removed 5,268 exact whole-row duplicates using a `ctid` row-partitioning sequence. |
| **Invalid Entries** | Removed 1,336 non-cancellation rows with negative quantity, no customer, and $0 price. Removed 3 'Adjust bad debt' ledger rows. Flagged genuine cancellations (`is_cancelled`), zero-price promotional items (`is_zero_price`), and confirmed non-product codes (`is_product = FALSE`) based on an explicit, individually verified list. |
| **Standardization** | Trimmed whitespace and standardised `country` to Title Case. |

---

## EDA summary

With the data cleaned, a set of analyses were run in SQL, then pulled into Python for visualization:

- Highest revenue-generating countries
- Top-selling products by total revenue
- Most purchased products by volume
- Distribution of order sizes (transaction quantities)
- Monthly sales revenue trend

## Findings and insights

- The UK generates roughly 30 times the revenue of the next closest country, the Netherlands, showing a customer base heavily concentrated in the home market.
- Revenue and volume aren't driven by the same products. "Regency Cakestand 3 Tier" is the single highest revenue-generating product, but it doesn't appear in the top 5 by units sold. "Medium Ceramic Top Storage Jar" and "Paper Craft, Little Birdie" drive volume instead, so a few premium items and a few high-volume items are doing different jobs for the business.
- Order sizes are heavily right-skewed, most order lines are for 1 to 12 units, dropping off sharply after that, consistent with individual retail purchases rather than wholesale buying.
- Monthly revenue is fairly stable through the first three quarters of the year, then rises sharply from September and peaks in November, a clear seasonal build-up tied to Christmas gift buying. December's drop reflects the data window ending mid-month, not an actual decline.
- About 25% of transactions have no identifiable customer, which limits how much customer-level analysis, like repeat purchase rate or lifetime value, the full dataset can support.

---
## Cross-checking the two versions

Both notebooks were compared number by number to confirm they agree: row counts, missing value counts, duplicate counts, invalid-row counts, and top products by revenue and by volume all matched exactly. Country revenue initially differed by about $1,084 because of the pattern-based `is_product` flag described above; after correcting it to the verified explicit list, both versions land within a few pounds of each other on total UK revenue (a gap explained by ordinary floating-point rounding differences between Postgres `NUMERIC` and pandas `float64`, not a methodology issue).

## Tools

PostgreSQL, SQL (via `jupysql`/`ipython-sql`), Python, pandas, plotnine

## Files

- `Retail_Data_Cleaning_and_Exploration_with_SQL_via_Python.ipynb`: main notebook, SQL-based cleaning and analysis, Python/plotnine for charts
- `E_Commerce.ipynb`: supplementary notebook, the same dataset cleaned and analyzed entirely in pandas
- `online_retail_cleaned.csv`: cleaned, analysis-ready dataset

---



## Author

**Johanna Ezedinma**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/) [![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma) 
[![← Back to Main Summary Repository](https://img.shields.io/badge/BACK_TO_MAIN_SUMMARY_REPO←-181717?style=for-the-badge&logo=README&logoColor=white)](../README.md) 
