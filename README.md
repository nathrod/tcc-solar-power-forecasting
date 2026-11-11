# Photovoltaic Generation Forecasting with Machine Learning

Forecasting photovoltaic power generation using machine learning models under multiple forecasting horizons and progressively richer meteorological feature sets.

This repository contains the code, notebooks and experimental pipeline developed for my undergraduate thesis in Information Systems at UNICAMP.

## Research objective

The main objective of this project is to investigate how the forecasting horizon and the progressive incorporation of meteorological variables affect the predictive performance of machine learning models applied to photovoltaic generation forecasting.

The contribution of this work is not the proposal of a new forecasting algorithm, but the experimental evaluation of the incremental contribution of different groups of meteorological information across multiple forecasting horizons.

The experiments compare:

- historical photovoltaic generation and temporal features;
- historical generation + solar radiation;
- historical generation + solar radiation + additional meteorological variables;

across forecasting horizons of:

- 1 hour;
- 12 hours;
- 24 hours.

## Research question

> How do forecasting horizon and the incorporation of meteorological variables influence the performance and robustness of machine learning models applied to photovoltaic generation forecasting?

## Study area

The study focuses on the photovoltaic generation complex located in **Dracena, São Paulo, Brazil**.

Generation and meteorological data were collected from public sources, including:

- **ONS — Operador Nacional do Sistema Elétrico**
- **INMET — Instituto Nacional de Meteorologia**
- **CAMS — Copernicus Atmosphere Monitoring Service**
- **ERA5 — ECMWF Reanalysis v5**

## Machine learning models

Three regression algorithms were evaluated:

- Support Vector Regression — SVR
- Random Forest
- XGBoost

Two persistence models were also used as forecasting baselines:

- Simple Persistence
- Daily Seasonal Persistence

## Experimental design

Three incremental feature sets were defined.

### E1 — Historical generation + temporal features

Generation features:

- `geracao_t`
- `geracao_t-1`
- `geracao_t-2`
- `geracao_t-3`
- `geracao_t-24`

Temporal cyclic features:

- `hour_sin`
- `hour_cos`
- `day_of_year_sin`
- `day_of_year_cos`

### E2 — E1 + solar radiation

Additional features:

- `radiacao_t`
- `radiacao_t-1`
- `radiacao_t-2`

### E3 — E2 + meteorological variables

Additional features:

- `temperatura_t`
- `umidade_t`
- `pressao_t`
- `precipitacao_t`

This results in:

**3 algorithms × 3 feature sets × 3 forecasting horizons = 27 trained machine learning models.**

## Forecasting strategy

The project uses a direct forecasting strategy:

`X(t) → y(t+h)`

where `h` represents the forecasting horizon.

The evaluated horizons are:

- `h = 1`
- `h = 12`
- `h = 24`

Models are trained independently for each forecasting horizon.

Observed future meteorological values are not used as model inputs.

Therefore, the 12-hour and 24-hour experiments should be interpreted as multi-horizon forecasting experiments based on information available at the forecasting origin, rather than complete operational day-ahead systems using future Numerical Weather Prediction data.

## Temporal validation

The dataset was divided chronologically in order to preserve the temporal structure of the problem.

The final common target periods were:

- Training: January 3, 2025 — August 31, 2025
- Validation: September 1, 2025 — October 31, 2025
- Test: November 1, 2025 — May 31, 2026

Hyperparameter selection was performed using temporal validation.

The final test period remained separated from model selection and hyperparameter tuning.

Time-series cross-validation was performed using `TimeSeriesSplit` with a forecasting-horizon-dependent gap.

## Evaluation metrics

The models were evaluated using:

- Mean Absolute Error — MAE
- Root Mean Squared Error — RMSE
- Coefficient of Determination — R²

RMSE was particularly useful for identifying configurations affected by larger forecasting errors.

## Main results

The best-performing configuration varied according to the forecasting horizon.

| Horizon | Best model | Feature set | MAE | RMSE | R² |
|---|---|---|---:|---:|---:|
| 1 h | Random Forest | E2 | 4.051 | 7.850 | 0.903 |
| 12 h | Random Forest | E3 | 6.952 | 12.318 | 0.761 |
| 24 h | XGBoost | E3 | 6.658 | 11.659 | 0.785 |

The main findings were:

- the 1-hour horizon produced the best predictive performance;
- performance deteriorated substantially at 12 hours;
- the 24-hour horizon performed better than the 12-hour horizon;
- adding solar radiation was especially useful for the 1-hour horizon;
- the complete meteorological feature set produced the most consistent improvement at 12 hours;
- meteorological improvements at 24 hours were smaller and model-dependent;
- Random Forest showed the most consistent performance across horizons;
- XGBoost achieved the lowest RMSE at 24 hours, although its result was very close to Random Forest.

## Important interpretation of the forecasting horizons

The non-monotonic behavior between 12-hour and 24-hour forecasts must be interpreted carefully.

The input feature set contains fixed generation lags relative to the forecasting origin:

- `t`
- `t-1`
- `t-2`
- `t-3`
- `t-24`

For a 1-hour forecast, `geracao_t` is immediately adjacent to the target.

For a 24-hour forecast, `geracao_t` corresponds exactly to the same hour one day before the target, providing a strong seasonal reference.

For a 12-hour forecast, the equivalent daily seasonal reference would be `geracao_t-12`, which was not included in the feature set.

Therefore, the stronger degradation observed at 12 hours may reflect both the daily photovoltaic cycle and the alignment of the selected historical features with each forecasting horizon.

This limitation is explicitly considered in the interpretation of the results.

## Uncertainty analysis

An additional empirical uncertainty analysis was performed using the Random Forest E3 configuration.

Prediction intervals were estimated from validation residuals using conditional residual quantiles.

Residuals were grouped according to predicted photovoltaic generation:

- 0–5 MWmed
- 5–20 MWmed
- 20–40 MWmed
- above 40 MWmed

The 2.5th and 97.5th percentiles of the validation residuals were used to construct empirical prediction ranges.

Summary results:

| Horizon | Mean width | Median width | Test coverage |
|---|---:|---:|---:|
| 1 h | 16.185 | 4.430 | 91.23% |
| 12 h | 27.131 | 4.073 | 87.68% |
| 24 h | 25.036 | 1.727 | 90.00% |

These intervals are empirical residual-based ranges and should not be interpreted as formal 95% confidence intervals.

## Repository structure

```text
.
├── notebooks/
│   ├── 00_meteorological_data_preparation_2025.ipynb
│   ├── 01_missing_data_treatment_2025.ipynb
│   ├── 02_target_dataset_construction_2025.ipynb
│   ├── 03_exploratory_data_analysis_2025.ipynb
│   ├── 04_dataset_extension_2026.ipynb
│   └── 05_multi_horizon_modeling_and_evaluation.ipynb
│
├── data
│
├── images
│
├── models/
│   ├── svr/
│   ├── random_forest/
│   └── xgboost/
│
├── metadata/
│   ├── feature_sets.json
│   ├── hyperparameters.json
│   ├── test_metrics.csv
│   └── library_versions.json
│
├── README.md
```

## Notebook workflow

### `00_meteorological_data_preparation_2025.ipynb`

Initial preparation and integration of meteorological data.

### `01_missing_data_treatment_2025.ipynb`

Treatment of missing values and data quality issues.

### `02_target_dataset_construction_2025.ipynb`

Construction of the photovoltaic generation target dataset and treatment of periods affected by operational generation restrictions.

### `03_exploratory_data_analysis_2025.ipynb`

Exploratory analysis of:

- photovoltaic generation;
- meteorological variables;
- correlations;
- temporal autocorrelation;
- daily generation patterns.

### `04_dataset_extension_2026.ipynb`

Extension and integration of the dataset with 2026 observations.

### `05_multi_horizon_modeling_and_evaluation.ipynb`

Main experimental notebook containing:

- feature engineering;
- horizon construction;
- baselines;
- temporal validation;
- hyperparameter optimization;
- SVR training;
- Random Forest training;
- XGBoost training;
- test evaluation;
- comparison between feature sets;
- comparison between horizons;
- comparison between algorithms;
- empirical uncertainty analysis.

## Model files

The final trained models are organized as:

```text
models/svr/svr_h{h}_e{experiment}.joblib
models/random_forest/rf_h{h}_e{experiment}.joblib
models/xgboost/xgb_h{h}_e{experiment}.joblib
```

Example:

```text
models/random_forest/rf_h1_e2.joblib
```

represents:

- Random Forest
- 1-hour forecasting horizon
- feature set E2

## Loading a trained model

Example:

```python
import joblib

model = joblib.load(
    "models/random_forest/rf_h1_e2.joblib"
)

prediction = model.predict(X)
```

The input columns must follow exactly the feature order used during training.

The corresponding feature sets are available in:

```text
metadata/feature_sets.json
```

## Physical constraints

The regression models were not explicitly constrained to generate non-negative outputs.

SVR and XGBoost may therefore occasionally produce negative photovoltaic generation predictions.

The original exported models reproduce exactly the configurations evaluated in the thesis.

For practical applications, a physical post-processing rule may be applied:

```python
import numpy as np

prediction = np.clip(prediction, 0, None)
```

This clipping was not used to calculate the official thesis metrics.

## Limitations

The main limitations of this study include:

- evaluation at a single photovoltaic generation site;
- relatively short historical period for climatological analysis;
- absence of future Numerical Weather Prediction inputs;
- fixed historical lag structure across forecasting horizons;
- no dedicated evaluation under extreme meteorological events;
- no validation across different photovoltaic plants or climate regions;
- uncertainty analysis based on empirical validation residuals rather than a dedicated probabilistic forecasting model.

## Future work

Several extensions can be explored:

- forecasting horizons beyond 24 hours, including 48 h and 72 h;
- incorporation of Numerical Weather Prediction data;
- hybrid models combining physical and data-driven information;
- probabilistic forecasting;
- quantile regression;
- model ensembles;
- larger historical datasets;
- evaluation across multiple photovoltaic plants;
- transfer learning between different regions;
- robustness analysis under adverse meteorological conditions;
- specific evaluation under extreme weather events.

A particularly relevant future research direction is the study of model robustness during meteorological extremes.

Such an analysis would require a longer climatological series and an objective definition of extreme events based on regional climatology, rather than arbitrary thresholds or statistical outliers alone.

This could support future research investigating how forecasting errors change under conditions such as:

- unusually high temperatures;
- intense precipitation;
- exceptionally low solar radiation;
- rapid irradiance ramps;
- highly variable cloud conditions.

## Thesis

This repository contains the experimental implementation developed for an undergraduate thesis in Information Systems at the University of Campinas — UNICAMP.

The thesis investigates the relationship between:

**forecast horizon × meteorological information × machine learning model performance**

in photovoltaic generation forecasting.

## Author

**Nathalia Rodrigues**

Information Systems  
University of Campinas — UNICAMP

## License

The source code can be released under an open-source license such as MIT.
