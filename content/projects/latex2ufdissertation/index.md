+++
title = 'latex2ufdissertation'
date = 2026-05-20T12:00:00-04:00
draft = false
weight = 5
description = 'A safety-net validator for UF doctoral dissertations. Produces a severity-tiered report citing the originating UF rule for each finding.'

+++

> Open-source · [github.com/YuZh98/latex2ufdissertation](https://github.com/YuZh98/latex2ufdissertation)

[![CI](https://github.com/YuZh98/latex2ufdissertation/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/YuZh98/latex2ufdissertation/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/YuZh98/latex2ufdissertation/blob/main/LICENSE)
[![Python: 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/downloads/)

A safety-net validator for UF doctoral dissertations using the Fall 2025+ University of Florida LaTeX template. Given a project archive, project directory, git URL, or compiled PDF, it produces a severity-tiered report citing the originating UF rule for each finding — one more pair of eyes before clicking submit.

## Install

```bash
pip install "git+https://github.com/YuZh98/latex2ufdissertation.git@v0.2.0"
```

Requires Python 3.10+ and (for the compile path) LuaLaTeX with TeX Live 2025.

## Quickstart

```bash
# Scaffold a new project from the bundled UF template
latex2ufdissertation --init my-thesis/
cd my-thesis/

# Validate + compile
latex2ufdissertation .

# Validate only (no TeX installation needed)
latex2ufdissertation --dry-run .
```

## Inputs

| Input | Source validation | PDF validation | Compile |
|---|---|---|---|
| Project directory | ✓ | ✓ | ✓ |
| `*.zip` archive | ✓ | ✓ | ✓ |
| Git URL | ✓ | ✓ | ✓ |
| `*.pdf` | — | ✓ | — |

## Severity tiers

- **`must-fix`** — documented UF rule violation that the Editorial Office is expected to require fixing.
- **`review`** — likely issue requiring human judgment; the tool flags, the student decides.

Every finding carries a `UF-*` rule ID and a link back to the rule's catalog entry.

## Flags

| Flag | What it does |
|---|---|
| (no flag) | Validate + compile → emit PDF |
| `--init DIR` | Scaffold a new project |
| `--demo` | Print location of the bundled demo dissertation |
| `--dry-run` | Validate only, skip compile |
| `--json` | Machine-readable summary on stdout |
| `--version` | Print version and exit |
