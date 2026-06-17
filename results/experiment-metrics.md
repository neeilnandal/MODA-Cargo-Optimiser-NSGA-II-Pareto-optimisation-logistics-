# Experiment Summary

## Objective

The MODA Cargo Ship Optimization project explores how multi-objective optimization can support better cargo shipping decisions.

The goal is to identify route and loading strategies that balance competing objectives:

- Minimize total operating cost
- Minimize fuel consumption
- Minimize delivery time
- Maximize cargo utilization
- Reduce operational risk

This reflects a realistic logistics problem where the cheapest solution is not always the best solution.

## Method

The experiment uses a multi-objective optimization approach inspired by Pareto analysis.

Instead of optimizing for a single target, the model evaluates several candidate solutions and identifies trade-offs between cost, time, fuel consumption, emissions, and utilization.

A solution is considered Pareto-efficient if no other solution improves one objective without worsening at least one other objective.

## Output File

The main result file is:

```text
results/pareto_solutions.csv
