# 🌫️ Kolhapur Air Quality Index (AQI) Analysis (2023–2025)

A data analysis project tracking daily Air Quality Index (AQI) levels in **Kolhapur, Maharashtra, India** across three years using city-level monitoring data.

---

## 📁 Dataset Overview

| File | Year | Coverage | Records |
|------|------|----------|---------|
| `AQI_dailycity_levelkolhapur2023.xlsx` | 2023 | May – December | ~8 months |
| `AQI_dailycity_levelkolhapur2024.xlsx` | 2024 | January – December | Full Year |
| `AQI_dailycity_levelkolhapur2025.xlsx` | 2025 | January – March | 3 months (ongoing) |

Each file contains two sheets:
- **AQI** — Daily AQI values for each month
- **Prominent Parameters** — Dominant pollutants driving the AQI (e.g., PM10, PM2.5, OZONE, NO2)

### 🧹 Cleaned & Merged Dataset

| File | Records | Columns |
|------|---------|---------|
| `cleaned_kolhapur_aqi.csv` | 696 rows | Day, Month, AQI, Year, Date, AQI_Category, Season, Month_Num |

The raw Excel files were cleaned and merged into a single analysis-ready CSV. Key transformations applied:
- Reshaped from wide (day × month) to long format (one row per observation)
- Removed rows with missing AQI values
- Added `Date` column in `DD-MM-YYYY` format
- Added `AQI_Category` based on CPCB classification
- Added `Season` label (Summer / Monsoon / Post-Monsoon / Winter)
- Added `Month_Num` for chronological sorting

---

## 📊 Key Findings

### Yearly AQI Averages

| Year | Avg AQI | Min AQI | Max AQI | Status |
|------|---------|---------|---------|--------|
| 2023 | 75.6    | 0       | 338     | Moderate |
| 2024 | 93.1    | 1       | 268     | Satisfactory–Moderate |
| 2025 | 89.8 (Jan–Mar) | 2 | 184 | Moderate |

### Monthly AQI Averages (2023 vs 2024)

| Month | 2023 Avg AQI | 2024 Avg AQI |
|-------|-------------|-------------|
| January | 15.5 | 125.4 |
| February | 14.0 | 131.1 |
| March | 15.5 | 127.4 |
| April | 52.1 | 122.1 |
| May | 69.0 | 88.5 |
| June | 48.3 | 53.4 |
| July | 42.0 | 39.0 |
| August | 43.2 | 42.6 |
| September | 42.0 | 51.8 |
| October | 112.3 | 98.2 |
| November | 123.4 | 147.1 |
| December | 139.3 | 88.9 |

> ⚠️ **Note:** 2023 data is missing January–March, so the yearly average appears lower than actual.

### AQI Category Distribution (Cleaned Dataset — 696 days)

| Category | Days | % of Total |
|----------|------|------------|
| Good (0–50) | 173 | 24.9% |
| Satisfactory (51–100) | 254 | 36.5% |
| Moderate (101–200) | 250 | 35.9% |
| Poor (201–300) | 18 | 2.6% |
| Very Poor (301–400) | 1 | 0.1% |

### AQI by Season (Cleaned Dataset)

| Season | Days Recorded |
|--------|--------------|
| Monsoon | 239 |
| Winter | 180 |
| Summer | 158 |
| Post-Monsoon | 119 |

### Dominant Pollutants
- **PM10** — Most frequently the prominent parameter across all months and years
- **PM2.5** — Common in winter months (November–February)
- **OZONE** — Dominant in summer months (April–June)
- **NO2** — Occasionally prominent in monsoon months

---

## 📈 AQI Category Reference (CPCB India)

| AQI Range | Category | Health Impact |
|-----------|----------|---------------|
| 0–50 | Good | Minimal |
| 51–100 | Satisfactory | Minor breathing discomfort for sensitive people |
| 101–200 | Moderate | Breathing discomfort on prolonged exposure |
| 201–300 | Poor | Breathing discomfort for most |
| 301–400 | Very Poor | Respiratory illness on prolonged exposure |
| 401–500 | Severe | Affects healthy population; serious impact on sensitive groups |

---

## 🗂️ Repository Structure

```
kolhapur-aqi-analysis/
│
├── data/
│   ├── raw/
│   │   ├── AQI_dailycity_levelkolhapur2023.xlsx
│   │   ├── AQI_dailycity_levelkolhapur2024.xlsx
│   │   └── AQI_dailycity_levelkolhapur2025.xlsx
│   └── processed/
│       └── cleaned_kolhapur_aqi.csv   ← merged, analysis-ready dataset
│
├── scripts/
│   └── clean_aqi_data.py              ← data cleaning & visualization script
│
├── charts/
│   └── chart1_histogram.png
│
├── dashboard/
│   ├── AQI.pbix                       ← Power BI dashboard file
│   └── dashboard_preview.png          ← screenshot of the dashboard
│
├── notebooks/          # (Add analysis notebooks here)
├── README.md
└── LICENSE
```

---

## 🧹 Data Cleaning Script

The following script loads the three raw Excel files, reshapes and merges them, assigns AQI categories and seasons, validates the data, and exports the final cleaned CSV.

```python
import pandas as pd
import os

# File paths
files = {
    2023: r'data/raw/AQI_dailycity_levelkolhapur2023.xlsx',
    2024: r'data/raw/AQI_dailycity_levelkolhapur2024.xlsx',
    2025: r'data/raw/AQI_dailycity_levelkolhapur2025.xlsx',
}

def load_and_reshape(filepath, year):
    df = pd.read_excel(filepath)

    # Melt wide → long
    df_long = df.melt(id_vars='Day', var_name='Month', value_name='AQI')

    # Add year column
    df_long['Year'] = year

    # Create a proper date column
    df_long['Date'] = pd.to_datetime(
        df_long['Year'].astype(str) + '-' +
        df_long['Month'] + '-' +
        df_long['Day'].astype(str),
        format='%Y-%B-%d',
        errors='coerce'   # invalid dates like Feb 30 become NaT
    )

    return df_long

# Load all three years
dfs = [load_and_reshape(path, year) for year, path in files.items()]

# Stack all three dataframes into one
master_df = pd.concat(dfs, ignore_index=True)

# Drop rows where date couldn't be parsed (e.g. Feb 30, Apr 31)
master_df = master_df.dropna(subset=['Date'])

# Sort by date
master_df = master_df.sort_values('Date').reset_index(drop=True)

print(master_df.shape)
print(master_df.head(10))

# Drop rows where AQI is NaN (months with no data at all)
master_df = master_df.dropna(subset=['AQI'])

# AQI Category (based on India's NAQI scale)
def aqi_category(aqi):
    if aqi <= 50:    return 'Good'
    elif aqi <= 100: return 'Satisfactory'
    elif aqi <= 200: return 'Moderate'
    elif aqi <= 300: return 'Poor'
    elif aqi <= 400: return 'Very Poor'
    else:            return 'Severe'

master_df['AQI_Category'] = master_df['AQI'].apply(aqi_category)

# Season (India-specific)
def get_season(month):
    if month in ['December', 'January', 'February']:        return 'Winter'
    elif month in ['March', 'April', 'May']:                return 'Summer'
    elif month in ['June', 'July', 'August', 'September']:  return 'Monsoon'
    else:                                                    return 'Post-Monsoon'

master_df['Season'] = master_df['Month'].apply(get_season)

# Month number (for sorting charts correctly)
master_df['Month_Num'] = master_df['Date'].dt.month

# ── Validation Checks ──────────────────────────────────────────────────────────

# 1. Check AQI value range — should be 0 to 500
print("AQI range:", master_df['AQI'].min(), "to", master_df['AQI'].max())

# 2. Check for duplicate dates
duplicates = master_df[master_df.duplicated(subset='Date')]
print("Duplicate dates:", len(duplicates))

# 3. Check years covered
print("Years in data:", master_df['Year'].unique())

# 4. Check AQI category distribution
print(master_df['AQI_Category'].value_counts())

# 5. Preview final dataframe
print(master_df.dtypes)
master_df.head()

# ── Export ─────────────────────────────────────────────────────────────────────
output_path = r'data/processed/cleaned_kolhapur_aqi.csv'
master_df.to_csv(output_path, index=False)
print(f"Saved {len(master_df)} rows to {output_path}")
```

---

## 📊 Visualization Script

Loads the cleaned CSV and generates a histogram of daily AQI distribution for each year.

```python
import pandas as pd
import matplotlib.pyplot as plt
import matplotlib.ticker as ticker
import seaborn as sns
import warnings
warnings.filterwarnings('ignore')

# Makes all charts look clean and consistent
sns.set_theme(style='whitegrid', palette='muted')
plt.rcParams['figure.dpi'] = 120
plt.rcParams['font.family'] = 'DejaVu Sans'

df = pd.read_csv(r'data/processed/cleaned_kolhapur_aqi.csv', parse_dates=['Date'])

print(df.shape)
print(df.head())

# ── Year-wise Summary Statistics ───────────────────────────────────────────────
stats = df.groupby('Year')['AQI'].agg(
    Total_Days='count',
    Mean='mean',
    Median='median',
    Std_Dev='std',
    Min='min',
    Max='max'
).round(1)

print("=== Year-wise AQI Summary ===")
print(stats)

# ── Chart 1: AQI Distribution Histogram (per year) ────────────────────────────
fig, axes = plt.subplots(1, 3, figsize=(15, 4), sharey=True)

colors = {2023: '#4C9BE8', 2024: '#E8834C', 2025: '#4CE8A0'}

for ax, (year, group) in zip(axes, df.groupby('Year')):
    ax.hist(group['AQI'], bins=20, color=colors[year], edgecolor='white', linewidth=0.5)
    ax.axvline(group['AQI'].mean(), color='red', linestyle='--', linewidth=1.5,
               label=f"Mean: {group['AQI'].mean():.0f}")
    ax.set_title(f'{year}', fontsize=13, fontweight='bold')
    ax.set_xlabel('AQI Value')
    ax.set_ylabel('Number of Days')
    ax.legend(fontsize=9)

fig.suptitle('Distribution of Daily AQI Values — Kolhapur (2023–2025)',
             fontsize=14, fontweight='bold', y=1.02)
plt.tight_layout()
plt.savefig(r'charts/chart1_histogram.png', bbox_inches='tight')
plt.show()
```

---

## 🚀 Getting Started

### Prerequisites
```bash
pip install pandas openpyxl matplotlib seaborn
```

### Load the Raw Data
```python
import pandas as pd

df_2023 = pd.read_excel('data/raw/AQI_dailycity_levelkolhapur2023.xlsx', sheet_name='AQI')
df_2024 = pd.read_excel('data/raw/AQI_dailycity_levelkolhapur2024.xlsx', sheet_name='AQI')
df_2025 = pd.read_excel('data/raw/AQI_dailycity_levelkolhapur2025.xlsx', sheet_name='AQI')
```

### Load the Cleaned Dataset (Recommended)
```python
import pandas as pd

df = pd.read_csv('data/cleaned_kolhapur_aqi.csv')
df['Date'] = pd.to_datetime(df['Date'], format='%d-%m-%Y')

# Quick exploration
print(df.shape)           # (696, 8)
print(df['AQI_Category'].value_counts())
print(df.groupby('Year')['AQI'].mean())
```

---

## 📊 Power BI Dashboard

An interactive Power BI dashboard built on top of the cleaned dataset.

![Kolhapur AQI Dashboard](https://github.com/LunarPhoenix42/kolhapur-aqi-analysis/blob/main/Kolhapur_AQI%20.png
)
### Dashboard Highlights

| Visual | Description |
|--------|-------------|
| **KPI Cards** | Days Recorded (696), Avg AQI (93.5), Max AQI (338), Cleanest Month (July), Worst Month (November) |
| **Avg AQI by Year** | Bar chart comparing 2023, 2024, 2025 yearly averages |
| **Avg AQI by Season** | Post-Monsoon and Winter have the highest seasonal AQI |
| **AQI Health Gauge** | Needle gauge showing overall avg AQI (93.54) against the safe limit of 100 |
| **AQI Category Split** | Pie chart — Satisfactory (36.55%), Moderate (35.97%), Good (24.89%), Poor (2.59%) |
| **Avg AQI by Month** | Bar chart across all 12 months; November peaks, July is lowest |

### Filters Available
- AQI Category
- Year
- Month
- Season


---

## 📌 Observations

- **Winter months (Nov–Feb)** consistently show higher AQI due to temperature inversions and increased particulate matter.
- **Monsoon months (June–September)** show the cleanest air, with AQI often in the *Good* to *Satisfactory* range — monsoon has the most recorded days (239) in the cleaned dataset.
- **~61% of days** fall in *Good* or *Satisfactory* range; only ~2.7% reach *Poor* or *Very Poor* levels.
- **2024** shows significantly higher AQI in winter/spring compared to 2023, suggesting worsening air quality trends.
- **PM10** is the dominant pollutant in Kolhapur, likely linked to construction, road dust, and industrial activity.

---

## 📍 Data Source

Data sourced from **Central Pollution Control Board (CPCB), India** — city-level daily AQI monitoring for Kolhapur, Maharashtra.

---

