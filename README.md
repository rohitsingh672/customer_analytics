# 🛒 Customer Transaction & Behavior Analysis

> End-to-end customer analytics pipeline — RFM segmentation, purchase pattern analysis, and churn risk scoring using Python and JupyterLab.

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Dataset](#dataset)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [How to Run](#how-to-run)
- [Methodology](#methodology)
- [Results & Key Findings](#results--key-findings)
- [Output Files](#output-files)
- [Troubleshooting](#troubleshooting)
- [Tech Stack](#tech-stack)

---

## Project Overview

This project analyzes e-commerce customer transaction data to answer three core business questions:

1. **Who are our customers?** — Segment customers into meaningful groups based on behavior
2. **How do they buy?** — Identify purchase patterns across time, category, and demographics
3. **Who is at risk of leaving?** — Score every customer on churn probability

The full pipeline runs inside a single JupyterLab notebook (`customer_analytics.ipynb`) and produces a visual dashboard, segment profiles, and exportable CSV reports.

---

## Dataset

| File | Rows | Description |
|------|------|-------------|
| `ecommerce_customer_data_large.csv` | 250,000 | Primary transaction log |
| `ecommerce_customer_data_custom_ratios.csv` | 250,000 | Extended dataset with custom ratios |

Both files share the same 13-column schema and are **automatically merged** in the notebook.

### Column Reference

| Column | Type | Description |
|--------|------|-------------|
| `Customer ID` | int | Unique customer identifier |
| `Purchase Date` | datetime | Timestamp of transaction |
| `Product Category` | str | Electronics / Clothing / Home / Books |
| `Product Price` | int | Unit price of item |
| `Quantity` | int | Units purchased |
| `Total Purchase Amount` | int | Transaction value ($) |
| `Payment Method` | str | Credit Card / PayPal / Cash |
| `Customer Age` | int | Age at time of purchase |
| `Returns` | float | Number of returns (nullable) |
| `Customer Name` | str | Full name |
| `Age` | int | Customer age (same as Customer Age) |
| `Gender` | str | Male / Female |
| `Churn` | int | Binary churn label (0 = active, 1 = churned) |

---

## Project Structure

```
📁 project/
│
├── 📓 customer_analytics.ipynb          ← Main analysis notebook (run this)
├── 📄 README.md                         ← You are here
│
├── 📄 ecommerce_customer_data_large.csv          ← Dataset 1
├── 📄 ecommerce_customer_data_custom_ratios.csv  ← Dataset 2
│
└── 📁 outputs/  (generated after running the notebook)
    ├── 🖼️  customer_analytics_dashboard.png   ← 8-panel visual dashboard
    ├── 📄 customer_rfm_segments.csv           ← Full RFM table per customer
    ├── 📄 segment_profile.csv                 ← Segment-level summary stats
    ├── 📄 high_churn_risk_customers.csv        ← High-risk customer list
    └── 📄 monthly_revenue_trend.csv            ← Monthly revenue time series
```

---

## Installation

### Prerequisites

- Python 3.7+
- JupyterLab or Jupyter Notebook
- Anaconda (recommended) or pip

### Install Dependencies

```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyterlab
```

Or with conda:

```bash
conda install pandas numpy matplotlib seaborn scikit-learn jupyterlab
```

> **pandas version note:** If your pandas version is below 2.2.0, the notebook automatically uses `'M'` instead of `'ME'` for monthly resampling. If you see a `ValueError: Invalid frequency: ME` error, replace all `resample('ME')` with `resample('M')` in the notebook.

---

## How to Run

### Step 1 — Place files in the same folder

Put all three files in the same directory:
```
customer_analytics.ipynb
ecommerce_customer_data_large.csv
ecommerce_customer_data_custom_ratios.csv
```

### Step 2 — Launch JupyterLab

```bash
jupyter lab
```

Or Jupyter Notebook:

```bash
jupyter notebook
```

### Step 3 — Open and run the notebook

Open `customer_analytics.ipynb` and run:

```
Kernel → Restart & Run All
```

### Using a custom file path (Windows)

If your CSVs are in a different folder, update the path variables in **Step 2** of the notebook:

```python
FILE_1 = r'C:\Users\YourName\Downloads\ecommerce_customer_data_large.csv'
FILE_2 = r'C:\Users\YourName\Downloads\ecommerce_customer_data_custom_ratios.csv'
```

> The `r` prefix before the string is required on Windows to handle backslashes correctly.

---

## Methodology

### Step 1 — Data Loading & Exploration
Both CSV files are loaded, merged with `pd.concat()`, and deduplicated. Basic statistics, null counts, and categorical distributions are printed.

### Step 2 — Cleaning & Feature Engineering
- `Purchase Date` parsed as datetime
- `Returns` null values filled with 0
- Duplicate `Customer Age` column dropped
- Derived columns: `Year`, `Month`, `Hour`, `DayOfWeek`, `Age_Group`

### Step 3 — RFM Analysis
Each customer is scored on three dimensions computed from their transaction history:

| Metric | Formula | Interpretation |
|--------|---------|----------------|
| **Recency** | Days since last purchase | Lower = more recent = better |
| **Frequency** | Total number of transactions | Higher = more engaged |
| **Monetary** | Sum of all purchase amounts | Higher = more valuable |

Each metric is then scored 1–5 using quintile binning, and an overall `RFM_Score` (3–15) is calculated.

### Step 4 — K-Means Clustering
- Features scaled with `StandardScaler`
- Optimal k selected using **Elbow Method** and **Silhouette Score**
- Final model: **k = 4 clusters**
- Clusters auto-labeled by ranking their combined RFM score

### Step 5 — Churn Risk Scoring
A composite 0–1 churn risk score is computed per customer using weighted features:

| Feature | Weight | Rationale |
|---------|--------|-----------|
| Recency (normalized) | 40% | Inactive customers are most likely to churn |
| Inverse Frequency | 30% | Low purchase frequency signals disengagement |
| Inverse Monetary | 20% | Low spenders have less loyalty |
| Return Rate | 10% | High returns indicate dissatisfaction |

Customers are then labeled **Low / Medium / High** risk using tertile thresholds.

### Step 6 — Purchase Pattern Analysis
- Monthly revenue trend (time series)
- Revenue by product category
- Order volume by day of week and hour of day
- Category × Segment revenue heatmap
- Payment method distribution by segment
- Average order value by age group

---

## Results & Key Findings

### Customer Segments

| Segment | Customers | Avg LTV | Revenue Share | Avg Recency |
|---------|-----------|---------|---------------|-------------|
| 🏆 Champions | 8,236 | $24,491 | 29.6% | 140 days |
| 💚 Loyal Customers | 18,638 | $15,703 | 43.0% | 181 days |
| ⚠️ At-Risk | 15,199 | $8,407 | 18.8% | 198 days |
| 😴 Hibernating | 7,588 | $7,802 | 8.7% | 723 days |

### Churn Risk

| Risk Level | Customers | Share |
|------------|-----------|-------|
| 🟢 Low | 3,777 | 7.6% |
| 🟡 Medium | 42,239 | 85.1% |
| 🔴 High | 3,645 | 7.3% |

### Key Insights

- **Loyal Customers** generate **43% of total revenue** despite not being the highest individual spenders — they are the most important segment to protect
- **Champions** have 2× higher lifetime value ($24K) than the average customer — reward them with VIP treatment
- **7,588 Hibernating customers** with previously high LTV represent the highest-value win-back opportunity
- **15,199 At-Risk customers** need nurturing before they go fully dormant
- Revenue is **evenly distributed** across Electronics, Clothing, Home, and Books — no single category dependency or risk
- Overall churn rate is **~20% uniformly across all segments**, suggesting a systemic experience issue rather than a segment-specific one

### Actionable Recommendations

| Segment | Action |
|---------|--------|
| 🏆 Champions | VIP loyalty program, early product access, referral incentives |
| 💚 Loyal Customers | Upsell campaigns, personalized recommendations, member-only discounts |
| ⚠️ At-Risk | Win-back emails, discount codes, satisfaction surveys |
| 😴 Hibernating | Heavy re-engagement offers, bundle deals — prioritize high-LTV customers first |

---

## Output Files

| File | Description |
|------|-------------|
| `customer_analytics_dashboard.png` | 8-panel master dashboard with KPI cards, segment donut, revenue charts, scatter plots, churn risk bars, trend line, category heatmap |
| `customer_rfm_segments.csv` | One row per customer with RFM scores, cluster, segment label, churn risk score and label |
| `segment_profile.csv` | Aggregated stats per segment: customer count, avg recency/frequency/LTV, return rate, churn rate, revenue share |
| `high_churn_risk_customers.csv` | High-risk customers sorted by lifetime value — ready for CRM upload |
| `monthly_revenue_trend.csv` | Month-by-month revenue in $M — ready for BI tools |

---

## Troubleshooting

### `FileNotFoundError: No such file or directory`
The notebook cannot find the CSV files. Either:
- Place both CSVs in the **same folder** as the notebook, or
- Update `FILE_1` and `FILE_2` in Step 2 to use the full file path

```python
# Windows example
FILE_1 = r'C:\Users\YourName\Downloads\ecommerce_customer_data_large.csv'
FILE_2 = r'C:\Users\YourName\Downloads\ecommerce_customer_data_custom_ratios.csv'

# Mac/Linux example
FILE_1 = '/home/yourname/downloads/ecommerce_customer_data_large.csv'
FILE_2 = '/home/yourname/downloads/ecommerce_customer_data_custom_ratios.csv'
```

### `ValueError: Invalid frequency: ME`
Your pandas version is below 2.2.0. Replace all instances of `resample('ME')` with `resample('M')` in the notebook (appears in Step 7 and Step 8).

### `ModuleNotFoundError`
Install the missing library:
```bash
pip install <library-name>
```

---

## Tech Stack

| Library | Version | Purpose |
|---------|---------|---------|
| `pandas` | ≥ 1.3 | Data loading, cleaning, aggregation |
| `numpy` | ≥ 1.21 | Numerical operations |
| `matplotlib` | ≥ 3.4 | All charts and dashboard |
| `seaborn` | ≥ 0.11 | Heatmap visualization |
| `scikit-learn` | ≥ 0.24 | K-Means clustering, StandardScaler, Silhouette Score |
| `jupyterlab` | ≥ 3.0 | Interactive notebook environment |

---

*Generated from 500,000 transaction records across 2 datasets | Jan 2020 – Sep 2023*
