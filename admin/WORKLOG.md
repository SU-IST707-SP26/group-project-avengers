# Work Log

This log documents significant work completed on the NYC Taxi Trip Duration and Congestion Pricing Prediction project.

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

