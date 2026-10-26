+++
title = 'RecVAE — Recurrent 3D-Conv VAE for fMRI'
date = 2026-05-20T12:00:00-04:00
draft = false
weight = 5
group = "research"
description = 'A PyTorch reference implementation plus an 18-lesson tutorial series for modelling temporal fMRI data with deep variational autoencoders.'
aliases = ["/projects/vae-fmri-alzheimer/"]

+++

> Open-source · [github.com/YuZh98/VAE-fMRI-Alzheimer](https://github.com/YuZh98/VAE-fMRI-Alzheimer)

[![CI](https://github.com/YuZh98/VAE-fMRI-Alzheimer/actions/workflows/tutorials.yml/badge.svg)](https://github.com/YuZh98/VAE-fMRI-Alzheimer/actions/workflows/tutorials.yml)
[![License: Apache-2.0](https://img.shields.io/badge/license-Apache--2.0-blue)](https://github.com/YuZh98/VAE-fMRI-Alzheimer/blob/main/LICENSE)
[![Python ≥3.9](https://img.shields.io/badge/python-3.9%2B-blue)](https://www.python.org/)
[![PyTorch ≥2.0](https://img.shields.io/badge/pytorch-2.0%2B-ee4c2c)](https://pytorch.org/)
[![Open in Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/YuZh98/VAE-fMRI-Alzheimer/blob/main/notebooks/RecVAE_on_synthetic.ipynb)

**A recurrent 3D-conv VAE for resting-state fMRI volumes**, in the same family as Kim et al. (2021) but with a linear latent transition solved in closed form by ridge regression rather than learned by SGD. Ships with a synthetic data path so you can run it without ADNI access; every example fits on a laptop CPU.

```bash
git clone https://github.com/YuZh98/VAE-fMRI-Alzheimer
cd VAE-fMRI-Alzheimer
python -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"
pytest -v                                              # 36 tests, ~5-20s, CPU-only
python tutorials/15_train_end_to_end/train_tiny.py     # synthetic, ~3s
```

## Tutorials

[`tutorials/`](https://github.com/YuZh98/VAE-fMRI-Alzheimer/tree/main/tutorials) has 18 hands-on lessons covering tensor shapes, 3D-conv arithmetic, the reparameterization trick, recurrent rollouts, alternating optimization, reproducibility, testing DL code, and research extensions.

## Architecture

A 3D-convolutional VAE with a latent recurrence:

```
                    ┌──────────────┐
   x_t ─────────────► 3D-CNN encode├──┐
   (1,91,109,91)    └──────────────┘  │  (B, 100)
                                      ▼
                       ┌────────────────────────────┐
   h_{t-1} ───────────►│  hidden2mu, hidden2log_var │
   (B, 10)             └────────────────────────────┘
                                      │
                                      ▼      ε ~ N(0, I)   (reparam)
                                 mu_h, log_var_h ────────────┐
                                                              ▼
                                                      h_t = mu_h + σ_h ε   (B, 10)
                                                              │
                                       z_s (subject noise) ───┤
                                                              ▼
                                                      h_t + z_s  ─────────┐
                                                                          ▼
                                                              ┌─────────────────────┐
                                                              │  3D-CNN decode      │
                                                              └─────────────────────┘
                                                                          │
                                                                          ▼
                                                                  μ_t  (B,1,91,109,91)

   Linear temporal prior: g(h) = h F^T  →  used in loss term  ‖h_t − g(h_{t-1})‖²
```

## Loss

| Term     | Form                                    | How it's optimized       |
|----------|-----------------------------------------|--------------------------|
| `loss1`  | Per-volume reconstruction MSE / σ_x²    | SGD                      |
| `loss2`  | ‖h_t − g(h_{t-1})‖² / σ_h² (temporal)   | SGD                      |
| `loss_z` | λ_z · ‖z‖₁ (subject-noise sparsity)     | SGD                      |
| `loss_F` | ρ · ‖F‖_F² (reported, not back-propped) | Closed form (ridge)      |

## Visuals

| Loss curve | Reconstruction | Latent trajectory |
|---|---|---|
| ![loss](https://raw.githubusercontent.com/YuZh98/VAE-fMRI-Alzheimer/main/docs/assets/loss_curve.png) | ![recon](https://raw.githubusercontent.com/YuZh98/VAE-fMRI-Alzheimer/main/docs/assets/recon_slice.png) | ![latents](https://raw.githubusercontent.com/YuZh98/VAE-fMRI-Alzheimer/main/docs/assets/latent_trajectory.png) |

## Citation

If you use this code in research or teaching, please cite via the GitHub "Cite this repository" button (driven by [`CITATION.cff`](https://github.com/YuZh98/VAE-fMRI-Alzheimer/blob/main/CITATION.cff)).
