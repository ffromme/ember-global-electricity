# ⚡ Global Electricity Transition Analysis (Ember Dataset)

An exploratory data analysis and preprocessing pipeline for the **Ember Global Electricity** dataset, focused on understanding worldwide patterns in electricity generation and quantifying the ongoing transition from fossil fuels to clean energy.

---

## 📌 Project Overview

This project processes and explores the Ember monthly electricity dataset to uncover structural shifts in the global energy mix. The pipeline covers end-to-end data preparation — from raw ingestion through EDA, missing value diagnosis, structured subsetting, and feature selection — producing a clean, analysis-ready dataset for downstream time-series modelling.

---

## 📊 Dataset

| Property | Detail |
|---|---|
| **Source** | [Ember – Global Electricity Data](https://ember-climate.org/data/) |
| **File** | `monthly_full_release_long_format.csv` |
| **Format** | Long format — one row per country × variable × month |
| **Coverage** | 1998 – 2026 (monthly) |
| **Key Columns** | `Area`, `Area type`, `Date`, `Category`, `Subcategory`, `Variable`, `Unit`, `Value` |

---

## 🔍 EDA Findings

### 1. Missing Value Analysis

Missing values in the `Value` column were dissected across two dimensions:

**By Time**
- Missing rate for the 1998–2025 period is negligible (**< 0.3%**).
- The year 2026 shows a spike to **30.27%** missing — concentrated entirely in the most recent months.

**By Country**
- When filtered to country-level rows only, no individual country exceeds **~1% missing** across its full historical record.

**Conclusion:** Missingness is not systematic data corruption. It is concentrated in the latest provisional months, strongly indicating **reporting delays** rather than structural data issues. This informed the decision to **drop** rather than impute missing `Value` rows (see Preprocessing §A).

---

### 2. Global Energy Transition Trend (World, TWh)

A line chart of global Fossil vs. Clean electricity generation reveals a clear structural shift:

- **Fossil** generation remains dominant in absolute volume but shows a **stagnant, highly seasonal** pattern (sharp peaks driven by winter/summer demand cycles) with no long-term upward trend.
- **Clean** energy shows **consistent, accelerating growth** year-over-year.

**Conclusion:** The gap between fossil and clean generation is visibly narrowing, providing empirical evidence of a genuine transition in the global electricity structure.

---

### 3. Absolute vs. Relative Metrics — Top 10 Countries (2024)

Two views of clean energy leadership in 2024 tell fundamentally different stories:

| Metric | Top Countries | Interpretation |
|---|---|---|
| **Absolute (TWh)** | China, USA, Brazil, Canada… | Reflects infrastructure scale & population size — not grid composition |
| **Relative (% of total generation)** | Norway, Sweden, Uruguay, Albania… | Reflects true clean energy share of a country's grid |

**Conclusion:** TWh is the right unit for measuring *capacity and volume*; percentage is the correct unit for measuring *energy transition progress*. Downstream modelling must specify which question is being answered before choosing a metric.

---

## 🛠️ Preprocessing Pipeline

All preprocessing decisions were grounded directly in EDA findings above.

### A. Missing Value Handling

| Column Group | Action | Rationale |
|---|---|---|
| `ISO 3 code`, `Continent`, org flags | **Keep** as-is (55,795 null rows) | These rows have `Area type = 'Region'` — regional/world aggregates do not have ISO codes by definition. Nulls are contextually valid. |
| `Value` | **Drop** rows where null | Imputing with `0` would falsely imply zero electricity generation or zero emissions — distorting all historical averages. Dropping is the safest option given that missingness is concentrated in provisional months only. |

### B. Date Parsing & Feature Extraction

- `Date` column converted from `string` → `datetime64` to enable chronological ordering for time-series analysis.
- Derived temporal columns extracted: `Year`, `Month`, `Quarter`, `YearMonth`.

### C. Structured Subsetting (Anti-Double-Counting)

Filtered to rows matching **all three** conditions simultaneously:

```
Area type  == 'Country or economy'
Category   == 'Electricity generation'
Subcategory == 'Total'
```

**Rationale:** The raw dataset is in long format where regional aggregates (World, EU, G20) co-exist with individual countries, and fuel-level breakdowns (Coal, Gas, Wind) co-exist with their totals. Without subsetting, any aggregation would count the same electricity multiple times. This filter isolates the single, unambiguous metric: *total national electricity generation per country per month*.

### D. Feature Selection

Columns removed from the final dataset:

| Column Group | Columns | Reason |
|---|---|---|
| **Zero-variance constants** | `Category`, `Subcategory`, `Variable` | After subsetting, these columns hold a single value across all rows — zero variance, no modelling utility |
| **Economic bloc flags** | `EU`, `OECD`, `G20`, `G7`, `ASEAN` | Analysis targets individual country behaviour independently; bloc membership is not a modelling feature |
| **Derivable YoY metrics** | `YoY absolute change`, `YoY % change` | >37% missing from the start; both can be accurately recalculated from `Date` and `Value` when needed |

---

## 🗂️ Project Structure

```
ember-electricity-analysis/
│
├── data/
│   └── monthly_full_release_long_format.csv   # Raw Ember dataset (not tracked by git)
│
├── notebooks/
│   └── main.ipynb                             # EDA & preprocessing notebook
│
├── requirements.txt                           # Python dependencies
└── README.md
```

> **Note:** The raw dataset file is large. Download it directly from [Ember's data page](https://ember-climate.org/data/) and place it in the `data/` folder before running the notebook.

---

## 🚀 Getting Started

### Prerequisites
- Python 3.9+
- pip

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-username/ember-electricity-analysis.git
cd ember-electricity-analysis

# 2. (Optional) Create a virtual environment
python -m venv venv
source venv/bin/activate        # On Windows: venv\Scripts\activate

# 3. Install dependencies
pip install -r requirements.txt

# 4. Place the dataset
# Download from https://ember-climate.org/data/
# Save as: data/monthly_full_release_long_format.csv

# 5. Launch Jupyter and open the notebook
jupyter notebook notebooks/main.ipynb
```

---

## 🔮 Next Steps

- **Time-Series Modelling** — Apply forecasting models (Prophet, ARIMA, or LSTM) on the cleaned country-level generation data to project clean energy trajectories.
- **Country Clustering** — Group countries by their transition speed and energy mix composition using unsupervised learning.
- **Per-Capita Normalisation** — Join with World Bank population data to produce generation per capita — a third dimension beyond TWh and percentage.
- **Dashboard** — Build an interactive visualisation layer (Plotly / Streamlit) to explore country-level transitions dynamically.

---

## 🧰 Tech Stack

![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-11557c?style=for-the-badge&logo=python&logoColor=white)
![Jupyter](https://img.shields.io/badge/Jupyter-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

| Library | Purpose |
|---|---|
| `pandas` | Data loading, cleaning, filtering, and transformation |
| `matplotlib` | EDA visualisations (line charts, bar charts) |
| `numpy` | Numerical support |

---

## 📄 License

This project is open-source and available under the [MIT License](LICENSE).

---

## 🙏 Data Credit

Dataset provided by **[Ember](https://ember-climate.org/)** — an independent energy think tank. All rights to the underlying data belong to Ember.
