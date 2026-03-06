# Missense Variant Classification

Binary classifier (Pathogenic vs Benign) for missense variants using features from **dbNSFP**, queried via the [myvariant.info](https://myvariant.info/) REST API.

## Overview

- Fetches variant-level annotations (conservation scores, in-silico predictors, population allele frequencies) for 25 disease-associated genes
- Engineers biochemical delta features (Grantham distance, hydrophobicity, volume, charge, etc.)
- Trains and compares Logistic Regression, Random Forest, XGBoost, and LightGBM
- Handles class imbalance with SMOTE and evaluates with F1, AUC-ROC, and MCC

## Setup

```bash
pip install myvariant pandas numpy scikit-learn matplotlib seaborn xgboost lightgbm imbalanced-learn tqdm
```

## Usage

Run all cells in `main.ipynb` sequentially. The first run queries myvariant.info and caches results to `dbnsfp_raw_variants.json`; subsequent runs load from cache.

## Project Structure

```
main.ipynb                  # Full pipeline (data → EDA → model → evaluation)
dbnsfp_raw_variants.json    # Cached API responses (auto-generated)
best_model.pkl              # Saved best model (auto-generated)
scaler.pkl                  # Saved StandardScaler (auto-generated)
feature_columns.json        # Feature list (auto-generated)
```