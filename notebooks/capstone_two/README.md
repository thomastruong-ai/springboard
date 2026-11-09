# SECOM Yield Prediction

Fault detection on the [UCI SECOM](https://archive.ics.uci.edu/dataset/179/secom) semiconductor manufacturing dataset: predicting pass/fail outcomes from in-line sensor measurements.

## Dataset

- 1,567 production entities, 590 sensor signals each, recorded with timestamps.
- Labels: `-1` = pass, `1` = fail. Only 104 failures (~6.6%), so the task is strongly imbalanced.
- Many columns contain missing values, constant values, or near-duplicate signals.

Download `secom.data` and `secom_labels.data` from UCI and place them in `data/raw/`:

```bash
bash scripts/download_data.sh
```

## Pipeline

1. **Cleaning** — drop constant and high-missingness columns, impute the rest (median / KNN).
2. **Feature selection** — variance threshold, correlation pruning, then univariate or model-based selection.
3. **Imbalance handling** — class weights, SMOTE, or undersampling (configurable).
4. **Models** — logistic regression, random forest, gradient boosting (XGBoost / LightGBM), and an MLP baseline.
5. **Evaluation** — stratified k-fold; reported with PR-AUC, recall on the failure class, F1, and MCC. Accuracy is not used, since predicting "pass" everywhere already scores ~93%.

## Usage

```bash
...
```

## Results

| Model | PR-AUC | Recall (fail) | F1 | MCC |
|---|---|---|---|---|
| Logistic regression | – | – | – | – |
| Random forest | – | – | – | – |
| XGBoost | – | – | – | – |

Means over 5 stratified folds. (Fill in once runs are complete.)

## Structure

```
data/        raw and processed data
runs/        checkpoints and logs
notebooks/   exploratory analysis, preprocessing, training, evaluation
outputs/     store data in each processing step
```

## Citation

McCann, M. and Johnston, A. (2008). *SECOM*. UCI Machine Learning Repository.

## License

MIT — see [LICENSE](LICENSE).
