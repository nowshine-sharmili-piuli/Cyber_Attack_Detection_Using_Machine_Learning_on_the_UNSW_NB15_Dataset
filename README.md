# Cyber Attack Detection using Machine Learning on the UNSW-NB15 Dataset

A machine learning pipeline that classifies network traffic into normal
activity or one of several attack categories using the
[UNSW-NB15](https://research.unsw.edu.au/projects/unsw-nb15-dataset) 
dataset. Five classifiers — Random Forest, Decision Tree, Logistic
Regression, Naive Bayes, and XGBoost — are trained and compared on accuracy,
classification reports, confusion matrices, ROC curves, and precision-recall
curves.


## Contents

- [Dataset](#dataset)
- [Project structure](#project-structure)
- [Setup](#setup)
- [Usage](#usage)
- [Pipeline overview](#pipeline-overview)
- [Known limitations](#known-limitations)
- [Requirements](#requirements)

## Dataset

This project uses the **UNSW-NB15** dataset, created by the Australian Centre
for Cyber Security (ACCS). It contains a mix of real normal network traffic
and synthetically generated attack traffic across 9 attack categories
(Fuzzers, Analysis, Backdoors, DoS, Exploits, Generic, Reconnaissance,
Shellcode, Worms) plus Normal traffic.

The notebook expects the two official CSV splits:

- `UNSW_NB15_training-set.csv`
- `UNSW_NB15_testing-set.csv`

Download them from the [official dataset page](https://www.kaggle.com/datasets/mrwellsdavid/unsw-nb15)
and update the paths in the **Load Dataset** cell to point to your local copy:

```python
train = pd.read_csv("path/to/UNSW_NB15_training-set.csv")
test  = pd.read_csv("path/to/UNSW_NB15_testing-set.csv")
```

The notebook was originally written for Google Colab with the dataset stored
on Google Drive — update these paths if you're running locally or on a
different platform.

## Project structure

```
.
├── Cyber_Attack_Detection_Using_Machine_Learning_on_the_UNSW_NB15_Dataset.ipynb
└── README.md
```

## Setup

```bash
git clone <this-repo-url>
cd <this-repo>
pip install -r requirements.txt   # or see Requirements section below
```

Then place the UNSW-NB15 CSVs somewhere accessible and update the file paths
in the notebook's **Load Dataset** cell.

## Usage

Open the notebook in Jupyter, JupyterLab, or Google Colab and run all cells
top to bottom:

```bash
jupyter notebook Cyber_Attack_Detection_Using_Machine_Learning_on_the_UNSW_NB15_Dataset.ipynb
```

The notebook runs end-to-end: load data → EDA → preprocessing → train 5
models → compare results → visualize.

## Pipeline overview

1. **Load & merge data** — reads the training/testing CSVs and concatenates
   them into a single DataFrame.
2. **Exploratory Data Analysis (EDA)** — shape, dtypes, summary statistics,
   missing values, class distribution, correlation heatmap, feature
   histograms, boxplots, and a pairplot on a 1,000-row sample.
3. **Preprocessing**
   - Drop `id` and `label` (see [Corrections](#corrections-from-the-original-notebook))
   - Label-encode categorical columns (`proto`, `service`, `state`,
     `attack_cat`, etc.)
   - Split features (`X`) from the target (`y = attack_cat`)
   - 80/20 stratified train/test split
4. **Model training** — Random Forest, Decision Tree, Logistic Regression
   (with feature scaling), Naive Bayes, and XGBoost, each evaluated with
   accuracy.
5. **Evaluation & visualization**
   - Accuracy comparison table and bar chart
   - Per-model classification reports (precision, recall, F1)
   - Per-model confusion matrices
   - Feature importance (Random Forest and XGBoost)
   - Multi-class ROC curves and precision-recall curves (XGBoost)

## Corrections from the original notebook

This version fixes issues found in the original draft:

1. **Data leakage (critical).** The original code only dropped the `id`
   column before building the feature matrix `X`, leaving the `label` column
   (0 = Normal, 1 = Attack) in as a predictor. Since `label` is derived
   directly from the target `attack_cat`, every model could trivially tell
   Normal from Attack traffic, artificially inflating all accuracy and F1
   scores. **Fix:** `label` is now dropped along with `id`.
2. **Mislabeled plot.** The Logistic Regression confusion matrix was titled
   "Linear Regression Confusion Matrix." **Fix:** corrected to "Logistic
   Regression Confusion Matrix."
3. **Unscaled features for Logistic Regression.** Logistic Regression is
   sensitive to feature magnitude, and columns like `sbytes`/`dbytes` are on
   a much larger scale than binary flag columns, which can slow convergence
   and bias coefficients. **Fix:** added `StandardScaler`, applied only to
   the Logistic Regression model (tree-based models don't need it).

## Known limitations

- **Train/test split methodology.** The notebook merges the official
  UNSW-NB15 train and test CSVs and then creates a new random 80/20 split.
  The original benchmark split is intentionally designed so the test set
  contains attack patterns underrepresented in training, to measure
  generalization. Re-splitting randomly makes the task easier, so results
  here are **not directly comparable** to published UNSW-NB15 benchmark
  scores. If you need comparable numbers, keep `train` and `test` separate
  instead of merging them.
- **Class imbalance.** Some attack categories (e.g., Worms) have very few
  samples relative to Normal traffic and common attack types. No
  class-balancing technique (class weights, SMOTE, etc.) is applied, so
  minority-class metrics should be read with that in mind.
- **No cross-validation / hyperparameter tuning.** Models are trained and
  evaluated on a single split with largely default hyperparameters, which is
  fine for comparison purposes but means reported numbers may vary across
  runs and aren't fully tuned.
- **Label encoding on the merged frame.** Categorical columns are label
  encoded on the combined train+test data before splitting, which avoids
  unseen-category errors here but means the same approach would need
  adjusting (e.g., fit encoder on train only) if you separate the train/test
  sets per the limitation above.

## Requirements

```
pandas
numpy
matplotlib
seaborn
scikit-learn
xgboost
```

Install with:

```bash
pip install pandas numpy matplotlib seaborn scikit-learn xgboost
```

