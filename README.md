# uk-road-safety-analytics
Analysing UK road accident patterns and severity using DfT STATS19 data, with a machine learning severity prediction model
# UK Road Safety Analytics

An end-to-end data analytics and machine learning project analysing 
640,449 road casualties across Great Britain from 2020 to 2024, using 
Department for Transport STATS19 open data.

---

## The Problem

Every year thousands of people are killed or seriously injured on British 
roads. The government has a target to halve the number of people killed 
and seriously injured by 2030. Based on this data, that target is not 
being met — serious casualties have risen since COVID and remain elevated.

This project analyses 5 years of official police-recorded accident data 
to answer four questions:

- How has road casualty severity changed since COVID?
- When and where are British roads most dangerous?
- What conditions produce the worst accidents?
- Can machine learning predict whether an accident will be fatal or serious?

---

## Key Findings

- Serious casualties rose from 24,493 in 2020 to 32,071 in 2024 and 
  have not come back down since the COVID period
- Fatal casualties remain elevated at 2,508 in 2024 — higher than 
  pre-COVID levels
- **5pm on a Friday** is consistently the most dangerous time on British 
  roads — 55,216 casualties at that hour across 5 years
- **60mph roads** have a higher fatal and serious rate (37.3%) than 70mph 
  motorways (27.4%) — rural single carriageways are the deadliest road type
- **August** has the highest fatal and serious casualty rate (27.1%) 
  despite not being the busiest month — summer driving is more dangerous 
  than winter driving
- **Number of vehicles involved** is the strongest predictor of severity 
  (importance 0.253), followed by speed limit (0.175) and rural location 
  (0.139)
- Four machine learning models (Logistic Regression, Random Forest, 
  XGBoost, LightGBM) all converged to ROC-AUC 0.629-0.634 — confirming 
  the performance ceiling is data-limited not model-limited

---

## Machine Learning Model

| Model | ROC-AUC | F1 Score | Recall |
|---|---|---|---|
| Logistic Regression | 0.606 | 0.412 | 0.525 |
| Random Forest (selected) | **0.634** | **0.440** | **0.579** |
| XGBoost | 0.629 | 0.436 | 0.573 |
| LightGBM | 0.632 | 0.439 | 0.577 |

**Target:** Binary classification — Fatal or Serious (1) vs Slight (0)  
**Train:** 2020-2022 (379,213 rows) | **Test:** 2023-2024 (261,236 rows)  
**Class imbalance handled** using class_weight='balanced'

---

## Charts

### Casualty Severity Trend 2020-2024
![Severity Trend](outputs/charts/01_severity_trend_2020_2024.png)

### Time Patterns — Hour of Day and Day of Week
![Time Patterns](outputs/charts/02_time_patterns.png)

### Road Conditions and Speed Limits
![Road Conditions](outputs/charts/03_road_conditions.png)

### Seasonal Pattern
![Seasonal Pattern](outputs/charts/04_seasonal_pattern.png)

### Geographic Distribution — Great Britain
![Geographic Map](outputs/charts/05_geographic_map.png)

### Model Evaluation — ROC Curves and Feature Importance
![Model Evaluation](outputs/charts/06_model_evaluation.png)

---

## Interactive Map

An interactive Folium map of all 161,400 fatal and serious accidents 
is available in the repository at:

`outputs/road_accidents_interactive_map.html`

Download the file and open it in any browser to explore individual 
accident locations across Great Britain. Three layers available:
- Accident density heatmap (all 161,400 fatal and serious accidents)
- Fatal accident pinpoints (all 12,389 fatal accidents, click for details)
- Serious accident clusters (20,000 sample, grouped by location)

---

## Project Structure

    uk-road-safety-analytics/
    ├── data/
    │   ├── raw/              <- DfT STATS19 CSV files and data guide
    │   └── processed/        <- Cleaned merged dataset, train and test sets
    ├── notebooks/
    │   ├── 01_data_loading.ipynb
    │   ├── 02_exploratory_data_analysis.ipynb
    │   ├── 03_feature_engineering.ipynb
    │   ├── 04_modelling.ipynb
    │   └── 05_conclusions.ipynb
    ├── outputs/
    │   ├── charts/           <- 6 publication quality charts
    │   └── road_accidents_interactive_map.html
    └── README.md


    ---

## Tools Used

Python · pandas · scikit-learn · matplotlib · seaborn · folium · 
xgboost · lightgbm · numpy · Jupyter Notebook · GitHub

---

## Data Source

**Department for Transport — Road Safety Open Data (STATS19)**  
Record level data on personal injury road collisions in Great Britain 
from 2020 to 2024, the vehicles involved, and the consequential casualties.  
Licence: Open Government Licence v3.0  
gov.uk/government/statistical-data-sets/road-safety-open-data

Three files used:
- Road Safety Data — Collisions — last 5 years (93.4MB)
- Road Safety Data — Casualties — last 5 years (49.5MB)
- Road Safety Data — Vehicles — last 5 years (96.9MB)

**Note on data quality:** All decode maps were cross-validated against 
the official DfT Open Dataset Data Guide (2024 edition) before use. 
One error was identified and corrected during validation — casualty_type 
code 9 was initially mapped incorrectly and corrected to Car occupant 
(affecting 344,129 rows).

---

## How to Run

1. Clone this repository
2. Install requirements:
   `pip install pandas matplotlib seaborn scikit-learn folium xgboost lightgbm numpy openpyxl`
3. Download STATS19 last 5 years files from DfT and place in `/data/raw/`
4. Run notebooks in order: 01 → 02 → 03 → 04 → 05
5. Open `outputs/road_accidents_interactive_map.html` in browser for the map
