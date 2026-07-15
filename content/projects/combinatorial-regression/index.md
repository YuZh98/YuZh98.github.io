+++
title = 'Combinatorial Regression'
date = 2026-05-20T12:00:00-04:00
draft = false
weight = 4
group = "research"
description = 'Reproducibility code for "Statistical Modeling for Combinatorial Response Data" — MH-within-Gibbs sampler, kernel comparison, and waterfowl data analysis.'

+++

> Open-source · [github.com/YuZh98/combinatorial-regression](https://github.com/YuZh98/combinatorial-regression)

[![License](https://img.shields.io/github/license/YuZh98/combinatorial-regression)](https://github.com/YuZh98/combinatorial-regression/blob/main/LICENSE)

Reproducibility code for the paper *Statistical Modeling for Combinatorial Response Data*. Includes the MH-within-Gibbs sampler, kernel comparison experiments, and waterfowl matching data analysis.

---

## Quick Start

```bash
# Sanity check
make smoke
make probit_smoke

# Main simulation study (MH-within-Gibbs)
make full

# Kernel comparison (supplementary)
make kernel_compare

# Data analysis (waterfowl matching)
make duck_full      # full model
make duck_reduced   # reduced model
```

## Simulation study

The default `make full` runs both exponential and half-Gaussian kernels. To run only the main-paper kernel:

```bash
JASA_METHODS=exponential make full
```

Outputs are saved under `results/runs/mh_within_gibbs/`.

## Custom settings

All scripts support environment-variable overrides:

```bash
JASA_N_ITER=20000 make full
JASA_METHODS=exponential,half_gaussian make full
```

## Documentation

See [`REPRODUCIBILITY.md`](https://github.com/YuZh98/combinatorial-regression/blob/main/REPRODUCIBILITY.md) for detailed mapping between paper sections and scripts, and [`INSTALL.md`](https://github.com/YuZh98/combinatorial-regression/blob/main/INSTALL.md) for R and Python dependency setup.
