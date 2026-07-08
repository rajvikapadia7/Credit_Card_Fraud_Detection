# Credit Card Fraud Detection - SQL & R Analytics

An end-to-end fraud detection analysis using the [IEEE-CIS Fraud Detection dataset](https://www.kaggle.com/c/ieee-fraud-detection) from Kaggle, combining SQL querying, exploratory data analysis, feature engineering, and predictive modeling in a R Markdown report.

## What's in this repo

- `fraud_detection.Rmd` - full analysis source (SQL, EDA, feature engineering, modeling)
- `fraud_detection.html` - rendered report with all outputs and visualizations
- Data files (`train_transaction.csv`, `train_identity.csv`) are not included. Download them from Kaggle and place them in the project folder to reproduce.

## SQL

Uses an in-memory SQLite database to demonstrate:
- JOINs across transaction and identity tables
- Aggregation with `GROUP BY`
- `CASE WHEN` risk-tiering logic
- Correlated subqueries
- Window functions (cumulative fraud count over time)
- CTEs
- A reusable `VIEW` for flagging suspicious transactions

## EDA Highlights

- **Class imbalance**: 96.5% legitimate vs. 3.5% fraudulent transactions
- **Product risk**: Product category `C` shows a notably higher fraud rate than other categories
- **Device risk**: mobile transactions show a higher fraud rate than desktop
- **Missing values as signal**: about 450K rows have missing `DeviceType` (only 24% complete). Rather than dropping or imputing, this was encoded as its own feature (`no_identity`)
- **Time-of-day pattern**: fraud rate spikes noticeably around hour 9, which led to the engineered `is_morning_peak` feature
- **Transaction amount**: the log-scale density plot shows legitimate transactions forming sharper peaks at certain values (consistent with round-dollar amounts), while the fraud distribution is flatter and more spread out

## Feature Engineering

| Feature | Description |
|---|---|
| `hour_of_day`, `is_night`, `is_morning_peak` | Time-based features derived from `TransactionDT` |
| `amt_log`, `amt_ratio`, `is_high_amount`, `is_round_amount` | Transaction amount features |
| `no_identity`, `is_mobile` | Device/identity features |
| `is_product_c` | Product category risk flag |


## Modeling

A logistic regression model was fit to predict `isFraud` using all engineered features: `amt_log`, `amt_ratio`, `is_round_amount`, `is_night`, `is_morning_peak`, `is_high_amount`, `no_identity`, `is_mobile`, and `is_product_c`. The model was trained on an 80/20  split and evaluated using the ROC/AUC.

**Result: AUC = 0.7562**

**Coefficients (from model summary):**

| Feature | Estimate | Significance |
|---|---|---|
| `is_product_c` | 2.747 | p < 0.001 |
| `is_round_amount` | 1.382 | p < 0.001 |
| `is_morning_peak` | 1.081 | p < 0.001 |
| `is_mobile` | 0.424 | p < 0.001 |
| `amt_log` | 0.399 | p < 0.001 |
| `no_identity` | -0.329 | p < 0.001 |
| `is_high_amount` | -0.082 | not significant (p = 0.126) |
| `is_night` | 0.039 | p = 0.022 |
| `amt_ratio` | 0.009 | p = 0.007 |

Positive coefficients indicate higher odds of fraud; negative coefficients indicate lower odds. `is_product_c` and `is_round_amount` are the strongest predictors. 

## Tools

R, RSQLite/DBI, tidyverse, ggplot2, pROC, skimr, R Markdown

## Reproducing this analysis

1. Download `train_transaction.csv` and `train_identity.csv` from the [Kaggle competition page](https://www.kaggle.com/c/ieee-fraud-detection) and place them in the project root.
2. Open `fraud_detection.Rmd` in RStudio or VS Code.
3. Install dependencies:
   ```r
   install.packages(c("rmarkdown", "knitr", "DBI", "RSQLite", "tidyverse", "skimr", "pROC"))
   ```
4. Knit the document to HTML.
