# AI & Machine Learning Portfolio — Fuyao Qiu

A collection of applied AI/ML and analytics projects from my MMAI (Master of Management in Artificial Intelligence) coursework and volunteer work, built to show how I actually work through a problem: state a hypothesis, apply the right machine learning technique, catch and fix my own mistakes, and verify claims rather than assume them.

## Projects

### [`credit-card-default-ablation/`](./credit-card-default-ablation/) — Predicting Credit Card Defaults
Supervised classification (Logistic Regression, Ridge, and a Decision Tree) predicting credit default, with the last few iterations **catching and fixing a real cross-validation leakage bug** (scaling the whole training set before CV instead of inside each fold). A **controlled ablation study** then showed the delay-summary features I engineered (`MAX_DELAY`, `AVG_DELAY`, `MONTHS_DELAYED`, `EVER_DELAYED`) were the only feature group that consistently drove predictive power — a direct, actionable finding that lets an engineering team build and maintain fewer features in production without sacrificing accuracy.

### [`apple-pricing-time-series/`](./apple-pricing-time-series/) — Apple Product Fair-Value Pricing
Approaching this from the perspective of an e-commerce platform, I explored how Machine Learning directly solves user friction. I built a temporal fair-pricing engine using model-agnostic attributes—like generation, storage, and launch-price anchoring—to help casual and SMB sellers list quickly without manual research, while allowing the platform to zero-shot price brand-new hardware on Day 1.

### [`nyc-airbnb-affordability/`](./nyc-airbnb-affordability/) — NYC Airbnb Shadow Inventory
Quantifying how commercial short-term-rental operators exploit NYC's Local Law 18 by shifting minimum stays just above its 30-day threshold instead of returning units to the long-term housing market. Engineered an `occupancy_rate` proxy (from review counts and minimum-stay length) to show that 81.9% of active listings sit in that "policy cushion" zone at only 14.2% annual occupancy — a pattern that held flat across seven months of data, challenging the policy assumption of "problem solved" with empirical evidence that the loophole is durable and structural.

### [`mentor-mentee-matching/`](./mentor-mentee-matching/) — International Student Mentor-Mentee Matching
An **NLP-powered** weighted-similarity + greedy matching algorithm for Schulich's International Student Services mentorship program. Uses a pre-trained sentence-transformer model to generate **semantic embeddings** of each mentee's and mentor's stated goals, combining that similarity score (40%) with shared cultural background (30%) and domestic/international preference alignment (30%) — designed deliberately for a transparent, explainable match over an opaque optimization score. Replaces hours of manual spreadsheet sorting with an automated match of the entire cohort in seconds, while maximizing mutual alignment across the whole pool. Ships with fully synthetic data (no real student information) so it runs end-to-end out of the box.

### [`social-golfer-problem/`](./social-golfer-problem/) — Fair Group Presentation Scheduler
A small constructive algorithm solving a real scheduling problem from a case-analysis course: assign students into two rounds of presentation groups so no two students are ever paired together twice. Built to be reused every term, not just for one roster — the professor can plug in a new class size for any section and get a validated, conflict-free schedule back in seconds instead of re-deriving it by hand.

## A note on data privacy

Every project that originally touched real personal or confidential data has been handled with fully synthetic replacement data rather than anonymized real records. The credit card default and Airbnb projects use public datasets (UCI/Kaggle and Inside Airbnb respectively) with no privacy concerns.
