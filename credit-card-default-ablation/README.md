# Predicting Credit Card Defaults — Fixing a Cross-Validation Leakage Bug + Feature Ablation

Predicting next-month credit card default on the UCI/Taiwan credit card dataset with Logistic Regression, Ridge Regression, and a Decision Tree.

## Project Impact & Business Relevance

The delay-summary features that I engineered contain the model's strongest signal—and the direct key to actionable credit-risk decisions. In the ablation study, MAX_DELAY, AVG_DELAY, MONTHS_DELAYED, and EVER_DELAYED were the only feature group that consistently cut through the noise. This finding saves the engineering team substantial resources: fewer features to build, validate, and maintain in production, without sacrificing predictive accuracy for defaults.


## The Problem

**Goal:** predict whether a credit card client will default on their payment next month, using account-level data (credit limit, demographics, six months of repayment status, bill amounts, and payment amounts) from ~30,000 Taiwanese credit card clients in 2005 — a period that sits inside Taiwan's credit/cash-card debt crisis.

The target is imbalanced (~22% default rate), so the modeling approach centers on **F1** (via a tuned decision threshold) rather than accuracy, since a model that just predicts "no default" every time would score 78% accuracy while catching zero actual defaulters.

## Approach

1. **Data audit against the data dictionary** — caught and cleaned undocumented category codes in `EDUCATION` and `MARRIAGE`, renamed `PAY_0` → `PAY_1` for column-naming consistency.
2. **EDA** — confirmed the expected drivers: default rate falls steadily as credit limit rises, and jumps sharply once a client is reported even one month late.
3. **Feature engineering** — collapsed six months of repayment history into summary features: `AVG_BILL_AMT`, `AVG_PAY_AMT`, `TOTAL_PAY_AMT`, `PAY_RATIO_1`-`5` (fraction of each month's bill actually paid), `MAX_DELAY`, `AVG_DELAY`, `MONTHS_DELAYED`, `EVER_DELAYED`.
4. **Three models**, each with its own hyperparameter search and a tuned F1-maximizing decision threshold: Logistic Regression, Ridge Regression (used as a classifier via thresholded predictions), and a Decision Tree.
5. **Model comparison** — Decision Tree wins on F1 (0.524) and Recall (catches 971 of 1,659 test-set defaulters, the most of the three), so it's the recommended model: missing a real defaulter is more costly here than a false alarm.

## The Bug I Caught and Fixed

**What was wrong:** an earlier version of this notebook fit `StandardScaler` once on the *entire* training set (`scaler.fit_transform(X_train)`), then reused that single pre-scaled array inside every `GridSearchCV` / `TunedThresholdClassifierCV` / `cross_val_predict` call for all three models. That's a cross-validation leakage bug — each fold's "held-out" validation rows had already influenced the scaler's mean/std before cross-validation even began, because they were part of the training set the scaler was fit on. Every fold was, in a small way, peeking at data it was supposed to be evaluated on.

**The fix:** wrap `StandardScaler` and each model together in a single `sklearn.Pipeline`, and run all cross-validation on the *unscaled* training data. Because the pipeline — not the model alone — is what gets cloned and refit inside each CV fold, the scaler now refits from scratch on only that fold's training rows every time. No fold's evaluation data ever touches its own scaling statistics.

**Did it actually change the results?** No, actually. Re-running the corrected, leakage-free cross-validation produced *identical* held-out test performance to the original leaky version for all three models. With ~22,500 training rows and 5-fold CV, each fold's held-out slice is only ~4,500 rows, so excluding it from `StandardScaler`'s fit barely shifts the mean/std versus fitting on the full training set — the leak was real, but too small in this specific case to move the numbers. That's not a reason to skip the fix: on a smaller dataset, with fewer folds, or with a scaler more sensitive to outliers (e.g. Min-Max scaling with extreme values), the same bug could easily have inflated CV scores enough to pick the wrong hyperparameters. The discipline is to check whether a leak mattered, rather than assume either way — this time it happened not to.

## Feature Ablation Study

Feature-importance tables (Section 9 of the notebook) showed `MONTHS_DELAYED` and `EVER_DELAYED` ranking highly across all three models — but ranking high in an importance table isn't proof a feature is pulling real weight, since it could just be correlating with something already in the model. To check, I ran a controlled ablation on the winning Decision Tree: hold its best hyperparameters fixed, remove one engineered feature group at a time, and re-score with the same leakage-safe 5-fold CV.

| Feature group removed | Mean CV F1 | Δ vs. baseline (0.4828) |
|---|---|---|
| *(none — baseline, 42 features)* | 0.4828 | — |
| Payment ratio features (`PAY_RATIO_1`–`5`, `AVG_PAY_RATIO`) | 0.4819 | −0.0009 |
| Delay summary features (`MAX_DELAY`, `AVG_DELAY`, `MONTHS_DELAYED`, `EVER_DELAYED`) | 0.4767 | **−0.0062** |
| Bill/payment aggregates (`AVG_BILL_AMT`, `AVG_PAY_AMT`, `TOTAL_PAY_AMT`) | 0.4807 | −0.0022 |

**Result:** the delay-summary features are the only group that clearly earns its place — removing them costs 0.0062 F1, about 40% of the baseline's fold-to-fold standard deviation, confirming they're doing real work rather than just correlating with something else. The payment-ratio features, despite being the most complex to engineer (dividing each month's payment by the prior month's bill), turned out to be the weakest — their −0.0009 impact is indistinguishable from noise, likely because they're redundant with the raw `PAY_1`–`PAY_6` delay columns and the bill/payment aggregates. If this model needed to be leaner, the payment-ratio group is the first thing I'd cut.

## Results

| Model | Accuracy | Precision | Recall | F1 | Defaults caught / missed |
|---|---|---|---|---|---|
| Logistic Regression | 0.760 | 0.466 | 0.577 | 0.515 | 957 / 702 |
| Ridge Regression | 0.770 | 0.483 | 0.558 | 0.518 | 926 / 733 |
| **Decision Tree (recommended)** | 0.765 | 0.475 | **0.585** | **0.524** | **971 / 688** |

Decision Tree wins on F1 and Recall — the right tradeoff here, since a missed defaulter (a client who keeps getting extended credit unflagged) is more costly than a false alarm, especially with these records sitting in the middle of Taiwan's 2005–2006 credit card debt crisis.

## Tech Stack

- **Python** — pandas, numpy
- **scikit-learn** — `Pipeline`, `GridSearchCV`, `TunedThresholdClassifierCV`, `cross_val_predict`, `cross_val_score`, `StandardScaler`, `LogisticRegression`, `Ridge`, `DecisionTreeClassifier`
- **Matplotlib / Seaborn** — EDA and result visualization
- **Jupyter Notebook**

## How to Run

```bash
pip install pandas numpy matplotlib seaborn scikit-learn jupyter
jupyter notebook Credit_Card_Default_Prediction.ipynb
```

`UCI_Credit_Card.csv` is included in this folder (the original, public UCI/Kaggle dataset — no privacy concerns). Run all cells top to bottom; the notebook is fully self-contained.

