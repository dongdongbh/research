# Direction Re-evaluation — 2026-08-02

> **SUPERSEDED 2026-08-03.** The star ranking on this page was replaced by
> [[Unified-Direction-Ranking-2026-08]] after a full check of all eight
> research areas from the researcher scan. The largest changes were:
>
> - T1, which asks when to freeze or train the vision encoder under different
>   training goals and stages, fell from ★★★★½ to ★★★. Three of its four
>   experiment combinations appeared at CVPR/ICLR/ICML 2026, and we had worried
>   about the wrong group publishing first.
> - The idea of giving KV-cache space to layers based on safety, not only text
>   prediction quality, was stopped. June work had already done it before this
>   gate ran.
> - Three new top ideas appeared: control the false discovery rate (FDR) when
>   automated research systems test many ideas, ★★★★★; audit whether AI judges
>   measure what they claim, ★★★★½; and compare combinations of design choices
>   in parallel reinforcement learning (parallel RL), ★★★★½.
>
> This page remains as the record of the August 2 re-check.

Updated 2026-08-02 for the general research wiki. At that time, **this page was
the current source for the direction star ranking**. Older star tables were
replaced by it. The banner above explains the newer 2026-08-03 replacement.

Status: **Complete.** Eight Opus agents searched deeply in parallel, one for
each direction. They judged ideas by how much useful work remained, not by how
many people were working nearby. This followed the PI's corrected rule:
*"a topic can not be filtered out only by its hot/crowd; a topic is only
filtered out if there are fewer opportunities — the direction is saturated."*
Here, *saturated* means already studied so heavily that little useful work is
left. The reverse rule also applied: before we give credit to an empty research
area, we must explain why it is empty.

Tooling note: the WebSearch budget ran out. The paper-search script also hit
Semantic Scholar 429 rate-limit errors for most agents. Evidence therefore
came from arXiv abstract and HTML pages, the Hugging Face (HF) papers API, the
Semantic Scholar graph API, and OpenAlex. The search covered arXiv well, but it
did not fully check conference-only proceedings such as GECCO and PPSN.

## Results

| Direction | Old | New | Change |
|---|---|---|---|
| T1 freeze × objective × stage | ★★★★★ | **★★★★½** | Confirmed; experiment design changed |
| T3 multimodal mixture crossover | ★★★ | **★★★★½** | Raised; only the text-only area was heavily studied |
| T2 perception audit + estimator | ★★★★ | **★★★★** | Stayed; changed to an audit across methods |
| Self-training collapse careful comparison | do not enter | **★★★★** | Reversed |
| AHD cost crossover + novelty audit | ★★ | **★★★★** | Raised; new claims grow about 10 times faster than controls |
| KV-cache allocation | ★★★ | **★★★★ narrowed** (safety-aware) / ★★★ as first planned | Raised only for the narrower version |
| B1 diversity attribution | ★★★★ | **★★★** | Lowered; already done in 2026-04 |
| T4 anneal-window data | ★★★★★ | **★★** | Lowered; done three times, with a clear reason small-scale work is weak |

The SVIB write-up (★★★★☆) was not checked again. Its score stayed the same.

## Evidence for each direction

### T1 — confirmed ★★★★½, HIGH evidence density

- CoVFT ([2603.21077](https://arxiv.org/abs/2603.21077), March 2026) says the
  freeze-versus-fine-tune question "remains unresolved." Its benchmark uses
  only supervised fine-tuning (SFT-only). Vision fine-tuning (VFT) beats freezing
  on only 6 of 12 benchmarks. The paper had zero citations.
- Frankenstein ([2602.12395](https://arxiv.org/abs/2602.12395)) studies what
  reinforcement learning (RL) improves, but never changes whether the vision
  encoder is frozen. Two 2026 results disagree: [PIVOT](https://arxiv.org/abs/2510.16333)
  says RL strengthens the encoder, while Frankenstein says most gains happen
  in the middle-to-late LLM layers.
- [2605.20177](https://arxiv.org/abs/2605.20177) mixes up two choices—freezing
  and training stage—in its main staged-training claim. This hidden mixing of
  causes is a *confound*.
- There is no survey that combines VLM training recipes. The 7B model-size
  range is still open.
- **Design change:** use three objective choices: SFT / RL / SFT plus a
  perception-focused extra loss, in a VIRAL-style design. This is needed because PIVOT
  already covers the combinations {unfrozen} × {SFT, DPO}. The best question
  is whether freezing changes from helpful to harmful, or vice versa, under
  an objective that is not language generation.
- Main risk: the CoVFT group could add an RL experiment. Use its public test
  harness.

### T3 — raised to ★★★★½, HIGH evidence density

- DataComp-VLM cites eight mixture methods. Seven are text-only. The eighth
  ([2602.04937](https://arxiv.org/abs/2602.04937)) studies the SFT-stage.
  [DataComp-VLM](https://arxiv.org/abs/2606.28551) says: "there exists no
  systematic study on filtering and mixing strategies in the VLM setting."
  Its 347 references contain no survey of this question.
- Two live results disagree. Shukor
  [2507.09404](https://arxiv.org/abs/2507.09404) says mixture scaling laws can
  predict larger runs and tests this on native multimodal data. DataComp-VLM
  sees the ranking reverse: caption-heavy data wins at `1B×6.25B`, but
  instruction-heavy data wins at `2B/4B×25B+`. Neither paper cites the other
  on this conflict.
- DataComp-VLM v1 has 13 broken LaTeX references, including the promised
  appendix about mixtures across several choices. That analysis has not been
  written.
- Public checkpoints at four model scales reduce a study that originally cost
  about 25,000 H100-h to about 500 H100-h.
- Best question: can we predict when the best mixture changes from small to
  large models without paying for the large runs? Possible tools are
  Shukor-style loss laws and Berasi-style model-merging shortcuts. A negative
  result would show that small proxy runs cannot guide mixture search.
- Watch Farina's group. Farina co-authored both 2602.04937 and work near the
  20/20 setting, and the consortium treats this as a competition.

### T2 — stayed ★★★★, with a new framing

- Part of the first idea was already done. [2602.12395](https://arxiv.org/abs/2602.12395)
  supported Perception-R1's null result with a mechanism study.
  [2603.01301](https://arxiv.org/abs/2603.01301) separated the possible
  sharpening effects in medical models.
- The open question compares many methods. About 50 RL methods claim to improve
  perception; [PAPO](https://arxiv.org/abs/2507.06448) alone has 89 citations.
  Yet three 2026 diagnosis papers report that gains remain when images are
  masked or damaged ([2605.09266](https://arxiv.org/abs/2605.09266),
  [2604.03179](https://arxiv.org/abs/2604.03179)). **Nobody has run those
  controls on the methods that claim to fix perception.** On July 30, paper
  [2607.28336](https://arxiv.org/abs/2607.28336) also reported that the PSR
  estimator is broken because it mixes up "perceptual insufficiency with
  reasoning difficulty."
- The diagnosis part is still unclaimed. The diagnosis papers have 3 / 0 / ~0
  citations.
- An audit across methods needs only inference on open model weights. Small
  3B–7B GRPO training runs provide the control groups. Estimated cost is about
  256 GPU-h, which is ICLR-2027-feasible.

### Self-training collapse — reversed to ★★★★

- [2606.21090](https://arxiv.org/abs/2606.21090) finds a sudden collapse under
  a binary code grader. It has zero citations, never measures entropy, guesses
  at the cause, and says "we have not tested this." It leaves an important
  prediction to test. The already-deployed early-warning system (ES) catches 22.2%
  of cases; a hindsight system could catch 38–48%.
- A check of 17 papers found an empty intersection across three choices: no
  entropy paper studies repeated campaigns, code graders, or sudden cliffs
  around 20 steps.
- Two kinds of collapse have explanations that cannot both apply. Under a
  guessed reward, collapse can be reward hacking
  ([2505.21444](https://arxiv.org/abs/2505.21444)). Under correct ground truth,
  that explanation is impossible. Nobody has settled the conflict. The timing
  also differs: entropy usually falls slowly over 100s–1000s of steps, but the
  cliff is a phase change in which 78% of the drop happens in
  one step.
- The field is growing, not closing. It names about 16 failure modes, adding
  more almost every month, but has no study that decides between causes. A
  1,250-paper recursive-self-improvement (RSI) survey still calls this an
  "underpopulated niche." Weng's 2026-07-04 article lists seven open problems.
  Google Science One adds a new verifiability problem instead of solving the
  old ones.
- Design: register four possible causes before running anything—entropy running
  out, winner-take-all behavior, repeated shrinking of the search space, and
  **format degeneration**, which no one has checked. Week 1 must first show
  that the test setup is trustworthy. The source paper is weak: placeholder
  citations, `n=3–5`, and `pass@1=1.00`, which suggests a tiny test set. Either
  result can publish. The work must be framed as a careful test that decides
  between explanations. If it becomes a new-method paper, the 11 nearby groups
  are better placed to win.
- A second open track needs almost no GPU time: teach research-agent loops when
  to give up on a hypothesis and produce an honest negative result. Nearby
  2026 papers only diagnose the problem. [2604.18805](https://arxiv.org/abs/2604.18805)
  finds that agents ignore evidence in 68% of traces and offers no fix.
  [2607.13083](https://arxiv.org/abs/2607.13083) "propose[s] no corrective
  intervention." A statistical stopping rule versus an LLM judge remains
  open, and directly questions Science One's own stated reliance on LLM judges.

### AHD — raised to ★★★★

Automated heuristic design (AHD) uses systems, often LLMs, to invent algorithms
or rules for solving problems.

- About 13 control papers in 24 months each cover one small case. For example,
  [2605.15221](https://arxiv.org/abs/2605.15221) studies only circle packing at
  `n=26` and says it "lacks comparison to domain-specific classical
  optimization methods." In the same period, about 20 new use areas opened.
  Abstracts containing "automated heuristic design" number 1/1/5/**17** in
  2023/2024/2025/January–July 2026. There are about 10–15 claims for every
  control study.
- No one has drawn the LLM-versus-classical performance line after accounting
  for cost. Three focused searches found zero papers. Only LLaMEA, the system
  proposing the method, compared with CMA-ES/DE. DeepMind's own
  [2602.16928](https://arxiv.org/abs/2602.16928) says that after distillation,
  "the true driver of generalization lies in a minimal algorithmic core."
- The paper has two parts. (a) Build a cost-crossover curve that puts tokens and
  CPU time in one money unit, and test whether the roughness of the problem
  landscape predicts where the best tool changes. CostAda
  [2607.26828](https://arxiv.org/abs/2607.26828) provides the cost method, and
  BLADE provides the test harness. (b) Audit novelty: grade results as old
  ideas rediscovered, old parts recombined, or truly new. Compare with RAISE's
  19× worse out-of-distribution (OOD) performance. No paper studies
  part (b).
- Limits: van Stein/Bäck and DeepMind work close to this topic. The search used
  arXiv only, so check GECCO and PPSN before starting.

### KV cache — ★★★★ only for safety-aware allocation

A KV cache stores earlier attention keys and values so an LLM can produce the
next token faster. Allocation decides how much cache space each layer gets.

- The worry that this work would require new low-level kernels was wrong.
  [KVTuner (ICML 25)](https://arxiv.org/abs/2502.04420),
  [EvolKV (EMNLP)](https://arxiv.org/abs/2509.08315), and
  [SCBench (ICLR 25)](https://arxiv.org/abs/2412.10319) all published without
  kernel work. The real bar is a level of detail that fits serving systems and
  gives higher throughput on existing kernels.
- The long-horizon opening is taken by CONF-KV
  [2605.24786](https://arxiv.org/abs/2605.24786). Three methods address errors
  that build up over time: [SQuat](https://arxiv.org/abs/2503.24358), KVarN,
  and [VeriCache](https://arxiv.org/abs/2605.17613). The original T idea is gone.
- The open part is safety. Six searches found no allocator whose goal is safety.
  KVFundaBench v2 removed safety from its abstract, suggesting that work was
  dropped. [2510.00231](https://arxiv.org/abs/2510.00231) reports that
  instructions can be "completely ignored" after compression. CAQ
  ([2511.07842](https://arxiv.org/abs/2511.07842)) shows that a paper about this
  kind of goal mismatch can publish for weight post-training quantization (PTQ).
- Run one clear sweep: compare the best per-layer allocation for safety with the
  best one for perplexity. If they match, stop cheaply. If they differ, the
  result is a new map plus a new allocator. Register
  [2605.18053](https://arxiv.org/abs/2605.18053), whose protection method beats
  scoring, as the control.

### B1 — lowered to ★★★

- Already done: [2604.16027](https://arxiv.org/abs/2604.16027) by Karouzos,
  Tan, and Aletras, posted 2026-04-17, follows the three
  [Olmo 3](https://arxiv.org/abs/2512.13961) training paths. Its main result is
  that training data writes diversity collapse into model weights; the output
  format does not cause it.
- A smaller opening remains because the study only observes existing models.
  Its training paths differ in both data and algorithm, so they do not show
  which one causes the result. The direct crossover experiment has not run.
  But the authors already named it as their next step, and Apple/CMU
  ([2605.09995](https://arxiv.org/abs/2605.09995)) added an untested model-size
  claim: the problem "worsens with scale."
- Our cost estimate was too low. A full SFT/DPO/RLVR crossover plus several
  model sizes costs more than 400 GPU-hours.
- The broad area has useful openings—about 10 papers per month and no survey—
  but this exact question is mostly closed. New work would need to be a
  contested, careful test between claims.

### T4 — lowered to ★★

- Three papers already did the main work. DiReCT
  ([2605.31175](https://arxiv.org/abs/2605.31175), May 29) uses our motivation
  almost exactly—selecting training data during this phase is important but
  lacks a sound method—and tests Llama-3-8B on 300B tokens with theory and code.
  QAFSL ([2605.25698](https://arxiv.org/abs/2605.25698)) makes the claim that
  decay weakens learning exactly when high-quality data arrives, and gains
  +1.70 over WSD on a 15B-MoE model. MIRA
  ([2605.30288](https://arxiv.org/abs/2605.30288)) shows that selecting data
  during mid-training is its own problem.
- **Our July search made a real mistake.** It called the area empty even though
  DiReCT had been public for about eight weeks.
- The area looked empty because labs publish recipes, not general methods. For
  example, Olmo 3's Dolmino gives pool sizes but no selector. Small-scale work
  now has a clear weakness: [2606.07597](https://arxiv.org/abs/2606.07597)
  finds that forked-decay prediction "frequently fails" when high-quality data
  repeats, which is exactly our protocol. The question may also disappear if
  decay itself is unnecessary. Three separate groups—
  [WSM](https://arxiv.org/abs/2507.17634), WSO, and
  [2604.13627](https://arxiv.org/abs/2604.13627)—all move toward less or no decay.
- Small, 1B-scale *data-selection* papers often publish at ICML/ICLR. But no
  paper that connects selection to the learning-rate schedule has a top-venue
  acceptance. [2511.18903](https://arxiv.org/abs/2511.18903), our planned
  baseline, remains unpublished after three versions in 9 months.
- PI-supplied rush check: Compute-Constrained Data Selection (ICLR 25)
  ignores training phase and studies fine-tuning only. It leaves the exact
  opening technically free, but it also shows that gradient-based selectors,
  T4's main tool, are almost never the best use of compute. Any remaining study
  must include its cost-aware baseline curve.
- One cheap, unclaimed question remains: does the value ranking of individual
  documents change between the stable-learning-rate phase and the decay phase?
  Measure rank correlation on one shared model trunk. This separates the
  "wasted data" explanation from the "sharpness" explanation. A null result
  would weaken the papers that took the main opening.

## Lessons from the re-check

1. **The number of nearby papers predicted nothing.** All four directions that
   we lowered for being crowded came back up: T3, AHD, self-improvement, and KV.
   Both directions that we liked for being empty fell: T4 and B1 were already
   done. A better test is whether new questions appear faster than papers answer
   them, and whether someone has already claimed this exact question.
2. **New-method areas fill in months; careful tests between explanations and
   diagnosis often stay empty.** This happened in five of eight areas: T1, T2,
   T3, collapse, AHD, and T4's remaining question. A likely reason is that a
   method group cannot easily publish a null result about its own explanation.
   Our tools are built for studies where a null is still useful.
3. **A search becomes stale quickly.** T4 and B1 were taken between April and
   June. The July survey was already wrong in one area when written. Re-run any
   gate older than about six weeks before spending compute.
4. Related working rules remain: judge by remaining opportunity, check prior
   work before experiments, and never trust a paper's own novelty claim without
   checking it.

## Related

[[Self-Improving-AI-Survey]] (its July collapse decision is superseded) ·
[[Top-Researcher-Scan-2026-08]] (researcher-level openings from the same date) ·
[[Method-Opportunities]] · [[Live-Research-Opportunities]] ·
[[Status-And-Survivors]] (its star table was superseded by this page, which was
then superseded on 2026-08-03 as explained at the top)
