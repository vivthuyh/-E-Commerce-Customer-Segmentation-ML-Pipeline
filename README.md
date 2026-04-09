# 🛒 E-Commerce Customer Segmentation ML Pipeline

**Course:** INFO 531 — Final Project  
**Author:** Vivian Huynh  
**Language:** Python 3.11

---

## Overview

This project builds a full end-to-end data pipeline — from a raw, denormalized e-commerce CSV all the way to trained and evaluated machine learning models. The goal is to classify customers as **high-value** or regular based on their purchasing behavior, using Logistic Regression and Random Forest classifiers.

The pipeline covers data normalization, star schema design, MySQL integration, feature engineering, and model evaluation.

---

## Pipeline Summary

**0NF CSV → Normalized Line Items → Star Schema (MySQL) → Feature Engineering → ML Models**

1. **Load & clean** the raw 0NF dataset (one row per invoice, multi-valued `Items` field)
2. **Normalize** the `Items` string into a proper 1NF line-item table (one row per product per invoice)
3. **Build a warehouse-style star schema** and load it into MySQL
4. **Engineer customer-level features** from aggregated transaction data
5. **Train and evaluate** Logistic Regression and Random Forest classifiers

---

## Dataset

**File:** `ecom_data_0nf.csv`

The raw data is in 0NF format — each invoice row contains a packed `Items` string encoding multiple products:

```
StockCode|Description|Quantity|UnitPrice;StockCode2|Description2|...
```

| Column | Description |
|---|---|
| `InvoiceNo` | Invoice identifier |
| `CustomerID` | Numeric customer ID |
| `Country` | Customer's country |
| `InvoiceDateFirst` | Invoice timestamp |
| `Items` | Multi-valued string of all products on the invoice |

After normalization, the dataset expands to **~397,884 line-item rows** across **22,190 unique invoices** and **4,338 customers**.

---

## Database Schema (Star Schema)

Three tables are created and populated in a MySQL database (`ECommerce`):

| Table | Grain | Key Columns |
|---|---|---|
| `fact_sales` | One row per invoice | `InvoiceNo`, `CustomerID`, `TotalAmount`, `TotalQuantity` |
| `dim_customer` | One row per customer | `CustomerID`, `Country` |
| `dim_product` | One row per product | `StockCode`, `Description` |

---

## Feature Engineering

Customer-level features are derived from the `fact_sales` table:

| Feature | Description |
|---|---|
| `TotalSpent` | Sum of all invoice amounts |
| `PurchaseCount` | Number of unique invoices |
| `AvgOrderValue` | `TotalSpent / PurchaseCount` |
| `Recency` | Days since the customer's last purchase |
| `Frequency` | Purchases per month over the observed period |
| `Country_*` | One-hot encoded country indicators |

**Target variable:** `CustomerSegment` — binary label where `1` = top 20% of spenders (high-value), `0` = all others.

---

## Models & Results

### Logistic Regression

| Metric | Score |
|---|---|
| Accuracy | 96.6% |
| ROC AUC | 0.995 |

The strongest positive predictors were `TotalSpent`, `PurchaseCount`, `Frequency`, and `AvgOrderValue`. `Recency` had the largest negative coefficient — inactive customers are less likely to be high-value.

### Random Forest Classifier (200 trees)

| Metric | Score |
|---|---|
| Accuracy | 100% |
| ROC AUC | 1.0 |

Feature importances confirm that `TotalSpent` (51%) dominates, followed by `Frequency` and `PurchaseCount`. Country indicators had minimal importance, indicating purchasing behavior is far more predictive than geography.

---

## Setup & Requirements

### Install dependencies

```bash
pip install pandas numpy scikit-learn matplotlib mysql-connector-python
```

### MySQL setup

1. Create a database named `ECommerce` in MySQL Workbench (or via SQL)
2. Update the credentials in the notebook:
   ```python
   mysql_address  = '127.0.0.1'
   mysql_username = 'your_username'
   mysql_password = 'your_password'
   mysql_database = 'ECommerce'
   ```
3. Run all notebook cells in order — tables will be created and populated automatically

> **Note:** MySQL integration is optional. The feature engineering and ML sections run independently of the database steps.

### Run the notebook

```bash
jupyter notebook INFO531_final_project_Huynh.ipynb
```

---

## Files

| File | Description |
|---|---|
| `INFO531_final_project_Huynh.ipynb` | Main project notebook |
| `ecom_data_0nf.csv` | Raw input data in 0NF format |
| `data.csv` | Supporting data file |

---

## Key Takeaways

High-value customers are defined by behavior, not geography. The models consistently show that customers who spend more, buy more frequently, place larger orders, and remain recently active are the strongest candidates for high-value classification. These insights can directly support CRM and marketing strategies such as loyalty programs, personalized promotions, and reactivation campaigns for lapsing customers.
