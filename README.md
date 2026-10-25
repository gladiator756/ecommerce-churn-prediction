# E-Commerce Customer Churn Prediction

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/gladiator756/ecommerce-churn-prediction/blob/main/source-code.ipynb)

Compares six classifiers (Naive Bayes, MLP, SVM, Random Forest, AdaBoost, and a Logistic Regression baseline) for predicting customer churn on the Kaggle E-Commerce Customer Churn dataset. The notebook is saved with all outputs, so the results and figures are visible without running it.

**Try it in your browser:** click **Open in Colab** above, then choose *Runtime → Run all*. Nothing needs to be installed or uploaded: the notebook installs the right package versions and downloads the dataset from this repository. On Colab it runs in a **quick mode** (smaller hyperparameter grids, 3 outer folds) that finishes in about 3–5 minutes. Set `QUICK_MODE = False` in the first cell to reproduce the full results below.

## Results

Nested 5-fold CV on the training set (4,058 customers), plus a single evaluation on a held-out test set (1,015 customers, 168 churners):

| Model | CV ROC-AUC | CV PR-AUC | CV F1 | Test ROC-AUC | Test PR-AUC | Test F1 |
|---|---|---|---|---|---|---|
| Random Forest | 0.975 ± 0.006 | 0.899 | 0.827 | 0.982 | 0.945 | 0.888 |
| MLP | 0.965 ± 0.010 | 0.881 | 0.814 | 0.982 | 0.920 | 0.872 |
| SVM | 0.963 ± 0.010 | 0.860 | 0.808 | 0.974 | 0.914 | 0.865 |
| AdaBoost | 0.936 ± 0.022 | 0.781 | 0.691 | 0.941 | 0.791 | 0.716 |
| Logistic Regression | 0.883 ± 0.020 | 0.683 | 0.583 | 0.862 | 0.664 | 0.544 |
| Naive Bayes | 0.769 ± 0.031 | 0.493 | 0.411 | 0.767 | 0.482 | 0.395 |

Random Forest is selected on the nested-CV scores. MLP and SVM are within about one standard deviation of it and catch slightly more churners on the test set. Permutation importance ranks `Tenure` and `Complain` as the features the model relies on most. The notebook's last section discusses overfitting, stability and limitations.

## Method

- **Cleaning:** `CustomerID` dropped, inconsistent category spellings merged (`Mobile Phone`/`Phone`, `CC`/`Credit Card`, `COD`/`Cash on Delivery`, …), and **557 duplicate rows removed** so copies of the same customer can't appear in both training and validation data.
- **Hold-out test set:** a stratified 20% split, used once for the final evaluation and never for tuning or model selection.
- **Leak-free pipeline** (`imblearn.pipeline`), fitted inside every training fold:
  1. median imputation for numeric features, mode imputation for categorical ones
  2. **SMOTENC** oversampling of churners; synthetic rows always get real categories
  3. `StandardScaler` for numeric features, **one-hot encoding** for nominal categories
  4. classifier
- **Nested cross-validation:** `GridSearchCV` (3-fold, optimising ROC-AUC) is passed as the estimator to `cross_validate` (5-fold), so hyperparameters are never chosen on the fold they are scored on.
- **Diagnostics:** train vs validation gaps, learning curves, fold-to-fold stability boxplots, confusion matrices and ROC/PR curves on the test set, and permutation feature importance.

## Run it locally

1. Install Python 3.11+ and the dependencies:

   ```bash
   pip install -r requirements.txt
   ```

2. Open `source-code.ipynb` in Jupyter and run all cells. The dataset is read from `data/E_Commerce_Dataset.xlsx`. A full run takes about 3 minutes on a 12-core CPU. Results are reproducible with `RANDOM_STATE = 42`.

## Files

| File | Contents |
|---|---|
| `source-code.ipynb` | The full analysis, with outputs |
| `data/E_Commerce_Dataset.xlsx` | The dataset (sheet `E Comm`), see the credit below |
| `requirements.txt` | Pinned package versions used for the saved run |

## Dataset credit and licence

The dataset is [Ecommerce Customer Churn Analysis and Prediction](https://www.kaggle.com/datasets/ankitverma2010/ecommerce-customer-churn-analysis-and-prediction) by Ankit Verma, published on Kaggle under the [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) licence. It is included here unmodified (only the file was renamed) for non-commercial, educational use, under the same licence. All cleaning is done in the notebook, not in the file.
