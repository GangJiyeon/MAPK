# MAPK — Cell-to-Cell Drug Response Explorer

> Exploring how differences in signaling protein abundance produce
> heterogeneous ERK suppression under the same KRAS mutation and drug conditions.

## Overview

MAPK is a mechanistic simulation and scientific-tool engineering project.

Its central question is:

**Under the same KRAS mutation and drug conditions, how much
heterogeneity in ERK suppression can arise from cell-to-cell differences
in signaling protein abundance, and which factors contribute most?**

The project explores this hypothesis within a simplified model.
It does not establish the causes of real cellular drug-response
heterogeneity.

Molecular activity prediction and a tool-using AI agent extend the
core simulator. Validation, reproducibility, and failure detection
are primary engineering outcomes.

## Current Status

**Planning and feasibility investigation — October 2026.**

The repository currently contains a React frontend scaffold and empty
backend implementation files. The components described below are planned,
not completed.

The initial data audit is due on **October 18, 2026**.
The available evidence at that deadline will determine the initial
implementation scope in ADR-000.

## Scientific Basis

Two studies motivate prioritizing extrinsic variability:

- Filippi et al., Cell Reports (2016).
  DOI: 10.1016/j.celrep.2016.05.024
  PMID: 27264188

  In the studied MEK–ERK system, extrinsic variability, particularly
  upstream of MEK, dominated cell-to-cell variation in ERK phosphorylation.

- Iwamoto et al., PLOS Computational Biology (2016).
  DOI: 10.1371/journal.pcbi.1005222
  PMCID: PMC5130170

  An EGF signaling model showed heterogeneous responses arising from
  variation in particular signaling proteins. Its nuclear ERK switch
  behavior depended on model structure not included in the simplified
  cascade used here.

These findings motivate the modeling approach. They do not establish
that protein abundance explains all drug-response differences in
KRAS G12D cancer cells.

## Architecture

1. Molecular activity prediction or experimental-value lookup.
2. Phenomenological activity-to-inhibition mapping.
3. MAPK ODE simulation for a shared virtual-cell population.
4. ERK suppression distributions and sensitivity analysis.
5. Validation and agent evaluation.

### Separation Principles

- The core simulator runs without a trained molecular model.
- Molecular predictions and lookup values share a common interface.
- Every molecular result identifies its source:
  `lookup`, `rf`, or `gcn`.
- Lookup-mode harness results are reported separately from molecular
  prediction performance.

## 1. Molecular Activity Prediction — molbind

### Data Scope

Initial targets: KRAS WT, G12C, and G12D.
Mutation labels are preserved.

Sources: BindingDB and ChEMBL.

Regression data requirements:

- Confirmed biochemical IC50 measurements.
- Exact measurements (`=`) only.
- Censored measurements (`>` / `<`) stored separately.
- IC50 converted to nM and then pIC50:

  `pIC50 = 9 - log10(IC50_nM)`

- Assay endpoints, conditions, mechanisms where available, and source
  identifiers preserved.
- Cellular and ambiguous assays excluded from the initial regression set.
- Different biochemical endpoints are not automatically treated as
  interchangeable measurements.

Structures are standardized using RDKit while preserving the original
structures and measurement records.

Cross-source duplicate candidates are identified using standardized
InChIKey, target/mutation, and normalized literature identifiers.
Aggregation is restricted to compatible assays and conditions.

### Models

- Baseline: molecular fingerprints + Random Forest.
- Comparison: GCN with a manually implemented message-passing layer.

Both models use the same frozen scaffold split.

Report MAE, RMSE, R², split sizes, scaffold counts, and limitations.
The GCN will be implemented even if it does not outperform the RF.

The initial 300-measurement / 30-scaffold threshold is a planning
heuristic, not a model-adequacy guarantee.

Transfer learning is not predetermined.

### Lookup Backup

If available data cannot support useful model training, the harness can
use a frozen table of eligible experimental pIC50 measurements.

The number of eligible lookup candidates is determined by the audit;
300 entries are a target, not an assumption.

Unknown structures return an explicit unavailable result.
Lookup values do not establish predictive generalization.

MRTX1133 is a planned sanity-check compound. Its structure, assay
records, and eligible activity values must be verified before use.

## 2. MAPK ODE

The reference model is the Kholodenko2000 simplified MAPK cascade.

The planned reduced implementation uses five independent states:
MAPKKK*, MAPKK-P, MAPKK-PP, MAPK-P, and MAPK-PP.

RAF, MEK, and ERK are biological interpretations of these cascade levels,
not a claim that the reference model is calibrated to a specific
KRAS-mutant cancer cell.

An effective upstream input represents KRAS-associated activity.
The model does not explicitly simulate the KRAS nucleotide cycle.

### Reference Verification

- Preserve the BioModels SBML identifier and version.
- Execute the reference model using COPASI/basico.
- Record parameters, initial conditions, units, solver settings,
  simulation horizon, and output grid.
- Compare the scipy implementation with the reference trajectory.
- Compare peak time, peak value, and steady-state value when applicable.
- Report final-time values when steady state is not reached.

Oscillations and absence of steady-state convergence are not
automatically numerical failures.

## 3. Activity-to-Inhibition Mapping

Initial mapping:

`input_eff = input / (1 + dose_nM / IC50_nM)`

This is a **phenomenological mapping** from biochemical activity to
effective upstream inhibition.

It is not a validated competitive-inhibition mechanism, a direct
IC50-to-Ki conversion, or a calibrated prediction of cellular potency.

Initial dose scenarios: 10 nM, 100 nM, and 1 µM.

ADR-002 documents:

- Assay dependence of the activity value.
- The effective exposure assumption.
- Omitted pharmacokinetics and intracellular exposure differences.
- Mapping biochemical activity to upstream signaling.
- Sensitivity to alternative mapping parameters.

## 4. Virtual-Cell Population

Virtual cells differ in modeled protein totals and/or explicitly
documented effective input and rate parameters.

Initial distributions and correlations are informed by relevant
literature where possible. Unsupported choices are scenario assumptions,
not measured cellular distributions.

### Required Rules

1. Map every sampled quantity to an actual model term.
   Distinguish protein totals from effective rate parameters.

2. Preserve each cell's totals and consistent initial-state constraints.

3. Use the same virtual cells for untreated/treated comparisons
   and across candidate molecules.

4. Define response as ERK suppression, not cell death or clinical benefit.

5. Treat 1,000 cells as an initial population size.
   Check stability of distributions, response fractions, and rankings
   as population size increases.

### Response Measure

For each cell:

`AUC reduction = 1 - AUC_treated / AUC_untreated`

The simulation horizon, treatment initialization, and response threshold
are fixed and documented.

Near-zero untreated AUC is handled explicitly.

Response-fraction results include threshold sensitivity.
CV = 0 must reproduce the corresponding single-cell ODE result.

## 5. Sensitivity Analysis

Identify model inputs associated with variation in ERK suppression.

Initial analysis includes correlation and regression.
Results are interpreted as model-dependent sensitivity, not biological
causal proof.

Report protein-total effects separately from effective-rate effects.

## 6. Validation and Failure Injection

| Scenario | Expected behavior |
|---|---|
| Invalid SMILES | Parsing failure and explicit rejection |
| Low similarity to training data | Applicability warning |
| Prediction outside the training label range | Extrapolation warning |
| Numerical or state-constraint failure | Invalid simulation result |
| Unsupported agent claim | Evidence-check failure |

A Tanimoto threshold such as 0.4 is an initial heuristic to evaluate,
not a universal applicability boundary.

Numerical checks distinguish solver failure, non-finite outputs,
state-bound violations, and valid non-steady-state dynamics.

Metrics:

- Failure detection recall.
- False positive rate.
- Fraction of injected failures reaching the final answer.

## 7. Agent Harness

A Claude-based ReAct agent uses:

- `predict_pic50`
- `simulate_population`
- `check_druglike`

Tool schemas include mutation/endpoint context where required.
Simulation tools reference a frozen population and configuration.

Drug-likeness checks report defined properties and rule violations;
they do not establish safety, efficacy, or developability.

The agent searches a fixed candidate pool under a tool-call budget.
Molecular generation is out of scope.

### Evaluation Tasks

Initial task:
find candidates with the highest simulated ERK-suppression response fraction.

Additional tasks, such as selectivity or minimum effective dose,
are adopted only when data and preliminary experiments support them.

Adding an axis does not automatically create a nontrivial search problem.

### Baselines

- Random selection.
- pIC50 sorting.
- Simple search under the same tool-call budget.
- ReAct agent.

If pIC50 sorting solves the task, report that result rather than
manufacturing agent complexity.

### Reference Ranking

Exhaustive execution of the same frozen tools provides the benchmark
reference ranking.

**The reference is simulator-internal. It evaluates tool use against
exhaustive execution of the same tools, not real-world drug efficacy.**

Report Top-5 overlap, objective gap, tool calls, runtime, API cost,
and failure handling. Ties are handled explicitly.

## Candidate Pool

Target size: 300–500 molecules, subject to eligible data and runtime.

In prediction mode, candidate structures are excluded from model training.
Scaffold overlap and training-set similarity are reported.

In lookup mode, candidate labels are available to the lookup tool;
this is a harness benchmark, not a held-out prediction evaluation.

## Implementation Schedule

### October
Data audit, reference-model investigation, environment verification,
candidate-pool feasibility, and variability assumptions.

### November
RF and GCN, verified scipy ODE, virtual-cell population,
sensitivity analysis, and initial harness integration as progress allows.

### December
Harness evaluation, baseline comparisons, failure injection,
and reproducibility improvements.

### January
Documentation, showcase, technical writing, and job applications.
New features are deferred.

The November planning budget is approximately 250 hours.
Progress is assessed through verified deliverables, not hours alone.

## Design Decisions

- ADR-000: Data feasibility and initial implementation scope.
- ADR-001: Choosing the Kholodenko simplified cascade.
- ADR-002: Phenomenological activity-to-inhibition mapping.
- ADR-003: Fixed candidate pool instead of molecular generation.
- ADR-004: GCN vs RF results.
- ADR-005: Prioritizing parameter ensembles over Gillespie.
- ADR-006: Excluding quantum computation from this MVP.

## Out of Scope

- Gillespie simulation in the initial MVP.
- Quantum computation.
- Molecular generation.
- Patient-specific prediction.
- Clinical efficacy or treatment recommendations.

## Limitations

This is an experimentally unvalidated mechanistic model.

Independent reference execution verifies implementation consistency,
not biological validity.

Population distributions, inhibition mapping, and simplified pathway
structure constrain the interpretation of all results.

## Reproduction

Verified installation and execution instructions will be added
as components are completed.

## License

MIT for project code.

Third-party data and models remain subject to their own licenses
and attribution requirements.
