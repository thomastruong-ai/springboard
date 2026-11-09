# Predicting Semiconductor Wafer Failures from Process Sensors

*Capstone Project Two, Springboard Data Science Career Track.* Semiconductor fabs run hundreds of in-line sensors on every unit, but pass/fail is only known at final test, after most of the cost is spent. This project asks whether the sensor readings can flag likely failures early enough to be useful, using the UCI SECOM dataset. The answer is that the signal is weak but real, and honest time-ordered evaluation is what shows it.

## 1. Data

- **Source:** [UCI SECOM](https://archive.ics.uci.edu/dataset/179/secom): 1,567 production units, 590 sensor readings each, a pass/fail label and a timestamp.
- **Target:** 104 failures (6.6%). Always predicting "pass" scores about 93% accuracy and catches nothing, so accuracy is not used.
- **Files** (in `data/`): `secom.data` (sensors, no header, `NaN` = missing), `secom_labels.data` (label and timestamp), `secom.names` (description).
- **Labels:** `-1` = pass and `1` = fail in the raw file. They are recoded to `0/1` so failure is the positive class.

## 2. Method

The task is rare-event binary classification on time-ordered data. Three rules shaped every step:

1. **Split by time first.** Train is the earliest 80% of units (1,253) and test is the latest 20% (314, with 17 failures). A random split would train on the future and look better than it deserves.
2. **Learn statistics from training data only.** Medians, clipping limits and scalers are fitted on train and applied to test.
3. **Never delete outliers.** With so few failures, extreme readings are likely to be the failures. They are clipped, not removed.

Models compared: Logistic Regression (L1 with feature selection), Random Forest and LightGBM. Earlier scripts also tried XGBoost, CatBoost, ExtraTrees, SVM, kNN and SMOTE variants. All use class weights. Hyperparameters were tuned with `GridSearchCV` or `RandomizedSearchCV` on `TimeSeriesSplit` (5 folds, gap 10), scored on PR AUC (average precision). A random model scores about 0.054 on that metric.

## 3. Data Wrangling

Notebook: [`01_data_wrangling.ipynb`](notebooks/01_data_wrangling.ipynb)

| Step | Result |
|---|---|
| Started with | 590 sensors, 1,567 units |
| Dropped mostly-missing sensors (>50%) | -24 |
| Dropped constant sensors | -122 |
| Dropped duplicated sensors (r > 0.9999) | -8 |
| Median imputation (from train) | all gaps filled |
| Added missing-value flags (`_isna`) for sensors missing >2% | +28 |
| Clipped to the 0.5% and 99.5% points of train | no units removed |
| **Final** | **464 columns**, 1,253 train and 314 test units |

Missing-value flags are kept because a sensor going offline can itself signal a problem. Scaling, feature selection and class balancing are deliberately left out of the saved data. They belong inside the cross-validation pipeline to avoid leakage.

## 4. EDA

Notebook: [`02_eda_secom.ipynb`](notebooks/02_eda_secom.ipynb). It uses the training partition only.

- Sensors are heavily skewed and their scales differ by orders of magnitude, so scaling happens inside the pipeline.
- Each sensor was tested with Mann-Whitney U (equal to per-sensor AUC) and Benjamini-Hochberg FDR correction. A few sensors pass, but AUC piles up near 0.5 and no single sensor separates the classes.
- Failures are shifted and more dispersed than passes, which suggests engineered features such as |z-score| or rolling SD.
- Several missing-value flags show a clear failure-rate lift.
- Correlation has block structure: most pairs are unrelated but some tight groups are redundant, probably sensors from the same tool. Regularised or ensemble models are therefore preferred.

## 5. Preprocessing

Notebook: [`03_preprocessing.ipynb`](notebooks/03_preprocessing.ipynb)

The dataset has no categorical columns, so no dummy encoding was needed. Numeric features are standardized with a scaler fitted on train only. The outputs are `train_processed.csv` and `test_processed.csv`.

## 6. Models

Notebook: [`04_models.ipynb`](notebooks/04_models.ipynb)

Test results (test set used once, after tuning):

| Model | Best settings | CV PR AUC | Test PR AUC | Test ROC AUC | Recall (fail) | Precision (fail) | F1 |
|---|---|---|---|---|---|---|---|
| **Logistic Regression** | L1, C=1, top 50 features | 0.182 | **0.166** | 0.574 | 0.294 | 0.106 | 0.156 |
| LightGBM | 100 trees, lr 0.03, 8 leaves | **0.247** | 0.078 | 0.580 | 0.000 | 0.000 | 0.000 |
| Random Forest | 500 trees, depth 8, `max_features=0.3` | 0.229 | 0.083 | 0.605 | 0.000 | 0.000 | 0.000 |

- LightGBM and Random Forest looked best in cross-validation but fell close to the random baseline on the later test period. This is the usual sign of patterns that do not carry forward in time.
- **Logistic Regression is the final model.** It is the only one with a clearly better-than-random test PR AUC (about 3x baseline) and the only one that detects failures. It is also the simplest to explain.

## 7. Limitations

- Only 17 failures in the test set, so one detection more or fewer moves recall by about 6 points. Differences between models are uncertain.
- Cross-validation preferred LightGBM, so the choice of Logistic Regression rests on the single test result. It should be confirmed with more data or a bootstrap interval.
- Precision is low. About 9 in 10 alerts are false alarms, so the model is a screening aid and not an automatic reject rule.

## 8. Future Improvements

- Add bootstrap confidence intervals on test F1, PR AUC and recall.
- Engineer instability features (rolling SD, |z-score|, lags) suggested by the EDA.
- Evaluate with rolling-origin (walk-forward) validation across many time windows to use more of the 104 failures.
- Calibrate predicted probabilities and weigh the cost of a missed failure against the cost of a false alarm.
- Try anomaly-detection and PCA residual methods.

## 9. Repository Structure

```
data/        raw SECOM files
notebooks/   01 wrangling, 02 EDA, 03 preprocessing, 04 models, and experiment scripts 05-10
outputs/     cleaned and processed data, CV and test result tables, plots
```

## 10. Credits

McCann, M. and Johnston, A. (2008). *SECOM*. UCI Machine Learning Repository. Thanks to my Springboard mentor for guidance.
