# Apple Product Fair-Value Pricing — Time-Series Leakage Case Study

Predicting a fair resale/marketplace price for Apple products (2020–2026) with Ridge regression and KNN

## Project Impact & Business Relevance
Stepping into a product design mindset, I set out to remove pricing friction for casual and SMB sellers. I engineered a temporal fair-pricing engine that uses model-agnostic feature decompositions (generation, storage, launch-price anchoring) to evaluate existing devices and perform zero-shot pricing on newly released models without needing prior sales history.

## The Problem
Individual sellers listing used or refurbished Apple products on marketplaces like Amazon or Flipkart have to research a fair asking price themselves — slow, error-prone, and a real barrier to casual sellers.

## Approach

- **Data:** ~80,000 marketplace listings, Sept 2020 – mid 2026, across iPhone/iPad/Mac/Watch, scraped-style records with launch price, current price, condition, platform, stock status, ratings, and sale events.
- **Feature engineering:** `Model_Name` is deliberately *not* used directly as a feature. Instead it's decomposed into model-agnostic attributes — `variant_tier`, `storage_gb`, `generation` — parsed from the name string, plus `Launch_Price_USD` as the price anchor and `days_since_launch` as the depreciation clock. This lets the model score a brand-new, never-seen model at launch. Category × time and category × launch-price interaction terms let Ridge express that iPhones, iPads, Macs, and Watches depreciate at different rates.
- **EDA:** target profiling, cardinality analysis (does `Product_Category + Launch_Price_USD` recover most of what `Model_Name` explains? — yes, ~most of it, with none of the deployment problem), a per-category ANOVA test confirming price decreases significantly with `days_since_launch` in every category, and a sanity check on whether `Reviews_Count` is genuinely cumulative (it isn't — treated as noise).
- **Models:** Ridge regression (scaled numerics, one-hot categoricals, mandatory category interaction terms, alpha tuned by grid search) and KNN (scaled features, k searched over {3, 5, 10, 20, 40, 80} × {uniform, distance} weighting), both benchmarked against a simple per-`(Product_Category, Condition)` group-mean baseline.
- **Evaluation slices:** the full forward-in-time test window, plus a dedicated slice isolating iPhone 17 listings — a model that only exists inside the test period, i.e., a true "day-one, never-seen-in-training" case.

## The Leakage Bug — What I Caught and Fixed

The mechanism here isn't a feature secretly peeking at future prices — it's leakage baked into *how the data would naturally get split* if I'd used the standard tools without thinking about the fact that this is a time series with repeated observations.


**How I diagnosed it:** during the EDA pass I explicitly counted duplicate/near-duplicate keys before doing any modeling, specifically because I was suspicious that a time-ordered pricing panel like this would have this structure. Finding 21,389 exact-key duplicates (and knowing that near-duplicates a few days apart are far more common than that) was the tell that a random split or random K-Fold would silently leak.

**How I fixed it:**
1. **A hard chronological cutoff, not a random split.** Train = everything before 2025-08-01; test = everything on or after, with an explicit assertion (`train['Date'].max() < test['Date'].min()`) so the split can never silently regress to overlapping in time.
2. **`TimeSeriesSplit` (expanding-window CV) instead of K-Fold for hyperparameter tuning**, for both Ridge's alpha and KNN's k/weighting grid search — so *even within the training set*, no fold's "validation" rows are chronologically interleaved with rows the model trained on.
3. **A held-out slice built for the worst case**, not just the easy one: iPhone 17 only appears inside the test window, so scoring it in isolation is a clean test of "can this model price a model it has never once seen in training" — the actual production scenario, not an average-case number that a leaky setup could flatter.
4. Also caught and dropped as *direct* leakage before any of this: `Current_Price_INR` (the same target variable in another currency, correlation ≈ 1.0 with the target) and `Discount_Pct` (algebraically derived from the target: `(1 − Current/Launch) × 100`). Both would have handed the model the answer.

## Results

On the forward-in-time test slice, both Ridge and KNN clear the group-mean baseline, confirming the models are learning real depreciation dynamics rather than just reproducing average category prices. Ridge's coefficients (in standardized units) confirm the intended story: `Launch_Price_USD` and its category interactions dominate as the price anchor, `days_since_launch` interactions carry the expected negative sign (price falls with age) at different rates per category, and `Condition` behaves as a roughly stable multiplier as hypothesized in EDA.

The harder test — iPhone 17, a model absent from training entirely — is where the two models separate, and where KNN's dependence on having "nearby" training examples shows its limits versus Ridge's ability to extrapolate from the interaction structure.
I also conducted a residual diagnostic by `variant_tier` (deliberately excluded as a feature) to check whether the "premium tiers are priced in via `Launch_Price_USD`" assumption actually holds. 

## Tech Stack

Python, pandas, NumPy, scikit-learn (`Pipeline`, `ColumnTransformer`, `Ridge`, `KNeighborsRegressor`, `GridSearchCV`, `TimeSeriesSplit`), scipy.stats, matplotlib, seaborn, Jupyter.

## How to Run

```bash
pip install pandas numpy scikit-learn scipy matplotlib seaborn jupyter
jupyter notebook Apple_Pricing_EDA.ipynb          # run first — writes apple_pricing_train.csv / apple_pricing_test.csv
jupyter notebook "Ridge&KNN_Pricing_Modeling.ipynb"  # loads the raw CSV directly and re-derives the same split/features
```

The EDA notebook must be run first if you want the saved `apple_pricing_train.csv` / `apple_pricing_test.csv` baseline artifacts — the modeling notebook is self-contained and re-derives its features and split from the raw CSV, so it can also be run on its own.
