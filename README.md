# Anomaly Detection in Arms Trade

## Overview

This project explores anomaly detection techniques applied to international arms trade data. The goal is to identify unusual export/import patterns that may indicate reporting errors, unexpected transaction spikes, or potential illicit activity.

The repository contains the data, analysis notebook, model code and result visualizations produced during a master's-level scientific project.

## Goals

- Investigate anomalies in historical arms trade records.
- Apply density-based outlier detection (Local Outlier Factor, LOF) repeatedly to assess consistency.
- Transform LOF raw outputs into score-transformed arrays (stf) and compute model stability across runs.
- Produce visualizations and a reproducible analysis pipeline.

## Key Concepts

- LOF (Local Outlier Factor): an unsupervised anomaly detection algorithm that assigns negative_outlier_factor_ values (lower = more anomalous).
- STF (score-transformed): for consistency with other scoring methods in the project we convert LOF's `negative_outlier_factor_` to a positive anomaly score using `stf = -lof.negative_outlier_factor_`.
- Model stability: computed across multiple repeated runs of the model on the same data to measure how consistent the ranking/scores are. Important: LOF stability must be computed on the STF arrays (the converted positive scores), not on LOF internal state or raw negative values.
- LOF variance scaling: when gathering evaluation metrics we compute per-run variance of LOF-derived scores; we then apply MinMax scaling to the `lof_variance` column to produce `lof_variance_scaled` for normalized comparisons.

## Data

Data files (stored in `data/`):

- `trade-register.csv` — main arms trade records used for anomaly detection.
- `countries-in-conflict-data.csv` — supplemental country/context data used for filtering and plotting.

(See the Jupyter notebook for pre-processing steps applied to these files.)

## Methodology (high level)

1. Data cleaning and feature engineering: prepare numeric features suitable for LOF.
2. Model runs: repeat LOF multiple times (n_runs) using the chosen `n_neighbors` (k) and capture scores for each run.
   - Important: for each run, compute stf = -lof.negative_outlier_factor_ and store the stf vector.
   - Build a list `stf_scores_list` containing these stf arrays for all runs.
3. Model stability: pass the `stf_scores_list` to `calculate_model_stability(...)` so the stability reflects consistency of the score-transformed outputs across runs.

Example (conceptual):

- Generate stf list:

  stf_scores_list = []
  for i in range(n_runs):
      lof = LocalOutlierFactor(n_neighbors=best_k)
      lof.fit(X)
      stf = -lof.negative_outlier_factor_
      stf_scores_list.append(stf)

- Compute stability:

  lof_stability = calculate_model_stability(stf_scores_list)

4. Evaluation metrics and scaling:

- Create an `evaluation_df` (pandas DataFrame) from collected metrics (one row per experiment/setting).
- MinMax-scale the `lof_variance` column (ignoring NaNs) and store the result in a new column `lof_variance_scaled`.

Conceptual snippet for scaling (`sklearn.preprocessing.MinMaxScaler`):

  from sklearn.preprocessing import MinMaxScaler
  import numpy as np

  scaler = MinMaxScaler()
  col = "lof_variance"
  mask = evaluation_df[col].notna()
  if mask.any():
      evaluation_df.loc[mask, "lof_variance_scaled"] = scaler.fit_transform(evaluation_df.loc[mask, [col]]).ravel()
  else:
      evaluation_df["lof_variance_scaled"] = np.nan

This preserves NaNs and only scales existing numeric entries.

## Files and layout

- `Anomaly_Detection_In_Arms_Trade.ipynb` — primary analysis notebook (preprocessing, modeling, evaluation, plots).
- `data/` — input CSVs used by the notebook.
- `models/` — saved model artifacts (if produced).
- `results/` — generated plots and evaluation artifacts.
- `requirements.txt` — Python dependencies used for reproducibility.

## Reproduce the analysis

1. Create a virtual environment and install dependencies:

   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt

2. Open the notebook and run cells top-to-bottom. The notebook contains the exact preprocessing and parameters used to produce the figures in `results/`.

3. If you prefer to run scripts instead of the notebook, search for helper scripts in the repository or convert the notebook cells into a script.

## Notes & Implementation Tips

- Ensure LOF stability is computed on the STF arrays. If you currently collect LOF `negative_outlier_factor_` directly into the stability calculator, change to storing `-negative_outlier_factor_` instead.
- When scaling `lof_variance`, fit the MinMaxScaler only on non-null values to avoid propagating NaNs into valid scaled results.
- When comparing results between countries or time windows, always verify the same scaling and transformations are applied consistently.

## Results

A number of result images are included in `results/` (heatmaps, lineplots and evaluation plots). Check the notebook for generation code and parameters.

## Contributing

If you want to contribute fixes or improvements:

- Open an issue describing the change.
- Send a pull request with tests or a short demonstration (notebook cells) that reproduce the improvement.

## Contact

For questions about the analysis, methodology, or reproduction, open an issue in this repository or contact the project owner (see repository metadata).