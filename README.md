# Fuzzy Demand Classification & Scheduling

## **Problem statement:** How can household energy consumption be classified into low, medium, and high demand levels to enable cost optimization?

> Goal: Classify hourly/instantaneous demand as Low/Medium/High in a way that reflects baseline usage patterns and dynamic electricity pricing, so a fuzzy-logic scheduler can shift deferrable loads and reduce peak demand and cost.

---

## Project overview (what this repo does)

- Load minute-level house data and compute hour-of-day baselines using percentiles (P20, P50, P80).
- Build a fuzzy demand classifier with inputs: relative load (vs hourly baseline), load trend, and price level.
- Infer a demand index (0–100), then map to categories: Low (<35), Medium (35–70), High (>70).
- Simulate a simple scheduling policy for deferrable loads (e.g., washer/dishwasher) from peak-price/high-demand hours to off-peak/loWe use fuzzy sets for: relative load vs baseline {low, medium, high}, ,load trend {falling, stable, rising}, price level {low, medium, high}, output demand {low, medium, high}. For absolute categories(e.g., kitchen, living room, bedroom), we also used {low, medium, high}.

---

## Core fuzzy concepts — mapped to this project

| Step | Concept                        | What it means                                       | This project (how it’s used)                                                                                                                                                                                                                                                                                 |
| ---- | ------------------------------ | --------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1    | **Fuzzy sets**                 | Vague categories with soft boundaries               | We use sets for: relative load vs baseline {low, medium, high}, load trend {falling, stable, rising}, price level {low, medium, high}, and output demand {low, medium, high}. (Optional appliance labeling uses {off, low, medium, high} on absolute power.)                                                 |
| 2    | **Membership functions (MFs)** | Map a number → degree in [0–1] for each set         | Each set is shaped by triangles/trapezoids. We use: rel_load (−2000…2000 W) low=trapmf[−2000,−2000,−200,−20], med=trimf[−200,0,200], high=trapmf[20,200,2000,2000]; trend (−600…600 W) falling/stable/rising with width w learned from data; price {1,2,3} low/med/high; output demand (0–100) low/med/high. |
| 3    | **Linguistic variables**       | Human-friendly variables that use fuzzy sets        | Inputs: rel_load, load_trend, price_level. Output: demand (index 0–100) and bucket {Low, Medium, High}.                                                                                                                                                                                                      |
| 4    | **Fuzzification**              | Convert crisp inputs (Watts) → fuzzy memberships    | For each timestamp: rel_load = aggregate − P50(hour); trend = rolling_5min − rolling_30min; price_level ∈ {1,2,3}. Evaluate their MFs to get degrees (e.g., μ_rel_high=0.7).                                                                                                                                 |
| 5    | **Inference (reasoning)**      | Apply fuzzy if-Then rules to input and fuzzy output | Rules combine inputs, e.g., (price high ∧ rel_load high) → demand high; (price low ∧ rel_load low) → demand low; (rel_load med ∧ trend stable) → demand med.                                                                                                                                                 |
| 6    | **Defuzzification**            | Convert fuzzy outputs → a crisp value               | Aggregate output sets and compute the centroid → demand_index ∈ [0,100], then bucket: Low <35, Medium 35–70, High >70.                                                                                                                                                                                       |

### Short notes — What “membership” means here

- A membership value (between 0 and 1) expresses how strongly a measurement belongs to a fuzzy set. Example: rel_load = +150 W might give μ_med=0.4 and μ_high=0.6 simultaneously.

- These memberships feed the rules, which determine how much of the output sets (demand low/med/high) are activated before converting to the final demand index used for scheduling.

- The demand index is a single score from 0 to 100 that summarizes "how demandy" this moment.
