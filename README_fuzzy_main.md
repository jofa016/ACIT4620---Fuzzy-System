# Fuzzy Main Notebook Overview

This README summarizes the logic, workflow, and key components implemented in `fuzzy_main.ipynb`, which builds a fuzzy logic layer over household energy usage and electricity spot prices.

## 1. Purpose

Model household lighting/activity patterns and electricity prices using fuzzy sets, derive interpretable rule-based outputs (energy waste, habit cost, efficiency), and convert them to crisp scores for analysis and visualization.

## 2. Data Inputs

Primary inputs originate from cleaned minute appliance data and spot price series. Core columns used:

- `kitchen_lights`, `lounge_lights`: aggregated light power/activity proxies.
- `kitchen_activity`, `lounge_activity`: appliance-derived activity indicators.
- `spot_price`: electricity price (normalized/scaled).
- Time attributes: `hour`, `day`, `month`.

## 3. Fuzzification

Triangular membership functions convert numeric columns to linguistic labels (overlapping for smooth transitions). Examples:

- Lights: off / low / medium / high.
- Activity: low / medium / high (plus derived none via NOT of any activity labels).
- Time-of-day: night / morning / afternoon / evening.
- Spot price: low / medium / high.

Implementation pattern:

1. Define (a, b, c) triples per label.
2. Generate a synthetic `x` range (linspace) only for plotting membership curves.
3. Apply a vectorized `triangular_membership(x, a, b, c)` across actual column values row-wise to create new membership degree columns.

## 4. Mamdani Rule Layer

Rules combine fuzzified inputs using fuzzy operators:

- AND: `min`
- OR: `max`
- NOT: `1 - μ`

Derived outputs (each represented by multiple membership columns):

- Energy waste (room-level + household aggregated).
- Kitchen light efficiency (very_poor / fair / good / very_good).
- Habit cost (price × activity interplay).
- Overall efficiency (combines waste and habit cost with contextual time).

To highlight inefficiency such as high lights during inactivity or night, and contextual price pressure.

## 5. Defuzzification Methods

Two crisp conversion strategies:

1. Weighted Average: Map each label to a numeric weight (e.g., low=0.0, medium=0.5, high=1.0) and compute `(Σ μ_i * w_i) / (Σ μ_i)` per timestamp.
2. Centroid (Mamdani):
   - Define ideal output label shapes over universe U=[0,1].
   - Clip each shape by its membership degree at the timestamp.
   - Aggregate (max) clipped shapes into a composite fuzzy set.
   - Compute centroid `∫ x * μ(x) dx / ∫ μ(x) dx`.

Outputs: `*_score` (weighted) and `*_centroid` (centroid) numeric columns.

## 6. Label Selection & Priority

For each concept, the dominant fuzzy label is chosen with `argmax` across its membership columns. A composite priority metric (`waste_cost_priority = energy_waste_score * habit_cost_score`) highlights hours combining high waste and high cost pressure.

## 7. Visualization

Key plot types:

- Membership function curves (line plots) for each variable.
- Combined grid: histograms of real data with overlaid fuzzy membership lines.
- Hourly average curves: group by `hour` and plot mean membership degrees per group (e.g., kitchen activity, lounge lights, time-of-day, price).
- Defuzzification comparison plots: base label triangles, shaded clipped activations, and vertical lines for weighted vs centroid crisp values.

Each defuzzification plot allows visual inspection of how fuzzy activations translate to final scores.

## 8. Usage Flow (Notebook Outline)

1. Load and aggregate raw data.
2. Define membership limits and fuzzify numeric columns.
3. Plot membership sets (optional interpretability step).
4. Compute Mamdani rule outputs with `compute_mamdani_rules`.
5. Derive weighted-average scores and dominant labels.
6. Build centroid scores for each concept.
7. Summarize per hour (means) and compute composite priority.
8. Select a representative row and visualize defuzzification.

## 9. Dependencies

Core libraries:

- `pandas`, `numpy`, `matplotlib`
  Optional:
- `scikit-fuzzy` (for alternative rule formalism)

Install example (conda environment):

```
conda install pandas numpy matplotlib
conda install -c conda-forge scikit-fuzzy  # optional
```

## 10. Quick Start Summary

Open `fuzzy_main.ipynb`, run cells top to bottom:

1. Data load & aggregation.
2. Membership definition & fuzzification.
3. Rule computation.
4. Weighted and centroid defuzzification.
5. Visualization & summary inspection.
