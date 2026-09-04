# Fair Group Presentation Scheduler

A scheduling algorithm to help Professor Joe Fayt assign groups in my case-analysis course, MGMT 6300.

## Project Impact & Business Relevance

This tool is built to be reused, not just to solve one term's roster. Prof Joe teaches two sections of MGMT 6300(sometimes more) per term, and he faces the exact same problem in every single one: assign students to groups so the assignment is fair and every student gets to work with as many different classmates as possible. Instead of re-deriving the group assignments by hand each time — for a different class size, in a different section — he can just plug in the new student count and get a validated, conflict-free schedule back in seconds. That's real time and headache saved every term, for every class.

## The Ask

Teamwork is a key concept we learned in class, and the professor wants us to practice applying it in different group settings while maximizing each student’s opportunity to work with different classmates. This means: **no two students should ever be paired together twice.**

Background: the professor's class runs two rounds of presentations (worth 30% each and 60% of total final grades) across a handful of class days (three, currently — 28 students today, 30 expected next term, and the count will keep changing). Two constraints have to hold simultaneously:

1. **Zero-overlap fairness** — no student works with the same partner twice across their two presentations.
2. **No same-day conflicts** — no student can be scheduled to present twice on the same class day.

## Approach

The notebook computes a schedule for **any class size** — change one number (`N_STUDENTS`) and re-run. The number of groups per round and their sizes are derived from that count (targeting ~4-5 students per group), not hardcoded.

It splits into two pieces:
1. **Group construction** — a pure grid/transpose formula, no search. Round A groups are the rows of a grid; Round B groups are built by dealing students round-robin into columns. Any row and column share at most one cell, so the zero-overlap constraint holds automatically for any N — no combinatorial search needed for this part.
2. **Day assignment** — the one genuinely combinatorial step, but a small one (a dozen-ish groups, a few days): solved by a quick backtracking search so no student is double-booked on a class day.

At first I thought this was the classic hard "social golfer problem" that needs a SAT/CP solver. After some thgouths I realized since Prof Joe only has two rounds of graded presentations, the pairing constraint is structurally free, and the leftover day-scheduling problem is tiny enough for plain backtracking, so the notebook stays dependency-free (`pandas` + the standard library — no OR-Tools/SAT solver).

## Tech Stack

- Python 3
- pandas
- Jupyter Notebook

