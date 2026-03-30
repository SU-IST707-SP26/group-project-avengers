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

3. Tarun:

4. Moses:

## 2026-03-04 - Baseline Modeling
**Context:** **Context:** Following data cleaning and feature engineering, baseline models were developed to establish benchmarks for both regression and classification tasks. A Linear Regression model was first used to predict continuous outcomes (e.g., trip duration), followed by a Logistic Regression model to predict whether a taxi trip incurs a congestion fee. These models provide interpretable benchmarks and serve as reference points for evaluating more complex models.

**Individual Contributions**

1. Abishek: 

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

4. Moses:

---
## 2026-02-25 - Data Cleaning and Individual Feature Engineering

**Context:** Each team member independently did feature enginnering on the cleaned data set. Temporal/time, location/spatial, trip characteristics/dervied metrics, and encoding with interaction features were created.

**Individual Contributions**

1. Abishek: 

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

3. Tarun: 

4. Moses: 
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

