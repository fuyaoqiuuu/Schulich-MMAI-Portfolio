# NYC Airbnb Shadow Inventory: A Housing Affordability Analysis

**Key finding: New York City's Local Law 18 does not appear to be an effective way to boost long-term housing supply.** The law defines a short-term rental as any stay under 30 days and imposes strict registration on hosts below that threshold — but this analysis shows commercial operators simply moved their minimum-stay setting to 30+ days to exit the law's jurisdiction, without converting those units into real long-term housing. 81.9% of active listings now sit just above the 30-day line, and the commercial-investor segment of that group runs at only 14.2% annual occupancy. The law moved listings out of the short-term-rental bucket on paper without returning that housing stock to residents.

## The Problem

If commercial operators are gaming the 30-day threshold, it should leave a specific signature in the data: units that are legally exempt from short-term rental restrictions but aren't behaving like long-term rentals either — off the 1-year-lease market, but not booked often enough to be real tourism inventory. This analysis finds and sizes that gray zone using April 2026 Inside Airbnb listings data for New York City.

## The Key Insight — Utilization Rate as an Affordability Proxy

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

## Findings

- **81.9%** of active listings (28,679 of 35,036) sit in the 30–45 day segment — just above Local Law 18's threshold.
- Short-term listings (0–29 days) run **~47–50%** occupancy across every host type — genuinely active tourism inventory.
- 30–45 day **Commercial Investor** listings — 11,457 units, 33% of the market — average only **14.2%** occupancy, versus 47.6% for short-term listings from the same investor class.
- **Geographic concentration**: Manhattan holds the largest share (6,088 units, 53%) at the lowest occupancy (12.9%). Brooklyn's 3,112 units perform somewhat better (18.0%). Queens, the Bronx, and Staten Island idle at 11.7–13.7%.

## Data Source

Inside Airbnb (insideairbnb.com) NYC listings, scraped April 14, 2026 (`2026-04-14_listings.csv`). **The raw CSV is not included** — it exceeds GitHub's 100MB limit. Download it from [insideairbnb.com/get-the-data](http://insideairbnb.com/get-the-data.html) and place it in this directory before running the notebook.

## Tech Stack

Python (pandas, numpy) · Seaborn / Matplotlib · Plotly Express · Jupyter Notebook

## How to Run

1. Download the April 14, 2026 NYC listings snapshot and save it as `2026-04-14_listings.csv` in this directory.
2. `pip install pandas numpy matplotlib seaborn plotly nbformat`
3. Open and run `Housing_affordability_vs_tourism_economy.ipynb` top to bottom.

See `Executive Summary_ A study of an Airbnb Shadow Inventory.pdf` for the condensed write-up.
