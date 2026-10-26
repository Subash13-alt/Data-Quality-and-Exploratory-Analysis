# Data Quality Summary: Customer Churn

**Files audited:** `customer_churn_dataset-training-master.csv` (440,833 rows) and `customer_churn_dataset-testing-master.csv` (64,374 rows), 12 columns each.
**Method:** full pipeline in `churn_data_quality_eda.ipynb`; raw files in `data_raw/` are never modified.

## 1. Verdict

The data is **structurally clean**: only one defect (a blank row), no duplicates, no formatting variants, no outliers, and every value is inside its plausible range. The real risks are not missing or dirty values. They are **(a)** `CustomerID` is not a shared key across the two files, **(b)** the train and test files describe noticeably different populations, and **(c)** several fields produce a 100% churn rate above a threshold, which looks synthetic and would inflate any model's apparent accuracy.

## 2. Profile results

| Check | Train | Test | Result |
|---|---|---|---|
| Rows (raw) | 440,833 | 64,374 | |
| Fully blank rows | **1** | 0 | Dropped |
| Other missing values | 0 | 0 | Pass |
| Exact duplicate rows | 0 | 0 | Pass |
| Duplicate `CustomerID` within file | 0 | 0 | Pass |
| Duplicates ignoring ID | 0 | 0 | Pass |
| Whitespace / case variants in categoricals | 0 | 0 | Pass |
| IQR outliers (7 numeric columns) | 0 | not run (all values in range) | Pass |
| Rows (clean) | **440,832** | **64,374** | |

Valid ranges observed: Age 18-65, Tenure 1-60 months, Usage Frequency 1-30, Support Calls 0-10, Payment Delay 0-30 days, Last Interaction 1-30 days, Total Spend 100-1,000. Categories: Gender (Male/Female), Subscription Type (Basic/Standard/Premium), Contract Length (Monthly/Quarterly/Annual).

## 3. Cleaning decisions and assumptions

| ID | Issue | Action | Why |
|---|---|---|---|
| A1 | One row where all 12 fields are null (train) | **Dropped** | Nothing to impute from; filling it would fabricate a customer. 440,833 -> 440,832 |
| A2 | Train columns read as `float64` | Cast count-like fields to `int64` | Floats existed only because of the blank row; all values were whole numbers |
| A3 | Categorical hygiene | Strip, Title Case, `category` dtype | No variants found; applied defensively so the pipeline is reusable |
| A4 | Duplicates | `drop_duplicates` on rows and ID | Safety net; removed 0 rows |
| A5 | `CustomerID` collides across files | Added `customer_key` (`train_2`, `test_2`, ...) and a `source` column | 62,995 IDs appear in both files, but only ~0.01% of those pairs match on age, gender and tenure, so they are different people |
| A6 | Outliers | **Kept** | None by the IQR rule; all values are plausible business values |
| A7 | `Total Spend` kept as float | No rounding | Values carry decimals (about 54% of rows); likely a currency amount (unit not documented) |

**Assumption to confirm:** we assumed the two files are separate extracts, not one population. Do not concatenate them and treat `CustomerID` as unique.

## 4. Validation checks (all PASS on both files)

No missing values; `customer_key` unique; every numeric column inside its expected range; `Churn` only 0/1; categorical values only from the allowed sets; row counts reconcile (440,833 -> 440,832 and 64,374 -> 64,374).

## 5. Initial patterns (training file, n = 440,832)

**Overall churn: 56.7%** (test file: 47.4%).

| Segment | Churn rate |
|---|---|
| Contract: Monthly | **100%** |
| Contract: Quarterly / Annual | 46% / 46% |
| Support Calls 6-10 | **100%** (0-2 calls: 31%) |
| Age 56-65 | **100%** (36-45: 43%) |
| Payment Delay 21-30 days | **100%** (0-20 days: 47%) |
| Gender: Female / Male | 67% / 49% |
| Subscription: Basic / Standard / Premium | 58% / 56% / 56% (weak signal) |
| Tenure bands | 54%-63% (weak signal) |

Correlation with churn: Support Calls +0.57, Total Spend -0.43, Payment Delay +0.31, Age +0.22, Last Interaction +0.15; Tenure and Usage Frequency are near zero. Churned customers average 5.1 support calls vs 1.6 for retained, and spend 541 vs 750 on average.

## 6. Risks and caveats

1. **Hard churn thresholds (looks synthetic).** Any one of Monthly contract, Support Calls >= 6, Age >= 56 or Payment Delay >= 21 gives 100% churn. Together they cover 47.6% of customers. Among the remaining 52.4%, churn is 17.4%. A model will score very high by learning these cut-offs, so report metrics on the "none of the four rules" group too, and do not read the 100% figures as real-world behaviour.
2. **Train/test shift.** Test customers make 50% more support calls (5.4 vs 3.6), pay 32% later (17.1 vs 13.0 days), spend 14% less, and churn less (47.4% vs 56.7%). Evaluating a model trained on train against test will mix model error with population change.
3. **Gender gap is large (67% vs 49%).** Treat it as a descriptive finding only; do not use it as a decision input without a fairness review.
4. **No data dictionary was supplied** for the churn guide, so field meanings (units, whether Tenure is months) are inferred from names and ranges.

## 7. Deliverables

- `churn_data_quality_eda.ipynb` (executed notebook, reproducible top to bottom)
- `data_clean/churn_train_clean.csv`, `data_clean/churn_test_clean.csv`, `data_clean/cleaning_log.csv`
- `charts/` (6 PNGs)
- `data_raw/` (original files, unchanged)
