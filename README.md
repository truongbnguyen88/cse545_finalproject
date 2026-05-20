# Integer Programming with a Hybrid Genetic Algorithm

This repository contains the final project for solving bounded integer programming problems using a hybrid **Genetic Algorithm (GA)** framework. The implementation combines classical evolutionary search with **feasibility-aware repair heuristics**, **local improvement**, and **linear programming (LP) relaxation guidance** to produce high-quality approximate solutions for large-scale instances.

## Problem Statement

The project studies integer programming problems of the form:

\[
\max \; c^T x
\]

subject to

\[
Ax \leq b
\]

with decision variables satisfying:

- \(x \geq 0\)
- \(x \in \mathbb{Z}^n\)
- \(x_j \in [lb, ub]\) for all \(j\)

In this implementation, variables are bounded between **0 and 50**.

## Repository Contents

- `integer_programming.ipynb` — main notebook containing data generation, exact optimization, genetic algorithm design, evaluation, and visualization
- `inputs.yml` — configuration file for problem size and GA hyperparameters
- `data/` — stored problem instances and saved optimal/exact solutions

## Project Workflow

The notebook follows the pipeline below:

### 1. Data Preparation
The project supports either:
- loading a pre-generated integer programming instance from the `data/` folder, or
- generating a new random instance and saving it for reuse

The generated instance includes:
- objective coefficient vector `c`
- constraint matrix `A`
- right-hand-side vector `b`

For the main experiment, the configuration uses:
- problem size: **256 variables**
- preloaded data: **enabled**

The number of constraints is scaled with problem size during data generation.

### 2. Exact Integer Programming Baseline
The notebook first solves the integer program using **PuLP** with the **CBC** solver as an exact benchmark when the instance size is manageable. For large instances such as `n = 256`, the notebook loads a previously computed exact solution from disk.

This exact solution is used as the reference point for evaluating the genetic algorithm.

For the 256-variable instance documented in the notebook:
- exact integer programming objective value: **9194.0**

### 3. Hybrid Genetic Algorithm Design
The main contribution of the project is a custom GA tailored for constrained integer optimization.

Key design components include:

#### Feasible Initialization
Instead of starting from arbitrary random solutions, the algorithm constructs feasible individuals using a greedy heuristic based on remaining constraint slack. This improves the quality of the initial population and reduces wasted search effort on invalid solutions.

#### Fitness Evaluation
Fitness is defined as:
- the objective value \(c^T x\) for feasible solutions
- a large negative penalty for infeasible solutions

This strongly biases the search toward feasible candidates.

#### Selection
The implementation uses:
- **elitist retention** of top-performing individuals
- **tournament selection** for choosing strong parents

#### Crossover Operators
Several crossover strategies are implemented and explored, including:
- order crossover (OX)
- partially mapped crossover (PMX)
- uniform crossover
- one-point crossover
- blend crossover

The notebook notes that for integer programming, operators such as **uniform crossover**, **one-point crossover**, and **blend crossover** are especially appropriate because they preserve the numeric structure of integer-valued decision vectors.

#### Mutation Operators
The implementation includes:
- **swap mutation**
- **inversion mutation**
- **delta mutation**

Delta mutation is especially useful for integer programming because it perturbs a small number of variables by bounded integer steps, enabling local exploration without fully disrupting a promising solution.

### 4. Feasibility Repair and Improvement Heuristics
A major part of the notebook is devoted to maintaining and improving feasibility after crossover and mutation.

The GA includes several improvement procedures:

- **solution_improve**  
  Repairs infeasible solutions by greedily reducing variables that most effectively remove constraint violations while minimizing loss in objective value.

- **fill_slack_improve**  
  Once a solution is feasible, this heuristic tries to increase selected variables to consume unused slack and improve the objective.

- **local_improve**  
  Performs small bounded local perturbations on promising variables to search for additional objective gains while preserving feasibility.

These steps make the algorithm substantially more problem-aware than a basic GA.

### 5. LP Relaxation Hybridization
The notebook also integrates **LP relaxation** into the GA.

This hybrid approach works by:
- solving the LP relaxation with `scipy.optimize.linprog` using the **HiGHS** method
- rounding LP solutions to integer candidates
- repairing and improving rounded solutions
- injecting strong LP-derived candidates into the population during initialization and periodically during evolution

This LP-guided reinjection helps maintain both solution quality and population diversity, and accelerates convergence.

### 6. Multi-Run Evaluation
To reduce sensitivity to random initialization, the GA is executed multiple times using different random seeds.

Configuration from `inputs.yml`:
- population size: **300**
- number of generations: **300**
- tournament size: **5**
- top-solution retention fraction: **0.15**
- swap mutation probability: **0.3**
- inversion mutation probability: **0.2**
- delta mutation probability: **0.35**
- stall patience: **50**
- number of GA runs: **10**

The notebook records runtime and best objective value from each run, then keeps the best overall result.

Example run summary shown in the notebook:
- average runtime across 10 runs: **376.26 seconds**
- best GA objective found: **8697.0**

### 7. Comparison Against Exact Solution
The best GA solution is compared against the exact integer programming optimum.

Reported results in the notebook:
- GA objective value: **8697.0**
- exact IP objective value: **9194.0**
- relative error: **0.0541**
- feasibility check: **True**

This shows that the hybrid GA produced a feasible solution within approximately **5.4%** of the exact optimum on a 256-variable problem.

### 8. Visualization
The notebook concludes by plotting the convergence history of the best-performing GA run, showing how the best fitness evolves across generations.

## Configuration

Main settings are stored in `inputs.yml`. Current values include:

- `n: 256`
- `data_load: True`
- `use_lp_relaxation: True`
- `lp_reinject_every: 20`
- `n_lp_solutions_frac: 0.1`
- `population_size: 300`
- `num_generations: 300`
- `tournament_size: 5`
- `topsoln_frac: 0.15`
- `swap_p: 0.3`
- `inv_p: 0.2`
- `delta_mutate_p: 0.35`
- `keep_children: 5`
- `stall_patience: 50`
- `n_ga_runs: 10`

## Key Takeaways

This project demonstrates that a carefully designed GA can be a practical approximation method for large bounded integer programming problems when exact optimization becomes expensive.

The notebook specifically shows the value of combining:
- feasible constructive initialization
- crossover and mutation adapted to integer vectors
- repair-based constraint handling
- local search improvements
- LP-relaxation-based candidate injection

Together, these techniques produce a robust hybrid metaheuristic capable of finding high-quality feasible solutions on nontrivial problem instances.

## Requirements

The notebook uses the following Python libraries:

- `numpy`
- `pandas`
- `matplotlib`
- `seaborn`
- `yaml`
- `pickle`
- `pulp`
- `scipy`

## How to Run

1. Update parameters in `inputs.yml`
2. Open and run `integer_programming.ipynb`
3. If generating a new dataset, first run with `data_load: False` to save the instance
4. Then set `data_load: True` to load the saved data and run optimization experiments

## Summary

In summary, this repository presents a hybrid optimization framework for integer programming that combines exact benchmarking with a customized genetic algorithm enhanced by repair heuristics, local search, and LP relaxation. The final implementation achieves strong approximate performance on a 256-variable constrained integer optimization problem while maintaining solution feasibility.
