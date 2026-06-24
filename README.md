# MODA Cargo Optimiser: NSGA-II Pareto Optimisation for Container Loading

A constrained multi-objective optimisation prototype for container placement on a cargo vessel.

The project uses **NSGA-II** to generate feasible loading layouts that balance unloading practicality, centre-of-gravity deviation, slot utilisation, and stacking feasibility. Instead of hiding the decision in one weighted score, it presents a Pareto front of defensible operational trade-offs.

---

## Project Snapshot

| Area                 | Details                                                                                  |
| -------------------- | ---------------------------------------------------------------------------------------- |
| **Status**           | Complete optimisation prototype                                                          |
| **Domain**           | Logistics, vessel loading, operations research                                           |
| **Algorithm**        | NSGA-II                                                                                  |
| **Library**          | `pymoo`                                                                                  |
| **Language**         | Python                                                                                   |
| **Data**             | Synthetic container weights and unloading priorities                                     |
| **Primary artefact** | [`container_ship_nsga_v2.ipynb`](container_ship_nsga_v2.ipynb)                           |
| **Outputs**          | Pareto solution CSV, optimisation summary, raw output, and 3D layout visualisations      |
| **Scope**            | Decision-support prototype; not an industrial vessel-stability or port-scheduling solver |

---

## The Decision Problem

Container loading is not merely a packing exercise.

A workable loading plan must balance competing requirements:

* reduce unloading conflicts;
* keep vessel weight distribution near the centre;
* use available slots efficiently;
* prevent duplicate slot assignments;
* prevent unsupported stacks;
* avoid unsafe heavy-over-light configurations;
* produce layouts that remain interpretable for a planner.

These objectives conflict. A solution that reduces unloading conflicts may be less balanced. A solution that improves balance may make unloading less convenient. That is why this problem is modelled as multi-objective optimisation rather than a single-score ranking exercise.

---

## Optimisation Formulation

Each candidate solution assigns every container to a vessel slot:

```text
x[i] = slot assigned to container i
```

The optimiser evaluates three objectives:

| Objective                   | Direction | Interpretation                                       |
| --------------------------- | --------- | ---------------------------------------------------- |
| Unloading violations        | Minimise  | Reduce operationally awkward unloading order         |
| Centre-of-gravity deviation | Minimise  | Keep vessel weight distribution closer to the centre |
| Used slots                  | Maximise  | Improve vessel-capacity utilisation                  |

`pymoo` performs minimisation by default, so utilisation is represented internally as:

```text
-used_slots
```

A lower internal value therefore corresponds to higher slot utilisation.

---

## Vessel Model

The default vessel is represented as a 3D grid:

```text
8 bays × 4 rows × 3 tiers = 96 available positions
```

Each slot is represented as:

```text
(bay, row, tier)
```

Each synthetic container has:

* a unique identifier;
* a weight;
* an unloading-priority group.

---

## Constraints

| Constraint                | Purpose                                                        |
| ------------------------- | -------------------------------------------------------------- |
| Duplicate slot assignment | Prevent multiple containers from occupying the same slot       |
| Unsupported stack         | Prevent containers from being placed above an empty lower tier |
| Unsafe weight stack       | Penalise unsafe heavy-over-light stack configurations          |

In `pymoo`, a solution is feasible when all constraint values satisfy:

```text
G <= 0
```

For the violation-count constraints used here, `0` means no violation and a positive value indicates one or more violations.

---

## Repository Structure

```text
MODA-Cargo-Optimiser-NSGA-II-Pareto-optimisation-logistics-/
├── README.md
├── requirements.txt
├── LICENSE
├── .gitignore
├── MODA25A2_42.pdf
├── container_ship_nsga_v2.ipynb
│
├── assets/
│   ├── pareto-front.png
│   ├── selected-layout-3d.png
│   ├── optimisation-workflow.png
│   └── objective-comparison.png
│
└── results/
    ├── pareto_solutions.csv
    ├── experiment-summary.md
    ├── results-discussion.md
    └── raw_nsga2_output.txt
```

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/neeilnandal/MODA-Cargo-Optimiser-NSGA-II-Pareto-optimisation-logistics-.git
cd MODA-Cargo-Optimiser-NSGA-II-Pareto-optimisation-logistics-
```

### 2. Create and activate a virtual environment

**macOS / Linux**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

**Windows PowerShell**

```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
```

**Windows Command Prompt**

```bat
python -m venv .venv
.\.venv\Scripts\activate.bat
```

### 3. Install dependencies

```bash
python -m pip install --upgrade pip
pip install -r requirements.txt
```

---

## Run the Optimiser

Launch Jupyter:

```bash
jupyter notebook
```

Then open and run:

```text
container_ship_nsga_v2.ipynb
```

The notebook performs this workflow:

```text
Generate synthetic containers
        ↓
Create vessel slot grid
        ↓
Define objectives and constraints
        ↓
Run NSGA-II
        ↓
Extract feasible Pareto solutions
        ↓
Select representative layouts
        ↓
Export results and visualisations
```

---

## Results

The optimiser generated a Pareto front of feasible candidate loading layouts.

The decision-ready outputs are available in:

```text
results/pareto_solutions.csv
results/experiment-summary.md
results/results-discussion.md
```

### Example Pareto Solutions

| Solution | Unloading violations | CoG deviation | Used slots | Interpretation                                                 |
| -------- | -------------------: | ------------: | ---------: | -------------------------------------------------------------- |
| S001     |                   29 |      0.646651 |         96 | Balanced solution with strong slot use                         |
| S002     |                   15 |      0.753381 |         96 | Best unloading-order option in the saved set                   |
| S003     |                   30 |      0.639909 |         96 | Best balance-focused option in the saved set                   |
| S004     |                   19 |      0.700328 |         96 | Middle-ground compromise                                       |
| S005     |                   17 |      0.715040 |         96 | Low unloading-violation option with moderate balance trade-off |

The utilisation objective appears internally as `-96` because the optimiser minimises all objectives. Operationally, that means all 96 available vessel slots were used.

### Decision Guidance

| Operational priority         | Suggested solution         |
| ---------------------------- | -------------------------- |
| Minimise unloading conflicts | S002                       |
| Prioritise vessel balance    | S003                       |
| Choose a balanced compromise | S004                       |
| Maximise slot utilisation    | Any listed Pareto solution |

A logistics planner should choose a solution based on the current operational priority rather than treating the lowest value in a single column as automatically optimal.

---

## Visual Outputs

### Pareto Front

![Pareto Front](assets/pareto-front.png)

### Selected 3D Layout

![Selected 3D Layout](assets/selected-layout-3d.png)

### Optimisation Workflow

![Optimisation Workflow](assets/optimisation-workflow.png)

### Objective Comparison

![Objective Comparison](assets/objective-comparison.png)

---

## Why NSGA-II

NSGA-II is appropriate here because:

* the assignment search space is large;
* objectives conflict;
* feasibility constraints are non-trivial;
* there is no single universally correct loading layout;
* decision-makers need transparent options rather than a black-box score.

The point of the project is not that NSGA-II magically finds “the best” plan. The point is that it exposes the trade-offs a planner must make.

---

## Reproducibility and Data Notes

* The project uses synthetic data only.
* There are no API keys, customer records, or private logistics manifests.
* Exact outputs can vary with random seed, population size, generation count, package version, and machine environment.
* For strict reproducibility, pin package versions and record the random seed used for each run.
* Do not present the synthetic results as real vessel-operational performance.

---

## Limitations

* Simplified vessel geometry.
* Simplified centre-of-gravity proxy.
* No real hydrostatic stability or trim model.
* No crane sequencing.
* No port-call schedule constraints.
* No hazardous-material separation constraints.
* No refrigerated-container placement rules.
* No repeated-seed sensitivity analysis.
* Notebook-first implementation rather than a production service.

---

## Future Improvements

* Convert the notebook into a tested Python package or CLI.
* Run repeated experiments over multiple random seeds.
* Add sensitivity analysis for population size and number of generations.
* Add crane-move minimisation and port-sequence constraints.
* Add hazardous-material and reefer-placement rules.
* Add realistic vessel-stability constraints.
* Add an interactive Pareto explorer with Streamlit or Plotly.
* Add smoke tests for result generation and feasibility checks.

---

## Skills Demonstrated

`Python` · `NSGA-II` · `pymoo` · `Multi-Objective Optimisation` · `Constraint Modelling` · `Evolutionary Algorithms` · `Pareto Analysis` · `Logistics Analytics` · `Decision Support` · `3D Visualisation`

---

## License

Released under the [Apache-2.0 License](LICENSE).
