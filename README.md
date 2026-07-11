# Data Cleaning & Exploratory Data Analysis Projects(SQL + Python)


**Author:** Johanna Ezedinma  
**Date:** July 2026   

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/) 
[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@johannaezedinma) [![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma)

> Turning a raw, real-world datasets into a structured, analysis-ready dataset and uncovering
> patterns and trends through exploratory data analysis.



Raw data rarely arrives clean or ready for modeling. This repository demonstrates practical, real-world data cleaning, 
validation, and exploratory techniques across two distinct operational datasets—highlighting two different engineering 
and analytical workflows:
1. **SQL-first Data Manipulation** using PostgreSQL integrated directly into Python.
2. **Automated & Modular Python Pipelines** using reusable custom helper functions.

---

## Projects Overview

| Project | Dataset | Core Focus | Primary Stack | Quick Links |
| :--- | :--- | :--- | :--- | :--- |
| **1. E-Commerce Transactions** | 541,909 rows (UCI / Kaggle) | Relational SQL cleaning, business rule validation, cross-tool verification | PostgreSQL, Python (`jupysql`, pandas), `plotnine` | [View Project ↗](./project-1-python-sql/README.md) |
| **2. Netflix Titles Catalog** | 8,807 rows (Netflix Titles) | Modular Python automation, custom utility functions, standardized EDA | Python, pandas, Custom Modules (`cleaning_utils`, `eda_utils`), `plotnine` | [View Project ↗](./project-2-python-automation/README.md) |

---

## Key Methodologies & Architectural Highlights

### 1. [E-commerce Transactions: Data Cleaning & EDA (SQL + Python)](./project-1-python-sql/README.md)
* **SQL-First Architecture:** Leveraged PostgreSQL connected via `jupysql` inside Jupyter Notebooks for initial data exploration, whole-row duplicate deletion using `ctid` partitioning, and complex subqueries.
* **Granular Anomaly Handling:** Applied careful domain logic to isolate true business events (such as cancellations and guest checkouts) from bad data, accounting adjustments, or non-product fee rows.
* **Dual-Pipeline Cross-Checking:** Ran a parallel, pure Pandas version (`E_Commerce.ipynb`) alongside the SQL notebook to benchmark and verify row counts, metrics, and aggregate results for 100% accuracy.

### 2. [Netflix Movies & TV Shows: Data Cleaning & EDA (Automated)](./project-2-python-automation/README.md)
* **Custom Python Libraries:** Abstracted repetitive tasks into two reusable Python modules—`cleaning_utils.py` (quality checks, type conversion, intelligent missing value imputation) and `eda_utils.py` (distribution analysis, automated plot selection).
* **Automated Data Quality Checks:** Used single-line function wrappers like `full_quality_check()` to inspect missing values, zero-variance columns, and IQR outliers simultaneously.
* **Positional Bug Fixing:** Identified and resolved shifted-column errors where duration values spilled into rating fields before executing structured string splits and unit standardized transformations.

---

## Technical Stack & Tools

* **Languages & Querying:** SQL (PostgreSQL), Python 3.x
* **Data Processing & Automation:** pandas, NumPy, Custom Python Utilities (`cleaning_utils.py`, `eda_utils.py`)
* **Database Integration:** `jupysql`, `ipython-sql`
* **Data Visualization:** `plotnine` (ggplot2 for Python), Matplotlib

---

## 👤 Author

**Johanna Ezedinma**
[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/) 
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma) 
[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@johannaezedinma)



