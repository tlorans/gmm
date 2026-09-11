# Empirical Finance, Reproduced

A chapter-by-chapter reproduction of the *Lecture Notes for Empirical Finance*,
published at <https://tlorans.github.io/gmm/>.

## Prerequisites

- [uv](https://docs.astral.sh/uv/)
- [Quarto](https://quarto.org/) 1.9.29 or later, installed separately (uv does not install it)

## Build

```bash
uv sync                 # install Python and the dependencies
uv run quarto preview   # live preview while writing
uv run quarto render    # full build into _book/; run before committing
```

Raw data goes in `data/raw/`, which git ignores. Record each dataset's source and
license in `data/README.md`.

## Frozen code output

Code runs only on this machine. Quarto saves each page's output in `_freeze/`, and
GitHub Actions publishes from it without Python or data. Commit `_freeze/` together
with your changes; every push to `main` republishes the site.

Quarto re-runs a page only when that page's own text changes. Editing shared code in
`src/gmm/` or a file in `data/raw/` does not re-run the pages that use it. After such a
change, render each affected page on its own, which always runs its code, then commit
`_freeze/`:

```bash
uv run quarto render chapters/03-linear-factor-models.qmd
```
