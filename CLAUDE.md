# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A one-page LaTeX resume (CV), written in Russian. There is no code, tests, or linter — only LaTeX sources that compile to a PDF.

## Build

The document requires **XeLaTeX** (it uses `fontspec` and `polyglossia`; pdflatex will not work). Build output goes into `build/` (gitignored):

```sh
mkdir -p build && xelatex -interaction=nonstopmode -file-line-error -output-directory=build backend.tex
```

There is one top-level `.tex` file per resume (`backend.tex`, `devops.tex`, …), and they all live side by side; to build a particular resume, pass its file to the command above instead of `backend.tex`. Run it twice for stable output (matches the default `xelatex x 2` recipe in `.vscode/.settings-for-*.json`, used by the LaTeX Workshop extension). No bibliography, so biber is not needed.

The committed `resume.pdf` and `resume.png` (shown in `README.md`) are manually updated snapshots of the compiled resume, not build artifacts — refresh them only when asked.

## Architecture

The project is a **resume constructor**: it holds several resumes for different roles (backend, devops, etc.) assembled from a shared pool of building blocks.

- **Top-level files = resume variants.** Each role gets its own top-level `.tex` file (`backend.tex` currently; e.g. `devops.tex` for others). It contains no content of its own — only `\input{preamble}` and a selection of `sections/*` and `items/*` in the desired order, grouped under `\cvsection{...}` + `cvlist`. To create a new variant, copy an existing top-level file and change which sections/items it includes.
- **`sections/`** — the structural parts of a resume (header, education, skills, appendix). Shared sections are reused as is; role-specific ones get a role suffix (`skills-backend.tex`, `skills-devops.tex`, …).
- **`items/`** — the pool of experience and achievements, one project/achievement per file. The same item can be included in several variants and under different headings (e.g. "Релевантный опыт" in one resume, "Достижения" in another); the top-level file decides placement, never the item.
- **`preamble.tex`**: packages, page geometry, fonts, languages, hyperref. Loads `config.tex` and `macros.tex` first.
- **`config.tex`**: personal data (name, GitHub/Telegram/email URLs and labels) as `\cv...` commands, plus OS-dependent font selection via `ifplatform` (Liberation fonts on Linux, Times New Roman / Courier New elsewhere). Change personal data here, not in `sections/header.tex`.
- **`macros.tex`**: the resume's visual vocabulary — colors `cvred`/`cvgray`, and `\cvsection{title}`, `\cventry{left}{right}`, `\cvproject[note]{name}{tech}`, `\cvsubtext{text}`, and the `cvlist` environment (compact `enumerate`). Use these rather than ad-hoc formatting.
- **Section file format**: each file in `sections/` starts with its own `\cvsection{...}` (except `header.tex`).
- **Item file format**: each file in `items/` is a single `\item` (typically `\item \cvproject{...}{...}` followed by "Задача / Стек / Результат" lines separated by `\\`), meant to be `\input` inside a `cvlist` environment in the top-level file.

## Conventions

- All content and comments are in Russian; each `.tex` file starts with a short `%` comment describing its purpose. Commit messages are also in Russian.
- Indentation is mandatory and reflects nesting: everything inside an environment (`\begin{document}`, `cvlist`, `center`, …) is indented one level deeper, the content of a section is indented one level under its `\cvsection{...}`, and the body lines of an item file are indented one level under its `\item` line:
  ```latex
  \cvsection{Достижения}
  \begin{cvlist}
      \input{items/sbom}
  \end{cvlist}

  \cvsection{Технические навыки}
      Python, Golang, ...
  ```
  One level = 4 spaces (existing files use spaces, not tab characters).
- The resume must stay on one page; check page count after content changes.
