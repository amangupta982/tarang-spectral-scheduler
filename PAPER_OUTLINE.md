# Paper Outline — Spectral Graph Scheduling for Task DAGs

Target: IEEE-style conference paper (~6–8 pages, double column).

## Title (working)
"Tarang: Spectral Graph-Theoretic Priority and Partitioning for DAG Task Scheduling"

## Abstract (write last)
One paragraph: problem (DAG scheduling on multi-worker systems), gap (classical
heuristics ignore the global spectral structure of the dependency graph), method
(use Laplacian eigenstructure — Fiedler value, eigenvector centrality, spectral
clustering — to set priorities and worker affinity), results (X% lower makespan
and Y% higher utilization vs FIFO/topological/critical-path baselines, all
statistically significant, p < 0.001 across DAGs of 100–5000 tasks).

## I. Introduction
- DAG scheduling is fundamental (build systems, scientific workflows, data
  pipelines, compilers).
- Classical list schedulers (HEFT, critical-path) use *local* or *path-based*
  information only.
- Insight: the DAG carries *global* structure — bottlenecks, clusters,
  influential hubs — that is invisible to path-length heuristics but visible in
  the Laplacian spectrum.
- Contribution bullets:
  1. A spectral priority function blending eigenvector centrality with
     critical-path urgency, adapted by the Fiedler value.
  2. A spectral-clustering worker-affinity scheme that minimizes cross-worker
     synchronization.
  3. A reproducible experimental study showing significant gains over three
     baselines across DAG scales.

## II. Background and Related Work
- DAG scheduling and list-scheduling heuristics (HEFT, CPOP).
- Spectral graph theory: Laplacian, Fiedler value, Cheeger's inequality,
  spectral clustering.
- **Crucial paragraph — the directed-Laplacian issue.** Spectral theory's clean
  guarantees are for symmetric (undirected) matrices; directed Laplacians can
  have complex eigenvalues. State your modeling choice explicitly: analyze the
  undirected skeleton for partitioning/centrality while retaining direction for
  execution ordering. Cite work establishing real non-negative eigenvalues for
  constructed DAG Laplacians (the 2023 DAG-diffusion line of work) to show
  awareness. This pre-empts the obvious reviewer objection.
- Gap statement: spectral structure has been used for partitioning and for
  GNNs, but not, to our knowledge, as the basis for runtime DAG scheduling
  priority + affinity decisions.

## III. Problem Formulation
- Define the DAG G = (V, E), task costs, worker set, dependency constraint.
- Objective: minimize makespan; secondary: maximize utilization, minimize
  cross-worker stalls.
- State it as the standard NP-hard multiprocessor DAG scheduling problem (so the
  use of heuristics is justified).

## IV. Spectral Scheduling Method (the core)
- IV-A Laplacian construction (symmetric skeleton): L = D − A.
- IV-B Spectral features: Fiedler value λ₂, Fiedler vector, eigenvector
  centrality, k-way spectral clustering. Give the equations.
- IV-C Priority function:
  `priority(t) = α·centrality_norm(t) + (1−α)·criticalpath_norm(t)`,
  with α adapted by the Fiedler value: `α_eff = α / (1 + λ₂)`. Explain the
  intuition (low connectivity ⇒ strong cluster structure ⇒ trust global
  centrality more).
- IV-D Worker affinity via spectral clusters.
- IV-E (optional theory flourish) Connect λ₂ to a bottleneck cut via Cheeger's
  inequality — one short subsection gives the paper mathematical weight.
- Include a small worked example figure (a ~10-task DAG with its bipartition).

## V. Experimental Setup
- Random layered DAG generator (describe parameters; emphasize seeded
  reproducibility).
- DAG sizes 100–5000, 20 trials each, 8 workers.
- Discrete-event execution model (deterministic ⇒ reproducible; same model for
  all schedulers ⇒ fair).
- Baselines: FIFO, Topological, CriticalPath (HEFT-style).
- Metrics: makespan, utilization, cross-worker stalls.
- Statistics: 95% CIs, paired t-tests.

## VI. Results
- Table I: mean makespan ± CI (from results_table.tex).
- Fig. 1: makespan vs size. Fig. 2: utilization vs size. Fig. 3: cross-worker
  stalls vs size.
- Report % improvement and significance for Spectral vs each baseline.
- Sensitivity analysis: vary α and edge density; show robustness.
- Discuss when spectral helps most (low-λ₂, strongly clustered workflows).

## VII. Threats to Validity / Limitations
- Simulation vs real hardware (justify, note future work with real threads).
- Symmetric-skeleton modeling choice.
- Eigendecomposition cost O(n³) — discuss scalability and approximate
  (Lanczos / sparse) eigen-solvers as future work; report plan-time overhead
  (the planMs column) honestly.

## VIII. Conclusion and Future Work
- Restate contribution and headline numbers.
- Future: sparse/approximate spectra for very large DAGs, directed-Laplacian
  formulations, real distributed execution, energy-aware objectives.

## References
- HEFT (Topcuoglu et al.), spectral clustering (Ng-Jordan-Weiss, von Luxburg
  tutorial), Fiedler (algebraic connectivity), Cheeger's inequality, the 2023
  DAG-Laplacian diffusion paper, classic DAG scheduling surveys.

---
### Honest contribution statement (for your own clarity)
The DAG engine and thread/execution scaffolding extend prior coursework
(TaskForge) and are NOT claimed as novel. The novelty is entirely in the spectral
decision layer (Section IV) and its experimental validation (Sections V–VI).
Be ready to state exactly this if asked in review or a viva.
```
