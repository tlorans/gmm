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
