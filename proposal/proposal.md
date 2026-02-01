# Predicting Taxi Trip Duration and Congestion Pricing Exposure in New York City

## Team
- **Abhishek Mohan Hundalekar** (GitHub: hundalekar) - **Point of Contact (POC)**
- **Moses Kamya** (Github: mkamya20)
- **Tarun Singh** (GitHub: tsingh15syr)
- **Morgan Cook** (GitHub: mcook12e)



---

## Introduction

This project uses Yellow Taxi trip records published by the New York City Taxi and Limousine Commission for January 2025 to study taxi trip outcomes in New York City. The dataset contains detailed trip information including pickup and drop off times, locations, trip distance, passenger count, fare components, and a newly introduced Central Business District (CBD) congestion pricing fee.

The primary goal of this project is to use machine learning methods to predict taxi trip duration and to identify the conditions under which congestion pricing is applied. These predictions are intended to support a better understanding of urban mobility patterns and the early impacts of congestion pricing, rather than to prescribe actions for individual drivers.


---

## Literature Review

Urban mobility analysis has traditionally relied on descriptive statistics and classical regression models to study travel time and pricing outcomes. While these approaches provide useful summaries, they often struggle to capture the non-linear relationships present in large scale transportation data.

Recent research has demonstrated that machine learning models such as decision trees, random forests, gradient boosting, and neural networks can improve the prediction of travel time, fare amounts, and traffic demand by modeling interactions between temporal, spatial, and trip features.[^1][^2] Studies using taxi and ride-hailing datasets have found that time of day, trip distance, and geographic location are among the most important predictors of trip outcomes.[^3]

However, many prior studies rely on older datasets that do not include congestion pricing mechanisms, limiting their ability to evaluate policy interventions. In addition, some machine learning approaches prioritize predictive accuracy at the expense of interpretability, making them less suitable for policy analysis. This project addresses these limitations by using recently released data that includes congestion pricing and by combining interpretable baseline models with more flexible machine learning methods.

### Stakeholders and Their Needs

- **City transportation agencies** are stakeholders because they are responsible for managing congestion and evaluating transportation policies. They need insight into how travel times and congestion pricing vary across time and location.
- **Taxi drivers and fleet operators** are stakeholders because trip duration and pricing directly affect earnings and operational decisions. They need an understanding of conditions associated with longer trips or additional charges.
- **Riders and city residents** are stakeholders because congestion pricing and travel time affect travel cost, reliability, and accessibility.
- **Urban researchers and planners** are stakeholders because they use transportation data to study mobility patterns and evaluate policy impacts.

---

## Data and Methods

### Data

The primary dataset for this project is the **NYC Yellow Taxi Trip Records for January 2025**, published by the New York City Taxi and Limousine Commission. The dataset contains millions of trip records and includes the following types of variables:

- Temporal features: `tpep_pickup_datetime`, `tpep_dropoff_datetime`
- Spatial features: `PULocationID`, `DOLocationID`
- Trip characteristics: `trip_distance`, `passenger_count`
- Fare components: `fare_amount`, `tip_amount`, `tolls_amount`, `congestion_surcharge`, `Airport_fee`, `total_amount`
- Policy-related variable: `cbd_congestion_fee`

The dataset is accompanied by detailed metadata and data dictionaries, and it is collected through mandatory reporting by licensed taxi operators, which supports its reliability and credibility. A taxi zone lookup table will also be used to interpret location identifiers.

The data are available at:  
https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page

### Methods

Trip duration will be derived from pickup and drop-off timestamps. Congestion pricing exposure will be represented as a binary indicator derived from the `cbd_congestion_fee` variable.

Data preprocessing will include handling missing values, removing implausible trips, extracting time-based features (hour of day, day of week), and appropriately encoding categorical variables such as payment type and location identifiers.

The modeling approach will include:
- **Baseline models**: linear regression for trip duration and logistic regression for congestion pricing exposure.
- **Machine learning models**: random forests and gradient boosting to capture non-linear relationships.
- **Evaluation**: regression models will be evaluated using metrics such as RMSE and MAE, while classification models will be evaluated using precision, recall, and ROC-AUC. Evaluation choices are aligned with stakeholder needs for both accuracy and interpretability.

---

## Project Plan

| Period | Activity | Milestone |
|------|--------|----------|
| Feb 2 – Feb 23 | Data acquisition, cleaning, and exploratory data analysis | Completed data description and initial insights |
| Feb 24 – Mar 16 | Feature engineering and baseline model development | Initial regression and classification models |
| Mar 17 – Apr 6 | Advanced modeling and evaluation | Model comparison and interpretation |
| Apr 7 – Apr 27 | Final analysis and report preparation | Draft report and presentation slides |
| Apr 28 – May 5 | Revision and polishing | Final submission |

---

## Risks

Several risks may affect the project. First, the analysis is limited to taxi trips and does not represent all modes of transportation, which may restrict broader conclusions about citywide congestion. Second, the congestion pricing fee is implemented as a fixed surcharge, limiting its usefulness as a continuous prediction target. Third, a small number of records contain negative congestion fee values, likely due to administrative adjustments, which must be handled carefully during preprocessing.

To mitigate these risks, the project will clearly define its scope, treat congestion pricing as a categorical indicator, and apply conservative data cleaning procedures. If more advanced models do not provide meaningful improvements over baseline approaches, the project will emphasize interpretability and policy relevance over model complexity.

---

## References

[^1]: Zhang, S., et al. *Hybrid feature selection-based machine learning classification system for prediction of travel outcomes*. PLoS One, 2022.  
[^2]: Fiorentini, N., & Losa, M. *Handling imbalanced data in transportation prediction problems*. Infrastructures, 2020.  
[^3]: Lin, C., et al. *Predicting travel outcomes using machine learning methods*. Applied Sciences, 2020.
