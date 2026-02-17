**R.E.E.F. : ML-Enabled Coral Reef Bleaching Prediction**

Rapid Environmental Early-warning Forecaster (R.E.E.F.) is a machine learning–powered early warning system designed to help shift coral reef conservation from reactive monitoring to proactive, data-driven intervention. It predicts coral bleaching intensity (and risk categories) using integrated reef survey observations and ocean biogeochemical variables, and delivers insights through an interactive Streamlit dashboard. 

**Why this project?**

Coral reefs support a large share of marine biodiversity and provide major ecological and economic value, yet bleaching events are increasing under climate stress. Traditional workflows often rely on retrospective reporting and manual verification. This project builds a predictive analytics pipeline + dashboard to support earlier detection of bleaching risk and better resource prioritization for conservation stakeholders. 

**Data sources**

This project integrates multiple scientific datasets:

1) BCO-DMO (Reef Check 1997–2022): in-situ reef monitoring and bleaching observations (target = Percent_Bleaching) 

2) GLODAP (NOAA + partners): ocean biogeochemical variables (e.g., dissolved oxygen, alkalinity, salinity, pH, carbon system variables) 

3) ICRI Coral Restoration Database: used to support interpretive, recommendation-oriented outputs in the dashboard 
 
**Key approach**

**1) Data cleaning & preprocessing**
- Removed non-informative text fields
- Handled missingness using:
      - Median imputation for highly skewed variables (e.g., depth / distance-to-shore / SST climatology)
      - MICE (Multivariate Imputation by Chained Equations) for variables missing-at-random
  - Reverse geocoding for country/state/city tagging (for regional analysis & dashboard filtering) 

**2) Spatio-temporal integration via 4D interpolation**
Direct spatial joins between reef observations and GLODAP were too sparse (only ~140 matches even within 20km). To address this, the project implemented 4D Inverse Distance Weighting (IDW) interpolation using (lat, lon, depth, time) to generate synthetic but scientifically grounded local biogeochemical features for each reef observation. IDW was selected over alternatives (TPS, Kriging) due to better accuracy + scalability on large GLODAP volumes. 

**3) Modelling**
Regression (predict bleaching %)
Best performers:
- **ExtraTreesRegressor**: R² ≈ 0.67, RMSE ≈ 10.02
- **RandomForestRegressor:** R² ≈ 0.67, RMSE ≈ 10.06 
**Classification (predict risk level)**
Risk thresholds were aligned to severity bands:
- Low: <10%
- Medium: 10–30%
- High: >30% 
Models evaluated included XGBoost, LightGBM, Random Forest, KNN, SVM (with class imbalance acknowledged and SMOTE explored). 

**4) Explainability**
Model drivers were interpreted with SHAP, consistently highlighting variables like temperature and cyclone frequency among the most influential predictors. 

**5) Time-based forecasting (exploratory)**
SARIMA and Prophet were tested but limited by irregular sampling / non-continuous time series; a simpler trend-based projection approach was used for practical scenario forecasting within the dashboard. 

**The R.E.E.F. dashboard (Streamlit**)
The final deliverable is a single-page Streamlit app that combines descriptive + predictive analytics for decision support. Key features include: 
- **KPI Cards:** dataset coverage, bleaching stats, scope overview
- **Descriptive Analytics:**
  - top affected realms
  - bleaching trendline (2000–2019)
  - SHAP feature importance summary 

- **Region & Year Forecast:**
  - select realm + forecast year
  - view projected environmental factors + predicted bleaching
  - interactive severity map of sampling sites 

- **Scenario / “What-if” Predictor:**
  - manually input environmental conditions
  - instant bleaching prediction + severity gauge 

- **Bulk CSV Prediction:**
  - upload many sites at once
  - auto-fills missing required columns with training means for usability
  - expects features such as: Temperature_Kelvin, SSTA, SST, Depth, Turbidity, Windspeed, Cyclone_Frequency, G2oxygen 
- Model Confidence indicator shown in the UI (reported ~71%)

- **Tech stack**
- Python (pandas, numpy, scikit-learn)
- Imputation: MICE
- Geospatial: cKDTree / spatial indexing approaches; 4D IDW interpolation
- Explainability: SHAP
- Dashboard: Streamlit 
