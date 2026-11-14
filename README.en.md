<p align="left"><a href="README.md">🇪🇸 Español</a> | <b>🇬🇧 English</b></p>

# Network Attack Detection in a Network Infrastructure

A supervised machine learning project that classifies network connections as **normal traffic** or **anomalous (attack)** based on their features, using the **NSL-KDD** dataset. Two algorithms are compared: **logistic regression** and **decision tree**.

All the work lives in a single notebook: [`Network attack detection.ipynb`](Network%20attack%20detection.ipynb) (English version of [`Detección de ataques en red.ipynb`](Detecci%C3%B3n%20de%20ataques%20en%20red.ipynb)).

## Repository contents

```
.
├── Network attack detection.ipynb                    # Notebook with the full analysis (English)
├── Detección de ataques en red.ipynb                 # Original notebook (Spanish)
└── NSL-KDD/                                          # Dataset (ARFF and TXT)
    ├── KDDTrain+.arff / .txt                         # Full training set (125,973 records)
    ├── KDDTrain+_20Percent.arff / .txt               # 20 % subset of the training set
    ├── KDDTest+.arff / .txt                          # Full test set (22,544 records)
    ├── KDDTest-21.arff / .txt                        # Test set without difficulty-21 records (11,850)
    └── index.html                                    # Official dataset description
```

## Dataset: NSL-KDD

NSL-KDD is a cleaned-up version of KDD'99 that removes redundant and duplicate records, making it a reasonable benchmark for comparing intrusion detection (IDS) methods. Each record describes a connection with **41 features** (duration, protocol, service, bytes sent/received, error rates, etc.) plus the `class` label (`normal` / `anomaly`).

- More information: see `NSL-KDD/index.html` or the [official UNB page](https://www.unb.ca/cic/datasets/nsl.html).
- ARFF format: Weka's text format; read with the `liac-arff` library.

## Notebook workflow

0. **Imports**: installing `liac-arff` and loading libraries.
1. **Data loading**: helper functions `load_kdd_data`, `get_df_from_dict`, `remove_label` and `evaluate_classifier`.
2. **Exploratory analysis**: dimensions (125,973 × 42), types, absence of nulls, statistics and categorical variables (`protocol_type`, `service`, `flag`).
3. **Preprocessing**
   - Null value check.
   - `OneHotEncoder` for categorical variables (avoids the ordinal bias of `LabelEncoder`).
   - Feature selection by correlation with the class (|r| > 0.5) → **12 variables**:
     `count`, `serror_rate`, `srv_serror_rate`, `dst_host_serror_rate`, `dst_host_srv_serror_rate`, `flag_S0`, `logged_in`, `same_srv_rate`, `dst_host_srv_count`, `dst_host_same_srv_rate`, `service_http`, `flag_SF`.
   - Scaling with `RobustScaler` (robust to outliers).
   - 80/20 train/test split (`random_state=42`).
4. **Models**
   - `LogisticRegression(max_iter=1000)` on the 12 scaled variables.
   - `DecisionTreeClassifier(max_depth=50, random_state=42)` on the unscaled, non-reduced variables.
5. **Evaluation**: `F1-score` and confusion matrix.

## Results

| Model                | F1 (train) | F1 (test) |
|----------------------|-----------:|----------:|
| Decision tree        |     0.9999 |    0.9983 |
| Logistic regression  |     0.9144 |    0.9143 |

The decision tree clearly outperforms logistic regression: it is more flexible, captures interactions between variables and is more robust to outliers, whereas logistic regression can only model linear boundaries.

## Requirements

- Python 3.12 (tested with 3.12.8)
- Jupyter Notebook / JupyterLab
- `pandas`, `scikit-learn`, `matplotlib`, `seaborn`, `liac-arff`

## Running

```bash
# 1. Create and activate a virtual environment
python3 -m venv .venv
source .venv/bin/activate

# 2. Install dependencies
pip install jupyterlab pandas scikit-learn matplotlib seaborn liac-arff

# 3. Open the notebook from the project root
jupyter lab "Network attack detection.ipynb"
```

Dataset paths are relative (`NSL-KDD/KDDTrain+.arff`), so the notebook must be run from the repository root.

## References

- M. Tavallaee, E. Bagheri, W. Lu, A. Ghorbani, *A Detailed Analysis of the KDD CUP 99 Data Set*, IEEE CISDA, 2009.
- ARFF format: <https://www.cs.waikato.ac.nz/ml/weka/arff.html>
