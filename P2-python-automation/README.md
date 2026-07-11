# Data Cleaning and EDA (Automated)
**Netflix Movies & TV Shows**


> Using functions to clean and explore the Netflix Movies & TV Shows catalog (8,807 titles). 
**The main approach here is automation**: instead of repeating the same missing-value checks, standardization steps, and
chart-building code by hand for every column, the cleaning and EDA logic was built once as reusable functions,
in two small modules, `cleaning_utils.py` and `eda_utils.py`, and called throughout the notebook.
Where the retail project's main version leans on SQL to do the heavy lifting, this project leans on a custom function library to do it.

**Author:** Johanna Ezedinma  
**Date:** July 2026   

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/) 
[![Medium](https://img.shields.io/badge/Medium-12100E?style=for-the-badge&logo=medium&logoColor=white)](https://medium.com/@johannaezedinma) [![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma)
[![← Back to Main Summary Repository](https://img.shields.io/badge/BACK_TO_MAIN_SUMMARY_REPO←-181717?style=for-the-badge&logo=README&logoColor=white)](../README.md) 

---

## Aim

To clean, validate, and explore a raw dataset properly, and to do it in a way that doesn't have to be rewritten from scratch next time. 
A lot of data cleaning is the same handful of operations repeated on different columns: check for missing values, strip whitespace, fill a gap, plot a distribution. 
Writing that logic once as a function, tested and confirmed correct, then calling it by name everywhere it's needed, cuts down on repeated code and on the chance of applying 
a slightly different (and inconsistent) fix to two columns that needed the same treatment.

---

## Dataset

Raw Dataset: [Netflix Movies & TV Shows Dataset (UCI/Kaggle)]([https://www.kaggle.com/datasets/vijayuv/onlineretail](https://www.kaggle.com/datasets/shivamb/netflix-shows))

- 8,807 rows, 12 columns
- One row per title (movie or TV show)
- `show_id` is a confirmed unique primary key
- Columns: `show_id`, `type`, `title`, `director`, `cast`, `country`, `date_added`, `release_year`, `rating`, `duration`, `listed_in`, `description`

---

## The automation layer

`cleaning_utils.py` and `eda_utils.py` are each a small library, not one-off helpers written just for Netflix. `cleaning_utils.py` documents its own suggested workflow (inspect, clean, validate, then a final all-in-one re-check), 
and `eda_utils.py` includes over a dozen chart types (boxplots, violin plots, scatter and bubble charts, correlation heatmaps, and an `auto_plot` function that picks the right chart type on its own based on the columns you give it). 
Only the subset actually needed for this dataset was used here.

**`full_quality_check(df)`** — runs the entire dataset-understanding checklist in one call: data types, missing values per column, duplicate row count, constant columns, and outliers (via IQR) across every numeric column. What would normally be five or six separate lines becomes one function call.

**`check_unique_values(df, column)`** — prints value counts for a column, used here to catch the shifted-column error in `rating` before it could quietly distort the ratings analysis.

**`strip_whitespace(df, columns=None)`** — strips leading/trailing whitespace from text columns. If no columns are specified, it defaults to every text column in the dataframe automatically, and it's careful to only strip actual strings, so real missing values aren't accidentally turned into the text `"nan"`.

**`convert_to_datetime(df, column, fmt=None)`** — parses a column into a real datetime type. If no format is given, pandas infers it automatically; unparseable values become proper missing values (`NaT`) instead of raising an error.

**`fill_missing(df, column, method='mean')`** — fills missing values in a column. `method` can be `'mean'`, `'median'`, `'mode'`, or any fixed custom value like `"Not Specified"`. Before filling, it also quietly catches stray text like `'nan'`, `'None'`, or empty strings and converts them to true missing values first, so they don't get double-counted or missed. Used here on `director`, `cast`, `country`, and `rating`, each with its own fill value, without writing four separate `.fillna()` calls.

**`summarize(df)`** — prints shape, data types, and full descriptive statistics in one call.

**`check_skewness(df)`** — doesn't just report a skewness number, it classifies the direction and explains what it means in plain language (for example, flagging a column as "highly right-skewed, consider log transformation").

**`plot_density`, `plot_distribution`, `plot_donut`, `plot_line`, `plot_bar`** — each takes a dataframe, the column(s) to plot, and a title, and returns a finished, styled chart, no repeated `ggplot(...) + geom_...` blocks. `plot_distribution` can also overlay mean/median lines and a bell curve for comparison. `plot_donut` uses matplotlib specifically because plotnine has no native pie/donut chart type. Every visualization in this project's EDA section is built from one of these five functions.

---

## Data cleaning

`director` was missing in 2,634 rows, `cast` in 825, `country` in 831, `date_added` in 10, and `rating` in 4. No duplicate rows were found.

**A shifted-column error.** Checking `rating`'s unique values (via `check_unique_values`) turned up three values that don't belong there at all: `"74 min"`, `"84 min"`, `"66 min"`. The corresponding `duration` values for those exact rows were blank. 
The duration had been shifted one column to the left for these three rows. The value was moved back into `duration`, and `rating` was cleared for those rows so it could be filled properly later.

**Missing `director`, `cast`, `country`, `rating`.** Filled with `"Not Specified"`, `"Not Specified"`, `"Unknown"`, and `"Not Rated"` respectively, using `fill_missing`. There's no reliable way to infer who directed a title or where it was produced from the other columns, and dropping a third of the rows to satisfy `director` alone would gut the dataset for every other analysis. Labeling explicitly keeps the gap visible as "unknown" rather than silently missing.

**Missing `date_added` and `duration`.** Left as genuinely missing rather than guessed, since there's no reasonable way to infer either. Two boolean flags, `date_added_missing` and `duration_missing`, mark these rows so any date-based or runtime-based analysis can explicitly account for them instead of accidentally treating a blank as zero or excluding rows silently.

**Splitting `duration`.** `duration` mixes two incompatible units depending on `type` ("90 min" for movies, "2 Seasons" for TV shows), so it was split into a numeric `duration_value` and a `duration_unit` column before any numeric analysis could be run on it.

**A validation check.** Fourteen rows have a `date_added` year earlier than `release_year`. it was investigated and left as-is, this is a legitimate pattern for renewed TV series, where `release_year` reflects the most recent season rather than when the show was first added.

---

## Cleaning summary

| Issue Found | Action Taken |
| :--- | :--- |
| Missing `director` (2,634 rows) | Filled with `"Not Specified"` |
| Missing `cast` (825 rows) | Filled with `"Not Specified"` |
| Missing `country` (831 rows) | Filled with `"Unknown"` |
| Missing `rating` (4 + 3 recovered) | Filled with `"Not Rated"` |
| Missing `date_added` (10 rows) | Left missing, flagged with `date_added_missing` |
| Shifted duration value in `rating` (3 rows) | Corrected, value moved back to `duration` |
| Mixed-unit `duration` | Split into `duration_value` (numeric) + `duration_unit` |
| Duplicate rows | None found |

## Findings and insights

- Movies outnumber TV shows roughly 2-to-1 in the catalog (6,131 movies to 2,676 TV shows).
- The United States and India are by far the two largest content-producing countries, well ahead of every other country in the catalog.
- TV-MA and TV-14 are the two most common ratings, showing the catalog skews toward mature and teen audiences over family-friendly content.
- International Movies, Dramas, and Comedies are the three most common genre tags, with international content well represented across the catalog.
- Movie runtimes cluster tightly around a standard feature-length range, showing the catalog leans on conventional film lengths rather than unusually short or long content.

## Tools

Python, pandas, plotnine, and two custom modules (`cleaning_utils.py`, `eda_utils.py`) used for cleaning and charting steps.

## Files

- `Netflix_Analysis.ipynb`: full notebook, dataset understanding, cleaning, EDA, visualizations
- `cleaning_utils.py`: reusable cleaning functions (quality checks, whitespace stripping, missing-value handling, type conversion)
- `eda_utils.py`: reusable EDA and charting functions (summary stats, skew checks, density/distribution/bar/line/donut plots)
- `netflix_titles_cleaned.csv`: cleaned, analysis-ready dataset


---



## Author

**Johanna Ezedinma**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0077B5?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/johanna-ezedinma/) 
[![GitHub](https://img.shields.io/badge/GitHub-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/Johanna-Ezedinma) 
[![← Back to Main Summary Repository](https://img.shields.io/badge/BACK_TO_MAIN_SUMMARY_REPO←-181717?style=for-the-badge&logo=README&logoColor=white)](../README.md) 
