# UK Road Safety Analytics: Casualty Patterns and Severity Prediction, 2020 to 2024

An MSc Business Analytics portfolio project by Obaloluwa Adelore. It analyses 640,449 road casualties in Great Britain using the Department for Transport's STATS19 open data, looks at when and where the worst collisions happen, and tests whether a model can predict which ones will be serious.

## 1. Background

Road collisions in Great Britain are recorded by the police and published every year as the STATS19 dataset. I used five years of it, 2020 to 2024, to ask four questions:

1. How has collision severity changed over the period?
2. When and where are the roads most dangerous?
3. Which road conditions go with the worst outcomes?
4. Can a model predict whether a collision will be fatal or serious?

### What "severity" means in this project

This matters for reading every number below. STATS19 records severity in two places. A collision has a severity set by its most seriously injured person, and each casualty has an individual severity. This project's severity labels, charts and model target all use the collision severity, attached to every casualty in that collision.

So a casualty counted as "fatal" in my tables is a casualty who was involved in a fatal collision. They may be a driver who was uninjured or slightly hurt in a collision where someone else died. The number of people who died is smaller. In the same data, the casualty level count of deaths is 7,954, against 12,389 casualties in fatal collisions. I use the phrases "casualties in fatal collisions" and "casualties in serious collisions" for this reason, and I do not call the 12,389 a death toll.

## 2. Data

| Item | Detail |
|---|---|
| Source | Department for Transport, Road Safety Open Data (STATS19) |
| Files loaded | Collisions (503,475 rows), casualties (640,522 rows) and vehicles (920,692 rows) |
| Files used in the analysis | Collisions and casualties, joined on the collision index. The vehicles file was loaded and checked but not used |
| Period | 1 January 2020 to 31 December 2024 |
| Licence | Open Government Licence v3.0 |
| Final dataset | 640,449 casualty rows, one per casualty |

Cleaning was light. Sixty five collisions had no location and were removed, and the casualty and vehicle files had no missing values or duplicates. Joining the casualties to the cleaned collisions left 640,449 rows, which is 73 fewer than the casualty file.

### A data check that caught an error

I decoded the numeric codes using the official DfT data guide for 2024. That check found a mistake in my own lookup table. Casualty type code 9 had been labelled "minibus occupant", but the guide defines it as car occupant. It affected 344,129 rows, more than half the data. I corrected the mapping and re-ran the analysis.

## 3. What the data shows

### 3.1 Severity over time

![Casualty severity trend](../outputs/charts/01_severity_trend_2020_2024.png)

| Collision severity | 2020 | 2024 | Change |
|---|---|---|---|
| Casualties in fatal collisions | 2,175 | 2,508 | +15.3% |
| Casualties in serious collisions | 24,493 | 32,071 | +30.9% |
| Casualties in slight collisions | 88,899 | 93,693 | +5.4% |

All three groups are higher in 2024 than in 2020, and the serious group grew fastest. Two cautions apply. The data starts in 2020, the year of the COVID lockdowns, when traffic was unusually low, so part of this rise is a recovery from an abnormal starting point. And the dataset has no earlier years, so it cannot show whether 2024 is higher than before the pandemic. I have also not adjusted for changes in how police forces record injury severity over time, which can affect trends in serious casualties.

### 3.2 When casualties happen

![Time patterns](../outputs/charts/02_time_patterns.png)

The busiest hour is 5pm, with 55,216 casualties across the five years, followed by 4pm (53,130) and 3pm (52,338). Friday is the busiest day at 105,883 casualties and Sunday the quietest at 76,194. These are two separate findings. The data does not show that Friday at 5pm is the single worst slot, because I counted hours and days separately.

These are counts, and busy times have more traffic. I did not calculate casualties per vehicle, so the results show when most casualties occur, not when a journey is riskiest.

### 3.3 Road type and speed limit

![Road conditions and speed limits](../outputs/charts/03_road_conditions.png)

The share of casualties that were in fatal or serious collisions rises with the speed limit up to 60mph.

| Speed limit | Casualties | Share in fatal or serious collisions |
|---|---|---|
| 20mph | 88,842 | 19.0% |
| 30mph | 328,180 | 22.4% |
| 40mph | 59,862 | 27.1% |
| 50mph | 31,942 | 30.5% |
| 60mph | 90,053 | 37.3% |
| 70mph | 41,555 | 27.4% |

Roads signed at 60mph have the highest share at 37.3%, above 70mph roads at 27.4%. The 60mph limit is the national limit for single carriageway roads outside built up areas, so this probably reflects rural roads, but I did not test that directly. By road surface, the share is 25.1% on dry roads and 26.6% on wet or damp ones. Flooding over 3cm shows 30.4%, from only 1,157 casualties.

### 3.4 Season

![Seasonal pattern](../outputs/charts/04_seasonal_pattern.png)

The share in fatal or serious collisions is highest in summer, at 27.1% in August and 26.4% in July, and lowest in winter, at 23.6% in January. October has the most casualties overall, at 58,143. So winter is not the most dangerous season by this measure, even though weather and darkness are often blamed. The data does not say why.

### 3.5 Where

![Geographic distribution](../outputs/charts/05_geographic_map.png)

The repository also holds an interactive map, `outputs/road_accidents_interactive_map.html`. Download it and open it in a browser. It has a density layer for all 161,400 casualties in fatal or serious collisions, pinpoints for the 12,389 in fatal collisions, and a 20,000 point sample of the serious ones, grouped into clusters. Each point is a casualty, so a collision with several casualties appears more than once.

## 4. Predicting severity

### 4.1 Set up

The target is binary: 1 if the collision was fatal or serious, 0 if slight. That is 25.2% of rows. The model uses 13 features about time, light, weather, road surface, speed limit, road type, rural or urban setting and the number of vehicles involved.

I split by time. The models trained on 2020 to 2022 (379,213 rows) and were tested on 2023 and 2024 (261,236 rows). The share of positive cases is 24.3% in training and 26.5% in testing. I set `class_weight='balanced'` to handle the imbalance.

### 4.2 Results

| Model | ROC AUC | F1 | Recall |
|---|---|---|---|
| Logistic regression | 0.606 | 0.412 | 0.525 |
| Random Forest (selected) | 0.634 | 0.440 | 0.579 |
| XGBoost | 0.629 | 0.436 | 0.573 |
| LightGBM | 0.632 | 0.439 | 0.577 |

The selected Random Forest also has accuracy 0.610 and precision 0.355. Train and test F1 differ by only 0.003, so it does not overfit.

![Model evaluation](../outputs/charts/06_model_evaluation.png)

### 4.3 How to read these results

The model has limited predictive power. An ROC AUC of 0.5 is chance, and 0.634 is only modestly above it. Precision of 0.355 means that of the casualties the model flags as likely serious, about two in three are in slight collisions. Recall of 0.579 means it catches a little over half of the real ones. I would not use it to make decisions about individual collisions.

All four models land between 0.606 and 0.634, and the three tree based models sit within 0.005 of each other. That suggests the features, not the algorithm, are the limit. The public data lacks things such as driver behaviour, vehicle condition and contributory factors, so I think better predictions would need new data and not a fancier model. This is my interpretation and I have not tested it.

### 4.4 What the model relies on

| Feature | Importance |
|---|---|
| Number of vehicles | 0.253 |
| Speed limit | 0.175 |
| Rural area | 0.139 |
| Single carriageway | 0.103 |
| Hour | 0.089 |
| High speed road | 0.067 |
| Month | 0.059 |
| Night | 0.029 |
| Dark | 0.025 |
| Weekend | 0.023 |
| Adverse surface | 0.015 |
| Rush hour | 0.012 |
| Adverse weather | 0.011 |

The number of vehicles involved matters most. Importance scores show which inputs the model used and do not prove cause. For example, a collision with only one vehicle often involves a pedestrian or a loss of control, and the data cannot separate those cases.

## 5. Conclusions

- Counts of casualties in serious and fatal collisions were higher in 2024 than in 2020, though 2020 was an unusual year.
- Higher speed limit roads, especially those signed at 60mph, show the highest share of serious and fatal outcomes.
- Summer months show a higher share of serious outcomes than winter months.
- Predicting severity from the public data is only modestly better than chance.

## 6. Limitations

| Limitation | Effect |
|---|---|
| Severity is collision level | Casualty counts by severity include people who were not seriously hurt. They are not counts of deaths or serious injuries. |
| Casualty rows are not independent | Casualties from the same collision share conditions, which makes patterns look stronger than they would at collision level. |
| Only five years, starting in 2020 | There is no pre-pandemic comparison, and 2020 was unusual. |
| No exposure data | Counts do not account for how much traffic there was, so busy times and roads look more dangerous. |
| Police recording changes | Changes in how forces classify injuries can affect trends in serious casualties. I have not adjusted for them. |
| Vehicles file unused | Vehicle and driver details were not part of the model. |
| Modest model | The best ROC AUC is 0.634. |

## 7. Tools

Python, pandas, NumPy, scikit-learn, XGBoost, LightGBM, matplotlib, seaborn, folium and Jupyter Notebook.

## 8. Source

Department for Transport, Road Safety Open Data (STATS19), collision, casualty and vehicle files for the last five years. Open Government Licence v3.0.
https://www.gov.uk/government/statistical-data-sets/road-safety-open-data
