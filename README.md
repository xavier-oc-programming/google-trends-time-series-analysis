# Google Trends Time Series Analysis

Three economic signals — Tesla stock price, Bitcoin price, and US unemployment — each measured against public search interest from Google Trends across different time frequencies. The analysis asks a simple question: does what people search for reflect, anticipate, or follow what markets and economic indicators actually do?

The core analytical challenge is frequency mismatch. Bitcoin price data arrives daily, while Google Trends search data and unemployment figures are monthly. Comparing them directly is meaningless until the timelines are aligned. The solution is resampling: daily Bitcoin prices are collapsed to month-end values using `.resample('ME').last()`, creating a single unified monthly frequency across all three signal pairs. Rolling average smoothing is applied to unemployment search data to separate the structural trend from seasonal noise.

The most visually striking finding is the COVID-19 unemployment spike in 2020. Search interest for "unemployment benefits" hit an all-time high simultaneously with the official unemployment rate — compressing a pattern that normally plays out over months into a matter of weeks. Adding the 2020 dataset also exposes a secondary effect: Google Trends normalises index values to 100 within the selected time window, so extending the range rescales all historical values, reshaping the entire pre-2020 baseline.

---

## Quick Start

```bash
git clone https://github.com/xavier-oc-programming/google-trends-time-series-analysis.git
cd google-trends-time-series-analysis
pip install -r requirements.txt
jupyter notebook notebooks/analysis/google_trends_analysis.ipynb
```

All datasets are committed to `data/` as CSV files. No API keys or credentials required.

---

## Analysis Flow

```
pipeline
    │
    │  ── [Ingestion] ────────────────────────────────────────────
    ├── pd.read_csv()  →  tesla_search_trend_vs_price.csv         →  df_tesla
    ├── pd.read_csv()  →  bitcoin_search_trend.csv                →  df_btc_search
    ├── pd.read_csv()  →  daily_bitcoin_price.csv                 →  df_btc_price
    ├── pd.read_csv()  →  ue_benefits_search_vs_ue_rate_2004_19.csv  →  df_unemployment
    ├── pd.read_csv()  →  ue_benefits_search_vs_ue_rate_2004_20.csv  →  df_unemployment_2020
    │
    │  ── [Parsing] ──────────────────────────────────────────────
    ├── pd.to_datetime()           →  convert MONTH / DATE strings to datetime
    │
    │  ── [Cleaning] ─────────────────────────────────────────────
    ├── .isnull() / .dropna()     →  remove 1 missing row from BTC price (2020-08-04)
    │
    │  ── [Resampling] ────────────────────────────────────────────
    ├── .resample('ME').last()    →  collapse daily BTC price to month-end
    │
    │  ── [Smoothing] ─────────────────────────────────────────────
    ├── .rolling(window=6).mean() →  6-month average to reduce unemployment search noise
    │
    │  ── [Visualisation] ─────────────────────────────────────────
    └── matplotlib (twinx dual-axis)
            ├── Tesla stock price  vs  Tesla search volume
            ├── Bitcoin monthly price  vs  Bitcoin news search
            ├── U/E rate  vs  unemployment benefits search (2004–2019)
            ├── U/E rate  vs  6-month rolling average search
            └── U/E rate  vs  search (2004–2020, COVID spike)
```

---

## Key Findings

- **COVID-19 shock (2020)**: The unemployment rate and search interest for "unemployment benefits" spiked simultaneously — the 2020 dataset shows a compression of a pattern that ordinarily unfolds over quarters. Adding 2020 data also rescales the entire 2004–2019 baseline due to Google Trends normalisation within the selected period.
- **Bitcoin search leads price in some cycles**: Search volume and Bitcoin price move together during peak speculation periods but diverge during consolidation. In several cycles, search interest reached its peak one to two months before price did.
- **Tesla search tracks news, not earnings**: Tesla search volume spikes align more closely with product announcements and public controversy than with earnings beats. Price movements following strong earnings often show no corresponding search increase.
- **Rolling average reveals structural unemployment trend**: The 6-month rolling average smooths out the seasonal search spikes and reveals the slower-moving structural employment cycle more clearly than the raw monthly data.

---

## Dataset Schema

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

## Architecture

```
google-trends-time-series-analysis/
│
├── notebooks/
│   ├── analysis/
│   │   └── google_trends_analysis.ipynb   # Full end-to-end analysis
│   └── concepts/                          # Topic notebooks — one concept per file
│       ├── 00__Overview.ipynb
│       ├── 01__Data_Exploration.ipynb
│       ├── 02__Data_Cleaning_Resampling.ipynb
│       ├── 03__Tesla_Line_Charts.ipynb
│       ├── 04__Locators_DateFormatters.ipynb
│       ├── 05__Bitcoin_Line_Style.ipynb
│       ├── 06__Unemployment_Grids.ipynb
│       ├── 07__Unemployment_New_Data.ipynb
│       └── 08__Summary.ipynb
│
├── data/
│   ├── tesla_search_trend_vs_price.csv
│   ├── bitcoin_search_trend.csv
│   ├── daily_bitcoin_price.csv
│   ├── ue_benefits_search_vs_ue_rate_2004_19.csv
│   └── ue_benefits_search_vs_ue_rate_2004_20.csv
│
├── plots/                                 # Charts saved at 150 dpi
│
├── docs/
│   └── COURSE_NOTES.md
│
├── notebook_web_render/
│   └── index.html                         # Rendered notebook (GitHub Pages)
│
├── .github/
│   └── workflows/
│       └── publish_notebook.yml           # Auto-publish on notebook change
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Visualisations

Charts are saved to `plots/` at 150 dpi when the analysis notebook is run.

| File | Description |
|---|---|
| `tesla_price_vs_search_v1.png` | Dual-axis: TSLA price vs search, initial layout (no colour) |
| `tesla_price_vs_search_v2.png` | Dual-axis: TSLA price vs search, colour-coded (red/blue) |
| `tesla_price_vs_search.png` | Dual-axis: TSLA closing price (red) vs Google search volume (blue), final styled, 2010–2021 |
| `bitcoin_price_vs_search.png` | Dual-axis: BTC month-end price (orange dashed) vs Bitcoin news search (blue circles), 2014–2020 |
| `unemployment_search_vs_rate_2004_19.png` | Dual-axis: U/E rate (purple dashed) vs "unemployment benefits" search (blue), with grey grid, 2004–2019 |
| `unemployment_rolling_avg.png` | Dual-axis: U/E rate (red) vs 6-month rolling average search (steel blue dashed), 2004–2019 |
| `unemployment_with_2020.png` | Dual-axis: U/E rate (red) vs search (grey dashed), 2004–2020, showing COVID spike |

---

## Operations Reference

| Value | Location | Description |
|---|---|---|
| `../../data/tesla_search_trend_vs_price.csv` | `notebooks/analysis/google_trends_analysis.ipynb` | Relative path to Tesla dataset |
| `../../data/bitcoin_search_trend.csv` | `notebooks/analysis/google_trends_analysis.ipynb` | Relative path to Bitcoin search dataset |
| `../../data/daily_bitcoin_price.csv` | `notebooks/analysis/google_trends_analysis.ipynb` | Relative path to Bitcoin price dataset |
| `../../data/ue_benefits_search_vs_ue_rate_2004_19.csv` | `notebooks/analysis/google_trends_analysis.ipynb` | Relative path to 2004–2019 unemployment dataset |
| `../../data/ue_benefits_search_vs_ue_rate_2004_20.csv` | `notebooks/analysis/google_trends_analysis.ipynb` | Relative path to 2004–2020 unemployment dataset |
| `figsize=(14, 8), dpi=120` | All chart cells | Figure size and inline display resolution |
| `dpi=150` | `plt.savefig()` calls | Resolution for saved chart files |
| `resample('ME')` | Cleaning section | Month-end resampling alias |
| `rolling(window=6)` | Unemployment section | 6-month rolling average window |

---

## Background

This project was built as part of a structured Python data analysis curriculum covering time series resampling and Matplotlib dual-axis visualisation. Full context and concept notes are in [docs/COURSE_NOTES.md](docs/COURSE_NOTES.md).

---

## Dependencies

| Module | Used in | Purpose |
|---|---|---|
| `pandas` | All notebooks | DataFrame operations, `to_datetime`, `resample`, `rolling` |
| `matplotlib` | concepts/03–07, analysis | Line charts, dual-axis plots, figure styling |
| `matplotlib.dates` | concepts/04, analysis | `YearLocator`, `MonthLocator`, `DateFormatter` for x-axis ticks |
| `numpy` | Implicit via pandas | Numerical operations |
| `notebook` | All `.ipynb` files | Jupyter notebook runtime |
