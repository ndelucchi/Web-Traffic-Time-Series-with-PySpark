# Web Traffic Time Series with PySpark

This repository contains the complete end-to-end pipeline for forecasting multi-series traffic volume data, designed and executed within PySpark, a distributed computing environment. The project leverages advanced time-series feature engineering and clustering techniques to improve predictive accuracy across diverse traffic patterns.

## Key Technologies

* **PySpark (MLlib):** Used for all data ingestion, distributed feature engineering, model training, and scaling the workflow across large datasets.
* **XGBoost (SparkXGBRegressor):** Selected as the primary machine learning model for its superior performance and handling of complex time-series features.
* **Hyperopt / Cross-Validation:** Employed for efficient tuning of model hyperparameters.
* **Pandas/Matplotlib:** Used for final model diagnostics and visualization (e.g., residual analysis).

## Methodology Highlights

1.  **Distributed Feature Engineering:** Implemented Spark window functions to compute critical time-series features across multiple partitions, including lagged features and rolling statistics (7-day and 30-day means/stds).
2.  **Clustering for Heterogeneity:** Applied Bisecting K-Means Clustering to group similar time series, using the resulting `cluster_id` as a powerful categorical feature to allow the model to learn context-specific dynamics.
3.  **Data Transformation:** The target variable (`TrafficCount`) was stabilized using a $\mathbf{\ln(x+1)}$ transformation to correct for heteroscedasticity and zero-bounded data.

## Results and Diagnostics

The final model achieved a **Mean Absolute Percentage Error (MAPE) of $\mathbf{48.867\%}$** on the test set. Crucially, a detailed diagnostic analysis revealed the source of error:

| Cluster ID | MAPE (%) | Finding |
| :---: | :---: | :--- |
| **0 & 1** | **$\mathbf{48.0\%} - \mathbf{48.5\%}$** | **High-Error Clusters.** These highly volatile, high-volume series are responsible for driving the overall error. |
| **2, 3, & 4** | **$\mathbf{16.89\%} - \mathbf{38.37\%}$** | **Model Strength.** The model performs reliably on series with stable or predictable traffic patterns. |

### Residual Analysis

Residual analysis confirmed two key limitations:
* **Heteroscedasticity:** Error magnitude scales with the predicted value, confirming high-volume series are the primary sources of variance.
* **Temporal Error:** Error spikes systematically align with periodic volatility (e.g., holidays/seasonality), confirming the need for enhanced event features.

## Future Work

To improve performance, the recommended next steps are:
1.  **Targeted Modeling:** Train specialized models with unique feature sets exclusively on the data belonging to the high-error **Clusters 0 and 1**.
2.  **Feature Enrichment:** Integrate external data (e.g., weather, local events) or enhanced lagged features (e.g., year-over-year lags) to better capture the sharp, non-linear traffic shocks observed in the residuals.