# 🌾 EDA Report — Indian Crop Production Statistics (1997–2020)

**Dataset**: `APY.csv` — Area, Production & Yield data across Indian states  
**Notebook**: `Crop_Production_EDA_India.ipynb`  
**Prepared for**: Ensemble ML modelling pipeline

---

## 1. Dataset Overview

| Attribute        | Value                                      |
|------------------|--------------------------------------------|
| Total Records    | 345,336 rows                               |
| Features         | 8 columns                                  |
| Year Range       | 1997 – 2020                                |
| States           | 37 unique states/territories               |
| Districts        | 707 unique districts                       |
| Crops            | 55 unique crop types                       |
| Seasons          | 6 (Kharif, Rabi, Autumn, Summer, Whole Year, Winter) |
| Duplicate Rows   | 0                                          |

### Column Schema

| Column         | Type    | Description                      |
|----------------|---------|----------------------------------|
| `State_Name`   | string  | Indian state / territory         |
| `District_Name`| string  | District within state            |
| `Crop`         | string  | Crop variety                     |
| `Crop_Year`    | int     | Harvest year                     |
| `Season`       | string  | Cropping season                  |
| `Area`         | float   | Cultivated area (hectares)       |
| `Production`   | float   | Total production (tonnes)        |
| `Yield_orig`   | float   | Raw yield (Production / Area)    |

---

## 2. Data Quality Analysis

### 2.1 Missing Values

| Column       | Missing Count | Missing % |
|--------------|--------------|-----------|
| `Production` | 4,948        | 1.43%     |
| `Crop`       | 9            | ~0.00%    |

> **Action**: Rows with null `Production` were dropped → **340,388 clean rows retained** (lost ~1.4%)

### 2.2 Numeric Range Validation

| Column       | Negatives | Zeros | Min    | Max           |
|--------------|-----------|-------|--------|---------------|
| `Area`       | 0         | 0     | 0.004  | 8,580,100     |
| `Production` | 0         | 1,466 | 0.000  | 1,597,800,000 |
| `Yield_orig` | 0         | 6,096 | 0.000  | 43,958.33     |

> No negative values in any numeric column. Zeros in Production/Yield are valid but flagged.

### 2.3 Categorical Cardinality

| Column         | Unique Values |
|----------------|--------------|
| `State_Name`   | 37           |
| `District_Name`| 707          |
| `Season`       | 6            |
| `Crop`         | 55           |

---

## 3. Feature Engineering

The following engineered features were added to the clean dataset:

| Feature          | Description                                              |
|------------------|----------------------------------------------------------|
| `Yield`          | `Production / Area` — recomputed to fix division-by-zero |
| `Decade`         | Time grouping: `1990s`, `2000s`, `2010s`, `2020s`       |
| `Season_clean`   | Whitespace-normalized, title-cased season labels         |
| `Log_Area`       | `log1p(Area)` — reduces right-skew                      |
| `Log_Production` | `log1p(Production)` — reduces right-skew                |
| `Log_Yield`      | `log1p(Yield)` — reduces right-skew                     |

---

## 4. Univariate Analysis

### 4.1 State Distribution (Top States by Record Count)

- **Uttar Pradesh** dominates with ~44,781 records
- Followed by Maharashtra, Karnataka, Bihar, Madhya Pradesh
- High record count ≈ more districts × more crops × more years reported

### 4.2 Season Distribution

- **Kharif** is the most dominant season (~138,369 records, ~40%)
- Followed by Rabi, Whole Year, Summer, Autumn, Winter

### 4.3 Crop Distribution

- **Rice** is the most frequently recorded crop (~21,611 entries)
- Other common crops: Wheat, Maize, Potato, Cotton (lint)

### 4.4 Numeric Distributions (Area, Production, Yield)

All three numeric variables show **severe right-skew** (long tails):

| Metric      | 25th %ile    | Median       | 75th %ile    | Max              |
|-------------|-------------|-------------|-------------|-----------------|
| Area (ha)   | 74           | 532          | 4,112        | 8,580,100        |
| Production  | 87           | 717          | 7,182        | 1,597,800,000    |
| Yield       | 0.55         | 1.00         | 2.47         | 43,958.33        |

> **Log-transforms applied** to all three for modelling readiness (see Feature Engineering).

---

## 5. Bivariate Analysis

### 5.1 Area vs. Production (Log Scale)

- Strong positive correlation between log-transformed Area and Production
- Most crop-district combinations are small-scale operations
- Large outliers are primarily **Coconut** (Tamil Nadu, Karnataka) and **Rice/Wheat** (UP)

### 5.2 Season vs. Yield

- **Rabi** crops tend to have higher median yields than **Kharif**
- **Summer** crops show the most variance in yield
- **Whole Year** crops (e.g., Coconut) have extreme outlier yields due to unit mismatch

### 6.3 Extreme Records

#### Top 5 by Production

| State       | Crop    | Year | Season     | Area (ha)  | Production (tonnes)   |
|-------------|---------|------|------------|------------|----------------------|
| Tamil Nadu  | Coconut | 2007 | Whole Year | 107,106    | 1,597,800,000        |
| Tamil Nadu  | Coconut | 2019 | Whole Year | 87,749     | 1,488,200,000        |
| Tamil Nadu  | Coconut | 2006 | Whole Year | 104,197    | 1,482,900,000        |
| Karnataka   | Coconut | 2019 | Whole Year | 176,243    | 1,452,725,000        |
| Tamil Nadu  | Coconut | 2015 | Whole Year | 85,448     | 1,401,300,000        |

> Coconut production is measured in **number of nuts**, not tonnes — causing extreme values.

#### Top 5 by Area

- **West Bengal — Niger seed (Rabi, 1997)**: 8,580,100 ha — this is a suspected data entry anomaly

---

## 6. Trend Analysis

### 6.1 Year-over-Year Production Trend (1997 – 2020)

- Overall **upward trend** in total production across all crops
- Notable dips in **2002–2003** (drought year) and **2014–2015**
- Acceleration in production post-2010, especially for cereals

### 6.2 Yield Improvement Over Decades

| Decade | Mean Yield (t/ha) |
|--------|-------------------|
| 1990s  | Lower             |
| 2000s  | Moderate growth   |
| 2010s  | Highest           |

---

## 7. Key Findings & Observations

| # | Observation |
|---|-------------|
| 1 | **Uttar Pradesh** and **Rice/Wheat** dominate record counts; dataset skews towards large North Indian states |
| 2 | **1.43% missing Production** values — handled via row-wise removal |
| 3 | **Extreme skew** in Area, Production and Yield — log-transform is mandatory pre-modelling |
| 4 | **Coconut records** (Tamil Nadu, Karnataka) have extreme Production values due to unit mismatch (nuts vs. tonnes) — consider filtering or separate treatment |
| 5 | **Niger seed — West Bengal (1997)**: Area of 8.58M ha is likely a data entry error |
| 6 | **Kharif season** accounts for ~40% of records; ensemble models should account for seasonal effects |
| 7 | Strong **log-linear relationship** between cultivated Area and Production |
| 8 | Yield has high variance and heavy right-tail — log-transform + outlier handling recommended |

---

## 8. Modelling Recommendations

### Feature Preprocessing
- **Log-transform**: `Area`, `Production`, `Yield` (reduces skew, improves model fit)
- **Encode categoricals**: `State_Name`, `District_Name`, `Crop`, `Season_clean` (label or target-encode)
- **Add time features**: `Crop_Year`, `Decade`

### Target Variable
- **Primary target**: `Yield` (log-transformed) — regression task
- Alternatively: **`Production`** (log-transformed) conditioned on `Area`

### Potential Data Issues to Address
1. Remove or cap extreme outliers (Coconut Production, Niger Seed Area anomaly)
2. Consider crop-specific models or crop-group features
3. Handle class imbalance across seasons (Kharif over-represented)

### Ensemble Methods Planned
| Model                | Type         |
|----------------------|--------------|
| Random Forest        | Bagging      |
| Gradient Boosting    | Boosting     |
| XGBoost              | Boosting     |
| LightGBM             | Boosting     |
| Stacking Regressor   | Meta-Ensemble|

---

*Report generated from `Crop_Production_EDA_India.ipynb`*
