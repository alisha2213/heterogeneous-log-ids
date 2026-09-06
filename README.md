# Heterogeneous Log-Based Intrusion Detection System

A hybrid intrusion detection pipeline that fuses structured network traffic features with unstructured log text, compresses the combined representation through a deep autoencoder, and classifies attacks using a stacked ensemble of tree-based and linear models.

## Overview

Most intrusion detection approaches rely solely on structured network features (protocol, byte counts, connection flags) or solely on raw log text — rarely both. This project builds a heterogeneous fusion pipeline that combines the two feature families and trains an ensemble classifier on the resulting compressed representation, evaluated on the NSL-KDD benchmark dataset.

## Data

- **Structured features:** NSL-KDD dataset (41 network traffic features per connection, e.g. protocol type, service, byte counts, login attempts)
- **Log text:** HDFS log lines (from the loghub HDFS_2k dataset), mapped to NSL-KDD rows to simulate paired structured/unstructured intrusion data
- **Labels:** Binary classification — Normal vs. Attack

## Pipeline

### 1. Log Parsing (Drain-Inspired)
A lightweight Drain-style parser normalizes raw log lines by masking IP addresses, ports, and numeric values into placeholder tokens, producing consistent log templates suitable for vectorization.

### 2. Feature Extraction
- **Network branch:** 41 structured NSL-KDD features, label-encoded and standardized
- **Log branch:** TF-IDF vectorization (50 features, unigrams + bigrams) over parsed log templates

### 3. Heterogeneous Fusion
Standardized network features and TF-IDF log features are concatenated into a single fused feature matrix.

### 4. Deep Autoencoder Compression
A dense autoencoder (64 → 32 → 10 → 32 → 64) compresses the fused feature space into a 10-dimensional bottleneck representation, trained unsupervised to reconstruct the input.

### 5. Ensemble Classification
The 10-dimensional encoded features are passed to:
- **Base models:** Random Forest, XGBoost, Logistic Regression
- **Meta-ensembles:** Stacking (logistic regression meta-learner) and Soft Voting

## Results

| Model | Accuracy | Precision | Recall | F1-Score | AUC-ROC |
|---|---|---|---|---|---|
| Random Forest | 0.9999 | 1.0000 | 0.9998 | 0.9999 | 1.0000 |
| XGBoost | 0.9998 | 0.9998 | 0.9997 | 0.9997 | 1.0000 |
| Stacking (meta-LR) | 0.9998 | 0.9998 | 0.9997 | 0.9997 | 1.0000 |
| Voting (Soft) | 0.9997 | 0.9998 | 0.9995 | 0.9997 | 1.0000 |
| Logistic Regression | 0.9990 | 0.9991 | 0.9986 | 0.9989 | 0.9986 |

The stacking ensemble misclassified only 3 out of 12,598 test samples (2 false negatives, 1 false positive).

## Visualizations

### Autoencoder Training Loss
![Autoencoder Training Loss](images_IDS/autoencoder_loss.png)

### Confusion Matrices — Stacking vs. Voting
![Confusion Matrices](images_IDS/confusion_matrices.png)

### ROC Curve — Stacking vs. Voting
![ROC Curve](images_IDS/roc_curve.png)

### Full Model Comparison
![Model Comparison Table](images_IDS/model_comparison.png)

## Tools Used

- **Data processing:** pandas, NumPy, scikit-learn (LabelEncoder, StandardScaler, TfidfVectorizer)
- **Deep learning:** TensorFlow/Keras (autoencoder)
- **Classical ML:** scikit-learn (Random Forest, Logistic Regression, Stacking, Voting), XGBoost
- **Visualization:** Matplotlib, Seaborn

## Notes

Results reflect performance on the NSL-KDD test split with HDFS log lines mapped in as a proxy for paired unstructured log data, since NSL-KDD does not natively include raw log text. Near-perfect metrics are consistent with NSL-KDD's known separability at the binary (Normal vs. Attack) classification level.
