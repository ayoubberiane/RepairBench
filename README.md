# RepairBench

## Benchmarking AI Systems for Automated Program Repair

RepairBench is an independent research project focused on the systematic evaluation of **AI-based Automated Program Repair (APR)** systems.

The project investigates how effectively modern AI systems can detect and repair software defects under standardized, controlled, and reproducible experimental conditions.

Rather than evaluating repair systems solely according to whether they generate a patch that passes an available test suite, RepairBench investigates automated repair across four complementary dimensions:

1. **Correctness**
2. **Robustness**
3. **Efficiency**
4. **Reproducibility**

The project also investigates the role of **test-guided and iterative feedback** in automated program repair.

> **Research status:** Early-stage independent research project. The benchmark, methodology, datasets, evaluation protocols, and experimental design are under active development.

---

# Table of Contents

- [Overview](#overview)
- [Motivation](#motivation)
- [Research Objective](#research-objective)
- [Research Questions](#research-questions)
- [The Four Evaluation Dimensions](#the-four-evaluation-dimensions)
  - [Correctness](#1-correctness)
  - [Robustness](#2-robustness)
  - [Efficiency](#3-efficiency)
  - [Reproducibility](#4-reproducibility)
- [Test-Guided Repair](#test-guided-repair)
- [Research Methodology](#research-methodology)
- [Benchmark Framework](#benchmark-framework)
- [System Architecture](#system-architecture)
- [Benchmark Pipeline](#benchmark-pipeline)
- [Dataset Design](#dataset-design)
- [Benchmark Instance](#benchmark-instance)
- [Patch Classification](#patch-classification)
- [Correctness Evaluation](#correctness-evaluation)
- [Robustness Evaluation](#robustness-evaluation)
- [Efficiency Evaluation](#efficiency-evaluation)
- [Reproducibility Framework](#reproducibility-framework)
- [Evaluation Metrics](#evaluation-metrics)
- [Experimental Design](#experimental-design)
- [Statistical Analysis](#statistical-analysis)
- [Research Workflow](#research-workflow)
- [Repository Structure](#repository-structure)
- [Development Roadmap](#development-roadmap)
- [Research Principles](#research-principles)
- [Limitations](#limitations)
- [Future Directions](#future-directions)
- [Expected Contributions](#expected-contributions)
- [Current Status](#current-status)
- [Contributing](#contributing)
- [Citation](#citation)
- [Disclaimer](#disclaimer)
- [License](#license)

---

# Overview

Automated Program Repair aims to automatically identify and correct software defects.

Traditional APR approaches have explored techniques based on:

- Program synthesis
- Search-based repair
- Template-based repair
- Constraint solving
- Symbolic execution
- Machine learning
- Neural program repair

More recently, advances in large language models and AI-assisted software engineering have created new approaches to automated program repair.

These systems can generate patches directly from:

- Buggy source code
- Error messages
- Test failures
- Repository context
- Documentation
- Natural-language descriptions
- Previous repair attempts

However, evaluating these systems presents a major methodological challenge.

Different studies can use different:

- Datasets
- Programming languages
- Test suites
- Models
- Model versions
- Prompts
- Sampling parameters
- Numbers of generated patches
- Computational budgets
- Evaluation environments
- Definitions of repair success

Consequently, reported performance can be difficult to compare.

RepairBench is designed to investigate this problem through a standardized, multi-dimensional evaluation framework.

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
