# Direction Re-evaluation — 2026-08-02

> **SUPERSEDED 2026-08-03.** The star ranking below was replaced by
> [[Unified-Direction-Ranking-2026-08]] after a full 8-lane gating sweep of
> the researcher-scan candidates. Biggest changes: T1 demoted ★★★★½ → ★★★
> (three of its four cells published at CVPR/ICLR/ICML 2026 — the flagged
> scoop risk was the wrong group); safety-aware KV allocation dead (scooped
> in June, before this page's gate ran); three new flagships identified
> (autoresearch FDR ★★★★★, judge-audit program ★★★★½, parallel-RL factorial
> ★★★★½). This page is kept as the record of the Aug 2 re-gate.


Updated 2026-08-02 for the general research wiki. **This page is the current
authority on the direction star ranking**; older star tables elsewhere in this
wiki are superseded by it.

Status: **Complete.** Eight parallel deep-search agents (Opus), one per
direction, each instructed to judge by **remaining opportunity** rather than
crowd count, per the PI's corrected criterion: *"a topic can not be filtered
out only by its hot/crowd; a topic is only filtered out if there are fewer
opportunities — the direction is saturated."* The inverse was also enforced:
an empty lane must explain why it is empty before being credited for emptiness.

Tooling note: WebSearch was budget-exhausted and the paper-search script died
on Semantic Scholar 429s for most agents; evidence came from arXiv abs/HTML,
the HF papers API, the Semantic Scholar graph API, and OpenAlex. Coverage is
arXiv-heavy; GECCO/PPSN and other venue-native proceedings were not swept.

## Outcome table

| Direction | Old | New | Movement |
|---|---|---|---|
| T1 freeze × objective × stage | ★★★★★ | **★★★★½** | Confirmed; design revised |
| T3 multimodal mixture crossover | ★★★ | **★★★★½** | Up — saturation was text-only |
| T2 perception audit + estimator | ★★★★ | **★★★★** | Held; reframed to cross-method audit |
| Self-training collapse arbitration | do not enter | **★★★★** | Reversed |
| AHD cost-crossover + novelty audit | ★★ | **★★★★** | Up — expansion outpaces controls ~10:1 |
| KV-cache allocation | ★★★ | **★★★★ narrowed** (safety-aware) / ★★★ as-scoped | Up if narrowed |
| B1 diversity attribution | ★★★★ | **★★★** | Down — scooped 2026-04 |
| T4 anneal-window data | ★★★★★ | **★★** | Down — scooped 3×, moat citable |

SVIB write-up (★★★★☆) was not re-evaluated; unchanged.

## Per-direction key evidence

### T1 — confirmed ★★★★½, HIGH density

- CoVFT ([2603.21077](https://arxiv.org/abs/2603.21077), Mar 2026) abstract: freeze-vs-finetune "remains
  unresolved"; its own benchmark is SFT-only, VFT beats frozen on only 6/12
  benchmarks. Zero citations.
- Frankenstein ([2602.12395](https://arxiv.org/abs/2602.12395)) analyzes what RL improves but never intervenes on
  the vision encoder. New 2026 contradiction: [PIVOT](https://arxiv.org/abs/2510.16333) (RL strengthens the
  encoder) vs Frankenstein (gains concentrate mid-late LLM layers).
- [2605.20177](https://arxiv.org/abs/2605.20177) confounds freeze with stage in its headline staged-training claim.
- No consolidating survey on VLM training recipes exists. 7B band uncontested.
- **Design revision:** objective axis becomes 3-level — SFT / RL / SFT +
  perceptual auxiliary (VIRAL-style) — because PIVOT occupies {unfrozen} ×
  {SFT, DPO}. Best question: does the freeze effect flip sign under a
  non-language-generation objective?
- Scoop path: CoVFT group adds an RL arm. Use their public harness.

### T3 — up to ★★★★½, HIGH density

- Of 8 mixture methods [DataComp-VLM](https://arxiv.org/abs/2606.28551) cites, 7 are text-only; the 8th
  ([2602.04937](https://arxiv.org/abs/2602.04937)) is SFT-stage. DataComp-VLM verbatim: "there exists no
  systematic study on filtering and mixing strategies in the VLM setting."
  Zero surveys in its 347 refs.
- Live contradiction: Shukor [2507.09404](https://arxiv.org/abs/2507.09404) (mixture scaling laws extrapolate,
  validated on native multimodal) vs DataComp-VLM's measured rank inversion
  (caption-heavy wins at 1B×6.25B, instruction-heavy at 2B/4B×25B+). Neither
  cites the other on this.
- DataComp-VLM v1 has 13 unresolved LaTeX labels including the promised
  multi-axis mixture appendix — the analysis is unwritten.
- Public checkpoints at 4 scales convert a ~25,000 H100-h study into ~500.
- Best question: can the small→large mixture-ranking crossover be predicted
  (Shukor-style loss laws, Berasi-style merging proxies) without paying for
  the large runs? Negative result invalidates small-proxy mixture search.
- Watch: Farina co-authors both 2602.04937 and 20/20-adjacent work; the
  consortium runs this as a competition.

### T2 — held ★★★★, reframed

- Original framing partially scooped: [2602.12395](https://arxiv.org/abs/2602.12395) corroborated Perception-R1's
  null mechanistically; [2603.01301](https://arxiv.org/abs/2603.01301) ran the sharpening decomposition (medical).
- Open, unclaimed: ~50 perception-targeted RL methods ([PAPO](https://arxiv.org/abs/2507.06448) 89 citations) vs
  three 2026 diagnostics showing gains survive image masking/corruption
  ([2605.09266](https://arxiv.org/abs/2605.09266), [2604.03179](https://arxiv.org/abs/2604.03179)) — **nobody has run those controls on the methods
  claiming the fix**. PSR estimator flagged as broken on Jul 30 ([2607.28336](https://arxiv.org/abs/2607.28336):
  conflates "perceptual insufficiency with reasoning difficulty").
- Diagnostic sub-lane unowned: diagnosis papers have 3 / 0 / ~0 citations.
- Cross-method audit is inference-only on open weights; control arms are small
  3B–7B GRPO runs. ~256 GPU-h. ICLR-2027-feasible.

### Self-training collapse — reversed to ★★★★

- [2606.21090](https://arxiv.org/abs/2606.21090) (cliff under binary code grader) has 0 citations; never measured
  entropy; conjectures its mechanism and says "we have not tested this";
  hands over leading-indicator prediction with pre-quantified headroom
  (deployed ES 22.2%, hindsight 38–48%).
- Intersection empty on 3 axes: no entropy paper tests sequential campaigns,
  code graders, or ~20-step cliffs (17 papers' settings verified).
- Two collapse regimes have mutually exclusive explanations (reward hacking
  under pseudo-reward, [2505.21444](https://arxiv.org/abs/2505.21444), vs impossible under ground truth) —
  unreconciled. Timescale mismatch: entropy decay is slow (100s–1000s steps);
  the cliff is a phase transition (78% of the drop in one step).
- Field expanding: ~16 named failure modes at monthly cadence, zero
  cross-mechanism arbitration; 1,250-paper RSI survey still finds an
  "underpopulated niche"; Weng (2026-07-04) lists 7 open bottlenecks; Google
  Science One opens verifiability rather than closing anything.
- Design: pre-register 4 mechanisms (entropy exhaustion / winner-take-all /
  recursive space contraction / **format degeneration** — unexamined);
  week 1 validates the testbed (source paper thin: placeholder citations,
  n=3–5, pass@1=1.00 suggests tiny eval set). Both outcomes publish. Must stay
  arbitration-framed; as a method paper the 11 nearby groups win.
- Secondary unoccupied track (near-zero GPU): hypothesis-abandonment /
  honest-negative production in agent loops — adjacent 2026 papers are
  diagnostic-only ([2604.18805](https://arxiv.org/abs/2604.18805): evidence ignored in 68% of traces, proposes
  nothing; [2607.13083](https://arxiv.org/abs/2607.13083): "propose[s] no corrective intervention"). Statistical
  stopping rule vs LLM judge is open, and attacks Science One's own named
  LLM-judge dependency.

### AHD — up to ★★★★

- ~13 control papers in 24 months each closed one sliver ([2605.15221](https://arxiv.org/abs/2605.15221) is circle
  packing n=26 only, "lacks comparison to domain-specific classical
  optimization methods"); ~20 new application domains opened in the same
  window. "Automated heuristic design" abstracts: 1/1/5/**17** (2023/24/25/Jan–
  Jul 26). Claims:controls ≈ 10–15:1.
- The cost-normalized LLM-vs-classical frontier does not exist; three targeted
  searches returned 0 results; only LLaMEA (the proposer) ever ran against
  CMA-ES/DE. DeepMind's own [2602.16928](https://arxiv.org/abs/2602.16928): after distillation "the true driver of
  generalization lies in a minimal algorithmic core."
- Paper shape: (a) cost-crossover frontier (tokens+CPU in one currency; test
  whether landscape ruggedness predicts the crossover; CostAda [2607.26828](https://arxiv.org/abs/2607.26828)
  supplies the cost formalism, BLADE the harness); (b) novelty audit —
  rediscovery/recombination/new grading vs the 19× OOD degradation RAISE
  measured. Zero papers on (b).
- Caveats: van Stein/Bäck and DeepMind adjacent; arXiv-only coverage — check
  GECCO/PPSN before committing.

### KV-cache — ★★★★ only if narrowed to safety-aware allocation

- Kernel objection refuted: [KVTuner (ICML 25)](https://arxiv.org/abs/2502.04420), [EvolKV (EMNLP)](https://arxiv.org/abs/2509.08315), [SCBench
  (ICLR 25)](https://arxiv.org/abs/2412.10319) published with zero kernel work; the actual bar is
  serving-compatible granularity + throughput atop existing kernels.
- Long-horizon slot claimed (CONF-KV [2605.24786](https://arxiv.org/abs/2605.24786)); compounding error remedied
  three ways ([SQuat](https://arxiv.org/abs/2503.24358)/KVarN/[VeriCache](https://arxiv.org/abs/2605.17613)). As-scoped T is gone.
- Unclaimed: no safety-objective allocator exists (6 searches, 0 hits);
  KVFundaBench v2 dropped safety from its abstract (thread abandoned);
  [2510.00231](https://arxiv.org/abs/2510.00231) documents instructions "completely ignored" under compression;
  CAQ ([2511.07842](https://arxiv.org/abs/2511.07842)) proves the objective-mismatch template publishable in
  weight PTQ.
- One falsifiable sweep: safety-optimal vs perplexity-optimal per-layer
  allocation — coincide (cheap death) or diverge (novel map + allocator).
  Pre-register [2605.18053](https://arxiv.org/abs/2605.18053) (protection > scoring) as the control.

### B1 — down to ★★★

- Scooped: [2604.16027](https://arxiv.org/abs/2604.16027) (Karouzos, Tan, Aletras; 2026-04-17) traces [Olmo 3](https://arxiv.org/abs/2512.13961)'s
  three lineages, headline "collapse is embedded in the model weights by
  training data, not imposed by the generation format."
- Remaining: it is observational (lineages differ in data AND algorithm); the
  causal crossover is unrun — but the authors named it as their next step, and
  Apple/CMU ([2605.09995](https://arxiv.org/abs/2605.09995)) added an untested scale axis ("worsens with scale").
- Our compute estimate was low: full SFT/DPO/RLVR crossover + scale arm
  exceeds 400 GPU-h.
- Field-level the area is opportunity-rich (~10 papers/month, no survey), but
  the specific question closed. Would now be adjudication-framed, contested.

### T4 — down to ★★

- Scooped 3×: DiReCT ([2605.31175](https://arxiv.org/abs/2605.31175), May 29) — "effectively selecting training
  data during this phase remains a key challenge... lack a principled
  grounding" is our motivation paragraph, at Llama-3-8B/300B with
  theory + code; QAFSL ([2605.25698](https://arxiv.org/abs/2605.25698)) owns "decay reduces update intensity
  exactly when high-quality data becomes available" with +1.70 over WSD at
  15B-MoE; MIRA ([2605.30288](https://arxiv.org/abs/2605.30288)) owns mid-training-selection-is-distinct.
- **Honest error:** our July sweep declared the lane empty while DiReCT had
  been public for ~8 weeks. The empty-lane claim was false when made.
- Why it looked empty: labs publish recipes not methods (Olmo 3's Dolmino:
  pool sizes, no selector); the small-scale moat is now citable ([2606.07597](https://arxiv.org/abs/2606.07597):
  forked-decay extrapolation "frequently fails" when high-quality data
  repeats — our exact protocol); and the object may dissolve — three
  independent groups ([WSM](https://arxiv.org/abs/2507.17634), WSO, [2604.13627](https://arxiv.org/abs/2604.13627)) converge on less/no decay.
- Venue evidence: 1B-scale *selection* papers publish at ICML/ICLR routinely,
  but the schedule-coupled subgenre has zero top-venue acceptances, and
  [2511.18903](https://arxiv.org/abs/2511.18903) (our baseline) is unpublished across 3 versions in 9 months.
- Rush check (PI-supplied): Compute-Constrained Data Selection (ICLR 25) is
  phase-agnostic fine-tuning-only — leaves the slot formally open but shows
  gradient-class selectors (T4's lever) are almost never compute-optimal. Any
  survivor must include its cost-aware baseline curve.
- Residual question (cheap, unclaimed, mechanism-shaped): does the
  per-document value ranking reorder between stable-phase and decay-phase
  learning rates? Rank-correlation on a shared trunk; discriminates the
  "wasted data" story from the "sharpness" story; null undercuts the papers
  that scooped the lane.

## The meta-lesson, recorded

1. **Crowding predicted nothing.** All four crowd-count downgrades reversed
   (T3, AHD, self-improvement, KV). Both emptiness-credited directions fell
   (T4 scooped, B1 scooped). The criterion that worked: is the question
   surface expanding faster than papers close it, and is the specific question
   claimed?
2. **Method spaces saturate in months; arbitration/diagnostic spaces stay
   empty.** Recurs in five of eight lanes (T1, T2, T3, collapse, AHD, T4's
   residual). Cause: method groups cannot afford nulls on their own
   mechanisms. Our toolkit is exactly the null-tolerant kind.
3. **Recency decays fast.** T4 and B1 were scooped between April and June;
   the July survey was already stale in one lane when written. Any gate older
   than ~6 weeks needs a re-run before compute is committed.
4. Related process rules: filter-by-saturation-not-crowding (memory),
   prior-art gate before experiments, never trust a paper's own novelty claim.

## Related

[[Self-Improving-AI-Survey]] (July verdict superseded for the collapse lane) ·
[[Top-Researcher-Scan-2026-08]] (people-level openings, same date) ·
[[Method-Opportunities]] · [[Live-Research-Opportunities]] ·
[[Status-And-Survivors]] (star table superseded by this page)
