# Fair Group Presentation Scheduler

A scheduling algorithm to help Professor Joe Fayt assign groups in my case-analysis course, SB MGMT 6300.

## The Ask

Teamwork is a key concept we learned in class, and the professor wants us to practice applying it in different group settings while maximizing each student’s opportunity to work with different classmates. This means: **no two students should ever be paired together twice.**

Background: We have 28 students, two rounds of presentations, and three class days. Each round needs six groups (four groups of 5, two groups of 4) — 12 group presentations in total, split evenly across the three class days. Two constraints have to hold simultaneously:

1. **Zero-overlap fairness** — no student works with the same partner twice across their two presentations.
2. **No same-day conflicts** — no student can be scheduled to present twice on the same class day.

## Approach

I need a **deterministic constructive algorithm**: the 28-student roster is partitioned into Round A groups by direct index slicing, and each Round B group is then built by pulling one student from a fixed index position out of each relevant Round A group. 

## Tech Stack

- Python 3
- pandas
- Jupyter Notebook

