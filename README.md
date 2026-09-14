# Unicauca Network Traffic — EDA and Multiclass Machine Learning

## Project Overview

This project analyzes network traffic flows from the Unicauca dataset using exploratory data analysis (EDA), feature engineering, supervised multiclass classification, and exploratory clustering.

**Research question:** To what extent can network-flow characteristics distinguish among application types, and which characteristics provide the greatest discriminatory information?

## Dataset

| Property | Value |
|---|---:|
| Original observations | 3,577,296 |
| Original attributes | 87 |
| Original application classes | 78 |
| Modeling sample | 300,000 |
| Classes retained for modeling | 55 |
| Rare classes excluded | 23 |
| Minimum retained class size | 20 |
| Target | `L7Protocol` |

The large source CSV is profiled in chunks. Computationally intensive modeling uses the reproducible 300,000-row sample.

## Cleaning and Leakage Prevention

The predictors exclude `Flow.ID`, `Source.IP`, `Destination.IP`, `Timestamp`, `Label`, `ProtocolName`, and `L7Protocol`. Numeric missing values are median-imputed and standardized. Engineered ratios/rates are protected against division by zero, with infinite values converted to missing values.

## Feature Engineering

Engineered features include forward/backward bytes per packet, packets per second, bytes per second, packet ratios, and byte ratios.

## EDA

The notebook examines class imbalance, feature distributions, outliers, correlations, application-level differences, and PCA structure.

## Modeling Requirements

### Baseline model

This is a **multiclass classification** task because `L7Protocol` represents an application/protocol class.

**Logistic Regression** is used as the baseline model. It is appropriate because it is a standard, comparatively simple classifier that provides a reference point before applying more flexible nonlinear models. The baseline is compared with a **Decision Tree** and **Random Forest** under the same preprocessing and stratified train/test design.

### Primary evaluation metric: Macro F1

The primary model-selection metric is **Macro F1**.

F1 combines precision and recall. Macro F1 calculates F1 independently for every modeled class and then averages the class-level scores with **equal weight for every class**.

### Rationale for Macro F1

The application classes are highly imbalanced. Accuracy alone can appear strong when a model performs well on large classes but poorly on smaller classes. Macro F1 gives each retained application class equal influence, making it appropriate for the goal of distinguishing among application types rather than optimizing only for the most common traffic.

Accuracy, balanced accuracy, precision, recall, and weighted F1 are also reported as complementary metrics.

### Valid interpretation

The Decision Tree's Macro F1 of **0.3950** means that the average class-level balance between precision and recall is approximately 0.395 when all 55 modeled classes receive equal weight.

It **does not mean 39.5% accuracy**. The Decision Tree's accuracy is **68.58%**.

### Actual classification results

| Model | Accuracy | Balanced Accuracy | Macro Precision | Macro Recall | Macro F1 | Weighted F1 |
|---|---:|---:|---:|---:|---:|---:|
| Decision Tree | 0.6858 | 0.3803 | 0.4401 | 0.3803 | **0.3950** | 0.6803 |
| Random Forest | **0.7429** | 0.3129 | **0.6498** | 0.3129 | 0.3751 | **0.7236** |
| Logistic Regression (Baseline) | 0.4810 | 0.0906 | 0.1309 | 0.0906 | 0.0913 | 0.4459 |

Using **Macro F1** as the primary metric, Decision Tree is selected because it has the highest Macro F1. Random Forest nevertheless has the highest overall accuracy and weighted F1. This difference demonstrates why metric choice matters for an imbalanced multiclass problem.

The tree-based models substantially improve on the Logistic Regression baseline, indicating that nonlinear relationships in the network-flow variables are important.

### Modeling rubric mapping

| Course requirement | Project response |
|---|---|
| Appropriate baseline model | Logistic Regression is the baseline for the multiclass `L7Protocol` task. |
| Clear evaluation metric | **Macro F1** is the primary model-selection metric. |
| Valid interpretation | Macro F1 averages class-specific F1 values with equal class weight; Decision Tree achieved 0.3950. |
| Clear rationale | Macro F1 is used because severe class imbalance makes accuracy alone potentially misleading. |

## Actual vs. Predicted and Error Analysis

The notebook saves observation-level actual-versus-predicted results to `actual_vs_predicted.csv`. For the selected model it also produces a confusion matrix, per-class precision/recall/F1/support, and top confusion pairs.

Important outputs include:

- `model_comparison.csv`
- `actual_vs_predicted.csv`
- `per_class_metrics.csv`
- `top_confusion_pairs.csv`

## Unsupervised Clustering

MiniBatchKMeans uses 55 clusters.

| Metric | Result |
|---|---:|
| Adjusted Rand Index | 0.0399 |
| Normalized Mutual Information | 0.1455 |
| Silhouette Score | 0.2430 |

The low ARI and NMI indicate that unsupervised clusters do not closely reproduce the known application labels.

## Reproducibility

The workflow uses `RANDOM_STATE = 42` for reproducible sampling, splitting, and applicable model operations. The supervised experiment uses an 80/20 stratified train/test split.

## Limitations

1. Modeling uses 300,000 observations rather than all 3,577,296.
2. Twenty-three rare classes are excluded, so supervised results apply to 55 classes.
3. Significant class imbalance remains.
4. Random splitting may place similar capture conditions in both train and test data.
5. Identifier and label-equivalent fields are deliberately excluded to reduce leakage.
6. K-Means is exploratory and shows weak agreement with the known labels.

## Key Findings

- Decision Tree has the highest **Macro F1: 0.3950**.
- Random Forest has the highest **accuracy: 0.7429** and **weighted F1: 0.7236**.
- Logistic Regression provides the baseline with **Macro F1: 0.0913**.
- The difference between macro and weighted metrics demonstrates the impact of class imbalance.
- K-Means does not closely reproduce application labels.

## Next Steps

Future work can evaluate hyperparameter tuning, class weighting, feature selection, stronger ensembles, and time-aware or group-aware validation.
