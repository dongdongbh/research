# research

General research repo for the lab: cross-project direction surveys, gates,
cluster setup docs, and shared results.

- `wiki/` — the knowledge base. Auto-syncs to the GitHub Wiki on push via
  `.github/workflows/wiki-sync.yml`. Start at `wiki/Home.md`.
- `results/` — shared result tables and analysis code (see its README for
  conventions). Large binaries stay on the cluster under
  `/anvil/projects/x-cis261253/artifacts/`.

Project-specific work lives in its own repo: `svib` (NeurIPS 2026 paper +
evidence base), `svib2` (distillation line). Their general wiki pages were
seeded into this repo on 2026-08-02 and are canonical here from now on.

Note: this repo is separate from `research-wiki` (the CollabMD space shared
with the professor), which carries short plain-language summaries; this repo
holds the full-detail record and syncs to GitHub.
