# wellbore-geology-predicton
Kaggle data science competition

# ROGII Wellbore Geology Prediction

## Project Overview

This project predicts **True Vertical Thickness (TVT)** for horizontal wells in the ROGII Wellbore Geology Prediction task.

Each well contains horizontal well measurements such as measured depth, spatial coordinates, gamma ray logs, and known TVT values up to the Prediction Start point. After the Prediction Start point, TVT is hidden and must be predicted.

The main objective is to use the horizontal well trajectory, gamma ray response, nearby geological information, and typewell data to estimate TVT accurately after the Prediction Start point.

## Problem Description

In each well, the model is given:

* `MD`: measured depth along the wellbore
* `X`, `Y`, `Z`: spatial coordinates of the well path
* `GR`: gamma ray values along the horizontal well
* `TVT_input`: known TVT values up to the Prediction Start point
* Typewell data containing known `TVT`, `GR`, and geological layer information

The target is to predict:

* `TVT` values after the Prediction Start point

Prediction quality is measured using **Root Mean Squared Error (RMSE)** between the predicted TVT and the true TVT values.

## Data Structure

The dataset contains two main files for each well:

```text
<well_id>__horizontal_well.csv
<well_id>__typewell.csv
```

### Horizontal Well File

The horizontal well file contains the well trajectory and log measurements.

Important columns include:

```text
MD
X
Y
Z
GR
TVT
TVT_input
```

### Typewell File

The typewell file represents a vertical reference well assigned to the horizontal well.

Important columns include:

```text
TVT
GR
Geology
```

The typewell provides a known relationship between gamma ray response and TVT, which helps estimate the TVT path of the horizontal well.

## Notebook Workflow

The notebook follows a dual-engine prediction workflow:

```text
Input CSV files
      ↓
Exploratory Data Analysis
      ↓
Feature Engineering
      ↓
Engine A: Ridge-SP
      ↓
Engine B: Drift-PF
      ↓
Final Blend
      ↓
Duplicate-well Recovery / Interpretation Hedge
      ↓
submission.csv
```

## Main Pipeline Components

### 1. Exploratory Data Analysis

The notebook first visualizes:

* The horizontal well trajectory
* Gamma ray behavior before and after the Prediction Start point
* Typewell gamma ray signature
* TVT drift after the Prediction Start point
* Spatial relationships between neighboring wells

This helps identify wells where simple baseline prediction fails due to geological drift.

### 2. Engine A — Ridge-SP

Engine A combines tracker-based predictions with machine learning models.

Main components:

* Multiple tracker variants
* Particle filter and beam-search estimates
* Selector routing based on well behavior
* LightGBM and CatBoost model stack
* Ridge meta-model blending
* Robust polynomial post-processing

Engine A is designed to stabilize predictions and reduce drift-related errors.

### 3. Engine B — Drift-PF

Engine B focuses on a likelihood-weighted particle filter and a separate ML stack.

Main components:

* Particle filter trackers
* Beam search trackers
* Cross-correlation features
* 128-seed likelihood-weighted particle filter
* Offset-well spatial priors
* Formation plane and nearby-well features
* LightGBM and CatBoost models
* Savitzky-Golay smoothing

Engine B is designed to be drift-resistant by combining physics-inspired tracking with machine learning.

### 4. Final Blend

The final prediction blends Engine A and Engine B:

```text
Final prediction = 0.55 × Engine A + 0.45 × Engine B
```

The purpose of blending is to reduce correlated errors because the two engines use different modelling strategies.

### 5. Duplicate-well Recovery and Interpretation Hedge

The notebook also includes a gated recovery step that checks for possible duplicate or highly similar wells.

Recovery is only applied when strict gates pass, such as:

* Very low pre-PS `TVT_input` error
* High full-length GR similarity
* High full-length Z similarity

This prevents unsafe copying from similar but geologically different wells.

## Model Outputs

The final output is:

```text
submission.csv
```

The file contains predicted TVT values for the hidden post-PS sections of the test wells.

Expected format:

```text
id,tvt
<well_id>_<row_index>,<predicted_tvt>
```

## Performance Summary

Example notebook performance summary:

| Stage               |       Score |
| ------------------- | ----------: |
| Last-known baseline | 15.91 ft CV |
| Engine A — Ridge-SP |    7.776 LB |
| Engine B — Drift-PF |    7.810 LB |
| Final hedge         |    7.528 LB |

## How to Run

1. Open the notebook in Kaggle or Jupyter.
2. Make sure the competition data is available in the expected input directory.
3. If using pretrained artifacts, attach the required artifact dataset.
4. Run all cells from top to bottom.
5. The notebook will generate `submission.csv`.
6. Submit `submission.csv` to Kaggle.

## Requirements

Main Python libraries used include:

```text
numpy
pandas
scikit-learn
lightgbm
catboost
scipy
numba
matplotlib
```

Depending on the environment, some models may use GPU acceleration.

## Project Notes

This solution combines geological reasoning, gamma ray matching, spatial well relationships, particle filtering, gradient boosting, and post-processing. The main challenge is preventing TVT drift after the Prediction Start point, especially in wells where gamma ray signatures are ambiguous.

The dual-engine structure improves robustness because each engine makes errors in different ways. Blending them gives a more stable final prediction.
