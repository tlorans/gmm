# Course reproduction site: design

Date: 2026-09-11
Status: approved in chat, pending spec review

## Goal

A GitHub Pages site where I reproduce, chapter by chapter, the course
"Lecture Notes for Empirical Finance"
(<http://efinance.org.cn/cn/AP/lecture%20notes%20for%20empirical%20finance.pdf>).
This task delivers the empty template only: structure, build, and publishing.
The notes' text is not copied; every page holds my own derivations and code.

## Decisions

| Topic | Decision |
|---|---|
| Tool | Quarto book (Quarto 1.9.29, already installed) |
| Language | Python, managed by uv in this project |
| Execution | Local only, `execute: freeze: auto`; `_freeze/` is committed |
| Publishing | GitHub Actions renders from `_freeze/` and pushes to `gh-pages` |
| Repo | `tlorans/gmm`, public (switched 2026-09-11) |
| Site URL | <https://tlorans.github.io/gmm/> |

## File layout

```
gmm/
├── _quarto.yml
├── index.qmd
├── chapters/01-gmm-mean-variance.qmd … 11-finite-difference.qmd
├── appendices/a-rational-expectations.qmd … d-econometrics.qmd
├── references.qmd
├── references.bib
├── src/gmm/__init__.py
├── data/README.md
├── .github/workflows/publish.yml
├── .gitignore
├── pyproject.toml
└── docs/superpowers/specs/   (not part of the site)
```

`main.py` is deleted.

## `_quarto.yml`

- `project.type: book`, `output-dir: _book`.
- `project.render` lists the book files explicitly (`index.qmd`, `chapters/*.qmd`,
  `appendices/*.qmd`, `references.qmd`) so `docs/`, `data/` and `README.md` are never rendered.
- `book.chapters`: `index.qmd`, the 11 chapter files in order, `references.qmd`.
- `book.appendices`: the 4 appendix files, lettered A–D by Quarto.
- `book.repo-url` pointing at the GitHub repo.
- `bibliography: references.bib`.
- `format.html`: default theme, `html-math-method: mathjax`, `number-sections: true`.
- `execute.freeze: auto`.

## Chapter and appendix pages

Each page contains a YAML title and the section headings from the notes' table of
contents (Quarto numbers them). Under each heading there is one placeholder line
(`To reproduce.`). No body text from the notes.

| File | Title | Sections |
|---|---|---|
| 01 | GMM Estimation of Mean-Variance Frontier | GMM Estimation of Means and Covariance Matrices; Asymptotic Distribution of the GMM Estimators; Mean-Variance Frontier |
| 02 | Predicting Asset Returns | A Little Financial Theory and Predictability; Empirical U.S. Evidence on Stock Return Predictability; Prices, Returns, and Predictability; Autocorrelations; Other Predictors; Trading Strategies; Maximally Predictable Portfolio |
| 03 | Linear Factor Models | Testing CAPM (Single Excess Return Factor); Testing Multi-Factor Models (Factors are Excess Returns); Testing Multi-Factor Models (General Factors); Fama-MacBeth |
| 04 | Linear Factor Models: SDF vs Beta Methods | Linear SDF ⇔ Linear Factor Model; Estimating Explicit SDF Models; SDF Models versus Linear Factor Models Again; Conditional SDF Models |
| 05 | Weighting Matrix in GMM | Arguments for a Prespecified Weighting Matrix; Near Singularity of the Weighting Matrix; Very Different Variances; $(E x_t x_t')^{-1}$ as Weighting Matrix |
| 06 | Consumption-Based Asset Pricing | Introduction; Problems with the Consumption-Based Asset Pricing Model; Assets in Simulation Models; Summary |
| 07 | ARCH and GARCH | Test of ARCH Effects; ARCH Models; GARCH Models; Non-Linear Extensions; (G)ARCH-M; Multivariate (G)ARCH |
| 08 | Financial Applications of ARCH and GARCH Models | Bansal and Lundblad, "Fundamental Values and Asset Returns in Global Equity Markets"; Heston and Nandi, "A Closed-Form GARCH Option Valuation Model" |
| 09 | Models of Short Interest Rates | SDF and Yield Curve Models; Chan et al. (1992), "An Empirical Comparison of Alternative Models of the Short-Term Interest Rate" |
| 10 | Kernel Density Estimation and Regression | Non-Parametric Regression; Estimating and Testing Distributions; Aït-Sahalia (1996), "Testing Continuous-Time Models of the Spot Interest Rate"; Aït-Sahalia and Lo (1998), "Nonparametric Estimation of State-Price Densities Implicit in Financial Asset Prices" |
| 11 | Finite-Difference Solution of Option Prices | Black-Scholes; Finite-Difference Methods; Early Exercise |
| A | Testing Rational Expectations | (none; appendix to chapter 2 in the notes) |
| B | Coding the GMM Problem in Section 3.2 | Exactly Identified System; Overidentified System |
| C | Data | (none; links to `data/README.md` on GitHub) |
| D | Econometrics | (none) |

Chapter 11 is numbered 21 in the notes. Appendices A and B are chapter appendices in
the notes (chapters 2 and 3); C and D follow chapter 6.

`references.qmd` has the heading "Reading list", a `::: {#refs}` block, and the topic
groups from the notes as a comment for later: Background, GMM, Predictability of Asset
Returns, Linear Factor Models, Consumption-Based Asset Pricing, Models of Changing
Volatility, Interest Rate Models, Testing Distributions and Kernel Regressions.
`references.bib` starts with one entry: the course notes themselves, cited in `index.qmd`.

## `index.qmd` (preface)

1. What the site is, with a link to the original notes.
2. How to build: `uv sync`, `uv run quarto preview`, `uv run quarto render`.
3. Conventions demo, written from scratch and short:
   a labeled equation with a cross-reference to it, and a Python cell that runs
   `import gmm` and prints `gmm.__version__`, which proves the uv environment is used.

## Python

- `pyproject.toml` gains a build system (`uv_build`) with the package in `src/gmm/`.
- `src/gmm/__init__.py` holds a docstring and `__version__ = "0.1.0"`, nothing else.
- Dependencies added with `uv add`: `numpy`, `pandas`, `scipy`, `matplotlib`, `jupyter`.

## Data

- `data/raw/` is gitignored. `data/README.md` has a table (dataset, source, license,
  used in chapter) with no rows filled in yet.
- Only rendered outputs are published. Licensed data must not appear in printed tables.

## `.gitignore`

Existing uv entries, plus `_book/`, `.quarto/`, `data/raw/`, `*.quarto_ipynb*`.
`_freeze/` is committed.

## Publishing

`.github/workflows/publish.yml`:

- Triggers: push to `main`, and `workflow_dispatch`.
- `permissions: contents: write`.
- Steps: `actions/checkout`, `quarto-dev/quarto-actions/setup` (Quarto 1.9.29),
  `quarto-dev/quarto-actions/publish` with `target: gh-pages`.
- No Python in CI. A page with unfrozen code fails the build instead of publishing.

One-time steps on my machine, after the first commit:

1. Push `main`.
2. `uv run quarto publish gh-pages --no-prompt`. This creates the `gh-pages` branch and
   `_publish.yml`.
3. Check Pages with `gh api repos/tlorans/gmm/pages`. If it is not enabled, set the source:
   `gh api -X POST repos/tlorans/gmm/pages -f "source[branch]=gh-pages" -f "source[path]=/"`.
4. Commit `_publish.yml` and push, so the workflow runs.

## Verification

- `uv run quarto render` exits 0.
- `_book/` has pages for the preface, 11 chapters, 4 appendices, and the reading list.
- The preface shows the printed `gmm.__version__` and a working equation cross-reference.
- The GitHub Actions run succeeds.
- <https://tlorans.github.io/gmm/> returns HTTP 200 and shows the book.

## Out of scope

Any chapter content, datasets, the GMM estimator itself, tests for `src/gmm`, custom theme.
