# Predicting Credit Card Defaults — Fixing Cross-Validation Leakage Bugs + Feature Ablation

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
5. **Model comparison** — Decision Tree wins on F1 (0.525) and Recall (catches 971 of 1,659 test-set defaulters, the most of the three), so it's the recommended model: missing a real defaulter is more costly here than a false alarm.


## Feature Ablation Study

Feature-importance tables (Section 9 of the notebook) showed `MONTHS_DELAYED` and `EVER_DELAYED` ranking highly across all three models — but ranking high in an importance table isn't proof a feature is pulling real weight, since it could just be correlating with something already in the model. To check, I ran a controlled ablation on the winning Decision Tree: hold its best hyperparameters fixed, remove one engineered feature group at a time, and re-score with the same leakage-safe 5-fold CV.

| Feature group removed | Mean CV F1 | Δ vs. baseline (0.4828) |
|---|---|---|
| *(none — baseline, 36 features)* | 0.4828 | — |
| Payment ratio features (`PAY_RATIO_1`–`5`, `AVG_PAY_RATIO`) | 0.4817 | −0.0010 |
| Delay summary features (`MAX_DELAY`, `AVG_DELAY`, `MONTHS_DELAYED`, `EVER_DELAYED`) | 0.4767 | **−0.0060** |
| Bill/payment aggregates (`AVG_BILL_AMT`, `AVG_PAY_AMT`, `TOTAL_PAY_AMT`) | 0.4806 | −0.0022 |

**Result:** the delay-summary features are the only group that clearly earns its place — removing them costs 0.0060 F1, about 38% of the baseline's fold-to-fold standard deviation, confirming they're doing real work rather than just correlating with something else. The payment-ratio features, despite being the most complex to engineer (dividing each month's payment by the prior month's bill), turned out to be the weakest — their −0.0010 impact is indistinguishable from noise, likely because they're redundant with the raw `PAY_1`–`PAY_6` delay columns and the bill/payment aggregates. If this model needed to be leaner, the payment-ratio group is the first thing I'd cut.

## Model Performance

| Model | Accuracy | Precision | Recall | F1 | Defaults caught / missed |
|---|---|---|---|---|---|
| Logistic Regression | 0.760 | 0.466 | 0.577 | 0.515 | 957 / 702 |
| Ridge Regression | 0.770 | 0.483 | 0.558 | 0.518 | 926 / 733 |
| **Decision Tree (recommended)** | 0.765 | 0.475 | **0.585** | **0.525** | **971 / 688** |

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

