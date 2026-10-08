# Student Dropout Prediction in Colombian Municipalities

Predicting the **secondary school dropout rate** in Colombian municipalities using regression-based machine learning models.

> Final project for **Mathematical Engineering** at **Universidad EAFIT** (School of Applied Sciences and Engineering), Medellín, Colombia, 2024.

---

## Table of Contents

- [Overview](#overview)
- [Business Context](#business-context)
- [Objectives](#objectives)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Results](#results)
- [Deployment](#deployment)
- [Limitations](#limitations)
- [Tech Stack](#tech-stack)
- [Authors](#authors)
- [References](#references)

---

## Overview

Secondary school dropout is a persistent problem in the Colombian education system. This project builds a predictive model that estimates the **intra-annual dropout rate of the public sector in secondary education** (`DESERCIÓN_SECUNDARIA`) for each municipality. The goal is to help the Ministry of Education identify high-risk areas and prioritize interventions.

The project follows the **CRISP-DM** process: business understanding, data understanding, data preparation, modeling, evaluation and deployment.

## Business Context

Dropout is influenced by socioeconomic level, teaching quality, geographic location, family situation and access to schools. Previous studies by the World Bank (2021) and the Colombian Ministry of Education (2022) suggest that early identification of at-risk populations is key to improving school retention.

## Objectives

**Business objectives**
- Reduce secondary school dropout by predicting dropout rates at the municipal level.
- Identify the municipalities with the highest projected dropout for targeted intervention.
- Explore which factors (e.g. geographic location, coverage) relate most strongly to dropout.

**Data mining objective**
- Build a regression model that estimates the secondary dropout rate from the available educational indicators.

**Success criterion**
- Mean Absolute Error (MAE) below **10%**. MSE and MAPE are used to understand model behavior, not as success criteria.

## Dataset

- **Source:** Open statistical data published by the Colombian Ministry of Education (MEN).
- **Coverage:** 2011–2023, Colombian municipalities.
- **Size:** 14,585 records × 41 columns.
- **Content:** Enrollment, coverage, dropout, approval, failure and repetition rates, among other education indicators.

### Data quality assessment

| Dimension | Findings |
|---|---|
| **Completeness** | `tamaño_promedio_de_grupo` (~48% null) and `sedes_conectadas_a_internet` (~47% null) are heavily incomplete. Most dropout, approval and coverage columns have <2% nulls. |
| **Validity** | Inconsistent department names (e.g. `Bogotá, D.C.` vs `Bogotá D.C.`, `Quindio` vs `Quindío`, fragmented `San Andrés` variants) and a non-official `NACIONAL` aggregate record. |
| **Consistency** | 2,344 records with enrollment rates above 100%, likely explained by internal migration not captured in DANE population projections. |
| **Accuracy** | No negative populations; approval and dropout rates do not exceed 100%. |
| **Uniqueness** | No duplicate records; repeated `código_municipio` / `año` values reflect the panel structure of the data. |
| **Integrity** | Not applicable (a single data source is used). |

## Methodology

### 1. Data preparation
- Validated consistency between related columns (`CÓDIGO_MUNICIPIO`/`MUNICIPIO`, `CÓDIGO_ETC`/`ETC`) and kept only one column from each redundant pair.
- For column groups sharing a prefix (e.g. `COBERTURA_NETA`), found linear relationships between them and kept only the `*_SECUNDARIA` columns.
- Removed records for a municipality that no longer exists, and dropped records and municipalities with excessive nulls in the target variable.
- Converted columns to numeric formats.
- Imputed remaining nulls using the mean of each municipality's non-null records.
- Detected outliers with **DBSCAN**.
- Analyzed covariance between the selected features and the target.
- Standardized the data and created two feature sets: one with all selected columns and one without the least relevant ones (`AÑO` and `POBLACIÓN`).

### 2. Modeling
Hyperparameter tuning with `GridSearchCV` (cross-validation, MAE as selection metric) for three regressors, each trained on both feature sets, for a total of **6 models**:

- Decision Tree Regressor
- Random Forest Regressor
- Gradient Boosting Regressor

`AÑO` and `POBLACIÓN` were excluded in the reduced feature set because they had the lowest correlation with the target.

### 3. Evaluation
Models were evaluated on a held-out test set using MAE, MSE and MAPE. Scatter plots (predicted vs. actual), error histograms and 95% confidence intervals were also produced. A naive baseline (constant prediction of the mean dropout rate) was used for comparison.

## Results

Best model: **Gradient Boosting Regressor**.

| Metric | Best model (test set) | Baseline (mean) |
|---|---|---|
| MAE | 8.93% | 7.23% |
| MSE | 12.6% | 52.41% |
| MAPE | 5% | 10.56% |

- The MAE of the best model is below the 10% success threshold.
- Predictions are accurate for most municipalities, with higher relative error in municipalities with extreme (very high or very low) dropout rates.
- Error histograms are centered around zero with few extreme errors.
- 95% confidence intervals were computed for the performance metrics (see the report/notebook for details).
- A hypothesis test was run comparing the best model against the mean baseline (see the report for details).

The final model was trained on the full dataset and saved for later use.

## Deployment

The selected model is deployed as a simple interface where the user enters the value of each input feature (`x`) of the dataset. The values are passed to the saved model, which predicts the secondary dropout rate (`y`) for that case. See the notebook for the implementation.

## Limitations

The results are affected by factors that are not measured in the dataset:

- Natural disasters and environmental emergencies (school closures and student relocation).
- Administrative delays in teacher hiring.
- Public order situations.
- Proportion of migrant population per municipality, and the limits of population measurement tools.
- Dropout broken down by gender.

Predictions should therefore be interpreted with this context in mind.

## Tech Stack

- **Language:** Python
- **Data processing:** Pandas, NumPy
- **Visualization:** Matplotlib
- **Modeling & evaluation:** Scikit-learn (Decision Tree, Random Forest, Gradient Boosting, GridSearchCV, DBSCAN)

## Authors

- Alain Phillip Jay Rodrigrez
- Jaime Andrés Jaramillo Ramírez
- Santiago Vera Ramírez

**Advisor:** Nicolas Pietro Escobar
Mathematical Engineering – Universidad EAFIT, Medellín, 2024

## References

- Banco Mundial. (2021). *Informe sobre la educación en América Latina.* Banco Mundial.
- Ministerio de Educación Nacional de Colombia. (2022). *Análisis de la situación educativa en Colombia.* Ministerio de Educación Nacional.

## Repository Structure

> Update this section to reflect your actual files.
