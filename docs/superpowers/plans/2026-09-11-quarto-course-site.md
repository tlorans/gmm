# Course Reproduction Site Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** An empty Quarto book template for reproducing "Lecture Notes for Empirical Finance", published to <https://tlorans.github.io/gmm/> by GitHub Actions.

**Architecture:** A Quarto book at the repo root with one `.qmd` page per chapter and appendix. Python code runs locally through the uv environment and outputs are frozen into `_freeze/`, which is committed. GitHub Actions renders from `_freeze/` without Python and pushes to the `gh-pages` branch.

**Tech Stack:** Quarto 1.9.29, Python 3.14, uv 0.11.7 (`uv_build` backend), GitHub Actions (`actions/checkout@v7`, `quarto-dev/quarto-actions@v2`).

**Spec:** `docs/superpowers/specs/2026-09-11-quarto-course-site-design.md`

## Global Constraints

- Working directory for every command: `C:\DBD\gmm`. Shell: Git Bash.
- Use `uv` for everything Python (`uv add`, `uv run`, `uv sync`). Never `pip`.
- Run Quarto as `uv run quarto ...` so it uses `.venv` Python.
- No body text from the course notes. Pages contain only titles, section headings, and the line `To reproduce.`
- Repo: `https://github.com/tlorans/gmm` (public). Site: `https://tlorans.github.io/gmm/`.
- `_freeze/` is committed. `_book/`, `.quarto/`, `data/raw/` are not.
- End every commit message with:
  ```
  Co-Authored-By: Claude Opus 5 (1M context) <noreply@anthropic.com>
  Claude-Session: https://claude.ai/code/session_01FCC4HmcjVQr6khXhkSuH1t
  ```
- Addition to the spec: `.gitattributes` forces LF line endings. This machine has `core.autocrlf=true`; CRLF files locally and LF files in CI would give different freeze hashes, and CI would then try to run Python and fail.

---

### Task 1: Python package and dependencies

**Files:**
- Modify: `pyproject.toml`
- Create: `src/gmm/__init__.py`
- Create: `.gitattributes`
- Modify: `.gitignore`
- Delete: `main.py`

**Interfaces:**
- Produces: importable package `gmm` with attribute `gmm.__version__ == "0.1.0"`, used by the Python cell in `index.qmd` (Task 2).

- [ ] **Step 1: Run the check to verify it fails**

Run: `uv run python -c "import gmm; print(gmm.__version__)"`
Expected: FAIL with `ModuleNotFoundError: No module named 'gmm'`

- [ ] **Step 2: Create the package**

`src/gmm/__init__.py`:

```python
"""Shared code for reproducing the Lecture Notes for Empirical Finance."""

__version__ = "0.1.0"
```

Append to `pyproject.toml`:

```toml

[build-system]
requires = ["uv_build>=0.11.7,<0.12"]
build-backend = "uv_build"
```

Delete the placeholder: `rm main.py`

- [ ] **Step 3: Add dependencies**

Run: `uv add numpy pandas scipy matplotlib jupyter`
Expected: `pyproject.toml` `dependencies` lists the five packages; `uv.lock` updated; `gmm` installed in editable mode.

- [ ] **Step 4: Run the check to verify it passes**

Run: `uv run python -c "import gmm; print(gmm.__version__)"`
Expected: `0.1.0`

Run: `uv run python -c "import numpy, pandas, scipy, matplotlib, ipykernel; print('ok')"`
Expected: `ok`

- [ ] **Step 5: Line endings and ignores**

`.gitattributes`:

```
* text=auto eol=lf
*.png binary
*.jpg binary
*.pdf binary
```

Append to `.gitignore`:

```

# Quarto
_book/
.quarto/
*.quarto_ipynb*

# Data (licensed or large; see data/README.md)
data/raw/
```

- [ ] **Step 6: Commit**

```bash
git add .gitattributes .gitignore .python-version pyproject.toml uv.lock src/gmm/__init__.py README.md
git commit -m "Set up gmm package and Python dependencies" -m "Co-Authored-By: Claude Opus 5 (1M context) <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01FCC4HmcjVQr6khXhkSuH1t"
git status --short
```

Expected: `git status --short` prints nothing (`main.py` was never tracked, so its deletion is not a change).

---

### Task 2: Quarto book skeleton

**Files:**
- Create: `_quarto.yml`
- Create: `index.qmd`
- Create: `references.qmd`, `references.bib`
- Create: `chapters/01-gmm-mean-variance.qmd` … `chapters/11-finite-difference.qmd` (11 files)
- Create: `appendices/a-rational-expectations.qmd` … `appendices/d-econometrics.qmd` (4 files)
- Create: `data/README.md`
- Modify: `README.md`
- Create (generated): `_freeze/`

**Interfaces:**
- Consumes: `gmm.__version__` from Task 1.
- Produces: a renderable book in `_book/`, and `_freeze/index/` holding the frozen output of the `index.qmd` Python cell. Task 3 publishes from these.

- [ ] **Step 1: Run the check to verify it fails**

Run: `uv run quarto render`
Expected: FAIL (no Quarto project; Quarto reports it cannot find anything to render, or exits non-zero).

- [ ] **Step 2: Write `_quarto.yml`**

```yaml
project:
  type: book
  output-dir: _book
  render:
    - index.qmd
    - chapters/*.qmd
    - appendices/*.qmd
    - references.qmd

book:
  title: "Empirical Finance, Reproduced"
  author: "Thomas Lorans"
  site-url: https://tlorans.github.io/gmm/
  repo-url: https://github.com/tlorans/gmm
  repo-actions: [source]
  chapters:
    - index.qmd
    - chapters/01-gmm-mean-variance.qmd
    - chapters/02-predicting-returns.qmd
    - chapters/03-linear-factor-models.qmd
    - chapters/04-sdf-vs-beta.qmd
    - chapters/05-weighting-matrix.qmd
    - chapters/06-consumption-based.qmd
    - chapters/07-arch-garch.qmd
    - chapters/08-garch-applications.qmd
    - chapters/09-short-rates.qmd
    - chapters/10-kernel-estimation.qmd
    - chapters/11-finite-difference.qmd
    - references.qmd
  appendices:
    - appendices/a-rational-expectations.qmd
    - appendices/b-gmm-coding.qmd
    - appendices/c-data.qmd
    - appendices/d-econometrics.qmd

bibliography: references.bib

format:
  html:
    theme: cosmo
    html-math-method: mathjax
    number-sections: true

execute:
  freeze: auto
```

- [ ] **Step 3: Write `index.qmd`**

````markdown
# Preface {.unnumbered}

This site is my chapter-by-chapter reproduction of the *Lecture Notes for Empirical
Finance* [@empirical-finance-notes]. Each page follows the structure of the notes and
holds my own derivations, code, and results. Read the original notes
[here](http://efinance.org.cn/cn/AP/lecture%20notes%20for%20empirical%20finance.pdf).

## How to build {.unnumbered}

```bash
uv sync                 # install Python and the dependencies
uv run quarto preview   # live preview while writing
uv run quarto render    # full build into _book/
```

Code runs only on my machine. Quarto saves the outputs in `_freeze/`, which is
committed, so GitHub Actions can publish the site without Python or data.

## Conventions {.unnumbered}

Label equations so other pages can refer to them. For example, a GMM estimator
minimizes a quadratic form in the sample moments:

$$
\hat\theta = \arg\min_\theta \; g_T(\theta)' \, W_T \, g_T(\theta),
\qquad
g_T(\theta) = \frac{1}{T} \sum_{t=1}^{T} f(x_t, \theta).
$$ {#eq-gmm}

A reference to @eq-gmm becomes a link. Shared code lives in `src/gmm/` and is imported
like any package:

```{python}
import gmm

print(gmm.__version__)
```
````

- [ ] **Step 4: Write `references.qmd` and `references.bib`**

`references.qmd`:

```markdown
# Reading list {.unnumbered}

<!-- Topic groups from the notes, to fill with citations:
Background; GMM; Predictability of Asset Returns; Linear Factor Models;
Consumption-Based Asset Pricing; Models of Changing Volatility;
Interest Rate Models; Testing Distributions and Kernel Regressions -->

::: {#refs}
:::
```

`references.bib`:

```bibtex
@misc{empirical-finance-notes,
  title = {Lecture Notes for Empirical Finance},
  url   = {http://efinance.org.cn/cn/AP/lecture%20notes%20for%20empirical%20finance.pdf},
  note  = {Course lecture notes}
}
```

- [ ] **Step 5: Write the 11 chapter pages**

Each file is its `#` title, then one `##` heading per section, each followed by a blank
line, `To reproduce.`, and a blank line. Exact contents:

`chapters/01-gmm-mean-variance.qmd`:

```markdown
# GMM Estimation of Mean-Variance Frontier

## GMM Estimation of Means and Covariance Matrices

To reproduce.

## Asymptotic Distribution of the GMM Estimators

To reproduce.

## Mean-Variance Frontier

To reproduce.
```

`chapters/02-predicting-returns.qmd`:

```markdown
# Predicting Asset Returns

## A Little Financial Theory and Predictability

To reproduce.

## Empirical U.S. Evidence on Stock Return Predictability

To reproduce.

## Prices, Returns, and Predictability

To reproduce.

## Autocorrelations

To reproduce.

## Other Predictors

To reproduce.

## Trading Strategies

To reproduce.

## Maximally Predictable Portfolio

To reproduce.
```

`chapters/03-linear-factor-models.qmd`:

```markdown
# Linear Factor Models

## Testing CAPM (Single Excess Return Factor)

To reproduce.

## Testing Multi-Factor Models (Factors are Excess Returns)

To reproduce.

## Testing Multi-Factor Models (General Factors)

To reproduce.

## Fama-MacBeth

To reproduce.
```

`chapters/04-sdf-vs-beta.qmd`:

```markdown
# Linear Factor Models: SDF vs Beta Methods

## Linear SDF $\Leftrightarrow$ Linear Factor Model

To reproduce.

## Estimating Explicit SDF Models

To reproduce.

## SDF Models versus Linear Factor Models Again

To reproduce.

## Conditional SDF Models

To reproduce.
```

`chapters/05-weighting-matrix.qmd`:

```markdown
# Weighting Matrix in GMM

## Arguments for a Prespecified Weighting Matrix

To reproduce.

## Near Singularity of the Weighting Matrix

To reproduce.

## Very Different Variances

To reproduce.

## $(E x_t x_t')^{-1}$ as Weighting Matrix

To reproduce.
```

`chapters/06-consumption-based.qmd`:

```markdown
# Consumption-Based Asset Pricing

## Introduction

To reproduce.

## Problems with the Consumption-Based Asset Pricing Model

To reproduce.

## Assets in Simulation Models

To reproduce.

## Summary

To reproduce.
```

`chapters/07-arch-garch.qmd`:

```markdown
# ARCH and GARCH

## Test of ARCH Effects

To reproduce.

## ARCH Models

To reproduce.

## GARCH Models

To reproduce.

## Non-Linear Extensions

To reproduce.

## (G)ARCH-M

To reproduce.

## Multivariate (G)ARCH

To reproduce.
```

`chapters/08-garch-applications.qmd`:

```markdown
# Financial Applications of ARCH and GARCH Models

## Bansal and Lundblad, "Fundamental Values and Asset Returns in Global Equity Markets"

To reproduce.

## Heston and Nandi, "A Closed-Form GARCH Option Valuation Model"

To reproduce.
```

`chapters/09-short-rates.qmd`:

```markdown
# Models of Short Interest Rates

## SDF and Yield Curve Models

To reproduce.

## Chan et al. (1992), "An Empirical Comparison of Alternative Models of the Short-Term Interest Rate"

To reproduce.
```

`chapters/10-kernel-estimation.qmd`:

```markdown
# Kernel Density Estimation and Regression

## Non-Parametric Regression

To reproduce.

## Estimating and Testing Distributions

To reproduce.

## Aït-Sahalia (1996), "Testing Continuous-Time Models of the Spot Interest Rate"

To reproduce.

## Aït-Sahalia and Lo (1998), "Nonparametric Estimation of State-Price Densities Implicit in Financial Asset Prices"

To reproduce.
```

`chapters/11-finite-difference.qmd`:

```markdown
# Finite-Difference Solution of Option Prices

## Black-Scholes

To reproduce.

## Finite-Difference Methods

To reproduce.

## Early Exercise

To reproduce.
```

- [ ] **Step 6: Write the 4 appendix pages**

`appendices/a-rational-expectations.qmd`:

```markdown
# Testing Rational Expectations

To reproduce.
```

`appendices/b-gmm-coding.qmd`:

```markdown
# Coding the GMM Problem in Section 3.2

## Exactly Identified System

To reproduce.

## Overidentified System

To reproduce.
```

`appendices/c-data.qmd`:

```markdown
# Data

The datasets used on this site, with their sources and licenses, are listed in
[`data/README.md`](https://github.com/tlorans/gmm/blob/main/data/README.md).

To reproduce.
```

`appendices/d-econometrics.qmd`:

```markdown
# Econometrics

To reproduce.
```

- [ ] **Step 7: Write `data/README.md` and `README.md`**

`data/README.md`:

```markdown
# Data

Raw files go in `data/raw/`, which git ignores. Record every dataset here so each
result on the site can be rebuilt. Do not publish licensed data (for example CRSP or
other WRDS data) in rendered tables.

| Dataset | Source | License | Used in |
|---|---|---|---|
```

`README.md` (currently empty):

````markdown
# Empirical Finance, Reproduced

A chapter-by-chapter reproduction of the *Lecture Notes for Empirical Finance*,
published at <https://tlorans.github.io/gmm/>.

## Build

```bash
uv sync
uv run quarto preview
```

Code runs locally and its output is frozen in `_freeze/`. Commit `_freeze/` with your
changes; GitHub Actions publishes the site on every push to `main`.
````

- [ ] **Step 8: Render and verify**

Run: `uv run quarto render`
Expected: exits 0; output ends with `Output created: _book\index.html`. No `WARNING` lines about unresolved citations or cross-references.

Run:

```bash
ls _book/index.html _book/references.html _book/chapters/*.html _book/appendices/*.html | wc -l
grep -c "0.1.0" _book/index.html
grep -c "?@eq-gmm" _book/index.html
grep -c "Lecture Notes for Empirical Finance" _book/references.html
ls _freeze/index/execute-results/
```

Expected, in order: `17`; at least `1` (the printed version); `0` (the equation reference resolved); at least `1` (the bibliography entry rendered); `html.json`.

If `references.html` shows the URL broken at `%`, change the `url` field to escape each `%` as `\%`, re-render, and re-run the check.

- [ ] **Step 9: Commit**

```bash
git add _quarto.yml index.qmd references.qmd references.bib chapters appendices data/README.md README.md _freeze
git commit -m "Add Quarto book skeleton for the course" -m "Co-Authored-By: Claude Opus 5 (1M context) <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01FCC4HmcjVQr6khXhkSuH1t"
git status --short
```

Expected: `git status --short` prints nothing.

---

### Task 3: Publish to GitHub Pages

**Files:**
- Create: `.github/workflows/publish.yml`
- Create (generated): `_publish.yml`

**Interfaces:**
- Consumes: the committed book and `_freeze/` from Task 2.
- Produces: `gh-pages` branch, Pages enabled, live site at `https://tlorans.github.io/gmm/`.

Order matters: push `main` before anything else, so GitHub makes `main` the default branch. Add the workflow only after `gh-pages` exists, so its first run does not fail.

- [ ] **Step 1: Verify the site is not live**

Run: `curl -s -o /dev/null -w "%{http_code}\n" https://tlorans.github.io/gmm/`
Expected: `404`

- [ ] **Step 2: Push `main`**

Run: `git push -u origin main`
Then: `gh repo view tlorans/gmm --json defaultBranchRef --jq .defaultBranchRef.name`
Expected: `main`

- [ ] **Step 3: First publish from this machine**

Run: `uv run quarto publish gh-pages --no-prompt`
Expected: renders, pushes the `gh-pages` branch, creates `_publish.yml`.

Run: `git ls-remote --heads origin gh-pages`
Expected: one line ending in `refs/heads/gh-pages`.

- [ ] **Step 4: Make sure Pages is enabled**

Run: `gh api repos/tlorans/gmm/pages --jq '.source'`
Expected: `{"branch":"gh-pages","path":"/"}`.

If it returns `404 Not Found` instead, run:
`gh api -X POST repos/tlorans/gmm/pages -f "source[branch]=gh-pages" -f "source[path]=/"`
and repeat the check.

- [ ] **Step 5: Add the workflow**

`.github/workflows/publish.yml`:

```yaml
name: Publish site

on:
  push:
    branches: [main]
  workflow_dispatch:

permissions:
  contents: write

jobs:
  publish:
    runs-on: ubuntu-latest
    steps:
      - name: Check out repository
        uses: actions/checkout@v7

      - name: Set up Quarto
        uses: quarto-dev/quarto-actions/setup@v2
        with:
          version: 1.9.29

      - name: Render and publish to gh-pages
        uses: quarto-dev/quarto-actions/publish@v2
        with:
          target: gh-pages
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

- [ ] **Step 6: Commit and push**

```bash
git add .github/workflows/publish.yml _publish.yml
git commit -m "Publish the site with GitHub Actions" -m "Co-Authored-By: Claude Opus 5 (1M context) <noreply@anthropic.com>
Claude-Session: https://claude.ai/code/session_01FCC4HmcjVQr6khXhkSuH1t"
git push
```

- [ ] **Step 7: Verify the workflow run**

Run: `gh run watch --repo tlorans/gmm --exit-status $(gh run list --repo tlorans/gmm --workflow publish.yml --limit 1 --json databaseId --jq '.[0].databaseId')`
Expected: exits 0, all steps green.

If the render step fails trying to start Jupyter, the freeze hash did not match: run
`uv run quarto render`, check `git status` for changes in `_freeze/`, commit them, and push again.

- [ ] **Step 8: Verify the live site**

Pages can take a minute or two after the run. Run:

```bash
curl -s -o /dev/null -w "%{http_code}\n" https://tlorans.github.io/gmm/
curl -s https://tlorans.github.io/gmm/ | grep -c "Empirical Finance, Reproduced"
curl -s -o /dev/null -w "%{http_code}\n" https://tlorans.github.io/gmm/chapters/03-linear-factor-models.html
```

Expected: `200`; at least `1`; `200`.
