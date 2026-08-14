# RepairBench

## Benchmarking AI Systems for Automated Program Repair

RepairBench is a research framework for evaluating AI systems that automatically generate repairs for software defects.

The project is motivated by a central problem in automated program repair: **a patch that passes the available test suite is not necessarily a correct repair**.

RepairBench therefore approaches AI-based program repair as a broader evaluation problem involving **correctness, robustness, efficiency, and reproducibility**, while also studying the role of **test-guided iterative repair**.

> **Research status:** RepairBench is an independent research project and is intended as an experimental benchmark and research framework. Results, methodologies, and implementations should be treated as research artifacts and independently validated.

---

# Table of Contents

- [Overview](#overview)
- [Motivation](#motivation)
- [Research Questions](#research-questions)
  - [RQ1: Correctness](#rq1-correctness)
  - [RQ2: Robustness](#rq2-robustness)
  - [RQ3: Efficiency](#rq3-efficiency)
  - [RQ4: Reproducibility](#rq4-reproducibility)
  - [RQ5: Test-Guided Repair](#rq5-test-guided-repair)
- [Four Evaluation Dimensions](#four-evaluation-dimensions)
- [Research Hypothesis](#research-hypothesis)
- [Methodology](#methodology)
- [Benchmark Design](#benchmark-design)
- [Evaluation Framework](#evaluation-framework)
- [Correctness Evaluation](#correctness-evaluation)
- [Robustness Evaluation](#robustness-evaluation)
- [Efficiency Evaluation](#efficiency-evaluation)
- [Reproducibility Evaluation](#reproducibility-evaluation)
- [Test-Guided Repair](#test-guided-repair)
- [Experimental Design](#experimental-design)
- [Baselines](#baselines)
- [Datasets](#datasets)
- [Models](#models)
- [Metrics](#metrics)
- [Experimental Pipeline](#experimental-pipeline)
- [Project Architecture](#project-architecture)
- [Repository Structure](#repository-structure)
- [Configuration and Reproducibility](#configuration-and-reproducibility)
- [Results](#results)
- [Limitations](#limitations)
- [Threats to Validity](#threats-to-validity)
- [Future Work](#future-work)
- [Research Principles](#research-principles)
- [Disclaimer](#disclaimer)
- [Citation](#citation)
- [Acknowledgements](#acknowledgements)

---

# Overview

Recent advances in artificial intelligence have substantially increased the ability of AI systems to understand, generate, and modify source code.

This creates new possibilities for **automated program repair (APR)**.

An AI system can receive a buggy program, identify a suspected defect, and generate a candidate patch:

```text
Buggy Program
      ↓
AI Repair System
      ↓
Generated Patch
      ↓
Evaluation
```

The most common first criterion for evaluating the generated patch is whether the available tests pass.

However:

```text
Tests Pass
     ↓
Plausible Patch
     ≠
Guaranteed Correct Repair
```

A test suite only observes the behaviors represented by its tests. A patch can therefore satisfy the available tests while still failing to implement the intended behavior of the program.

RepairBench investigates this problem by treating automated program repair as a **multi-dimensional evaluation task** rather than reducing repair quality to a single pass/fail measurement.

The framework focuses on four primary dimensions:

```text
                    RepairBench
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   Correctness      Robustness       Efficiency
        │                │                │
        └────────────────┼────────────────┘
                         │
                         ▼
                  Reproducibility
```

A fifth experimental component studies whether **iterative test-guided feedback** improves repair effectiveness.

---

# Motivation

A fundamental question in automated program repair is:

> **When an AI system generates a patch, how do we know that the patch actually repaired the bug?**

A simple evaluation pipeline might look like:

```text
Buggy Program
      ↓
AI System
      ↓
Generated Patch
      ↓
Test Suite
      ↓
Tests Pass
```

However, **passing the available test suite does not necessarily mean that the underlying defect has actually been repaired**.

A test suite only evaluates the behaviors that its tests cover. If important behaviors are not tested, an AI system may generate a patch that satisfies the available tests while failing to implement the intended behavior of the program.

This creates an important distinction:

```text
Tests Pass
    ↓
Plausible Patch
    ≠
Guaranteed Correct Repair
```

## The Problem of Plausible Repairs

A generated patch that passes all available tests can be considered a **plausible repair**. However, plausibility and correctness are not necessarily the same.

An AI-generated patch may:

- Fix the behavior covered by a failing test while leaving the underlying defect unresolved
- Overfit to the available test suite
- Modify or remove behavior rather than correctly implementing the intended behavior
- Introduce regressions that are not detected by the existing tests
- Fail on inputs or execution paths that are not represented in the benchmark

Therefore, the evaluation of AI-based program repair should not stop at:

> **"Did the tests pass?"**

It should also ask:

> **"Does the generated patch correctly address the underlying defect while preserving the intended behavior of the program?"**

## The Need for Multi-Dimensional Evaluation

A single repair-success score does not fully describe the quality of an automated repair system.

RepairBench therefore evaluates repair systems across complementary dimensions:

```text
Correctness
    │
    ├── Does the patch actually repair the defect?
    │
Robustness
    │
    ├── Does performance survive irrelevant changes?
    │
Efficiency
    │
    ├── What computational resources are required?
    │
Reproducibility
    │
    └── Can the reported results be independently reproduced?
```

These dimensions complement traditional test-based evaluation rather than replacing it.

---

# Research Questions

RepairBench is built around the following central research question:

> **How reliable are modern AI systems at automatically repairing software defects, and how can their repair performance be evaluated beyond simply measuring whether generated patches pass the available tests?**

The project investigates this question through five research questions.

## RQ1: Correctness

> **To what extent do AI-generated patches actually repair the underlying defect rather than merely satisfy the available test suite?**

Traditional automated program repair evaluation often relies on whether a generated patch passes the tests associated with a bug.

RepairBench investigates whether this criterion is sufficient for evaluating repair quality.

The evaluation distinguishes between:

```text
Test-Suite Success
       ↓
Plausible Repair
       ↓
Correct Repair
```

where stronger validation is available.

The objective is to determine whether a patch:

- Resolves the intended defect
- Preserves the intended program behavior
- Avoids introducing regressions
- Generalizes beyond the specific behaviors represented by the original tests

### RQ1 Metrics

Depending on the benchmark and available validation mechanisms, correctness may be evaluated using:

- Test-suite pass rate
- Number of plausible patches
- Number of validated correct patches
- Number of incorrect patches
- Regression rate
- Behavioral validation results
- Agreement between test-based and stronger correctness criteria

---

## RQ2: Robustness

> **How robust are AI-based repair systems to semantics-preserving changes in the source program?**

A reliable repair system should ideally respond to the underlying defect rather than superficial characteristics of the program.

RepairBench therefore evaluates repair performance under controlled transformations that preserve program semantics.

Examples include:

- Variable renaming
- Formatting changes
- Equivalent syntactic transformations
- Other validated semantics-preserving transformations

The comparison can be represented as:

```text
Original Bug
     ↓
Repair System
     ↓
Repair Performance
```

compared with:

```text
Semantically Equivalent Bug
     ↓
Repair System
     ↓
Repair Performance
```

The objective is to determine whether equivalent representations lead to substantially different repair outcomes.

### RQ2 Metrics

Potential robustness measurements include:

- Repair success rate before transformation
- Repair success rate after transformation
- Performance difference between equivalent representations
- Patch consistency
- Failure-rate changes
- Sensitivity to individual transformations

---

## RQ3: Efficiency

> **What computational resources are required for AI systems to successfully repair software defects?**

Repair effectiveness alone does not describe the complete cost of an AI-based repair system.

A system that achieves a high repair rate but requires substantially more computation may have different practical characteristics from a system that achieves similar performance with fewer resources.

RepairBench therefore measures repair performance together with computational cost.

Potential measurements include:

- Number of generated candidates
- Number of repair attempts
- Model inference time
- Test execution time
- Total repair time
- Token usage
- Computational resources
- Monetary cost, where applicable

The central comparison is:

```text
Repair Effectiveness
        ↕
Computational Cost
```

---

## RQ4: Reproducibility

> **To what extent can reported AI-based program repair results be reproduced under controlled experimental conditions?**

AI systems can produce different results depending on experimental configuration.

Relevant factors may include:

- Model version
- Prompt
- Sampling parameters
- Random seed
- Number of candidate generations
- Dataset version
- Test-suite version
- Software dependencies
- Hardware
- Computational budget

RepairBench therefore treats experimental configuration as part of the result rather than as an implementation detail.

A reproducible experiment should make it possible to reconstruct:

```text
Dataset
   +
Repair System
   +
Model Configuration
   +
Experimental Parameters
   +
Evaluation Procedure
   ↓
Reported Result
```

### RQ4 Metrics

Potential measurements include:

- Successful reproduction rate
- Configuration completeness
- Result variance across repeated runs
- Deterministic versus stochastic behavior
- Availability of experiment artifacts
- Availability of generated patches
- Availability of evaluation results

---

## RQ5: Test-Guided Repair

> **Does iterative test feedback improve automated program repair, and what additional computational cost does it introduce?**

AI repair can be performed using a single-shot approach:

```text
Buggy Program
      ↓
AI System
      ↓
Patch
```

or through an iterative test-guided process:

```text
Buggy Program
      ↓
AI System
      ↓
Candidate Patch
      ↓
Run Tests
      ↓
Feedback
      ↓
AI System
      ↓
New Candidate Patch
      ↓
Run Tests
      ↓
...
```

RepairBench investigates whether iterative feedback produces measurable improvements in repair performance.

The analysis considers both effectiveness and cost.

### RQ5 Metrics

Potential measurements include:

- Repair success rate
- Number of iterations
- Number of generated patches
- Time to successful repair
- Token consumption
- Test executions
- Computational cost
- Improvement over single-shot repair

---

# Four Evaluation Dimensions

RepairBench defines four primary dimensions.

## 1. Correctness

**Question:**

> Does the generated patch actually repair the underlying defect?

Correctness is concerned with the semantic validity of a repair rather than only whether it satisfies the available tests.

```text
Bug
 ↓
Patch
 ↓
Tests
 ↓
Additional Validation
 ↓
Repair Assessment
```

## 2. Robustness

**Question:**

> Does the repair system behave consistently when irrelevant characteristics of the program change?

The benchmark uses controlled transformations where possible.

```text
Program A
   │
   │ Semantics-Preserving Transformation
   ▼
Program B
```

## 3. Efficiency

**Question:**

> How much computational effort is required to obtain a successful repair?

Efficiency connects repair quality to resources such as:

- Time
- Tokens
- Number of candidates
- Number of test executions
- Model calls
- Computational resources
- Monetary cost where measurable

## 4. Reproducibility

**Question:**

> Can another researcher reproduce the reported result using the documented experimental configuration?

Reproducibility requires recording the relevant conditions under which an experiment was conducted.

---

# Research Hypothesis

RepairBench is designed around several testable hypotheses.

## H1: Test Passing Is Insufficient

> **Patches that pass the available test suite will not necessarily correspond to semantically correct repairs.**

## H2: AI Repair Is Sensitive to Program Representation

> **Repair performance may change when the same underlying defect is presented through semantics-preserving variations of the source program.**

## H3: Repair Effectiveness Has a Computational Cost

> **Higher repair effectiveness may require additional model inference, candidate generation, and test execution.**

## H4: Reproducibility Requires Explicit Experimental Control

> **Reported repair results can vary when model, dataset, configuration, or execution conditions change.**

## H5: Test Feedback Can Improve Iterative Repair

> **Iterative access to test feedback may improve repair effectiveness compared with single-shot repair, but may introduce additional computational cost.**

These hypotheses are empirical questions. RepairBench does not assume that they are true; the purpose of the benchmark is to test them systematically.

---

# Methodology

RepairBench follows a controlled experimental methodology.

The general workflow is:

```text
                 ┌─────────────────┐
                 │     Dataset     │
                 └────────┬────────┘
                          ↓
                 ┌─────────────────┐
                 │ Buggy Program   │
                 └────────┬────────┘
                          ↓
                 ┌─────────────────┐
                 │  Repair System  │
                 └────────┬────────┘
                          ↓
                 ┌─────────────────┐
                 │ Generated Patch │
                 └────────┬────────┘
                          ↓
                 ┌─────────────────┐
                 │ Test Execution  │
                 └────────┬────────┘
                          ↓
                 ┌─────────────────┐
                 │ Evaluation      │
                 └────────┬────────┘
                          ↓
             ┌────────────┼────────────┐
             ↓            ↓            ↓
        Correctness   Robustness   Efficiency
             │            │            │
             └────────────┼────────────┘
                          ↓
                 Reproducibility
                          ↓
                     Results
```

The methodology is intended to separate:

1. Dataset preparation
2. Repair generation
3. Patch execution
4. Test evaluation
5. Additional validation
6. Metric computation
7. Statistical analysis
8. Result storage

---

# Benchmark Design

The benchmark is designed around repair tasks that contain:

- A buggy program
- A defect or bug description where available
- A failing or relevant test suite
- A reference or known repair where available
- Metadata describing the task
- Evaluation procedures
- Optional transformed variants

A benchmark instance can conceptually be represented as:

```text
Benchmark Instance
│
├── Buggy Source
├── Tests
├── Bug Metadata
├── Reference Patch
├── Expected Behavior
├── Transformations
└── Evaluation Metadata
```

The benchmark should preserve the distinction between:

- Information available to the repair system
- Information available only to the evaluator
- Information used to validate the final result

This separation helps prevent evaluation leakage.

---

# Evaluation Framework

RepairBench separates **repair generation** from **repair evaluation**.

```text
                Repair Generation
                       │
                       ▼
                 Candidate Patch
                       │
                       ▼
                Patch Evaluation
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
      Tests Pass?   Correct?     Cost?
          │            │            │
          └────────────┼────────────┘
                       ↓
                 Final Result
```

A patch should therefore not be described simply as "successful" without specifying the criterion used.

The framework distinguishes concepts such as:

### Compilation Success

The patched program can be built or interpreted successfully.

### Test-Suite Success

The available evaluation tests pass.

### Plausible Repair

The patch satisfies the benchmark's test-based acceptance criteria.

### Validated Correct Repair

The patch satisfies a stronger correctness criterion when such validation is available.

These categories should not be conflated.

---

# Correctness Evaluation

Correctness is the most important distinction in RepairBench.

A basic evaluation may be:

```text
Patch
 ↓
Run Tests
 ↓
All Tests Pass?
```

However, a stronger evaluation can be:

```text
Patch
 ↓
Run Existing Tests
 ↓
Run Additional Tests
 ↓
Check Expected Behavior
 ↓
Check for Regression
 ↓
Correctness Assessment
```

Where appropriate, the framework may use:

- Hidden tests
- Additional behavioral tests
- Regression tests
- Reference behavior
- Differential testing
- Metamorphic testing
- Human validation
- Independent validation mechanisms

The selected mechanism must be documented for each experiment.

---

# Robustness Evaluation

Robustness experiments apply controlled transformations to benchmark instances.

The basic process is:

```text
Original Instance
       ↓
Validate Semantics
       ↓
Apply Transformation
       ↓
Transformed Instance
       ↓
Run Same Repair System
       ↓
Compare Results
```

Potential transformations include:

### Variable Renaming

Changing identifier names while preserving references and behavior.

### Formatting Changes

Changing whitespace, indentation, or formatting without changing program semantics.

### Equivalent Syntax

Replacing a construct with a behaviorally equivalent representation where this can be validated.

### Transformation Validation

Every transformation should be validated independently.

A transformation should not be considered semantics-preserving merely because the source code appears equivalent.

---

# Efficiency Evaluation

RepairBench records computational cost alongside repair outcomes.

Potential measurements include:

| Metric | Description |
|---|---|
| Repair attempts | Number of repair attempts made |
| Candidates | Number of generated patches |
| Model calls | Number of AI inference calls |
| Generation time | Time spent generating candidates |
| Test time | Time spent executing tests |
| Total time | End-to-end repair time |
| Tokens | Input and output token usage |
| Compute | Computational resources used |
| Cost | Monetary API cost where measurable |

The exact metrics used should depend on the experimental environment.

---

# Reproducibility Evaluation

Every experiment should record enough metadata to allow another researcher to reconstruct the experiment.

A configuration should ideally contain:

```text
Experiment
│
├── Dataset Version
├── Benchmark Version
├── Model
├── Model Version
├── Prompt
├── Sampling Parameters
├── Random Seed
├── Candidate Budget
├── Test Configuration
├── Software Dependencies
├── Hardware
└── Evaluation Configuration
```

Results should be stored together with the configuration that produced them.

---

# Test-Guided Repair

RepairBench supports comparison between different repair strategies.

## Single-Shot Repair

The repair system receives the task and generates a patch without iterative test feedback.

```text
Bug
 ↓
AI
 ↓
Patch
 ↓
Evaluation
```

## Test-Guided Repair

The repair system can use test results to refine its patch.

```text
Bug
 ↓
AI
 ↓
Patch
 ↓
Tests
 ↓
Feedback
 ↓
AI
 ↓
New Patch
 ↓
Tests
 ↓
...
```

The comparison should control for factors such as:

- Model
- Dataset
- Prompting strategy
- Candidate budget
- Maximum iterations
- Computational budget

The objective is to determine whether test feedback produces an improvement that justifies its additional cost.

---

# Experimental Design

Experiments should be designed so that comparisons isolate the variable being studied.

For example, when comparing two repair strategies:

```text
                 Same Dataset
                      │
          ┌───────────┴───────────┐
          ↓                       ↓
     Strategy A               Strategy B
          │                       │
          ↓                       ↓
      Evaluation              Evaluation
          │                       │
          └───────────┬───────────┘
                      ↓
                  Comparison
```

Where possible, experiments should use:

- The same benchmark instances
- The same evaluation tests
- The same computational budget
- The same model version
- The same environmental conditions
- Documented random seeds
- Multiple runs for stochastic systems

---

# Baselines

RepairBench should compare AI-based repair systems against clearly defined baselines.

## No-Repair Baseline

The original buggy program is evaluated without modification.

This establishes the baseline failure behavior.

## Random or Simple Transformation Baseline

A controlled non-AI modification can be used where scientifically appropriate to establish whether improvements are attributable to the repair process.

## Conventional APR Baseline

Where compatible implementations are available, established automated program repair approaches may be evaluated against the same benchmark.

## AI Single-Shot Baseline

An AI model generates a patch once without iterative feedback.

## AI Test-Guided Baseline

The same or comparable AI system receives iterative test feedback.

All baseline comparisons should clearly document differences in:

- Model
- Search strategy
- Candidate budget
- Test access
- Computational resources
- Evaluation criteria

---

# Datasets

The dataset layer is separated into raw and processed data.

```text
datasets/
├── raw/
└── processed/
```

## Raw Data

Raw benchmark data should preserve the original source material as closely as possible.

Raw data should not be modified unnecessarily.

## Processed Data

Processed datasets may contain:

- Normalized benchmark instances
- Extracted metadata
- Validated transformations
- Experiment-ready representations
- Train/test or evaluation splits where applicable

Dataset processing should be deterministic where possible.

---

# Models

The `models/` directory contains model-related components and configurations.

The framework should distinguish between:

- Model identity
- Model version
- API or local runtime
- Prompt configuration
- Sampling configuration
- Candidate budget

A model should never be identified only by a generic name when the exact version materially affects reproducibility.

---

# Metrics

RepairBench uses multiple metrics rather than a single repair score.

## Repair Rate

The proportion of benchmark instances for which the repair system produces an accepted repair under a specified evaluation criterion.

```text
Repair Rate =
Successful Repairs / Evaluated Instances
```

## Plausibility Rate

The proportion of instances for which at least one generated patch satisfies the available test-based acceptance criteria.

## Correctness Rate

The proportion of instances for which a generated patch satisfies the stronger correctness criterion used by the experiment.

## Robustness Difference

The difference in repair performance between original and transformed instances.

```text
Robustness Difference =
Performance(Original)
-
Performance(Transformed)
```

The exact interpretation depends on the metric.

## Repair Attempts

The number of candidate patches generated before termination or success.

## Time to Repair

The elapsed time required to obtain an accepted repair.

## Token Usage

The number of model input and output tokens consumed, where available.

## Reproducibility Rate

The proportion of reported results that can be reproduced under the documented experimental conditions.

---

# Experimental Pipeline

The complete experimental pipeline is:

```text
                ┌─────────────────────┐
                │   Dataset Loading   │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Benchmark Validation│
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Repair Configuration│
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Patch Generation    │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Patch Application   │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Test Execution      │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Correctness Check   │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Metric Computation  │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Results + Metadata  │
                └──────────┬──────────┘
                           ↓
                ┌─────────────────────┐
                │ Statistical Analysis│
                └─────────────────────┘
```

---

# Project Architecture

RepairBench is organized around separate components for benchmark data, repair systems, experiments, evaluation, and results.

```text
RepairBench
│
├── baselines/
│   └── repairbench/
│
├── datasets/
│   ├── raw/
│   └── processed/
│
├── evaluation/
│
├── experiments/
│   ├── configs/
│   └── scripts/
│
├── models/
│
├── results/
│   ├── tables/
│   └── figures/
│
├── scripts/
│
├── tests/
│
├── README.md
├── LICENSE
└── .gitignore
```

The architecture is intentionally modular so that individual components can be developed and evaluated independently.

---

# Repository Structure

```text
.
├── baselines/
│   └── repairbench/
│
├── datasets/
│   ├── raw/
│   └── processed/
│
├── evaluation/
│
├── experiments/
│   ├── configs/
│   └── scripts/
│
├── models/
│
├── results/
│   ├── tables/
│   └── figures/
│
├── scripts/
│
├── tests/
│
├── .gitignore
├── LICENSE
└── README.md
```

## `baselines/`

Contains baseline repair systems and their associated implementations.

## `datasets/`

Contains benchmark data.

## `evaluation/`

Contains evaluation logic and metric computation.

## `experiments/`

Contains experiment configurations and execution scripts.

## `models/`

Contains model-related interfaces and configurations.

## `results/`

Stores generated experimental outputs.

## `scripts/`

Contains utility scripts used throughout the project.

## `tests/`

Contains tests for the RepairBench implementation itself.

---

# Configuration and Reproducibility

Experiments should use explicit configuration files rather than relying on undocumented command-line state.

An experiment configuration may specify:

```yaml
experiment:
  name: example_experiment

dataset:
  name: example_dataset
  version: "1.0"

model:
  name: example_model
  version: example_version

generation:
  temperature: 0.0
  max_candidates: 10

evaluation:
  run_tests: true

reproducibility:
  seed: 42
```

The exact configuration format may evolve as the framework develops.

Sensitive information such as API keys must never be committed to the repository.

---

# Results

Experimental results should be stored separately from source code.

The results structure is:

```text
results/
├── tables/
└── figures/
```

## Tables

Tables should contain machine-readable or clearly documented summaries of experimental results.

Examples include:

- Repair success rates
- Correctness results
- Robustness comparisons
- Efficiency measurements
- Reproducibility measurements

## Figures

Figures may include:

- Repair-rate comparisons
- Cost-effectiveness plots
- Robustness comparisons
- Runtime distributions
- Candidate-generation distributions
- Reproducibility analyses

Every reported result should identify the experiment configuration that generated it.

---

# Limitations

RepairBench has several important limitations.

## Test Coverage

No finite test suite can guarantee that all program behaviors have been evaluated.

Therefore, test passing should not automatically be interpreted as semantic correctness.

## Benchmark Bias

Results can depend strongly on the benchmark selected.

A repair system may perform differently on different programming languages, project types, defect classes, and complexity levels.

## Model Dependence

AI repair performance can vary substantially across models and model versions.

Results should therefore not be generalized from one model to all AI systems.

## Transformation Validity

Robustness experiments depend on the correctness of the semantics-preserving transformations.

An incorrectly transformed program can invalidate the comparison.

## Computational Constraints

Large-scale repair experiments can require substantial computational resources.

Limited budgets may therefore constrain the number of models, benchmark instances, and repeated runs that can be evaluated.

## Correctness Validation

Strong correctness validation may not always be available.

In such cases, RepairBench must clearly distinguish between test-based plausibility and stronger correctness claims.

---

# Threats to Validity

## Internal Validity

Experimental conclusions may be affected by:

- Incorrect benchmark preprocessing
- Faulty transformations
- Bugs in evaluation scripts
- Configuration inconsistencies
- Incorrect metric calculations

The framework therefore emphasizes automated validation and independent checks.

## External Validity

Results from a particular benchmark may not generalize to:

- Other programming languages
- Other software domains
- Other defect types
- Other model families
- Production software

## Construct Validity

A metric may not perfectly represent the underlying concept it is intended to measure.

For example:

```text
Test Passing
     ≠
Semantic Correctness
```

Similarly:

```text
Runtime
     ≠
Complete Computational Cost
```

Metrics should therefore be interpreted within their experimental context.

## Reproducibility Threats

External APIs, model updates, infrastructure changes, and dependency versions can affect experimental outcomes.

Experiments should therefore record as much configuration information as practically possible.

---

# Future Work

Potential future extensions include:

- Additional programming languages
- Additional defect categories
- Larger benchmark datasets
- More semantics-preserving transformations
- Stronger correctness validation
- More AI models
- Open-source repair systems
- Human evaluation
- Differential testing
- Metamorphic testing
- Statistical significance testing
- Confidence intervals
- Cost-quality optimization
- Long-horizon iterative repair
- Cross-model comparisons
- Cross-benchmark evaluation
- Automated experiment reproduction

The framework is intended to evolve as the empirical understanding of AI-based program repair develops.

---

# Research Principles

RepairBench follows several principles.

## 1. Separate Plausibility From Correctness

A test-passing patch should not automatically be described as a correct repair.

## 2. Report More Than One Number

Repair effectiveness should be considered alongside robustness, efficiency, and reproducibility.

## 3. Control Experimental Variables

Comparisons should isolate the factor being studied.

## 4. Record Experimental Configuration

Models, datasets, prompts, parameters, and evaluation procedures should be documented.

## 5. Preserve Raw Data

Raw benchmark data should remain distinguishable from processed experimental data.

## 6. Avoid Unsupported Claims

Experimental results should be reported according to the evidence available.

## 7. Make Results Reproducible

Where practical, code, configurations, datasets, generated patches, and results should be preserved.

---

# Disclaimer

RepairBench is an independent research project.

It is **not affiliated with, endorsed by, sponsored by, or officially associated with any university, research laboratory, company, AI provider, software organization, benchmark organization, or other institution unless explicitly stated otherwise in the repository or associated research materials.

The project is provided for research and experimental purposes.

Experimental results should not be interpreted as universal claims about the capabilities or limitations of AI systems without considering the benchmark, models, configurations, datasets, and evaluation procedures used.

---

# Citation

If RepairBench is used in academic research, publications, experiments, or derivative research artifacts, please cite the project according to the citation information provided here once a formal research publication or citation record is available.

A placeholder BibTeX entry may be added when the associated research work has been formally published:

```bibtex
@misc{repairbench,
  title  = {RepairBench: Benchmarking AI Systems for Automated Program Repair},
  author = {Beriane, Ayoub},
  year   = {2026},
  note   = {Independent research project}
}
```

The citation information should be updated if a formal publication is subsequently associated with the project.

---

# Acknowledgements

RepairBench builds upon the broader research literature in:

- Automated program repair
- Software testing
- Program analysis
- Machine learning for code
- Large language models
- Software engineering
- Benchmark design
- Reproducible research

Specific datasets, tools, libraries, models, and prior research used by the project should be acknowledged in the corresponding documentation and experimental records.
