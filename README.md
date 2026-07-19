
<div align="center">

<img src="https://capsule-render.vercel.app/api?type=waving&color=0:145A32,100:8B0000&height=180&section=header&text=Data%20Profiler&fontSize=45&fontColor=ffffff&animation=fadeIn&fontAlignY=38&desc=Multi-Source%20Data%20Acquisition,%20Cleaning%20%26%20Automated%20Profiling&descAlignY=58&descSize=18" width="100%"/>

![Python](https://img.shields.io/badge/Python-145A32?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-8B0000?style=for-the-badge&logo=pandas&logoColor=white)
![Seaborn](https://img.shields.io/badge/Seaborn-145A32?style=for-the-badge&logo=python&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-8B0000?style=for-the-badge&logo=sqlite&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-145A32?style=for-the-badge&logo=jupyter&logoColor=white)
![YData Profiling](https://img.shields.io/badge/YData_Profiling-8B0000?style=for-the-badge&logo=databricks&logoColor=white)

</div>




## 🎯 Project Overview

This notebook walks through the complete data analysis lifecycle — from data acquisition across multiple formats, through cleaning and deduplication, exploratory data analysis (univariate → bivariate → multivariate), to automated data profiling.

It is organized into 3 structured parts:

| Part | Focus |
|------|-------|
| **B** | Data Acquisition — pulling the same customer data from CSV, JSON, SQL, and a live API, then merging them |
| **C** | Data Understanding & Cleaning — missing values, duplicate detection & removal, dtype verification |
| **D** | Exploratory Data Analysis — univariate, bivariate, and multivariate analysis |
| **E** | Data Profiling — an automated `ydata-profiling` report |

## 🏗️ Project Architecture

```
Raw Data (CSV + JSON + SQL + API)
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│  PART B — DATA ACQUISITION                                │
│  CSV (sep fix) → JSON → SQLite → DummyJSON API → merged_df│
└─────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│  PART C — CLEANING                                         │
│  Missing values → dtype check → duplicate removal          │
│  300 rows → 100 unique rows                                │
└─────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│  PART D — EXPLORATORY DATA ANALYSIS                        │
│  Univariate → Bivariate → Multivariate → Correlations      │
└─────────────────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────────────────┐
│  PART E — DATA PROFILING                                   │
│  Automated report → data_profiling_report.html             │
└─────────────────────────────────────────────────────────┘
```

## 📊 Dataset

| Property | Value |
|----------|-------|
| Primary sources | `customers.csv`, `customers.json`, `customers.db` |
| Unique customers | 100 |
| Columns | 7 — `CustomerID`, `Name`, `Age`, `Gender`, `City`, `Income`, `Purchased` |
| Rows after merge (before cleaning) | 300 (same 100 customers × 3 sources) |
| Rows after dedup | 100 |
| Target concept | `Purchased` (Yes / No) |

### Other Sources Merged In
- **`customers.json`** — same 100 customer records in structured key-value format
- **`customers.db`** — SQLite database, built in-notebook from the CSV data, queried via `SELECT * FROM Customers`
- **DummyJSON API** (`https://dummyjson.com/users`) — used to demonstrate live, nested API ingestion

## 🛠️ Tech Stack

| Layer | Tools |
|-------|-------|
| Language | Python |
| Data manipulation | Pandas, NumPy |
| Data sources | CSV, JSON, SQLite (`sqlite3`), REST API (`requests`) |
| Visualization | Matplotlib, Seaborn |
| Automated profiling | `ydata-profiling` |
| Notebook environment | Jupyter Notebook |

## 📁 Repository Structure

```
Data_Profiler/
│
├── 📄 README.md                       ← You are here
├── 📓 Data_Profiler.ipynb             ← Main analysis notebook (Parts B–E)
│
├── 📂 data/
│   ├── customers.csv                   ← Primary dataset (100 rows × 7 cols, tab-separated)
│   ├── customers.json                  ← Same 100 records in JSON format
│   └── customers.db                    ← SQLite database (created in-notebook)
│
├── 📂 reports/
│   └── data_profiling_report.html      ← Auto-generated profiling report
│
└── 📄 requirements.txt
```

`requirements.txt`
```
pandas
numpy
matplotlib
seaborn
requests
ydata-profiling
jupyter
```

## ⚡ Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/priyasavaliya20-collab/Data_Profiler.git
cd Data_Profiler

# 2. Install dependencies
pip install -r requirements.txt

# 3. Launch the notebook
jupyter notebook Data_Profiler.ipynb
```

Run all cells through Part E and the report is saved automatically as `data_profiling_report.html` — open it in any browser to explore missing values, descriptive statistics, correlations, and data-quality warnings interactively.

## 🔍 Workflow Walkthrough

### Part B — Data Acquisition

**1. CSV — the primary dataset**
```python
import pandas as pd

csv_data = pd.read_csv("customers.csv")
print(csv_data.head())
print(csv_data.shape)
csv_data.info()
```
> **Interpretation:** The file loaded, but `.info()` showed the shape as `(100, 1)` — all 7 columns had collapsed into one, because the file is actually **tab-separated**, not comma-separated. Fixed with `sep="\t"`.
>
> **Conclusion:** `pd.read_csv()` defaults to a comma delimiter. For a tab-separated file, `sep="\t"` must be passed explicitly, or every row is read as a single string column.

**2. JSON — the same records, structured**
```python
json_data = pd.read_json("customers.json")
print(json_data.shape)
json_data.info()
```
> **Interpretation:** Parsed cleanly — 100 rows × 7 columns (`CustomerID`, `Name`, `Age`, `Gender`, `City`, `Income`, `Purchased`), correct dtypes, zero missing values. No formatting issues like the CSV had.

**3. SQL — an in-notebook SQLite table**
```python
import sqlite3

csv_data = pd.read_csv("customers.csv", sep="\t")
conn = sqlite3.connect("customers.db")

csv_data.to_sql("Customers", conn, if_exists="replace", index=False)
sql_data = pd.read_sql("SELECT * FROM Customers", conn)
```
> **Interpretation:** The corrected CSV data was written into a SQLite table `Customers` and read back successfully — a full write → read roundtrip with no data loss.

**4. REST API — live, nested data**
```python
import requests

url = "https://dummyjson.com/users"
response = requests.get(url)
api_data = response.json()

df_api = pd.DataFrame(api_data["users"])
print(df_api.shape)
```
> **Interpretation:** 100+ dummy user records were pulled from DummyJSON, including nested fields like address, bank details, company, and university — a realistic example of semi-structured API data (best flattened with `pd.json_normalize()` in production).

**5. Merge CSV + JSON + SQL**
```python
merged_df = pd.concat([csv_data, json_data, sql_data], ignore_index=True)
print(merged_df.shape)   # (300, 7)
```
> Because all three sources represent the **same 100 customers**, the merge produces 300 rows — each customer appearing 3 times. This duplication is intentional groundwork for the cleaning step in Part C.

### Part C — Data Understanding & Cleaning

```python
print(merged_df.isnull().sum())
print("Total Duplicate Rows:", merged_df.duplicated().sum())
```
> **Interpretation:** Zero missing values across all columns, but **200 duplicate rows** — exactly as expected from merging 3 identical sources. `Age` ranges 18–60 (avg ≈ 39.3), `Income` ranges ₹19,000–₹119,000 (avg ≈ ₹72,470), and `Purchased` = "Yes" for 180 of 300 rows.

```python
merged_df.fillna(0, inplace=True)                       # safeguard — no actual NaNs present
merged_df["Age"] = merged_df["Age"].astype(int)          # dtype verification
merged_df["Income"] = merged_df["Income"].astype(int)
merged_df = merged_df.drop(columns=["UnnecessaryColumn"], errors="ignore")
```
> **Conclusion:** No genuine cleaning was needed for missing values or dtypes — both checks were precautionary. The real fix was structural: fixing the CSV separator upstream so all 7 columns parsed correctly in the first place.

Duplicate removal (performed after Part D's univariate step, so class-balance visuals are shown pre- and post-dedup):
```python
merged_df.drop_duplicates(inplace=True)
print(merged_df.shape)   # (100, 7)
```

### Part D — Exploratory Data Analysis (EDA)

**Univariate**
```python
sns.histplot(data=merged_df, x="Age", bins=10, kde=True, color="skyblue")
sns.histplot(data=merged_df, x="Income", bins=10, kde=True, color="orange")
sns.countplot(data=merged_df, x="Purchased")
```
> Age is fairly uniform across 18–60, no dominant segment. Income spans ₹19K–₹119K with no major spikes — a financially diverse base. `Purchased` shows a **mild class imbalance** (~60% Yes vs ~40% No).

**Bivariate**
```python
sns.countplot(data=merged_df, x="Gender", hue="Purchased")
sns.boxplot(data=merged_df, x="Purchased", y="Income")
```
> Gender shows no strong influence on purchase decisions. Income distributions for "Yes" vs "No" overlap heavily — income alone doesn't separate purchasers from non-purchasers.

**Multivariate**
```python
sns.heatmap(merged_df.corr(numeric_only=True), annot=True, cmap="coolwarm")
sns.pairplot(merged_df, vars=numeric_columns, hue="Purchased")
```
> `CustomerID`, `Age`, and `Income` show correlations close to zero — weak/negligible linear relationships, and no multicollinearity concern. The pair plot shows no distinct clustering by `Purchased` either.

### Part E — Data Profiling
```python
from ydata_profiling import ProfileReport

profile = ProfileReport(merged_df, title="Customer Data Profiling Report", explorative=True)
profile.to_file("data_profiling_report.html")
```
> A single interactive HTML report summarizing missing values, descriptive statistics, correlations, and automated data-quality warnings across all 7 columns of the cleaned, deduplicated dataset — used as a sanity check against the manual EDA findings above.

## 🔎 Key Findings

| Aspect | Insight |
|--------|---------|
| Merge duplication | 3 identical sources → 300 rows → **200 duplicates** → 100 unique customers after cleanup |
| Missing values | **None** found across any column |
| Age & Income | Broadly distributed, no major outliers |
| Purchased | Mild class imbalance (~60% Yes, ~40% No) — worth a stratified split if modeling later |
| Correlations | Weak across all numeric features — no single feature strongly predicts `Purchased` |
| CSV pitfall | Wrong delimiter silently collapsed 7 columns into 1 — always check `.info()` after load |

**Takeaway:** The dataset is clean and analysis-ready. The critical preprocessing step was **duplicate removal** after the 3-way merge, not missing-value handling. Since no single feature strongly explains purchase behavior, a future predictive model should combine **Age + Income + Gender + City** rather than rely on any one signal.

## 📈 Data Profiling Report

The notebook generates a full `ydata-profiling` report (`data_profiling_report.html`) covering:

- ✅ Missing-value breakdown per column
- ✅ Descriptive statistics (mean, median, std, quantiles)
- ✅ Full correlation matrix
- ✅ Automated data-quality warnings (skew, cardinality, duplicates)

Open the generated HTML file locally in your browser after running Part E of the notebook.

## 🧠 Skills Demonstrated

```
Data Acquisition        →  CSV, JSON, SQL, REST API ingestion & merging
Data Cleaning           →  Delimiter fixes, missing-value checks, duplicate removal, dtype verification
Exploratory Analysis    →  Univariate, bivariate, multivariate techniques
Data Visualization      →  Matplotlib, Seaborn (histograms, countplots, boxplots, heatmap, pairplot)
Statistical Reasoning   →  Correlation analysis, distribution interpretation, class-imbalance checks
Automated Profiling     →  ydata-profiling report generation
Documentation           →  Structured, professional README & notebook narrative
```

## 🗺️ Future Roadmap

- [ ] **Purchase Prediction Model** — train a classifier (Logistic Regression / Random Forest) using `Age`, `Income`, `Gender`, `City`
- [ ] **Stratified Split / Class Weighting** — address the mild class imbalance in `Purchased`
- [ ] **Feature Engineering** — encode `City` and `Gender`, bin `Age`/`Income` into segments
- [ ] **API Enrichment** — flatten nested DummyJSON fields with `pd.json_normalize()` and merge them in
- [ ] **Dashboard** — interactive Streamlit/Plotly Dash view of the profiling report

## 👤 Author

**Priya Savaliya**
Data Science & AI/ML Student

If this project helped you, consider giving the repo a ⭐

---

<div align="center">
<img src="https://capsule-render.vercel.app/api?type=waving&color=0:8B0000,100:145A32&height=100&section=footer" width="100%"/>
</div>
