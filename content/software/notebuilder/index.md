+++
title = 'NoteBuilder'
date = 2026-10-26T12:00:00-04:00
draft = false
weight = 6
group = "apps"
description = 'Local-first macOS app for interactive study notebooks: sliders, formulas, charts, and quizzes that react to each other, shared as one offline HTML file. No account, no cloud.'
+++

> Open-source · [github.com/YuZh98/notebuilder](https://github.com/YuZh98/notebuilder)

Study material that does something. Build notebooks from reactive cards, drill them in Study mode, and hand them to anyone as a single offline file.

![NoteBuilder Study mode: two sliders drive computed values, live prose, and a growth chart in one card](https://raw.githubusercontent.com/YuZh98/notebuilder/main/docs/screenshots/v0.2.0/study-live.png)

[![CI](https://github.com/YuZh98/notebuilder/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/YuZh98/notebuilder/actions/workflows/ci.yml) [![Tests](https://img.shields.io/badge/tests-2718-brightgreen)](https://github.com/YuZh98/notebuilder/tree/main/v2) [![Svelte 5](https://img.shields.io/badge/Svelte-5-FF3E00?logo=svelte&logoColor=white)](https://github.com/YuZh98/notebuilder/blob/main/v2/package.json) [![Tauri 2](https://img.shields.io/badge/Tauri-2-24C8DB?logo=tauri&logoColor=white)](https://github.com/YuZh98/notebuilder/tree/main/v2/src-tauri) [![macOS](https://img.shields.io/badge/platform-macOS-black?logo=apple)](https://github.com/YuZh98/notebuilder/releases/latest) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://github.com/YuZh98/notebuilder/blob/main/LICENSE)

> **[Download for macOS](https://github.com/YuZh98/notebuilder/releases/latest)** (Apple silicon, v0.3.0) · **[Website and live demo](https://yuzh98.github.io/notebuilder/)**. Notebooks live in a local SQLite database on your machine; nothing is sent anywhere.

---

## Who is this for?

- **Students and self-learners** who want flashcards that flip, quizzes that check themselves, practice problems with fresh numbers every attempt, and a progress meter that fills as they finish cards.
- **Teachers and tutors** who want to send one file that works in any browser, offline, with nothing to install, that remembers each student's progress and hands back a results summary.
- **Anyone explaining something quantitative.** A slider, a formula, and a chart in one card wire together and react live. Drag the slider, watch the curve move.

---

## Why not a document?

| Problem | Static notes | NoteBuilder |
| --- | --- | --- |
| Self-testing | Re-reading, highlighting | Flashcards flip, quizzes grade themselves, practice problems redraw their numbers |
| Quantitative intuition | A pasted screenshot of a plot | Slider drives formula drives chart, live, per card |
| Knowing where you stand | Gut feeling | Per-reader progress meters count what you have actually finished |
| Sharing | Recipient needs your app or an account | One self-contained `.html` that opens offline in any browser |
| Authoring cost | Hours per lesson | Any chat AI drafts it from the built-in prompt; you review the drafts |

---

## How it works

**Cards are built from slots.** A card is a small stack of primitives on a 12-column grid: text, slider, formula, chart, choice, practice problem, table, code, image, checklist, text answer, progress. Slots in the same card share variables; dependencies resolve by topological sort, and cycles or missing writers are caught at import.

**Edit and Study are hard-separated.** Edit shows structure and configuration; Study *is* the rendered, interactive result. `Cmd+Enter` flips between them.

**Any AI can draft a notebook.** **＋ → From AI…** writes a complete prompt for ChatGPT, Claude, Gemini, or any other chat AI. Paste the reply back; NoteBuilder validates it, explains any problems with where and how to fix them, and marks every new card as a **draft** until you review it. No AI runs inside the app, and the [notebook format](https://github.com/YuZh98/notebuilder/blob/main/docs/FORMAT.md) is published.

**Sharing is a file.** Export a notebook as a self-contained Reader `.html`. The recipient reads and interacts; their answers and progress are saved in their own browser, and **My results** gives them a summary to send back.

| ![Edit mode: slot configuration fields with tags, section operations, and the card-type store in the sidebar](https://raw.githubusercontent.com/YuZh98/notebuilder/main/docs/screenshots/v0.2.0/edit-mode.png) | ![A shared Reader file in a plain browser: locked chrome with only a Remix button](https://raw.githubusercontent.com/YuZh98/notebuilder/main/docs/screenshots/v0.2.0/reader.png) |
| --- | --- |
| *Edit mode: every slot becomes its configuration fields.* | *The shared `.html` in a plain browser: authoring chrome is gone, and the reader's state is their own.* |

---

## Engineering

- **Stack:** Svelte 5 + TypeScript (strict) front end in a Rust Tauri 2 shell; SQLite storage with migrations; native print and dialogs.
- **Durable saves:** every database access rides one queue, each save is a single transaction, and saves survive a crash or a Cmd+Q. Persisted-shape changes ship a migration, so old files keep loading.
- **Security:** no `eval` on user content; formulas run through a small purpose-built expression evaluator, all user text is escaped before Markdown or KaTeX rendering, and SQL is parameterized throughout.
- **Tests:** 2,718 tests across 204 files, svelte-check clean, CI on every push. Source-level guard tests lock architectural invariants so regressions fail loudly.

---

## License

[MIT](https://github.com/YuZh98/notebuilder/blob/main/LICENSE).
