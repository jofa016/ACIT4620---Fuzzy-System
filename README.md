# Fuzzy Demand Classification & Scheduling
## **Problem statement:** How can household energy consumption be classified into low, medium, and high demand levels to enable cost optimization?

> **Goal:** How can household energy consumption patterns be classified into Low, Medium, and High demand levels in a way that reflects both temporal usage variations and dynamic electricity pricing, so that fuzzy logic–based scheduling can enable cost optimization and reduce peak demand?



| Step | Concept                        | What it means                                         | This project  (now example text)                                 |
| ---- | ------------------------------ | ----------------------------------------------------- |------------------------------------------------------------------|
| 1    | **Fuzzy sets**                 | Define vague categories                               | “Low power”, “Medium power”, “High power”                        |
| 2    | **Membership functions (MFs)** | Translate numbers into degrees of belonging (0–1)     | Power = 1200 W → µ_high = 0.8                                    |
| 3    | **Linguistic variables**       | Human-friendly variables that use fuzzy sets          | “Power usage” = {low, medium, high}                              |
| 4    | **Fuzzification**              | Convert crisp inputs (Watts) → fuzzy memberships      | You already did this: `fuzzy_data`                               |
| 5    | **Inference (reasoning)**      | Apply fuzzy **rules** to derive new fuzzy conclusions | “If kitchen is high AND lights are medium → activity is cooking” |
| 6    | **Defuzzification**            | Convert fuzzy outputs → crisp values                  | “Activity level = 73/100 (high)”                                 |
