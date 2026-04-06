# Work Log

This log documents significant work completed on the NYC Taxi Trip Duration and Congestion Pricing Prediction project.
---


## 2026-03-18 - Advanced Modeling
**Context:** After establishing baseline regression and classification models, the team implemented advanced ensemble methods to improve predictive performance and better capture nonlinear relationships and complex feature interactions. Tree-based models, including Random Forest and XGBoost, were used to enhance model accuracy and provide deeper insight into the key drivers of congestion pricing.

**Individual Contributions**

1. Abishek: 2026-03-28
Built Random Forest starter pipelines for both project tasks (regression and classification), designed for Moses to tune further.
- Regression (Trip Duration):

 - RF Regressor with light regularization (max_depth=30, min_samples_leaf=5) on 500K sampled trips
 - RMSE = 3.99 min, MAE = 2.61 min, R² = 0.858 — a 26.8% RMSE improvement over the Linear Regression baseline
 - Stakeholder goal (RMSE < 10 min) achieved; overfitting controlled (train-test RMSE gap = 0.89 min)
 - Top feature: trip_distance (84% importance), followed by pickup_hour and location IDs

- Classification (Congestion Fee):

 - RF Classifier with class_weight='balanced' and same regularization; stratified split on target
 - Accuracy = 95.3%, AUC = 0.979, F1 = 0.968 — substantial improvement over Logistic Regression baseline
 - Minimal overfitting (train-test accuracy gap = 0.01)
 - Top features: PULocationID (40%) and DOLocationID (33%), confirming congestion fee is fundamentally location-driven

Data integrity: Dropped all post-trip financial features, leaky derived features (avg_speed_mph, cbd_fee_ratio, etc.), and payment_name (recorded at trip end). 21 clean features retained.

2. Morgan: Implemented a tree-based ensemble model to capture nonlinear relationships for both Classification and Regression ('has_congestion_fee', 'trip_duration_min')
  - Model Configurations:
    - 100 estimators
    - Max depth: 6
    - Learning rate: 0.1
  - Classification:
      * Objective: Improve classification performance on congestion fee prediction.
      * Approach:
        - Used XGBClassifier from XGBoost.
        - Split data (80/20) with stratify=y_congestion_fee.
        - Default hyperparameters: n_estimators=100, learning_rate=0.1, max_depth=5.
      * Evaluation:
        - Predicted congestion fee on the test set.
        - Calculated F1-score and accuracy.
  - Regression:
      * Objective: Model trip_duration as a continuous variable.
      * Approach:
        - Switched to XGBRegressor (regression mode, not classification).
        - Split data into training and test sets without stratification.
        - Used default hyperparameters: n_estimators=100, learning_rate=0.1, max_depth=5.
        - Evaluation:
        - Predictions on test set.
        - Evaluated using mean squared error (MSE).

3. Tarun: 2026-04-06
Built XGBoost models for both project tasks (classification and regression) with tuned hyperparameters and early-stopping validation (Tarun_XGB.ipynb).
  - Data and splits:
    - Stratified sample of 150,000 rows from data/processed/taxi_engineered.parquet (random_state=42), preserving has_congestion_fee class balance (74.5% / 25.5%).
    - Same leakage-safe column drops as team pipeline: all post-trip financial features, leaky derived features (avg_speed_mph, cbd_fee_ratio, etc.), and payment_name. 21 clean features retained.
    - 11 categorical columns (PULocationID, DOLocationID, RatecodeID, pickup_hour, pickup_day_of_week, time_slot, time_of_day, pickup_borough, dropoff_borough, distance_category, ratecode_name) handled natively via XGBoost enable_categorical=True.
    - Three-way split: 64% train / 16% validation / 20% test (stratified on target for classification).
  - Classification (Congestion Fee — XGBClassifier):
    - Hyperparameters: n_estimators=300, learning_rate=0.05, max_depth=6, min_child_weight=3, subsample=0.80, colsample_bytree=0.80, early_stopping_rounds=30.
    - Early stopping selected iteration 191 (validation log loss = 0.1145).
    - Test results: Accuracy = 96.3%, ROC-AUC = 0.986, F1 = 0.975, Precision = 0.984, Recall = 0.966.
    - Top features: PULocationID (32.3%), DOLocationID (22.6%), pickup_borough (11.2%), is_airport_pickup (11.2%) — confirming congestion fee is fundamentally location-driven.
    - Generated confusion matrix, ROC curve, and feature importance visualizations.
  - Regression (Trip Duration — XGBRegressor):
    - Hyperparameters: n_estimators=500, learning_rate=0.05, max_depth=6, min_child_weight=3, subsample=0.80, colsample_bytree=0.80, early_stopping_rounds=30.
    - Early stopping selected iteration 423 (validation RMSE = 3.92 min).
    - Test results: RMSE = 3.90 min, MAE = 2.50 min, R² = 0.866 — stakeholder goal (RMSE < 10 min) achieved; competitive with Abhishek's RF baseline (RMSE ≈ 3.99 min).
    - Top features: is_extreme_distance (38.2%), distance_category (29.5%), trip_distance (16.5%), is_same_borough (8.0%).
    - Generated actual-vs-predicted scatter, residual histogram, and feature importance visualizations.
  - Outputs: All metrics saved to tarun_xgb_metrics.json for downstream comparison.

4. Moses: 2026-03-18
Tuned Random Forest hyperparameters on Abhishek's starter (work/05_advanced_models/Moses_RF.ipynb) for trip duration (regression) and has_congestion_fee (classification).
  - Data and splits:
    - Load data/processed/taxi_engineered.parquet; sample 500,000 rows with random_state=42.
    - Same leakage-safe column drops as Abhishek; label-encode categoricals.
    - Outer split: 80% train / 20% test, random_state=42, stratified on has_congestion_fee.
    - Tuning split: 80% / 20% inside training only (X_tr / X_val) so the test set is held out until final evaluation.
  - Regression tuning:
    - Candidate grid (REG_CANDIDATES): n_estimators (100–500), max_depth, min_samples_leaf, optional min_samples_split, max_features (sqrt, 0.5, or full).
    - Pick best validation MAE on X_val; refit on full X_train; report test RMSE, MAE, R² vs. Abhishek's reference run (RMSE ≈ 3.99 min, MAE ≈ 2.61 min).
    - Optional: train on log1p(trip_duration_min) and map back with expm1 for metrics in minutes (skewed duration).
  - Classification tuning:
    - Candidate grid (CLF_CANDIDATES): same knobs plus class_weight balanced; rank by validation ROC-AUC; refit on full train; report test accuracy, ROC-AUC, F1.
  - Outputs:
    - Validation leaderboards, final test metrics, train vs. test gap, top 15 feature importances for the tuned regressor.

## 2026-03-04 - Baseline Modeling
**Context:** **Context:** Following data cleaning and feature engineering, baseline models were developed to establish benchmarks for both regression and classification tasks. A Linear Regression model was first used to predict continuous outcomes (e.g., trip duration), followed by a Logistic Regression model to predict whether a taxi trip incurs a congestion fee. These models provide interpretable benchmarks and serve as reference points for evaluating more complex models.

**Individual Contributions**

1. Abishek: 
2026-03-05 - Baseline Model: Linear Regression (Trip Duration Prediction)
Abhishek:
- Built baseline Linear Regression model to predict trip_duration_min using 500K sampled trips from the engineered dataset
- Identified and dropped 23 leaky/post-trip features (avg_speed_mph, fare_amount, tip_to_total_ratio, etc.) — kept only features a driver would know before accepting a trip
- Encoded 6 categorical features (time_of_day, pickup_borough, dropoff_borough, distance_category, payment_name, ratecode_name) and scaled with StandardScaler
- Results: Test RMSE = 5.45 min, MAE = 3.75 min, R² = 0.7386 — stakeholder goal of RMSE < 10 min achieved
- Model is 48.9% better than naive baseline (always predict mean = 10.66 min) with no overfitting (train-test RMSE gap = 0.09 min)
- Generated diagnostic plots (actual vs predicted, residual distribution, residuals vs predicted) and feature importance analysis via standardized coefficients
- Reviewed Morgan's logistic regression baseline and shared feedback on data leakage issues, missing location features, and class imbalance handling

2. Morgan: Built a baseline classification model to predict whether a trip incurs a congestion fee
  - Preprocessing:
    - Sampled 100,000 rows for efficiency
    - Removed data leakage features (post-trip financial variables)
    - Encoded categorical variables using Label Encoding
    - Standardized features using StandardScaler
    - Train/test split (80/20, stratified)
  - Model:
    - Logistic Regression with class balancing
  - Performance:
    - Accuracy: 0.669
    - ROC AUC: 0.734
    - Observations:
        - Strong performance in predicting congestion trips (high recall for class 1)
        - Lower precision for non-congestion trips (class imbalance impact)
        - Serves as a solid baseline for comparison

3. Tarun:

4. Moses: 2026-03-04
No separate Moses_Baseline.ipynb; contributed temporal features and team alignment so Abhishek's and Morgan's baselines use one consistent engineered table.
  - Features used by baselines:
    - Temporal columns from Moses_FE.ipynb merged into 03_feature_engineering.ipynb as taxi_engineered.parquet: pickup_hour, pickup_day_of_week, is_weekend, is_rush_hour and peak flags, time_of_day, hour_x_dayofweek, time_slot, cyclical sin/cos for hour and day-of-week.
  - Protocol alignment:
    - Coordinated target definitions (trip_duration_min, has_congestion_fee) and shared engineered Parquet for fair comparison.
    - Maintained admin/WORKPLAN.md milestones (data prep → EDA → FE → baseline → advanced).
  - Scope note:
    - Regression baseline: Abhishek_Baseline.ipynb; classification baseline: Morgan_Baseline.ipynb / checkpoint. Baseline-phase work here is feature and process support; primary tree modeling artifact is Moses_RF.ipynb (see Advanced Modeling).

---
	
## 2026-03-01 - Add model-focused EDA (In-Depth EDA Re-analysis)
Abhishek:
- Traced missing values (15.5%) to specific vendor systems — identified Vendor 6 as primary source
- Identified key data quality issues: negative fares, zero-distance trips, zero-passenger records, invalid RatecodeIDs, pre-congestion-pricing dates
- Analyzed distributions for trip duration, pickup hour, day of week, passenger count, and trip distance
- Explored relationships: duration varies ~40% by hour, Manhattan zones show 80-90% congestion fee rate vs airports at 30-44%, confirming location matters more than time for fee prediction
- Built data cleaning plan (12 actions) and feature engineering plan (6 features) with data leakage identification
- Documented congestion zone fee structure from MTA source ($0.75/trip, weekday 5AM-9PM, weekend 9AM-9PM, Manhattan south of 60th St)

---

---
## 2026-02-25 - Data Cleaning and Individual Feature Engineering

**Context:** Each team member independently did feature enginnering on the cleaned data set. Temporal/time, location/spatial, trip characteristics/dervied metrics, and encoding with interaction features were created.

**Individual Contributions**

1. Abishek: 
2026-02-20 - Feature Engineering: Trip Characteristics & Derived Metrics
Abhishek:
- Created avg_speed_mph (trip_distance / trip_duration_min * 60) with clipping at 60 mph and inf/NaN handling
- Built fare efficiency features: fare_per_mile and total_per_mile with division-by-zero protection and clipping
- Added passenger features: is_single_passenger flag and passenger_count_binned (solo/pair/group)
- Calculated cost ratios: tip_percentage and tip_to_total_ratio, both clipped to 0-100 range
- Created distance_category using pd.cut (short: 0-2mi, medium: 2-5mi, long: 5+ mi)
- Added outlier flags: is_extreme_distance and is_extreme_fare based on 95th percentile thresholds
- Total: 10 features created, 6 retained after team trimming for file size (avg_speed_mph, is_single_passenger, tip_to_total_ratio, distance_category, is_extreme_distance, is_extreme_fare)

2. Morgan: 2026-02-25
  - Missing Value Handling:
    - Analyzed missing values using raw dataset (~3.47M rows).
    - Identified ~15.5% missingness across several fields (passenger_count, RatecodeID, etc.).
    - Created missing value indicator features (5 total), documenting data quality considerations.
  - Categorical Encoding:
    - One-hot encoded key categorical variables:
    - Payment type (credit, cash, dispute, etc.)
    - VendorID 
    - RatecodeID (e.g., JFK, Newark, standard)
    - Encoded store_and_fwd_flag into a binary variable
  - Interaction Features (for non-linear relationships):
    - Distance x passenger count
    - Fare x payment type (credit vs cash)
    - Hour x day of week
    - Time slot feature (168 unique combinations)
  - Fare-Based Features (for better representation of trip cost structure):
    - Congestion surcharge ratio
    - Airport fee ratio
    - MTA tax ratio
    - CBD congestion fee ratio
    - Total surcharge ratio
    - Base fare ratio

3. Tarun: 2026-03-01
Feature Engineering: Geographical/Location Features (Tarun_FE.ipynb)
  - Borough Mapping:
    - Loaded NYC TLC Taxi Zone Lookup CSV (265 zones) and mapped each PULocationID and DOLocationID to its corresponding borough
    - Created pickup_borough and dropoff_borough columns; handled unmapped locations (288 pickup, 8,651 dropoff) by filling with 'Unknown'
  - Cross-Borough Trip Flags:
    - is_same_borough: binary flag for trips staying within one borough
    - is_cross_borough: binary flag for inter-borough trips (note: inverse of is_same_borough — one should be dropped before modeling to avoid multicollinearity)
    - borough_route: string feature capturing trip route (e.g., "Manhattan -> Brooklyn")
  - Airport Flags (9 features):
    - Mapped JFK (zone 132), LaGuardia (zone 138), and EWR (zone 1) using official TLC zone IDs
    - Created per-airport pickup/dropoff flags: is_jfk_pickup, is_jfk_dropoff, is_lga_pickup, is_lga_dropoff, is_ewr_pickup, is_ewr_dropoff
    - Combined flags: is_airport_pickup, is_airport_dropoff, is_airport_trip
  - Zone Popularity / Frequency Encoding:
    - pickup_zone_freq: normalized frequency of each pickup zone across all trips
    - dropoff_zone_freq: normalized frequency of each dropoff zone across all trips
    - route_id: combined PU-DO location pair as a single feature
    - route_freq: normalized frequency of each route pair
  - Output: 18 geographical features total; saved to data/processed/processed_taxi_with_geo_features.parquet
  - Notebook styled to match Moses's temporal FE notebook structure for team consistency

4. Moses: 2026-02-25
  - Data cleaning (supporting role on team pipeline):
    - Shared notebook work/02_data_cleaning/02_data_cleaning.ipynb outputs data/processed/processed_taxi_cleaned.parquet; focused on time-aware checks for downstream temporal features.
    - Confirmed tpep_pickup_datetime preserved and parsed so pickup_hour, DOW, and month stay valid after January 5–31, 2025 filter (post–CBD congestion pricing window).
    - Cross-checked trip_duration_min and binary has_congestion_fee (cbd_congestion_fee > 0) against VISION.md and proposal.
    - Reviewed invalid duration removal and distance/fare outlier rules so time-of-day and duration distributions support rush-hour and weekend flags in FE.
  - Feature engineering (temporal features; Moses_FE.ipynb):
    - Integrated into work/03_feature_engineering/03_feature_engineering.ipynb; output data/processed/taxi_engineered.parquet.
    - Calendar and clock: pickup_hour, pickup_day_of_week, pickup_month, pickup_day_of_month, pickup_day_name where used before encoding.
    - Flags from EDA: is_weekend; peak and rush definitions (evening peak 17:00–19:00, morning rush 7:00–9:00, evening rush 17:00–19:00, combined is_rush_hour).
    - Categorical time buckets: time_of_day, hour_x_dayofweek, time_slot (hour × DOW bins).
    - Cyclical encodings: sine/cosine of hour and day-of-week for continuity across midnight.
    - Combined notebook applies stakeholder-focused feature trimming per 03_feature_engineering documentation; temporal set feeds baselines and tree models unless a notebook drops columns for leakage.

---

## 2026-02-16 - Individual Exploratory Data Analysis

**Context:** Each team member independently explored the NYC Yellow Taxi dataset (January 2025, 3.47M trips, 20 columns) to understand data structure, quality issues, and key patterns before cleaning and modeling.

**Common Findings Across All Members:**
- Dataset: 3,475,226 trips with 20 features (~490-616 MB in memory)
- 5 columns with ~15.54% missing values (540,149 rows each): passenger_count, RatecodeID, store_and_fwd_flag, congestion_surcharge, Airport_fee
- Single-passenger trips dominate (~2.3M trips), confirmed via passenger count visualizations

**Individual Contributions:**
1. Abhishek: 2026-02-10
- Data quality issues identified: 144,118 negative fares, 24,656 zero-passenger trips, 162 trips over 100 miles, 55 trips over $500
- Max trip distance of 276423.57 miles — extreme outlier indicating data errors
- Max passenger count of 9 flagged as suspicious (taxis typically hold 4-6)
- Trip distance heavily right-skewed with mean of 5.86 miles

2. Morgan: 2026-02-15
- Identified 540,149 missing values in 5 columns: passenger_count, RatecodeID, store_and_fwd_flag, congestion_surcharge, Airport_fee
- Created side-by-side boxplots for trip_distance, fare_amount, and total_amount — all three show extreme outliers reaching into the hundreds of thousands, confirming need for outlier removal during cleaning
- Passenger count bar chart confirming single-rider dominance (~2.3M trips)

3. Tarun: 2026-02-15
- Engineered new features: trip_duration (minutes), pickup_hour, pickup_day, pickup_month
- Built correlation heatmap across all numeric features (including engineered ones)
- Scatter plots: Distance vs Fare, Fare vs Tip, Duration vs Total Amount (with 99th percentile capping for clarity)
- Location analysis: Top 20 pickup zones bar chart and Pickup→Dropoff heatmap (Top 15 zones) revealing trip flow patterns
- Categorical feature distributions for VendorID, payment_type, RatecodeID, store_and_fwd_flag

4. Moses: 2026-02-16
- Time pattern analysis: Identified peak hours at 18:00 (267,951 trips), 17:00 (253,518 trips), and 19:00 (221,055 trips) — evening rush hour dominance
- Trip duration analysis: Mean duration of 14.75 minutes, median 11.82 minutes, with most common duration of 9 minutes (filtered to 1-180 minute range)
- Payment and tipping behavior: 70.34% credit card payments, 67.83% of trips include tips with average tip of $4.36 when given
- Variable relationships: Strong correlation of 0.941 between trip distance and fare amount, and 0.803 correlation between trip distance and duration

**Impact:** Team has a shared understanding of data quality issues (missing values, negative fares, outliers), key distributions, and feature relationships. Findings will guide the data cleaning pipeline and feature engineering strategy in the next phase.

---

---

## 2026-02-08 - Notebook Structure Organization (Abhishek)

**Context**: Need to organize exploratory data analysis work and set up pipeline for future modeling phases.

**Solution Implemented**:
- Created structured notebook folders for all project phases:
  - `notebooks/01_data_exploration/` - Contains individual team member EDAs
  - `notebooks/02_data_cleaning/` - For data cleaning pipeline
  - `notebooks/03_feature_engineering/` - For feature creation
  - `notebooks/04_baseline_models/` - For linear/logistic regression
  - `notebooks/05_advanced_models/` - For random forests and gradient boosting
- Created placeholder notebooks in each folder
- Set up combined EDA notebook (`01_data_exploration.ipynb`)

**Impact**: Clear pipeline structure established. Team can work on different phases simultaneously without conflicts.

---

## 2026-02-08 - Source Code and Results Directories (Abhishek)

**Context**: Need to organize Python modules and model outputs separately from notebooks for better code reusability.

**Solution Implemented**:
- Created `src/` folder for Python source code modules:
  - `data_processing.py` - Data loading and preprocessing functions
  - `feature_engineering.py` - Feature creation functions
  - `models.py` - Model training and prediction functions
  - `evaluation.py` - Evaluation metrics and visualization
  - `__init__.py` - Package initialization
- Created `results/` folder for model outputs:
  - `figures/` - Visualization outputs
  - `model_performance/` - Performance metrics and comparisons

**Impact**: Modular code structure established. Functions can be imported across notebooks, reducing code duplication.

---

## 2026-02-08 - Project Management Documentation (Moses, Abhishek)

**Context**: Course requires VISION.md and WORKPLAN.md in admin folder for project tracking.

**Solution Implemented**:
- **Moses**: Created and committed `WORKPLAN.md` with project milestones and task breakdown
- **Abhishek**: Created `VISION.md` documenting project objectives, stakeholders, problem statement, and expected outcomes
- **Abhishek**: Added GitHub repository structure diagram to `GitHub Structure/` folder for documentation

**Impact**: Project vision and work plan clearly documented. Team aligned on objectives and timeline.

---

## 2026-02-08 - Data Infrastructure Setup (Abhishek, Moses)

**Context**: Project requires organized data storage with separation between raw and processed data.

**Solution Implemented**:
- **Abhishek**: Created data folder structure:
  - `data/raw/` - For original NYC Taxi parquet file
  - `data/processed/` - For cleaned and transformed datasets
  - Added README.md files documenting data sources and structure
- **Moses**: Uploaded `yellow_tripdata_2025-01.parquet` (87.3 MB) to `data/raw/`
- **Abhishek**: Set up folder for cleaned data outputs

**Impact**: NYC Yellow Taxi data (January 2025, 2.3M+ trips) successfully loaded and accessible to all team members. Data size within GitHub 100MB limit.

---

## 2026-01-31 - Project Proposal Development (Moses)

**Context**: Course checkpoint requires comprehensive project proposal including problem statement, literature review, methodology, stakeholders, and timeline.

**Solution Implemented**:
- Created detailed `proposal.md` with:
  - Team member information and point of contact
  - Introduction to NYC Taxi trip duration and congestion pricing prediction
  - Literature review on ML for transportation analysis
  - Stakeholder analysis (city agencies, drivers, riders, researchers)
  - Data description (NYC TLC Yellow Taxi data)
  - Methodology (baseline and ML models)
  - Project timeline (Feb-May 2025)
  - Risk assessment
- Updated proposal with refinements based on team feedback

**Impact**: Project scope, objectives, and approach clearly defined. Proposal approved by instructor. Clear roadmap for execution including baseline models (linear/logistic regression) and advanced models (random forests, gradient boosting).