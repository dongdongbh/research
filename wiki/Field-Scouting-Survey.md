# Field Scouting Survey — Beyond Compositional VLMs

Status: **In progress 2026-07-25.** Scientific-ML and agentic/world-model legs
complete. Representation-geometry, emerging-topics, and math-venue legs still
running.

Written after [[Calibration-Prior-Art-Gate]] closed the last candidate in the
compositional-VLM space. Constraints applied as filters throughout: **one
person, 4-16 GPUs (L40S/A100/H100/H200), no pretraining budget,
pre-registration-friendly, comparative advantage in controls and falsification.**

## The organising observation

Across every area surveyed so far, the papers that survived scrutiny share one
method: **a compute-matched or nuisance-matched control arm plus reported
run-to-run variance.** The scarce resource in 2026 is not benchmarks and not
models. It is controls. That is exactly the capability this group has spent
six months building.

The recurring quantitative signature of an opportunity is the
**method:critique ratio**. Where it runs 100:1 or worse, the critique side is
empty for lack of people, not lack of merit.

| Area | Method papers (2025) | Critique/audit papers |
|---|---:|---:|
| Physics-informed neural networks | 6,237 | ~10 |
| Neural operators | 5,691 | ~10 |
| World-model benchmarks ("we introduce") | 295 | **1** metric-validity paper |
| Neural combinatorial optimization | 524 | ~10 (and now well-covered) |
| LLM automatic heuristic design | 82+ | **1** (a workshop suite) |

## Ranked opportunities so far

### 1. World-model evaluation — highest leverage found

The single most striking number in the survey: **295** arXiv abstracts match
`"world model" AND "benchmark" AND "we introduce"`; **1** matches world model
plus meta-evaluation or metric validity. Roughly three hundred groups have
built a world-model benchmark and essentially nobody has asked whether any of
them agree with each other or predict anything downstream.

Industry owns the models completely (NVIDIA Cosmos, DeepMind Genie, Tencent,
Skywork, plus a dense 2026 wave). **That is precisely why the measurement layer
is vacant** — nobody holding a frontier world model is incentivised to publish
that the metrics don't work. Evaluation needs inference on released
checkpoints only, which is the right size for 4-16 GPUs.

Two existing position papers (`2601.15533` naming "visual conflation";
`2606.15032` naming a "claim/evidence mismatch") **assert** the problem without
measuring it. Converting an asserted position into evidence is a clean,
citable contribution.

Candidate pre-registrations: cross-benchmark convergent validity (registered
hypothesis: median pairwise Kendall tau below 0.5); nuisance-robustness of
physical-plausibility metrics under perceptually null re-encoding (bitrate,
fps, resolution); whether benchmark score predicts downstream planning utility.

Risk is **venue fit, not scoop risk** — CVPR/ICCV reward new benchmarks over
meta-evaluation. Target the World Model workshops, TMLR, or D&B tracks.

### 2. Weather/climate ML — a live contradiction, near-zero compute

Two 2026 papers using **the same WeatherBench 2 data** reach opposite
conclusions:

- *Physics-based models outperform AI weather forecasts of record-breaking
  extremes* (**Science Advances 2026**, `2508.15724`): HRES beats GraphCast,
  Pangu, FuXi on record-breaking events at nearly all lead times.
- *Towards Fair Comparisons... via the Weighted Potential CRPS* (`2606.21170`,
  Gneiting group): after isotonic-distributional-regression post-processing and
  weighted CRPS, **FuXi gives the most informative extreme-event forecasts**.

Both cannot be the correct summary. The difference is a methodological choice —
raw deterministic output versus calibrated probabilistic scoring — and nobody
has adjudicated it. A pre-registered factorial over {scoring rule} ×
{post-processing} × {reference} × {event definition} × {model}, with the
registered outcome being *which factor flips the sign of the conclusion*, is a
real scientific contribution.

**Compute is essentially zero**: WeatherBench 2 forecasts are precomputed public
Zarr on Google Cloud. This adjudicates claims that cost 10^5-10^6 GPU-hours to
produce — the best verify-cost-to-claim-cost ratio found anywhere.

Two companions: verification-target contamination (everything is scored against
ERA5, which is itself a model product, and `2601.04701` shows GraphCast can
detect ERA5's own errors — so re-score against independent station
observations); and initial-condition fairness (AI models initialise from ERA5,
operational NWP from real-time analysis).

Risk: competing with Gneiting's group and ECMWF, who are strong and fast.
Budget ~3 weeks to learn proper scoring rules and the double-penalty problem.

### 3. LLM automatic heuristic design — near-empty critique space

The FunSearch / EoH / ReEvo / AlphaEvolve lineage claims LLMs discover
superhuman heuristics. Targeted critique queries return **1** result (a GECCO
workshop suite); `"evolutionary" AND "LLM" AND "heuristic" AND "seed" AND
"variance"` returns **0**.

Enormous claims, no independent audit, and verification costs CPU-hours plus a
local open-weight model. Candidate controls, none of which anyone has run:
seed-variance and best-of-N inflation (does the published number sit above the
90th percentile of the seed distribution?); harness ablation (replace the LLM
proposer with a random program mutator over the same skeleton — if that
recovers the gain, the claim collapses from "LLM discovery" to "the harness did
the work"); budget-normalised comparison against handing the same wall-clock and
dollars to `irace`/SMAC tuning LKH-3 or HGS.

**Pinning an open-weight proposer with provenance hashing is itself a
methodological contribution here**, since every current AHD paper is
non-reproducible against vendor model drift.

### 4. PDE foundation models — the control nobody ran

`abs:"pretraining" AND abs:"physics" AND abs:"compute-matched"` returns
**literally 0** arXiv papers. Poseidon (NeurIPS 2024) and successors claim large
sample-efficiency gains from multi-physics pretraining across 15 downstream
tasks; nobody has run the arms that would test it — random init at equal total
FLOPs, pretraining on physics-scrambled corpora, pretraining on Gaussian random
fields with matched spectral statistics.

Also unclaimed: **break-even accounting**. Every surrogate paper reports
inference speedup; almost none counts solver cost to generate training data,
training cost, or accuracy-matched solver comparison.
`abs:"training data generation" AND abs:"cost" AND abs:"amortiz"` → **0**.

This field has already institutionally endorsed the methodology: the *Common
Task Framework for a Critical Evaluation of Scientific ML* (**NeurIPS D&B
2025**, `2510.23166`) names weak baselines and inconsistent evaluation as the
problem. Cite it as licence.

### 5. MLIP / materials — best verify-to-claim ratio, needs domain help

`abs:"Matbench Discovery" AND (abs:"leakage" OR abs:"compliance")` → **0**,
despite MLIP Arena naming leakage as a headline unresolved problem. Modern
universal MLIPs train on corpora that overlap the WBM test set structurally,
not just by formula. A leakage-stratified re-analysis is **inference-only over
257k structures, hours on a few GPUs, against models costing 10^5 GPU-hours**.

Caveat: anything touching DFT/XRD verification of discovery claims needs a
chemistry collaborator. Leakage stratification and energy-conservation
diagnostics are solo-doable.

## Areas to avoid, with reasons

| Area | Why |
|---|---|
| **Agent evaluation** | Genuinely broken but overrun: 370 agentic-benchmark abstracts, 10+ dedicated workshops in one cycle, a 27-author Princeton-led consortium running 21,730 rollouts, two competing "harness engineering" surveys, and a paper *auditing the auditors*. Also: SWE-Bench+ was withdrawn from ICLR 2026 and rejected from ICLR 2025 |
| **AI for math / theorem proving** | Saturated (`abs:"miniF2F"` → 107), no GPU advantage since verification is token-bound, and the obvious audit play was just published (*Faults in Our Formal Benchmarking*, ICML 2026, 4,833 findings) |
| **Chip placement** | Litigated to death and reputationally radioactive; Nature issued an Addendum in 2024 |
| **PINNs vs FEM** | Settled by Grossmann et al. 2024 |
| **Neural CO vs classical solvers** | Settled by MaxCut-Bench and FrontierCO (ICLR 2026) |
| **Multimodal benchmark blind-solvability** | The headline is a 2024 result and now table stakes. Only the nuisance-robustness flank (opened ~7 months ago) remains |

## Venues receptive to this work

**Tier A.** TMLR — scope *explicitly invites* reproducibility studies, accepts
on evidence quality rather than significance, rolling deadlines, and has a
Reproducibility Certification. NeurIPS D&B Track. ICML/ICLR main track do take
critique papers and give them orals (*The dark side of the forces*, ICML 2025
Oral; FrontierCO, ICLR 2026; *Faults in Our Formal Benchmarking*, ICML 2026) —
do not assume a method is required. ICML Position track for framing papers.

**Tier B, domain venues with higher credibility for scientific claims.** Nature
Machine Intelligence (took McGreivy & Hakim and Matbench Discovery), plus
*Matters Arising* for direct rebuttals; Machine Learning: Science and Technology
(explicitly welcomes negative results); npj Computational Materials; PRX Energy;
npj Climate and Atmospheric Science; AIES (AMS); Science Advances.

**Tier C.** ICBINB ("I Can't Believe It's Not Better") at NeurIPS, purpose-built
for negative results; ML4PS; AI4Mat.

## Open legs

Representation geometry (including the GeoLAN paper), emerging-topic scouting
across 2025-26 best-paper and new-workshop lists, and recent math-venue results
applicable to ML are still being surveyed. This page will be extended.

## Related

[[Calibration-Prior-Art-Gate]] — closed the last compositional-VLM candidate.
[[Next-Direction-Literature-Survey]] — the earlier, narrower survey.
