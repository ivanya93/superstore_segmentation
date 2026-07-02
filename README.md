# Superstore Analytics Project

**Business Statistics & Insights — Master in Business Analytics & AI**  
Porto Business School · Team 4

> *Which customer segments create the most value, and how should the organisation allocate its marketing budget to maximise profitability?*

---

## Team

| Name | Role |
|------|------|
| Ivana Caridad Lovera Ruiz | Data Analysis, Segmentation, CAC, Dashboard |
| Linda | Statistical Inference |
| Diogo Ruas | — |
| Daniel Santos | — |

**Professor:** Pedro José Ramos Moreira Campos

---

## Key Results

| Metric | Value |
|--------|-------|
| Total Sales | $2.30M |
| Total Profit | $286K |
| Overall Margin | 12.5% |
| Customers | 793 |
| Avg CAC | $18.51 |
| Avg LTV:CAC | 62× |
| Discount Cost | $567K |
| Champions (top segment) | 104 customers · 206× LTV:CAC |

---

## Deliverables

| Deliverable | Link / Path |
|------------|-------------|
| 📊 Looker Studio Dashboard | [Open Dashboard](https://datastudio.google.com/s/hrlTCDBNtw8) |
| 📄 Written Report | `reports/superstore_analytics_report.docx` |
| 🌐 Interactive HTML Dashboard | `reports/superstore_analytics_dashboard.html` |

---

## Project Structure

```
superstore_dataset/
│
├── notebooks/
│   ├── 00_synthetic_data_generation.ipynb   # CAC & channel data generation
│   ├── 01_data_preparation.ipynb            # Cleaning, feature engineering
│   ├── 02_descriptive_stats.ipynb           # EDA, discount cliff, seasonality
│   ├── 03_statistical_inference.ipynb       # Non-parametric tests (Ivana)
│   ├── 03_statistical_inference_merged.ipynb# Merged with Linda's notebook
│   └── 04_segmentation.ipynb               # RFM, K-Means, CLV, CAC analysis
│
├── data/
│   └── processed/
│       ├── superstore_clean.parquet         # Cleaned base dataset (9,994 rows)
│       ├── superstore_enriched.parquet      # + Discount_Cost, channel, clusters
│       ├── customer_acquisition.parquet     # CAC & channel per customer (793 rows)
│       ├── monthly_channel_spend.parquet    # Monthly spend by channel (288 rows)
│       └── rfm_cluster_profile.csv          # Cluster centroids & labels
│       └── bigquery_exports/               # Parquet files uploaded to BigQuery
│
├── reports/
│   ├── superstore_analytics_report.docx    # Full 15-page academic report
│   ├── superstore_analytics_dashboard.html # Self-contained interactive dashboard
│   └── Superstore_Analytics_Report.pdf
│
├── docs/
│   └── project_roadmap.md
│
└── requirements.txt
```

---

## Analysis Pipeline

### 1 · Data Preparation (`01_data_preparation.ipynb`)
- Zero nulls, zero duplicates confirmed
- 9 engineered features: Profit Margin, Delivery Days, Year/Quarter/Month, Discount Category, Discount_Cost, State_abb
- `Discount_Cost = (Sales / (1 − Discount)) × Discount` → $567K total revenue foregone

### 2 · Synthetic Marketing Data (`00_synthetic_data_generation.ipynb`)
The Superstore dataset contains no acquisition channel or spend data. Three synthetic tables were generated to enable CAC and LTV:CAC analysis, calibrated to industry benchmarks and clearly labelled as modelled estimates:

| Table | Rows | Description |
|-------|------|-------------|
| `customer_acquisition` | 793 | Channel, CAC, campaign per customer |
| `monthly_channel_spend` | 288 | 6 channels × 48 months with ROAS |
| `superstore_enriched` | 9,994 | Transactions + channel + cluster labels |

CAC distributions by channel: Direct $3.50, Email $6.00, Organic $10.00, Social $22.00, Paid Search $28.00, Display $40.00.

### 3 · Descriptive Statistics (`02_descriptive_stats.ipynb`)
- Discount cliff: profitability inverts sharply above 20% (margin drops from +17.4% → −16.7%)
- Q4 accounts for 40–45% of annual sales every year
- Tables sub-category: $207K revenue, −$17.7K profit (−8.6% margin)

### 4 · Statistical Inference (`03_statistical_inference_merged.ipynb`)
All tests non-parametric (normality rejected for all variables, p < 0.001):

| Test | Result | Finding |
|------|--------|---------|
| Kruskal-Wallis — Profit by Region | H = 227.9, p < 0.001 | West > Central significantly |
| Kruskal-Wallis — Sales by Segment | p = 0.710 | No significant difference |
| Spearman — Discount vs Profit Margin | ρ = −0.645, p < 0.001 | Strong negative relationship |
| Chi-Square — Segment × Category | p = 0.834, V = 0.009 | No association |
| Chi-Square — RFM × Discount Band | p < 0.001 | RFM segments differ behaviourally |

Bonferroni correction applied to all pairwise post-hoc tests.

### 5 · Customer Segmentation & CAC (`04_segmentation.ipynb`)

**RFM Segmentation**
- Reference date: 2018-01-01
- Monetary = total profit (not revenue)
- Quintile scoring (1–5) per dimension; composite score 3–15

**K-Means Clustering**
- Outlier detection: Tukey IQR fences
- Winsorisation applied (records kept, values capped)
- StandardScaler for equal feature weighting
- k = 5 selected by Silhouette score (evaluated k = 3–10)

**RFM Segment Profiles**

| Segment | Count | Avg Profit | LTV:CAC | Avg CAC |
|---------|-------|-----------|---------|---------|
| Champions | 104 | $818 | 206× | $19.28 |
| Loyal Customers | 255 | $529 | 93× | $18.47 |
| Potential Loyalists | 257 | $153 | 16× | $18.70 |
| Needs Attention | 150 | $3 | 4× | $18.31 |
| At Risk | 27 | −$206 | −27× | $15.30 |

**Key finding:** Consumer / Corporate / Home Office segments split ~50/30/20 identically across all 5 K-Means clusters — administrative segments carry no behavioural signal.

---

## BigQuery Setup

**Project:** `superstore-dataset-pbs` · **Dataset:** `superstore`

| Table | Rows | Description |
|-------|------|-------------|
| `enriched_with_clusters` | 9,994 | Transactions + all derived columns |
| `customer_master` | 793 | Customer-level RFM, CAC, cluster |
| `monthly_channel_spend` | 288 | Channel spend & ROAS by month |
| `rfm_cluster_profile` | 5 | Cluster centroids |

**Views (use these in Looker Studio — raw tables have column names with spaces):**

```sql
-- Fix for Looker Studio "invalid field name" errors
v_enriched          → aliases all spaced column names to snake_case
v_customer_master   → same for customer_master
```

---

## Recommendations

**1. Cap all discounts at 20%**
Projected margin recovery: $50–70K/year. Eliminate discounts entirely on Tables, Bookcases, Supplies.
Measurement: negative-margin order share below 5% within two quarters.

**2. Reallocate 30% of Display budget to Email and Direct**
Email: $5.97 CAC / 101× LTV:CAC vs Display: $40.15 CAC / 30× LTV:CAC — 6.7× efficiency gap.
Measurement: blended CAC below $15 within one year (current: $18.51).

**3. Re-engagement campaign for 177 lapsed customers**
Needs Attention (150) + At Risk (27). 20% reactivation target → ~$4,500 incremental profit.
Measurement: 90-day reactivation rate.

---

## Setup

```bash
git clone <repo>
cd superstore_dataset
pip install -r requirements.txt
```

Run notebooks in order: `00` → `01` → `02` → `03` → `04`

For BigQuery upload, set your service account key path in notebook 04 and ensure the `superstore-dataset-pbs` project exists.

---

## Dataset

[Superstore Sales Dataset — Kaggle](https://www.kaggle.com/datasets/vivek468/superstore-dataset-final)  
9,994 transactions · 793 customers · Jan 2014 – Dec 2017 · US retail
