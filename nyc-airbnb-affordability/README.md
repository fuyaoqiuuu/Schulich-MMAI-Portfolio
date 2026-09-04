# NYC Airbnb Shadow Inventory: A Housing Affordability Analysis

What began as a ordinary EDA unraveled intriguing: a hidden regulatory loophole and unintentional consequences of a local law in NYC.

## Project Impact & Business Relevance

**Key Finding: NYC’s Local Law 18 designed a fix for the long-term housing market, but the evidence reveals a different reality.**
Instead of converting to standard residential leases, commercial operators simply shifted minimum stays to 30+ days to bypass the regulation. Today, 81.9% of active listings cluster just above that threshold, running at a meager 14.2% annual occupancy. Over seven months of data (holding flat at ~12% occupancy) confirm this is a durable, structural loophole: operators would rather tolerate low medium-term occupancy than return units to long-term renters.

The core value of this post-mortem analysis is replacing policy assumptions with empirical evidence. Rather than declaring victory because short-term listings vanished from the books, the data highlights a persistent blind spot—proving policymakers need to re-examine the law rather than consider the housing problem solved.

## The Problem

If commercial operators are gaming the 30-day threshold, it should leave a specific signature in the data: units that are legally exempt from short-term rental restrictions but aren't behaving like long-term rentals either — off the 1-year-lease market, but not booked often enough to be real tourism inventory. This analysis finds and sizes that gray zone using April 2026 Inside Airbnb listings data for New York City.

## Feature Engineering — Utilization Rate as an Affordability Proxy

Airbnb doesn't publish occupancy, so I engineered a proxy for it, adapting Inside Airbnb's standard estimation approach:

```
occupancy_rate = (number_of_reviews_ltm × 2) × MAX(3, minimum_nights_avg_ntm) / 365
```

`number_of_reviews_ltm` (reviews in the last 12 months) is doubled under the standard assumption that ~50% of stays generate a review, giving an estimated booking count. Multiplying by minimum stay length (floored at 3 nights, per Inside Airbnb's methodology) converts that into booking-nights; dividing by 365 gives the share of the year the unit was actually occupied.

A unit's minimum-stay setting tells you what a host is *legally allowed to claim* it's being used for. The occupancy rate tells you what it's *actually* being used for. Crossing the two exposes the gap: I bucketed listings into three length-of-stay segments (0–29, 30–45, 46+ days — 30–45 being the "policy cushion" zone just above Local Law 18's threshold) and three host archetypes (Casual Home-Sharer, Professional Single-Home Host, Commercial Investor).

## Approach

1. **Feature engineering** — built `occupancy_rate` on the full active-listing dataset (n = 35,036 after filtering).
2. **Segmentation** — binned listings by length-of-stay via `minimum_nights_avg_ntm`, and classified hosts into `landlord_type` using room type, portfolio size, and annual availability.
3. **Aggregation** — grouped by segment × landlord type to compare occupancy and volume (Seaborn + Plotly bubble charts).
4. **Deep dive** — isolated the 30–45 day × Commercial Investor intersection as `shadow_residential_inventory`, aggregated by NYC borough.
5. **Longitudinal validation** — reran the identical pipeline on the earliest available scrape (Sept 1, 2025) and compared Commercial Investor occupancy against the April 2026 snapshot, both overall and by segment, to test whether the pattern is stable over time or a one-off artifact of a single scrape.

## Findings

- **81.9%** of active listings (28,679 of 35,036) sit in the 30–45 day segment — just above Local Law 18's threshold.
- Short-term listings (0–29 days) run **~47–50%** occupancy across every host type — genuinely active tourism inventory.
- 30–45 day **Commercial Investor** listings — 11,457 units, 33% of the market — average only **14.2%** occupancy, versus 47.6% for short-term listings from the same investor class.
- **Geographic concentration**: Manhattan holds the largest share (6,088 units, 53%) at the lowest occupancy (12.9%). Brooklyn's 3,112 units perform somewhat better (18.0%). Queens, the Bronx, and Staten Island idle at 11.7–13.7%.
- **The pattern is persistent, not transient**: comparing the earliest available scrape (Sept 1, 2025) to the April 2026 snapshot, overall Commercial Investor occupancy moved only marginally (20.4% → 21.7%, +1.2pp), and the 30–45 day loophole segment specifically was nearly flat (12.1% → 12.4%, +0.3pp) even as its listing volume shrank 5.8% (13,642 → 12,847 units). By contrast, genuine short-term inventory (0–29 days) improved more (48.3% → 49.9%), consistent with normal seasonal demand. Across the full ~7.5-month window, the medium-term shadow inventory did not resolve itself — it stayed stuck.


## Data Source

Inside Airbnb (insideairbnb.com) NYC listings, primarily the April 14, 2026 scrape (`2026-04-14_listings.csv`), cross-validated against the earliest available scrape, September 1, 2025 (`2025-09-01_listings.csv`). **The raw CSVs are not included** — they exceed GitHub's 100MB limit. Download them from [insideairbnb.com/get-the-data](http://insideairbnb.com/get-the-data.html) and place both in this directory before running the notebook.

## Tech Stack

Python (pandas, numpy) · Seaborn / Matplotlib · Plotly Express · Jupyter Notebook

## How to Run

1. Download the April 14, 2026 and September 1, 2025 NYC listings snapshots and save them as `2026-04-14_listings.csv` and `2025-09-01_listings.csv` in this directory.
2. `pip install pandas numpy matplotlib seaborn plotly nbformat`
3. Open and run `Housing_affordability_vs_tourism_economy.ipynb` top to bottom.

See `Executive Summary_ A study of an Airbnb Shadow Inventory.pdf` for the condensed write-up.
