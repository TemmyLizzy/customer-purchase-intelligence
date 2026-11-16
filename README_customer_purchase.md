# Customer Repeat Purchase Prediction

A machine learning project that predicts whether an existing online retail customer will make another purchase within the next **30 days**, based on their past transactions.

## Project overview

**Business question:** Can we identify customers who are likely to purchase again in the next 30 days?

I used historical retail transactions to build customer-level features, trained two classification models, and compared their predictive performance. The project focuses on an understandable, reproducible machine learning workflow rather than a production deployment.

## Dataset

- **Source:** [UCI Machine Learning Repository — Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii)
- **Period:** December 2009–December 2011
- **Original dataset:** 1,067,371 transaction lines
- **Eligible purchase lines after initial filtering:** 805,549
- **Customers in the September 1, 2011 snapshot:** 5,249

The original data includes invoice details, products, quantity, date, unit price, customer ID and country. Rows with missing customer IDs, nonpositive quantities, nonpositive prices or cancellation invoices were excluded from the customer-purchase modeling dataset. These rules define eligible purchases and are not a full financial reconciliation of returns.

**Data attribution:** Chen, D. (2012). *Online Retail II*. UCI Machine Learning Repository. [Dataset page and license information](https://archive.ics.uci.edu/dataset/502/online+retail+ii). Dataset license: CC BY 4.0.

The raw and processed datasets are not included in this repository. Download the original Excel workbook from UCI and save it as `data/raw/online_retail_II.xlsx`.

## Method

1. **Explore and clean the data:** inspect missing customer IDs, cancellations, nonpositive quantities and prices, and duplicates. Retain the source data separately.
2. **Create customer features:** use transactions **before September 1, 2011** to calculate five features.
3. **Create the label:** mark whether the customer made an eligible purchase from September 1 through September 30, 2011 (a 30-day horizon).
4. **Train and evaluate:** use a stratified 80/20 random split of the customer snapshot, with `random_state=42`. Compare Logistic Regression and Random Forest.

### Features

| Feature | Meaning |
|---|---|
| `recency_days` | Days since the customer's most recent eligible purchase |
| `frequency` | Number of distinct historical invoices |
| `monetary` | Total historical eligible purchase value |
| `product_diversity` | Number of distinct stock codes purchased |
| `avg_basket_value` | Monetary value divided by purchase frequency |

**Target:** `purchased_next_30d` (1 = purchased again, 0 = no eligible purchase observed).

In this snapshot, **1,077 customers (20.52%)** purchased again, while **4,172 (79.48%)** did not.

## Model results

Performance on the held-out 20% of customers (1,050 customers):

| Metric | Logistic Regression | Random Forest |
|---|---:|---:|
| Accuracy | **0.823** | 0.810 |
| Precision (returning customers) | **0.684** | 0.574 |
| Recall (returning customers) | 0.251 | **0.270** |
| F1-score (returning customers) | 0.367 | 0.367 |
| ROC-AUC | **0.789** | 0.773 |

**Result:** Logistic Regression performed slightly better on ROC-AUC and precision. Random Forest had slightly higher recall. At the default 0.5 decision threshold, both models miss many customers who actually return. A model always predicting “no purchase” would achieve about **79.5% accuracy** on this test set, so accuracy should not be considered in isolation.

## Limitations

- **Single historical snapshot:** The models were trained and tested using customers from one cutoff date. A random customer split does **not** demonstrate performance on a future time period.
- **Missing customer identifiers:** Some transaction lines cannot be assigned to individual customers and are excluded from modeling.
- **Purchase definition:** The analysis filters out cancellations and nonpositive quantities/prices; it does not calculate net revenue after all returns.
- **Invoice and duplicate assumptions:** Distinct invoice numbers are treated as purchases; identical rows have not been blindly dropped.
- **Class imbalance:** Only about one in five customers in this snapshot purchased again within 30 days.

A future improvement would evaluate on later historical cutoff dates and tune the classification threshold on a separate validation set.

## Repository structure

```text
customer-purchase-intelligence/
├── notebooks/
│   ├── 01_data_exploration.ipynb
│   ├── 02_feature_engineering.ipynb
│   └── 03_model_training.ipynb
├── data/
│   ├── raw/
│   └── processed/
├── requirements.txt
├── README.md
└── .gitignore
```

## How to reproduce

Requires Python 3.12.

1. Clone the repository and enter the project directory.
2. Create and activate a virtual environment:

   ```bash
   python3.12 -m venv .venv
   source .venv/bin/activate
   ```

   On Windows (PowerShell): `py -3.12 -m venv .venv`, then `.\.venv\Scripts\Activate.ps1`.

3. Install dependencies and start JupyterLab:

   ```bash
   python -m pip install -r requirements.txt
   jupyter lab
   ```

4. Download the Online Retail II Excel file from UCI and place it at `data/raw/online_retail_II.xlsx`.
5. **From the repository root**, run the notebooks in order:
   - `01_data_exploration.ipynb` → creates `data/processed/eligible_purchases.parquet`
   - `02_feature_engineering.ipynb` → creates `data/processed/customer_features.parquet`
   - `03_model_training.ipynb` → trains and compares the models

The notebooks assume the data files are in the paths shown above.

## Tools

Python, Pandas, NumPy, Matplotlib, Seaborn, scikit-learn, JupyterLab, PyArrow and OpenPyXL.
