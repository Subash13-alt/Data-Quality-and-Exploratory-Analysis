# Customer Churn: Data Quality and Exploratory Analysis

**Result:** 1 blank row removed; no duplicates, outliers or format issues; 13/13 validation checks pass on both files. Churn is 56.7% (train) vs 47.4% (test). Four fields give 100% churn above a threshold (Monthly contract, Support Calls >= 6, Age >= 56, Payment Delay >= 21), which looks synthetic and will inflate model accuracy. `CustomerID` collides across train and test, so use `customer_key`.

| File | What it is |
|---|---|
| `churn_data_quality_eda.ipynb` | Executed notebook: profile, clean, validate, explore |
| `data_quality_summary.md` | Defensible data-quality report |
| `data_clean/churn_train_clean.csv` | Cleaned training data (440,832 rows) |
| `data_clean/churn_test_clean.csv` | Cleaned testing data (64,374 rows) |
| `data_clean/cleaning_log.csv` | Every cleaning step with before/after row counts |
| `charts/` | Distribution, boxplot, segment, correlation and shift charts |
| `data_raw/` | Original files, untouched |

**Reproduce:** `pip install pandas matplotlib seaborn jupyter`, then run the notebook from this folder.
"# Data-Quality-and-Exploratory-Analysis" 
