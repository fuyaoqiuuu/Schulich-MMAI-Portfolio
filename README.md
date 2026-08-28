# AI & Machine Learning Portfolio — Fuyao Qiu

A collection of applied AI/ML and analytics projects from my MMAI (Master of Management in Artificial Intelligence) coursework and volunteer work, built to show how I actually work through a problem: state a hypothesis, apply the right machine learning technique, catch and fix my own mistakes, and verify claims rather than assume them.

## Projects

### [`credit-card-default-ablation/`](./credit-card-default-ablation/) — Predicting Credit Card Defaults
Supervised classification (Logistic Regression, Ridge, and a Decision Tree) predicting credit default, with the last few iterations **catching and fixing a real cross-validation leakage bug** (scaling the whole training set before CV instead of inside each fold), then a **controlled ablation study** to check which engineered features actually improve the model versus just looking important.

### [`apple-pricing-time-series/`](./apple-pricing-time-series/) — Apple Product Fair-Value Pricing
Ridge and KNN regression models suggesting a fair marketplace resale price for Apple products, built around diagnosing and fixing a **time-series leakage trap**: near-duplicate listings of the same product a few days apart would let a random split or K-Fold CV silently "cheat" by matching a listing to its own near-twin. Fixed with a chronological split and `TimeSeriesSplit`.

### [`nyc-airbnb-affordability/`](./nyc-airbnb-affordability/) — NYC Airbnb Shadow Inventory
Quantifying how commercial short-term-rental operators exploit a 30-day regulatory threshold to warehouse housing units that are neither real long-term housing nor real tourism inventory. What I'm proud of: engineering an `occupancy_rate` proxy (from review counts and minimum-stay length) to expose the gap between what a listing's minimum-stay setting *claims* and how the unit is *actually* used.

### [`mentor-mentee-matching/`](./mentor-mentee-matching/) — International Student Mentor-Mentee Matching
An **NLP-powered** weighted-similarity + greedy matching algorithm for Schulich's International Student Services mentorship program, built for the International Student Services department. Uses a pre-trained sentence-transformer model to generate **semantic embeddings** of each mentee's and mentor's stated goals, combining that similarity score (40%) with shared cultural background (30%) and gender alignment (30%) — designed deliberately for a transparent, explainable match over an opaque optimization score. Ships with fully synthetic data (no real student information) so it runs end-to-end out of the box.

### [`social-golfer-problem/`](./social-golfer-problem/) — Fair Group Presentation Scheduler
A small constructive algorithm solving a real scheduling problem from a case-analysis course: assign 28 students into two rounds of presentation groups so no two students are ever paired together twice, with a validator that programmatically proves the fairness constraint holds rather than assuming it.

## A note on data privacy

Every project that originally touched real personal or confidential data has been handled with fully synthetic replacement data rather than anonymized real records. The credit card default and Airbnb projects use public datasets (UCI/Kaggle and Inside Airbnb respectively) with no privacy concerns.
