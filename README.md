# Smart Meter Load Forecasting Across Moroccan Cities

## Overview

This project investigates electricity load forecasting across four Moroccan cities — **Laâyoune, Boujdour, Foum Eloued, and Marrakech** — using high-resolution smart meter data and a combination of classical statistical, machine learning, and deep learning approaches.

The project focuses on three main objectives:

* Understanding regional electricity consumption patterns.
* Comparing forecasting models across the four cities.
* Explaining model predictions using SHAP-based interpretability.

The workflow includes data preprocessing, unit standardization, time-series analysis, feature engineering, forecasting, model evaluation, and explainability.

---

## Research Questions

### RQ1 — Regional Consumption Patterns

What are the differences in electricity consumption patterns across Laâyoune, Boujdour, Foum Eloued, and Marrakech?

### RQ2 — Forecasting Accuracy

Which forecasting models provide the highest accuracy for electricity load prediction in each city?

### RQ3 — Model Interpretability

How can SHAP be used to explain feature contributions to electricity load forecasting across Random Forest, XGBoost, and GRU-based models?

---

## Cities Analyzed

| City        | Zones | Original Unit  | Sampling Frequency |
| ----------- | ----: | -------------- | ------------------ |
| Laâyoune    |     5 | Amperes (A)    | 10 min             |
| Boujdour    |     3 | Amperes (A)    | 10 min             |
| Foum Eloued |     7 | Amperes (A)    | 10 min             |
| Marrakech   |     2 | Kilowatts (kW) | 30 min             |

The three Ampere-based datasets are converted to kW using a three-phase power conversion formula. Marrakech is already provided in kW.

> **Methodological note:** A common power factor of 0.90 is assumed for the Ampere-based cities. City- or zone-specific power-factor measurements could improve conversion accuracy.

---

## Project Workflow

```text
High-Resolution Smart Meter Data
                ↓
        Data Inspection
                ↓
       Timestamp Cleaning
                ↓
     Duplicate Removal
                ↓
    Missing Timestamp Detection
                ↓
      Short-Gap Interpolation
                ↓
       Unit Standardization
             A → kW
                ↓
      Zone-Level Aggregation
                ↓
        Total City Load
                ↓
       Daily / Weekly Profiles
                ↓
     Exploratory Data Analysis
                ↓
       Feature Engineering
                ↓
   Train / Validation / Test
                ↓
      Forecasting Models
                ↓
 ┌──────────────┼──────────────┐
 ↓              ↓              ↓
ARIMA/SARIMA   RF/XGBoost    LSTM/GRU
 └──────────────┼──────────────┘
                ↓
     RMSE / MAE / MAPE
                ↓
       Model Comparison
                ↓
       SHAP Explainability
```

---

## Data Preprocessing

The raw data contains electricity measurements from multiple zones with different measurement units and sampling frequencies.

The preprocessing pipeline includes:

1. Loading the city-specific Excel sheets.
2. Standardizing timestamp representation.
3. Sorting observations chronologically.
4. Removing duplicate timestamps.
5. Reconstructing the expected time index.
6. Detecting missing timestamps.
7. Interpolating short gaps.
8. Converting Ampere measurements to kW.
9. Calculating total city-level load.
10. Aggregating data into daily and weekly profiles.

Short gaps are filled using time-based linear interpolation, while longer gaps are retained rather than artificially generating long sequences of observations.

---

## Unit Standardization

Laâyoune, Boujdour, and Foum Eloued contain measurements in Amperes, while Marrakech is already measured in kW.

The Ampere-based measurements are converted using:

```text
P(kW) = √3 × V(kV) × I(A) × PF
```

The implementation assumes:

```text
Line Voltage = 22 kV
Power Factor = 0.90
```

After conversion, the loads of all zones within each city are summed to create a `total_kW` city-level series.

---

## Exploratory Data Analysis

The project performs exploratory analysis to understand regional electricity consumption behavior.

The analysis includes:

* Descriptive statistics.
* Daily load trends.
* Zone-level load comparison.
* Hourly consumption profiles.
* Day-of-week patterns.
* Weekly behavior.
* Ramadan vs. non-Ramadan consumption.
* Regional comparison across the four cities.

These analyses support **RQ1** by identifying differences in load magnitude, variability, temporal behavior, and regional consumption patterns.

---

## Feature Engineering

The forecasting models use historical load and contextual information.

### Lag Features

The previous **14 daily observations** are used as historical predictors:

```text
Load(t-14)
Load(t-13)
...
Load(t-2)
Load(t-1)
       ↓
Predict Load(t)
```

### Calendar Features

The engineered dataset includes:

* `day_of_week`
* `month`
* `is_weekend`
* `is_ramadan`

### Rolling Features

The following seven-day statistics are also created:

* `roll_mean_7d`
* `roll_std_7d`

These capture recent average demand and short-term load variability.

---

## Train / Validation / Test Split

The time series is divided chronologically:

```text
70% → Training
15% → Validation
15% → Testing
```

The data is **not shuffled** because temporal order must be preserved.

The training set is used for model learning, the validation set is used for model selection and tuning, and the test set is reserved for final performance evaluation.

---

## Forecasting Models

Six primary forecasting approaches are evaluated.

### Classical Time-Series Models

* ARIMA
* SARIMA

### Machine Learning Models

* Random Forest
* XGBoost

### Deep Learning Models

* LSTM
* GRU

This provides a comparison between classical statistical forecasting, tree-based machine learning, and recurrent neural-network approaches.

---

## Model Evaluation

The models are evaluated using:

### RMSE

Root Mean Squared Error, which gives greater weight to larger prediction errors.

### MAE

Mean Absolute Error, representing the average absolute difference between actual and predicted load.

### MAPE

Mean Absolute Percentage Error, representing forecasting error as a percentage.

The models are compared across all four cities.

---

## Results

The best-performing model varies by city and evaluation metric.

| City        | Best RMSE Model |    RMSE | Best MAPE Model |   MAPE |
| ----------- | --------------- | ------: | --------------- | -----: |
| Laâyoune    | Random Forest   | 1889.23 | Random Forest   |  7.27% |
| Boujdour    | GRU             |  719.29 | XGBoost         |  7.79% |
| Foum Eloued | GRU             | 1646.85 | GRU             | 12.60% |
| Marrakech   | Random Forest   |  113.23 | Random Forest   |  7.15% |

The results indicate that there is **no single forecasting model that is universally optimal across all four cities**.

---

## SHAP Explainability

SHAP (**SHapley Additive exPlanations**) is used to understand how different features contribute to model predictions.

The interpretability analysis focuses on:

* Random Forest
* XGBoost
* Extended-feature GRU

The SHAP analysis includes:

* Global feature importance.
* Feature contribution distributions.
* Positive and negative contributions.
* SHAP dependence analysis.
* Cross-model feature importance comparison.

The analysis helps identify whether models rely primarily on historical load, rolling statistics, calendar information, or contextual variables such as weekends and Ramadan.

> SHAP describes model attribution and should not be interpreted as proof of causal relationships.

---

## Key Findings

* Electricity consumption patterns differ across the four Moroccan cities.
* Historical electricity load is an important source of forecasting information.
* Tree-based and recurrent deep-learning models generally provide stronger forecasting performance than the classical ARIMA/SARIMA baselines in the reported experiments.
* Random Forest performs particularly well for Laâyoune and Marrakech.
* GRU achieves the lowest RMSE for Boujdour and Foum Eloued.
* The model selected as "best" can differ depending on the evaluation metric.
* SHAP provides insight into the features that drive model predictions.
* Model behavior and forecasting accuracy vary across regions.

---

## Applications

The forecasting framework can support:

* Smart-grid planning.
* Electricity demand management.
* Infrastructure planning.
* Peak-load management.
* Demand-side management.
* Regional energy analysis.
* Energy policy planning.
* Forecasting applications in regions with similar smart-meter data.

---

## Technologies

* Python
* Pandas
* NumPy
* Matplotlib
* Seaborn
* Scikit-learn
* XGBoost
* Statsmodels
* TensorFlow / Keras
* SHAP

---

## Repository Structure

```text
moroccan-city-load-forecasting-ml/
│
├── README.md
├── Smart_Meter_Load_Forecasting_V7.ipynb
├── requirements.txt
│
├── data/
│   └── README.md
│
├── processed_data/
│   ├── daily/
│   ├── weekly/
│   └── native_resolution/
│
├── figures/
│   ├── eda/
│   ├── models/
│   └── shap/
│
└── results/
    ├── model_comparison.csv
    └── shap_feature_importance.csv
```

---

## Dataset

The project uses the **High Resolution Load Dataset from Smart Meters Across Various Cities in Morocco** from the UCI Machine Learning Repository.

Dataset:

https://archive.ics.uci.edu/dataset/1158/high-resolution+load+dataset+from+smart+meters+across+various+cities+in+morocco

---

## Research Article

**Smart Meter Load Forecasting across Moroccan Cities using Machine Learning Techniques**

Springer:

https://link.springer.com/article/10.1007/s12597-025-00950-w

---

## Limitations

* A uniform power factor of 0.90 is assumed for Ampere-based cities.
* City- or zone-specific power-factor measurements could improve unit conversion.
* Forecasting performance depends on the historical data and engineered features.
* SHAP explains model behavior but does not establish causality.
* Model performance varies across cities and evaluation metrics.

---

## Author

**Hassan Iqbal**

Artificial Intelligence & Generative AI

LinkedIn: `YOUR_LINKEDIN_URL`

---

## License

This repository is intended for research, educational, and portfolio purposes. Please refer to the original UCI dataset and research publication for their respective licensing and usage conditions.
