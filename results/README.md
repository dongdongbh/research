# Results

Shared, cross-project results and analysis artifacts live here in the repo
proper (the wiki is prose-only; anything tabular, scripted, or re-runnable
belongs here).

Conventions (following the svib repo's practice):

- One subdirectory per study, named `<topic>_<yyyymm>/`, containing the
  result tables (`.csv`/`.json`), the exact command or script that produced
  them, and a short `README.md` stating protocol, seeds, and provenance.
- Pre-registrations go in the study directory *before* the run, and the wiki
  page links to them.
- Large binaries (checkpoints, feature caches) do NOT go in git — put them
  under `/anvil/projects/x-cis261253/artifacts/` and record the path +
  checksum in the study README.
