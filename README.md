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
(See the Jupyter notebook for pre-processing steps applied to these files.)


## Files and layout

- `Anomaly_Detection_In_Arms_Trade.ipynb` — primary analysis notebook (preprocessing, modeling, evaluation, plots).
- `data/` — input CSVs used by the notebook.
- `requirements.txt` — Python dependencies used for reproducibility.

## Reproduce the analysis

1. Create a virtual environment and install dependencies:

   python -m venv .venv
   source .venv/bin/activate
   pip install -r requirements.txt

2. Open the notebook and run cells top-to-bottom. The notebook contains the exact preprocessing and parameters used to produce the figures in `results/`.

3. If you prefer to run scripts instead of the notebook, search for helper scripts in the repository or convert the notebook cells into a script.


## Results

A number of result images are included in `results/` (heatmaps, lineplots and evaluation plots). Check the notebook for generation code and parameters.

