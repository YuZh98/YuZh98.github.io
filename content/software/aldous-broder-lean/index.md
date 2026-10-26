+++
title = 'aldous-broder-lean'
date = 2026-10-26T13:00:00-04:00
draft = false
weight = 5
group = "research"
description = 'Lean 4 formalization of "The Order Mass Identity of Rooted Forests, with a Direct Proof of the Extended Aldous–Broder Theorem": the order-mass identity, the Aldous–Broder tree law, and the stopped-forest law, machine-checked with mathlib and no unfinished proofs.'
+++

> Open-source · [github.com/YuZh98/aldous-broder-lean](https://github.com/YuZh98/aldous-broder-lean)

[![CI](https://github.com/YuZh98/aldous-broder-lean/actions/workflows/ci.yml/badge.svg)](https://github.com/YuZh98/aldous-broder-lean/actions/workflows/ci.yml) [![Lean 4](https://img.shields.io/badge/Lean-4-blue)](https://github.com/YuZh98/aldous-broder-lean/blob/main/lean-toolchain) [![mathlib](https://img.shields.io/badge/mathlib-v4.30.0-blue)](https://github.com/YuZh98/aldous-broder-lean/blob/main/lake-manifest.json) [![License: Apache 2.0](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](https://github.com/YuZh98/aldous-broder-lean/blob/main/LICENSE)

Machine-checked proofs of the main results of my paper [*The Order Mass Identity of Rooted Forests, with a Direct Proof of the Extended Aldous–Broder Theorem*](https://arxiv.org/abs/2610.13063) (arXiv:2610.13063, 2026). To my knowledge, this is the first machine-checked proof of the Aldous–Broder theorem.

> **[Read the blueprint (PDF)](https://github.com/YuZh98/aldous-broder-lean/blob/main/blueprint/blueprint.pdf)**: every definition and proof in paper form, with labels cited by the Lean docstrings.

---

## What is proved

- **Order-mass identity.** For zero-row-sum matrices over *any field*, with a nonempty proper root set and nonzero principal minors on proper growth-order bases, the sum of growth-order weights entering at root $r$ equals the principal cofactor at $r$ of $I - \Gamma^B M_f$.
- **Aldous–Broder tree law.** The Aldous–Broder tree law holds for finite irreducible Markov chains started at a fixed state. Reversibility and two-way support are *not* required.
- **Stopped forest law.** The first-entrance forest obtained once all nonroots have been visited, including the completion identity for forests of zero weight.

The general directed Aldous–Broder theorem is due to [Hu, Lyons and Tang (2021), Theorem 3.4](https://doi.org/10.1214/20-AIHP1101); this project follows the paper's direct proof through the order-mass identity.

---

## Paper map

| Paper | Lean theorem (`EAB.Paper`) |
|---|---|
| Theorem 4.5: order-mass identity | `order_mass_identity`, `single_root_normalization` |
| Theorem 6.1: Aldous–Broder | `Chain.aldous_broder`, `Chain.aldous_broder_normalized` |
| Theorem 6.4: stopped forest | `Chain.stopped_forest_law` |

The [formalization reference](https://github.com/YuZh98/aldous-broder-lean/blob/main/docs/formalization.md) maps every supporting lemma to its Lean declaration and records modeling choices and the results not formalized. Path measures use mathlib's Ionescu–Tulcea construction.

---

## Why trust it

- **21 proof modules**, from matrix foundations and growth orders to the Markov property, excursions, and the forest laws.
- **No unfinished proofs:** a source scan in CI rejects any `sorry` or missing input.
- **Axiom audit:** every theorem in a fixed inventory is checked to depend only on `propext`, `Classical.choice`, and `Quot.sound`.
- **Pinned toolchain:** Lean and mathlib pinned to `v4.30.0`, dependency revisions recorded in `lake-manifest.json`.

---

## Verify it yourself

Requires [elan](https://github.com/leanprover/elan), git, and Python 3.9+.

```sh
git clone https://github.com/YuZh98/aldous-broder-lean.git
cd aldous-broder-lean
lake exe cache get
lake build
python3 scripts/check_proofs.py
python3 scripts/check_axioms.py
```

The paper's numerical appendix lives in the companion repo [aldous-broder-numerics](https://github.com/YuZh98/aldous-broder-numerics).

---

## License

[Apache 2.0](https://github.com/YuZh98/aldous-broder-lean/blob/main/LICENSE).
