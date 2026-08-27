# International Student Mentor-Mentee Matching Algorithm

A weighted-similarity matching system built for Schulich's International Student Services mentorship program, pairing incoming first-year international students with upper-year mentors on shared goals, background, and identity.

## The Problem

Every year, Schulich's International Student Services department runs a mentorship program that pairs incoming first-year international students with upper-year student mentors. Historically, this pairing was done manually — a time-consuming process with no consistent, defensible logic behind who got matched with whom, and no good way to weigh multiple factors (what a mentee actually needs help with, shared cultural background, gender preference) against each other at scale.

With roughly 20-30 mentors and a comparable number of mentees each intake, the matching problem is small enough to solve by hand, but large enough that "by hand" means intuition-driven guesswork rather than a repeatable process. As the program grows, that doesn't scale — and more importantly, it doesn't reliably produce good matches.

## Approach

The matching is framed as a **weighted bipartite similarity-scoring problem, solved with greedy matching**:

1. **Semantic similarity on stated goals (40% weight).** Each mentee's stated objectives ("Adjusting to life in Canada," "Making friends," "Resume/interview prep," etc.) and each mentor's stated motivation for mentoring are encoded into dense vector embeddings using a pre-trained sentence-transformer model (`all-MiniLM-L6-v2`). Cosine similarity between every mentee-mentor pair produces a semantic "goals alignment" score.
2. **Shared cultural background (30% weight).** A binary match score (1 if the mentee's and mentor's home country match, 0 otherwise), reflecting that shared cultural/national background is often a meaningful source of comfort and common ground for a student adjusting to a new country.
3. **Gender alignment (30% weight).** Gender is inferred programmatically from first names (via the `gender-guesser` library) where not explicitly provided, then scored as a match (1.0), mismatch (0.0), or neutral (0.5) when inference is inconclusive — reflecting that some mentees may have a gender preference for their mentor and shouldn't be penalized when that data is missing.
4. **Combine into one score matrix.** The three signals are combined into a single weighted similarity matrix (mentees × mentors), scored on a 0-1 scale.
5. **Greedy one-to-one matching.** Starting from the single highest-scoring pair in the entire matrix, the algorithm repeatedly selects the best remaining mentee-mentor pair, removes both from the pool, and repeats — until every mentee has one mentor. 

## Tech Stack

- **Python / pandas** — data loading, wrangling, and matrix construction
- **sentence-transformers** (`all-MiniLM-L6-v2`) — semantic embeddings of free-text goals and motivations
- **scikit-learn** — cosine similarity computation
- **NumPy** — score matrix construction and weighting
- **gender-guesser** — name-based gender inference for records with missing data
- **Jupyter Notebook** — end-to-end analysis, documented step by step


**Note on data:** this repo ships with **fully synthetic replacement data** (`Mentees.csv`, `Mentors.csv`) — fictional names, emails, and IDs generated

