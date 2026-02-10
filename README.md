# ❄️ NYC Taxi Demand Forecasting: Snowflake & Snowpark MLOps

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue?logo=python&logoColor=white)](https://www.python.org/)
[![Snowflake](https://img.shields.io/badge/Data_Cloud-Snowflake-29B5E8?logo=snowflake&logoColor=white)](https://www.snowflake.com/)
[![Snowpark](https://img.shields.io/badge/Compute-Snowpark_Python-29B5E8?logo=snowflake&logoColor=white)](https://docs.snowflake.com/en/developer-guide/snowpark/python/index)
[![LightGBM](https://img.shields.io/badge/Model-LightGBM-orange)](https://lightgbm.readthedocs.io/)
[![Status](https://img.shields.io/badge/Status-Development-yellow)]()

## 📋 Executive Summary
This project implements a secure, scalable **MLOps pipeline** for forecasting NYC Taxi demand directly within the **Snowflake Data Cloud**.

Unlike traditional pipelines that extract data to external servers for processing (incurring egress costs and security risks), this architecture leverages **Snowpark Python** to push compute *to* the data. It handles the full lifecycle—ingestion, cleaning, feature engineering, and model inference—without a single row of data ever leaving Snowflake's governed boundary.

---

## 🏗️ System Architecture

The pipeline follows a **Medallion Architecture** (Raw -> Silver -> Gold) pattern, orchestrated via Python scripts and Snowpark DataFrames.

```mermaid
graph LR
    A[Parquet Files] -->|PUT & COPY| B[(RAW Layer)]
    B -->|Snowpark Filter| C[(FILTERED Layer)]
    C -->|Snowpark Transform| D[(TRANSFORMED Layer)]
    D -->|Feature Eng.| E[Snowpark ML]
    E -->|Train (Optuna)| F[LightGBM Model]
    F -->|Inference| G[(PREDICTIONS Table)]
    G -->|SQL| H[Dashboard]
```
### Key Engineering Decisions
**1. Zero Data Movement:** All transformations (src/filter_data.py, src/transform_data.py) execute lazily on Snowflake warehouses. This allows processing of billions of rows without memory constraints on the client side.

**2. Snowpark for ETL:** Replaced legacy SQL Scripts with Snowpark DataFrames to allow for modular, testable, and Pythonic data transformations while maintaining SQL-like performance.

**3. Hyperparameter Tuning:** Integrated Optuna for automated hyperparameter optimization of the LightGBM regressor to minimize RMSE.

📂 Project Structure

```
├── sp25_taxi_snowflake/
│   ├── src/
│   │   ├── filter_data.py      # Cleansing logic (Outlier removal via approx_percentile)
│   │   ├── transform_data.py   # Feature engineering (Rolling windows, Lag features)
│   │   └── utils.py            # Shared Snowflake session management
│   ├── sqls/
│   │   └── dashboard.sql       # Analytical queries for comparing Actual vs. Predicted
│   ├── notebooks/              # Exploratory Data Analysis (EDA) & Prototype Training
│   └── requirements.txt        # Python dependencies (snowflake-snowpark-python, etc.)
├── .gitignore
├── README.md
└── .env.example                # Template for Snowflake credentials
```
### 🚀 Getting Started
**Prerequisites**
1. Python 3.9+
2. A Snowflake Account (Free Trial works) with `ACCOUNTADMIN` or `SYSADMIN` role.

**Installation**
1. Clone the repository
`
git clone [https://github.com/sharmiladevi23/nyc-taxi-snowflake-mlops.git] (https://github.com/sharmiladevi23/nyc-taxi-snowflake-mlops.git)`
`cd nyc-taxi-snowflake-mlops`
2. Set up Virtual Environment

`python -m venv .venv`
`source .venv/bin/activate`
`pip install -r sp25_taxi_snowflake/requirements.txt`

3. Configure Credentials Copy `.env.example` to `.env` and fill in your Snowflake details:

`cp .env.example .env` Edit .env with your SNOWFLAKE_USER, PASSWORD, ACCOUNT, WAREHOUSE

### Execution Guide
Step 1: Ingest Raw Data Uploads local parquet files to an internal Snowflake stage and loads the RAW table.

Run `notebooks/01_upload_data.ipynb`

Step 2: Cleanse & Filter (Raw -> Silver) Executes Snowpark logic to remove outliers using statistical thresholds.
`
python sp25_taxi_snowflake/src/filter_data.py`

Step 3: Feature Engineering (Silver -> Gold) Generates time-series features (grid generation, missing value imputation).


`python sp25_taxi_snowflake/src/transform_data.py`

Step 4: Train & Predict Trains the LightGBM model on the Gold layer and writes predictions back to the PREDICTIONS table.

Run `notebooks/02_train_and_predict.ipynb`

### 📊 Results & Dashboarding
You can visualize the model performance directly in Snowflake using Snowsight Dashboards or by running the SQL queries found in sqls/dashboard.sql.

1. Target Metric: Demand (Rides per Hour per Zone)

2. Evaluation: RMSE (Root Mean Squared Error) comparisons against a seasonal baseline.

### 🔮 Future Roadmap
1. Stored Procedures: Migrate training logic from Notebooks to Snowflake Stored Procedures to fully automate the pipeline via Snowflake Tasks.

2. CI/CD: Implement GitHub Actions to lint Snowpark code and run unit tests on Pull Requests.

3. Orchestration: Deploy an Apache Airflow DAG to trigger the pipeline steps on a daily schedule.

### Author: 
Sharmila Devi
