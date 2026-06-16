# Tarang: Spectral Graph Scheduling for Task DAGs

A dependency-aware job scheduler that uses **spectral graph theory** (the
eigenstructure of the task-DAG Laplacian) to assign task priorities and worker
affinities, compared rigorously against three classical baselines.

This is a research project intended for an IEEE-style conference paper. The novel
contribution is the **spectral decision layer**: using the Fiedler value,
eigenvector centrality, and spectral clustering of the task graph to drive
scheduling decisions — something classical DAG schedulers (FIFO, topological,
critical-path/HEFT) do not do.

## Project structure

```
spectral-scheduler/
  pom.xml                         Maven build (pulls in EJML + JUnit)
  src/main/java/com/tarang/
    core/        Task, Dag                      (Phase 1: DAG engine)
    spectral/    LaplacianBuilder, SpectralAnalyzer  (Phase 2: math core)
    scheduler/   Scheduler, SchedulePlan,
                 FifoScheduler, TopologicalScheduler,
                 CriticalPathScheduler, SpectralScheduler  (Phase 3)
    execution/   ExecutionEngine, ExecutionResult   (Phase 4: executor + metrics)
    experiment/  RandomDagGenerator, ExperimentRunner (Phase 5: harness)
    Demo.java    end-to-end sanity check
  src/test/java/com/tarang/       JUnit tests for core + spectral
  analysis/analyze.py             (Phase 6: statistics + charts in Python)
```

## Requirements

- **Java 17+** and **Maven** (the VS Code "Extension Pack for Java" includes Maven).
- **Python 3** with `pandas numpy scipy matplotlib` for the analysis step.

## How to run

### 1. Build and test the Java project
Open the `spectral-scheduler` folder in VS Code. Maven downloads EJML + JUnit
automatically. Then:

```bash
mvn test          # runs all unit tests (should all pass)
```

### 2. Quick sanity check
Run `com.tarang.Demo` (right-click -> Run, or `mvn compile exec:java` configured
to that main class). It prints the spectral features of one DAG and a head-to-head
comparison of all four schedulers.

### 3. Run the full experiment
Run `com.tarang.experiment.ExperimentRunner`. This sweeps DAG sizes
(100 -> 5000 tasks), 20 random seeds each, all four schedulers, and writes
`results/results.csv`.

```bash
mvn compile
mvn exec:java -Dexec.mainClass=com.tarang.experiment.ExperimentRunner
```

(Or just run the class from VS Code.)

### 4. Analyze and make charts
```bash
pip install pandas numpy scipy matplotlib
python analysis/analyze.py
```

This produces, in `results/`:
- `summary.csv` — means and 95% confidence intervals
- `ttests.csv` — paired t-tests, Spectral vs each baseline
- `chart_makespan.png`, `chart_utilization.png`, `chart_crossstalls.png`
- `results_table.tex` — a LaTeX table ready to paste into the paper

## The four schedulers

| Scheduler | Priority basis | Worker affinity | Role |
|-----------|----------------|-----------------|------|
| FIFO | task id | none | floor baseline |
| Topological | topological rank | none | textbook DAG approach |
| CriticalPath | upward rank (HEFT) | none | strongest classical baseline |
| **Spectral** | centrality + critical-path, Fiedler-adapted | **spectral clusters** | **the contribution** |

## The math (Phase 2)

- **Laplacian** `L = D - A` on the symmetrized DAG skeleton (kept symmetric so
  eigenvalues are real and non-negative — see `LaplacianBuilder` for why this is a
  principled modeling choice and not a hack).
- **Fiedler value** (`lambda_2`): algebraic connectivity; gates the adaptive blend.
- **Fiedler vector / spectral bipartition**: weakly-coupled task groups.
- **Spectral clustering** (k-means on the leading eigenvectors): worker assignment.
- **Eigenvector centrality**: global structural importance of each task.

## Notes for the paper

See `PAPER_OUTLINE.md` for the suggested section structure, the precise novelty
claim, the related-work positioning (including the directed-Laplacian issue and
the citation that addresses it), and which figure/table goes where.
```
