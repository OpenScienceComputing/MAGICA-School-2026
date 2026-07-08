# MAGICA School 2026 — Agent Instructions

## General

- Always strip notebook outputs before committing (`jupyter nbconvert --ClearOutputPreprocessor.enabled=True --inplace`).

## Python environment

- Use the existing `protocoast-notebook` conda environment to run Python — it already has all the libraries these notebooks and scripts need. Do not create a new environment or `pip install` packages unless explicitly asked.
- Run scripts with `conda run -n protocoast-notebook python <script>`.
- For interactive notebook work — editing cells and inspecting real outputs — connect to the `protocoast-notebook` Jupyter kernel.
- To just verify a notebook runs top-to-bottom (no interactive kernel needed), execute it to a scratch copy so the tracked file isn't touched: `conda run -n protocoast-notebook jupyter nbconvert --execute --to notebook --output-dir /tmp <notebook>`.
