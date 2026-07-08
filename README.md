# E-commerce Transactions: Data Cleaning & EDA

> Turning a raw, real-world transactional dataset (541,909 rows of UK online retail sales, Dec 2010 to Dec 2011) into a structured, analysis-ready dataset and uncovering patterns and trends through exploratory data analysis.

**Author:** Johanna Ezedinma  
**Date:** July 2026   
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/) [![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@johannaezedinma) [![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma) 


Raw Dataset: [Online Retail dataset (UCI/Kaggle)](https://www.kaggle.com/datasets/vijayuv/onlineretail)   
`500K+ transaction records from a UK-based online retailer (Dec 2010 to Dec 2011).`



## Aim

To clean, validate, and explore a raw dataset. Deciding what counts as an error versus a real event, what should be fixed, what should be flagged, and what should be left alone.

## Dataset

- 541,909 rows, 8 columns
- One row per product line, per invoice
- No single-column primary key. `InvoiceNo` and `StockCode` together identify one line item
- Columns: `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country`


## Missing values

`Description` was missing in 1,454 rows.    
`CustomerID` was missing in 135,080 rows; about 25% of the data.   
No other column had missing values.

### *What Was Done*

**Description.** Each `StockCode` represents one specific product, so every row with that code should carry the same description. Since this is in text format where the mean or median can be used, the fix didn't need statistics, but it also wasn't a simple overall "most common word" fix either. Instead, **the most frequent description already recorded for that exact `StockCode` elsewhere in the data was used to fill the gap.**    
This is a grouped lookup: it asks "what is this specific product usually called" rather than "what is the most common description in the whole dataset."   
A stock code is a fixed identifier, so it's a far safer key to fix from than using the free text itself, which varies in spelling and capitalization even for the same product.   
Any stock code with no known description anywhere was labelled `UNKNOWN ITEM` instead of guessed.

**CustomerID.** This was left as missing rather than dropped or imputed.   
A blank customer ID cab be a guest checkout, which is a real, valid transaction, not an error.  
There is no reliable way to guess who a guest was, and dropping a quarter of the data to satisfy this column would have thrown away good transactions.   
A new column, `is_guest_checkout`, flags these rows instead.   
Customer-level analysis (repeat purchases, spend per customer) can filter guests out. Revenue and product-level analysis can still use every row.

---
## Duplicates

Exact duplicate rows (same invoice, same product, same everything) were found and removed. These weren't repeat purchases. A genuine repeat purchase would show up as a separate invoice line, not an identical copy of an existing row. This pattern points to the same line item being recorded twice, most likely an export or scanning glitch, so it was safe to drop the repeat and keep one copy.

---

## Standardisation

Column names were converted to lower_snake_case.   
`InvoiceDate` was loaded as plain text (like `12/1/2010 8:26`) and converted into a real datetime column   
 `Description` and `Country` were stripped of stray whitespace and standardised in case, so the same product or country doesn't get split into several different-looking entries purely because of formatting (`"mug "` versus `"MUG"` versus `"Mug"`).

---

## Validation and anomalies

Looking closely at `Quantity` and `UnitPrice` turned up a few distinct issues that all needed different treatment.

**Cancelled orders.** Invoice numbers starting with `C` carry negative quantities. This is not an error, it's the dataset's own way of marking a cancellation or return. These are real business events, so they were kept, not deleted, and flagged with `is_cancelled` so any analysis can choose to include or exclude them depending on what it's measuring.

**Rows with no legitimate explanation.** A separate set of rows also had negative quantity, but were not flagged as cancellations, had no customer ID attached, and were priced at zero. On their own, none of these three details would justify removing a row. Together, they described a transaction that didn't behave like a cancellation, a guest checkout, or a normal sale.   
This combination pointed to internal stock write-offs rather than customer activity, so these rows were removed.    
> The lesson here: a row should only be removed once multiple independent signals agree, not from a single suspicious value.

**Ledger entries mixed into sales data.** A small number of rows used the stock code `B` for "Adjust bad debt," an accounting entry, not a product being sold. These were removed, since they don't represent a transaction at all.

**Non-product stock codes.** Codes like `POST`, `BANK CHARGES`, and `D` are fully valid, complete rows. Someone really was charged postage or a fee. But they aren't merchandise, and leaving them untagged would let something like postage quietly show up in a "top-selling products" result. These were kept and flagged with `is_product = False`, so product-specific analysis can exclude them without losing the row from the rest of the dataset.

**Zero-price rows.** Some rows have a real, positive quantity, but a unit price of exactly zero, most likely a free or promotional item. The transaction happened, it just generated no revenue.    
These were kept and flagged with `is_zero_price`, so revenue calculations can exclude them if needed, without deleting a real event.

> The pattern across all of these: flag instead of delete, unless a row is confirmed not to represent a real event. Missing or unusual values were rarely noise. They almost always meant something specific (a guest, a cancellation, a fee, a write-off), and erasing the row would have erased that meaning along with it.

---

## Feature engineering

A few columns were added on top of the cleaned data, each one built to serve a specific analysis later on, rather than left as a "maybe useful" afterthought.

- `is_cancelled`, `is_guest_checkout`, `is_product`, `is_zero_price`: boolean flags created during cleaning, described above, so filtering the dataset for a specific analysis is a one-line operation instead of re-deriving the logic every time.
- `total_price` (`quantity` multiplied by `unit_price`): added because revenue isn't provided directly anywhere in the raw data, and nearly every downstream analysis needed it.

---

## EDA summary

With the data cleaned, a set of analyses were run to surface patterns:

- Top-selling products by quantity
- Highest revenue-generating countries
- Monthly sales trend
- Most purchased products by number of orders
- Customer purchasing behaviour among identified customers

## Findings and insights

- The UK generates roughly 30 times the revenue of the next closest country, the Netherlands, showing a customer base heavily concentrated in the home market.
- Monthly revenue nearly doubles between the summer months and an autumn peak in September through November, a seasonal build-up tied to Christmas gift buying.
- A small set of products, like Paper Craft Little Birdie and Medium Ceramic Top Storage Jar, are bought in far larger quantities per order than typical items, pointing to wholesale or reseller buyers alongside individual customers.
- Order value is heavily right-skewed. Most orders are modest, but a long tail of large orders pulls the average well above the median, so average order value alone is a misleading number for this business.
- About 25% of transactions have no identifiable customer, which limits how much customer-level analysis, like repeat purchase rate or lifetime value, the full dataset can support.

---

## Tools

Python, pandas, plotnine

## Files

- `E_Commerce.ipynb`: full notebook covering dataset understanding, cleaning, EDA, and visualisations
- `online_retail_cleaned.csv`: cleaned, analysis-ready dataset


## Author

**Johanna Ezedinma**

[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma) [![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/)  

