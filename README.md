# ROGII Wellbore Geology Prediction

## Overview

This project is based on the **ROGII Wellbore Geology Prediction** competition task.

The goal of the competition is to predict the geological position of a horizontal wellbore after a given **Prediction Start (PS)** point. The target variable is **TVT**, or **True Vertical Thickness**, which represents the stratigraphic depth of the wellbore within the geological column.

In real drilling operations, understanding the TVT position of a horizontal well is important because the wellbore may move up or down through geological layers as it is drilled. The challenge is to estimate this hidden TVT path using available well trajectory data and gamma ray measurements.

## Problem Statement

For each horizontal well, TVT values are provided only up to the **Prediction Start** point through the `TVT_input` column. After this point, the true TVT values are hidden and must be predicted.

The task is to predict TVT for all required post-PS rows in the test wells.

The prediction problem can be summarized as:

```text
Given:
- Horizontal well trajectory
- Gamma ray measurements
- Known TVT before the Prediction Start point
- A corresponding vertical typewell

Predict:
- TVT values after the Prediction Start point
```

## Dataset Structure

Each well is represented by two main CSV files:

```text
<well_id>__horizontal_well.csv
<well_id>__typewell.csv
```

## Horizontal Well Data

The horizontal well file contains measurements along the drilled lateral section.

Important columns include:

| Column      | Description                                       |
| ----------- | ------------------------------------------------- |
| `MD`        | Measured depth along the wellbore                 |
| `X`         | X coordinate of the well path                     |
| `Y`         | Y coordinate of the well path                     |
| `Z`         | Z coordinate or vertical position                 |
| `GR`        | Gamma ray measurement along the horizontal well   |
| `TVT`       | True TVT value, available in training data        |
| `TVT_input` | TVT values known up to the Prediction Start point |

The `TVT_input` column is especially important because it marks the part of the well where TVT is known. After the Prediction Start point, `TVT_input` becomes missing, and those are the rows that must be predicted.

## Typewell Data

Each horizontal well is assigned a corresponding **typewell**, which is a vertical reference well.

The typewell file contains the relationship between geological depth and gamma ray response.

Important columns include:

| Column    | Description                                      |
| --------- | ------------------------------------------------ |
| `TVT`     | True Vertical Thickness in the vertical typewell |
| `GR`      | Gamma ray measurement at each TVT depth          |
| `Geology` | Geological layer or formation name               |

The typewell provides a reference gamma ray signature for the geological column. By comparing gamma ray patterns from the horizontal well with the typewell, it is possible to estimate where the horizontal well is positioned in TVT space.

## Key Concepts

### True Vertical Thickness, TVT

**TVT** represents the stratigraphic depth of the wellbore within the geological column. It is the main value that must be predicted after the Prediction Start point.

### Prediction Start, PS

The **Prediction Start** point is the location in each horizontal well where known TVT information stops. Before this point, `TVT_input` is available. After this point, TVT must be predicted.

### Gamma Ray, GR

**Gamma ray** measurements are used as a geological signal. Different rock layers can produce different gamma ray responses, so GR logs can help identify where the wellbore is located relative to the typewell.

### Typewell

A **typewell** is a vertical reference well assigned to a horizontal well. It provides a known mapping between TVT and gamma ray response, helping support TVT prediction along the horizontal well.

## Objective

The objective is to produce accurate TVT predictions for the hidden post-PS sections of each test well.

The final submission should contain:

```text
id,tvt
```

Where:

| Column | Description                                    |
| ------ | ---------------------------------------------- |
| `id`   | Row identifier from the sample submission file |
| `tvt`  | Predicted TVT value                            |

## Evaluation Metric

Prediction quality is evaluated using **Root Mean Squared Error (RMSE)** between the predicted TVT values and the true TVT values.

RMSE is calculated as:

```text
RMSE = sqrt(mean((true_TVT - predicted_TVT)^2))
```

Lower RMSE means better prediction accuracy.

## Why the Task Is Challenging

This task is difficult because the gamma ray signal can be ambiguous. Similar GR values may appear at different TVT depths, making it possible to match the wellbore to the wrong geological layer.

The horizontal well may also drift upward or downward through the geological column after the Prediction Start point. This means that simply holding the last known TVT value constant is often not accurate enough.

The model must infer the hidden TVT path using a combination of:

* Gamma ray behavior
* Horizontal well trajectory
* Known TVT before the Prediction Start point
* Typewell reference data
* Geological continuity and spatial context

## Project Goal

The goal of this project is to understand and solve a realistic geosteering prediction problem where geological position must be inferred from limited well data.

The competition combines concepts from:

* Petroleum engineering
* Geology
* Wellbore trajectory analysis
* Gamma ray log interpretation
* Machine learning
* Time-series and spatial prediction
