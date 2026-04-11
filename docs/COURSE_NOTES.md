# Course Notes — Day 75: Google Trends, Data Resampling and Visualising Time Series

**Course**: 100 Days of Code: The Complete Python Pro Bootcamp  
**Day**: 75  
**Topics**: Time series analysis, resampling, matplotlib dual-axis charts, date formatting

---

## Exercise Brief

Explore whether Google Trends search volume correlates with real-world data:

- **Tesla** — does search popularity for "TSLA" track the stock price?
- **Bitcoin** — does news search volume for "BTC" track daily/monthly price?
- **Unemployment** — does search interest in "Unemployment Benefits" lead or lag the actual U.S. unemployment rate?

### Challenges covered

1. Explore the shape, column names, descriptive statistics, and periodicity of four datasets.
2. Identify and remove missing values in the Bitcoin daily price dataset.
3. Convert string date columns to `datetime` objects with `pd.to_datetime()`.
4. Resample daily Bitcoin price data to monthly frequency using `.resample('ME').last()`.
5. Plot Tesla stock price vs. search volume using a dual-axis line chart (`twinx()`).
6. Style the chart: colours, line widths, axis labels, tick sizes, x-axis limits.
7. Use `matplotlib.dates` locators and formatters to control year/month tick marks.
8. Plot Bitcoin price vs. search volume with dashed line style and circle markers.
9. Plot unemployment rate vs. search trend with a grey dashed grid.
10. Compute a 6-month rolling average with `.rolling(window=6).mean()` and plot against the unemployment rate.
11. Re-read the unemployment dataset extended to 2020 and observe the COVID-19 spike.

---

## Key Concepts

### Time Series Alignment
Datasets with different frequencies (daily vs. monthly) must be resampled to a common frequency before comparison.

```python
df_btc_monthly = df_btc_price.resample('ME', on='DATE').last()
```

### Dual-Axis Charts
Use `twinx()` to overlay two metrics with different scales on the same x-axis:

```python
ax1 = plt.gca()
ax2 = ax1.twinx()
ax1.plot(...)   # left y-axis
ax2.plot(...)   # right y-axis
```

### Date Tick Formatting
```python
import matplotlib.dates as mdates
years = mdates.YearLocator()
months = mdates.MonthLocator()
years_fmt = mdates.DateFormatter('%Y')

ax.xaxis.set_major_locator(years)
ax.xaxis.set_major_formatter(years_fmt)
ax.xaxis.set_minor_locator(months)
```

### Rolling Average
Smooth noisy search data with a rolling window mean:

```python
df['UE_SEARCH_6M_AVG'] = df['UE_BENEFITS_WEB_SEARCH'].rolling(window=6).mean()
```

---

## Data Sources

| Dataset | Source | URL |
|---|---|---|
| U.S. Unemployment Rate | FRED (St. Louis Fed) | https://fred.stlouisfed.org/series/UNRATE/ |
| Google Trends | Google Trends | https://trends.google.com/trends/explore |
| Tesla Stock Price | Yahoo Finance | https://finance.yahoo.com/quote/TSLA/history |
| Bitcoin Price | Yahoo Finance | https://finance.yahoo.com/quote/BTC-USD/history |
