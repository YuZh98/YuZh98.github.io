+++
title = 'Anti-correlation Gaussian Data Augmentation'
date = 2026-05-20T12:00:00-04:00
draft = false
weight = 4
group = "research"
description = 'Code for the paper "Gibbs Sampling using Anti-correlation Gaussian Data Augmentation, with Applications to L1-ball-type Models".'

+++

> Open-source · [github.com/YuZh98/Anti-correlation-Gaussian](https://github.com/YuZh98/Anti-correlation-Gaussian)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/YuZh98/Anti-correlation-Gaussian/blob/main/LICENSE)
[![R ≥ 4.0](https://img.shields.io/badge/R-≥4.0-blue)](https://www.r-project.org/)

Code accompanying the paper [*Gibbs Sampling using Anti-correlation Gaussian Data Augmentation, with Applications to L1-ball-type Models*](https://arxiv.org/abs/2309.09371).

---

## Files

| File | Purpose |
|------|---------|
| `SourceCode.R` | `l1ball.linreg(X, y, ...)`: blocked Gibbs sampler for the L1-ball-prior linear regression model using the anti-correlation Gaussian data-augmentation step |
| `SliceSampler.R` | Scalar and directional slice samplers (Neal, 2003). Used to update κ; can be sourced standalone |
| `LinearReg.R` | Toy variable-selection demo (`n = 300`, `p = 500`) producing estimation, trace, and ACF plots |
| `TruncatedMVN.R` | Extension of the anti-correlation trick to sampling a multivariate normal truncated to a box, plus a 2-D demo |
| `tests/test_samplers.R` | Smoke + correctness tests |

## Quick start

```r
source("SliceSampler.R")
source("SourceCode.R")

set.seed(1)
n <- 100; p <- 20
X <- matrix(rnorm(n * p), n, p)
beta_true <- c(rep(2, 5), rep(0, p - 5))
y <- as.numeric(X %*% beta_true + rnorm(n))

fit <- l1ball.linreg(X, y, steps = 4000, burnin = 2000, thin = 1,
                     init_method = "random", verbose = FALSE)
round(fit$theta.est, 2)
```

To reproduce the paper's linear-regression figures:

```bash
Rscript LinearReg.R
```

## Testing

```bash
Rscript tests/test_samplers.R
```

Checks that `truncMVN` reproduces the moments of a known truncated MVN, and that `l1ball.linreg` recovers the support and signs of the true coefficients on a small synthetic problem.
