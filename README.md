# Google Trends Time Series Analysis

Explores correlations between Google search trends and stock prices and unemployment data using resampling and dual-axis Matplotlib visualisations.

This project investigates whether public interest — measured by Google Trends search volume — can predict or reflect real-world financial and economic outcomes. Three questions are posed: does Tesla search popularity track its stock price? Does Bitcoin news search volume move in step with its market price? And does the search interest in "unemployment benefits" anticipate or follow the official U.S. unemployment rate reported by the Federal Reserve?

The datasets come from Google Trends, Yahoo Finance, and the Federal Reserve Economic Data (FRED) service, all pre-downloaded as CSV files. The data spans multiple years and operates at different time frequencies: Tesla and unemployment data are monthly, while Bitcoin price is daily. The core analytical step converts daily Bitcoin prices to monthly frequency using pandas `.resample()`, so that all three comparisons operate on aligned timelines. Additional cleaning includes removing missing values from the Bitcoin price history and converting string date columns to proper datetime objects.

No external APIs or credentials are required. All datasets are committed directly to the repository as seed CSV files and can be loaded with relative paths from the practice notebook.

---

## Table of Contents

1. [Quick start](#1-quick-start)
2. [Analysis flow](#2-analysis-flow)
3. [Features](#3-features)
4. [Dataset schema](#4-dataset-schema)
5. [Architecture](#5-architecture)
6. [Notebook reference](#6-notebook-reference)
7. [Configuration reference](#7-configuration-reference)
8. [Course context](#8-course-context)
9. [Dependencies](#9-dependencies)

---

## 1. Quick start

```bash
git clone https://github.com/xavier-oc-programming/google-trends-time-series-analysis.git
cd google-trends-time-series-analysis
pip install -r requirements.txt
jupyter notebook practice/A_Google_Trends_Analysis.ipynb
```

Open `practice/A_Google_Trends_Analysis.ipynb` first — it contains the full analysis end-to-end. The `theory/` notebooks explain each step individually.

---

## 2. Analysis flow

```
data/tesla_search_trend_vs_price.csv  ─────────────────────┐
data/bitcoin_search_trend.csv          ──── pd.read_csv()   │
data/daily_bitcoin_price.csv           ──── pd.read_csv()   │──► DataFrames
data/ue_benefits_search_vs_ue_rate_2004_19.csv ─────────────┘
data/ue_benefits_search_vs_ue_rate_2004_20.csv

DataFrames
    │
    ├── pd.to_datetime()         ──► parse MONTH / DATE columns
    ├── .isnull() / .dropna()   ──► remove 1 missing BTC price row
    │
    ├── .resample('ME').last()  ──► daily BTC price → monthly
    │
    ├── .rolling(window=6).mean() ──► 6-month search trend average
    │
    └── matplotlib (twinx)
            ├── Tesla stock price  vs  Tesla search volume
            ├── Bitcoin monthly price  vs  Bitcoin news search
            ├── U/E rate  vs  unemployment benefits search (2004–2019)
            ├── U/E rate  vs  rolling 6-month search average
            └── U/E rate  vs  search (2004–2020, including COVID spike)
```

---

## 3. Features

- **Tesla correlation**: Dual-axis line chart overlaying TSLA closing price against Google search popularity, styled with red/blue colour coding.
- **Bitcoin resampling**: Converts daily price data to month-end frequency so it can be compared directly with the monthly search dataset.
- **Bitcoin correlation**: Dual-axis chart with a dashed price line and circle markers on the search data.
- **Unemployment search vs rate**: Dual-axis chart with a grey dashed grid to reveal seasonality in search interest.
- **Rolling average smoothing**: 6-month rolling mean applied to unemployment search data to expose the underlying trend versus the actual unemployment rate.
- **COVID shock**: Extended dataset to 2020 shows the unprecedented spike in both search interest and unemployment during the pandemic.

---

## 4. Dataset schema

### `tesla_search_trend_vs_price.csv`

| Column | Type | Description |
|---|---|---|
| MONTH | datetime (parsed) | Calendar month (monthly frequency) |
| TSLA_WEB_SEARCH | int | Google Trends relative search popularity (0–100) |
| TSLA_USD_CLOSE | float | Tesla closing stock price in USD |

### `bitcoin_search_trend.csv`

| Column | Type | Description |
|---|---|---|
| MONTH | datetime (parsed) | Calendar month (monthly frequency) |
| BTC_NEWS_SEARCH | int | Google Trends news search popularity (0–100) |

### `daily_bitcoin_price.csv`

| Column | Type | Description |
|---|---|---|
| DATE | datetime (parsed) | Calendar date (daily frequency) |
| CLOSE | float | Bitcoin closing price in USD |
| VOLUME | int | Daily trading volume |

**Computed at runtime**: `df_btc_monthly` — month-end BTC price resampled from daily data via `.resample('ME').last()`.

### `ue_benefits_search_vs_ue_rate_2004_19.csv`

| Column | Type | Description |
|---|---|---|
| MONTH | datetime (parsed) | Calendar month (monthly, 2004–2019) |
| UE_BENEFITS_WEB_SEARCH | int | Google Trends search popularity for "Unemployment Benefits" (0–100) |
| UNRATE | float | Official U.S. unemployment rate (%) from FRED |

**Computed at runtime**: `UE_SEARCH_6M_AVG` — 6-month rolling mean of `UE_BENEFITS_WEB_SEARCH`.

### `ue_benefits_search_vs_ue_rate_2004_20.csv`

| Column | Type | Description |
|---|---|---|
| MONTH | datetime (parsed) | Calendar month (monthly, 2004–2020) |
| UE_BENEFITS_WEB_SEARCH | int | Google Trends search popularity (rescaled, includes 2020) |
| UNRATE | float | Official U.S. unemployment rate (%) from FRED |

---

## 5. Architecture

```
google-trends-time-series-analysis/
│
├── theory/                          # Lesson notebooks — concepts explained
│   ├── 00__Overview.ipynb           # Day 75 goals and what will be built
│   ├── 01__Data_Exploration.ipynb   # Shape, dtypes, describe(), periodicity
│   ├── 02__Data_Cleaning_Resampling.ipynb  # Missing values, to_datetime, resample
│   ├── 03__Tesla_Line_Charts.ipynb  # Dual-axis line chart for Tesla
│   ├── 04__Locators_DateFormatters.ipynb   # YearLocator, DateFormatter tick control
│   ├── 05__Bitcoin_Line_Style.ipynb # Dashed lines, circle markers for Bitcoin
│   ├── 06__Unemployment_Grids.ipynb # Grid styling for unemployment chart
│   ├── 07__Unemployment_New_Data.ipynb     # Effect of adding 2020 COVID data
│   └── 08__Summary.ipynb            # Key learning points recap
│
├── practice/
│   └── A_Google_Trends_Analysis.ipynb  # Student code — full end-to-end analysis
│
├── data/
│   ├── tesla_search_trend_vs_price.csv         # Tesla monthly search + stock price
│   ├── bitcoin_search_trend.csv                # Bitcoin monthly news search
│   ├── daily_bitcoin_price.csv                 # Bitcoin daily OHLCV
│   ├── ue_benefits_search_vs_ue_rate_2004_19.csv  # Unemployment 2004–2019
│   └── ue_benefits_search_vs_ue_rate_2004_20.csv  # Unemployment 2004–2020
│
├── docs/
│   └── COURSE_NOTES.md              # Original exercise brief and key concepts
│
├── requirements.txt                 # Python package dependencies
├── .gitignore
└── README.md
```

---

## 6. Notebook reference

### theory/

| Notebook | Key methods covered | Question answered |
|---|---|---|
| 00__Overview.ipynb | — | What will be built today? |
| 01__Data_Exploration.ipynb | `.shape`, `.describe()`, `.info()`, `.min()`, `.max()` | What does each dataset look like? What is the periodicity? |
| 02__Data_Cleaning_Resampling.ipynb | `.isnull()`, `.dropna()`, `pd.to_datetime()`, `.resample('ME').last()` | How do we align daily and monthly data? |
| 03__Tesla_Line_Charts.ipynb | `plt.figure()`, `twinx()`, `.plot()`, `.set_ylabel()`, `.set_xlim()` | How do you plot two scales on one chart? |
| 04__Locators_DateFormatters.ipynb | `mdates.YearLocator()`, `mdates.MonthLocator()`, `mdates.DateFormatter()` | How do you control x-axis tick marks on a time series? |
| 05__Bitcoin_Line_Style.ipynb | `linestyle='--'`, `marker='o'` | How do line style and markers aid readability? |
| 06__Unemployment_Grids.ipynb | `ax.grid(color=, linestyle=, alpha=)` | How do grids help read data across a long time axis? |
| 07__Unemployment_New_Data.ipynb | `.read_csv()`, `.to_datetime()` | How does new data (2020 COVID spike) reshape the chart? |
| 08__Summary.ipynb | — | What are the key takeaways from Day 75? |

### practice/

| Notebook | Key methods covered | Question answered |
|---|---|---|
| A_Google_Trends_Analysis.ipynb | All of the above | Does search volume correlate with Tesla price, Bitcoin price, or the U.S. unemployment rate? |

---

## 7. Configuration reference

| Value | Location | Description |
|---|---|---|
| `../data/tesla_search_trend_vs_price.csv` | `practice/A_Google_Trends_Analysis.ipynb` | Relative path to Tesla dataset |
| `../data/bitcoin_search_trend.csv` | `practice/A_Google_Trends_Analysis.ipynb` | Relative path to Bitcoin search dataset |
| `../data/daily_bitcoin_price.csv` | `practice/A_Google_Trends_Analysis.ipynb` | Relative path to Bitcoin price dataset |
| `../data/ue_benefits_search_vs_ue_rate_2004_19.csv` | `practice/A_Google_Trends_Analysis.ipynb` | Relative path to 2004–2019 unemployment dataset |
| `../data/ue_benefits_search_vs_ue_rate_2004_20.csv` | `practice/A_Google_Trends_Analysis.ipynb` | Relative path to 2004–2020 unemployment dataset |
| `figsize=(14, 8), dpi=120` | All chart cells | Figure size and resolution |
| `resample('ME')` | Cleaning notebook | Month-end resampling alias |
| `rolling(window=6)` | Unemployment notebook | 6-month rolling average window |
| `{:,.2f}` | Import cell | Float display format for pandas |

---

## 8. Course context

100 Days of Code: The Complete Python Pro Bootcamp — Day 75: Google Trends, Data Resampling and Visualising Time Series.  
See [docs/COURSE_NOTES.md](docs/COURSE_NOTES.md) for the full exercise brief and concept explanations.

---

## 9. Dependencies

| Module | Used in | Purpose |
|---|---|---|
| `pandas` | All notebooks | DataFrame operations, `to_datetime`, `resample`, `rolling` |
| `matplotlib` | theory/03–07, practice | Line charts, dual-axis plots, figure styling |
| `matplotlib.dates` | theory/04, practice | `YearLocator`, `MonthLocator`, `DateFormatter` for x-axis ticks |
| `numpy` | Implicit via pandas | Numerical operations |
| `notebook` | All `.ipynb` files | Jupyter notebook runtime |
