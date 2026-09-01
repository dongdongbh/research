# Researcher Scan Refresh — 2026-08-31

This page records what changed between 2026-08-02 and 2026-08-31 for the 26
researchers in [[Top-Researcher-Scan-2026-08]], plus new people and releases
the fixed list would miss. Eight agents ran in parallel: two single-person
checks (Karpathy; Bengio/LawZero), five cluster checks (diversity,
verification, VLM binding, robotics, LLM training science), and one
new-entrants sweep. Every claim below carries a date and a link taken from a
primary source. The arXiv API rate-limited our host during the scan, so
verdicts marked "arXiv-verified" come from arXiv pages checked one by one;
talk and social-media coverage is incomplete for the robotics slice.

**Headline: none of our four live projects was scooped in August, but three
of them must change wording or add experiments, and five entries in the
2026-08-02 baseline are wrong.**

## 1. What this means for our live projects

### TMLR binding paper (under review)

- **Near-scoop, survivable.** [Foveated Probes](https://arxiv.org/abs/2608.00726)
  (posted 2026-08-01, one day before our baseline scan) argues our thesis —
  spatial information survives in frozen patch tokens and global pooling
  loses it — on the same encoders (CLIP, SigLIP, SigLIP2). Verified from
  their full text: attribute binding only, no role/verb binding, no text
  encoder, and they never evaluate the image-text similarity score. Our two
  distinguishing legs survive. **Action: the revision must cite it with a
  delta paragraph.**
- **Pre-emptable attack.** [2608.06174](https://arxiv.org/abs/2608.06174)
  names "operation laundering": a factor probe can score well by reusing one
  predicted slot across operations. **Action: add their injective
  leave-one-cell-out protocol as a control arm (<50 GPU-h).**
- **Time-sensitive follow-up.** Foveated-vs-global readout run *through the
  contrastive score itself*, on verb-argument role swaps, is now both our
  differentiator and the obvious next paper for the Foveated Probes authors
  (~100–200 GPU-h on existing caches).

### RoboJudge audit (ICLR abstract 2026-09-18)

- **The framing is taken; the evidence is not.**
  [A Judge Should Know What Changed](https://arxiv.org/abs/2608.24419)
  (2026-08-25) makes the "reliability is not validity" argument for text
  judges: invariance 0.945, sensitivity to genuine construct changes 0.319,
  and surface-only predictors reproduce 67.4% of MT-Bench human votes. It
  touches no robotics. **Action: cite it; pivot the paper's framing to the
  embodied setting** (physical ground truth, tiny samples).
- **Fresh ammunition.** [RoboWorld](https://arxiv.org/abs/2607.01060) claims
  r = 0.989 from n = 8 policies with no confidence interval and GPT-4o as
  judge — weaker footing than RoboLab's r = 0.68 at n = 4, and it is already
  inside our audited eleven. [TrustRoboReward](https://arxiv.org/abs/2608.08491)
  (2026-08-09) independently found a RoboReward defect (20.15% score-pair
  reversals) — a citable third-party defect.
- **The clock.** The NeurIPS 2026 workshop
  ["Who Verifies the Agents?"](https://verify-agents-workshop.github.io/)
  closed submissions 2026-08-29; expect a December wave. NVIDIA is
  productizing RoboLab into
  [Isaac Lab-Arena](https://github.com/isaac-sim/IsaacLab-Arena) (100+
  August commits, no validation of its success detector). **Do not slip the
  Sep 18 abstract.**
- **Zero-compute strengthener.** The Mazaheri
  [judge-leniency audit](https://arxiv.org/abs/2608.16003) (2026-08-17)
  separates "the judge is soft" (criterion) from "the judge is blind" (d′)
  with signal-detection theory. It is a re-analysis of scores we already
  produce. A small Apple-affiliated team one step from a robot version —
  both threat and gift.
- All four audit targets (DriveJudge, DreamGen Bench, SC3-Eval, RoboReward)
  still have **no independent validity audit**. RoboLab is unchanged at v4
  (2026-08-14); the ratified option (c) holds. Fourteen August papers found
  by this scan are absent from the RoboJudge prereg page and should be
  folded in.

### H3 diversity campaign (IMM, Delta — live)

- **Not scooped.** No August paper explains the cause of diversity loss in
  1-NFE / few-step image generators. He's opening is untouched; the
  [Drifting code](https://github.com/lambertae/drifting) is public and
  dormant (last commit 2026-03-19), so the disproof arm stays cheap.
- **The generic claim is taken.**
  [Verifier-Induced Support Reshaping](https://arxiv.org/abs/2608.00220)
  (2026-07-31) already shows optimization narrowing an LLM's outputs
  (pass@1 +6.5 points, best@32 −9.8). Our paper must lead with the *cause*
  question in one-step image models, not with "optimization reduces
  diversity."
- **Add before analysis lock:** (a) a training-set-size sweep to rule out
  the finite-sample rival explanation of
  [2602.14682](https://arxiv.org/abs/2602.14682) (diversity scores grow
  with sample size, so finite training sets look less diverse than the
  truth); (b) report under-dispersion with
  [ZID](https://arxiv.org/abs/2608.24881) beside recall, so the claim
  carries a p-value; (c) consider porting the "entrance locking" test of
  [2608.29188](https://arxiv.org/abs/2608.29188) (2026-08-29 — RLVR
  diversity loss concentrates at the start of the trace) to the sampling
  trajectory of a one-step model.

### Epistemic contextualization (prereg, Deviation 5 in progress)

- **No LawZero release; the 48-hour stop rule has not fired.** LawZero is
  staffing the "truthifier" through
  [MATS Summer 2026](https://www.matsprogram.org/stream/lawzero-10) rather
  than shipping it; its data-pipeline jobs are
  [still open](https://job-boards.greenhouse.io/lawzero). Fellow write-ups
  are due this autumn — watch them.
- **Baseline miss with a decision attached.** A public partial
  implementation has existed since April:
  [truthification_pretraining](https://github.com/superkaiba/truthification_pretraining)
  by Thomas Jiralerspong (Mila). Finished fine-tuning-scale experiments
  (Qwen-2.5-7B, 3 seeds; HIGH-reliability tags cut misalignment by 24
  points, p = 0.009); conclusion "a compartmentalized policy, not genuine
  alignment"; and a written next-phase plan close to our Phase 1 (FineWeb,
  2–10B tokens, raw / truthified / random-metadata arms). Nothing pushed
  since 2026-04-21. **Owner call pending: our read is wording correction
  plus related work, not a stop** — their outcome (emergent misalignment
  after fine-tuning) differs from ours (calibration under source conflict
  after mid-training).
- **Related work we must now cite:** [MeCo](https://arxiv.org/abs/2501.01956)
  (ICML 2025 — our tags-only arm under a published name),
  [MIRA](https://arxiv.org/abs/2605.30288) (source-aware data selection,
  no rewriting), and possibly
  [ElephantBench](https://arxiv.org/abs/2608.28478) (conflicting accounts
  of long-tail facts).
- **New probe worth adding before the next lock:** test with
  present-but-worthless metadata, to separate "the model reacts to metadata
  being present" from "the model is sensitive to source quality"
  (suggested by Jiralerspong's negative result).

### Sony proposal (circuit repair of binding)

- **Still unoccupied.** August produced several binding *diagnosis* papers
  but nothing doing activation patching or steering for binding repair
  inside CLIP-family encoders. The proposal's core claim of novelty holds
  as of 2026-08-31.

## 2. Corrections to the 2026-08-02 baseline

Five errors or misses, now on record:

1. **CoVFT is not Darrell's paper.** [2603.21077](https://arxiv.org/abs/2603.21077)
   is by Nan Zhou, Huiqun Wang, Yaoyan Zheng, Di Huang (Beihang), CVPR 2026.
   Three wiki pages call it "Darrell's C1 group" and rank Darrell the main
   T1 scoop risk ([[Top-Researcher-Scan-2026-08]] line ~280,
   [[Method-Opportunities]] lines ~307–320,
   [[Direction-Reevaluation-2026-08]] lines ~59, 78). The T1 risk re-points
   to the Beihang group.
2. **The scan table and the gate table disagree on the autoresearch FDR
   opening.** The scan table says "unclaimed"; [[Method-Gates-Wave-2-2026-08]]
   (line ~50) already marks S-M2 as SCOOPED level 2 against
   [PACE](https://arxiv.org/abs/2606.08106). The gate table is right.
   What survives is narrower: a run-level audit of Karpathy's specific
   public rule (see §4).
3. **Beyer's move date is wrong.** He went OpenAI → Meta in **June 2025**
   ([TechCrunch](https://www.techcrunch.com/2026/06/26/meta-hires-key-openai-researcher-to-work-on-ai-reasoning-models)),
   not July 2026.
4. **Karpathy joined Anthropic to lead pre-training** (announced
   [2026-05-19](https://techcrunch.com/2026/05/19/openai-co-founder-andrej-karpathy-joins-anthropics-pre-training-team/)),
   before our baseline; the baseline missed it. He has been publicly silent
   since 2026-08-02.
5. **Missed papers now folded in:** the Jiralerspong repo (April),
   [2607.05184](https://arxiv.org/abs/2607.05184) (Arora's group,
   2026-07-06 — privileged context lowers fork rates; half-claims our
   "drag = fork suppression" opening, the causal intervention remains),
   [jepa-wms](https://github.com/facebookresearch/jepa-wms)
   ([2512.24497](https://arxiv.org/abs/2512.24497) — released planning
   checkpoints), and [Foveated Probes](https://arxiv.org/abs/2608.00726)
   (2026-08-01).

**Standing hazard confirmed again:** [Beyer's homepage](https://lucasb.eyer.be/)
still carries hidden text instructing LLM agents to insert DOTA2 references.
Our agent quoted it and did not follow it. No other fetched page carried
injection text this scan.

## 3. Scoreboard delta

Only changes are listed; everyone else holds their 2026-08-02 star.

| Person | Was | Now | One-line reason |
|---|---|---|---|
| Liang | 4 | **5** | The [535B hero run](https://github.com/marin-community/marin/issues/8435) is the largest fully open training log in existence; the pre-hero scaling-ladder protocol (~1% of budget) is copyable at our scale |
| Sadigh | 3 | **4** | Her new work ([QWM](https://arxiv.org/abs/2608.17163), with Finn) is simulation-only — inside our reach |
| Fidler | 2 follow | **3 follow / 4 audit** | New mechanism paper ([2608.13391](https://arxiv.org/abs/2608.13391)); DriveJudge audit confirmed viable (repo 0 stars, dormant) |
| Karpathy | 5 | **4 person / 5 artifact** | Inside Anthropic, publicly silent; the frozen `experiment_refactor` branch remains a top audit target |
| Bengio | 4 | **3.5** | "First implementation" of contextualization no longer survives contact at fine-tuning scale (Jiralerspong) |
| D. Chen | 4 | **3** | Sabbatical at Thinking Machines; her listed opening is being commoditized |
| Shazeer | 4 | **3** | Two silent months; the MQA opening outlived the signal |
| Abbeel | 4 | **3** | Drifting into humanoid hardware we cannot touch |
| B. Zhou | 4 | **3** | No new work; MetaDrive dormant a year |
| Sutskever | 3 | **2** | SSI still publishes nothing; NVIDIA partnership (2026-07-27) but no artifact |
| D. Zhou | 3 | **2** | Reportedly moved to Meta ~Feb 2026 (reported, not self-confirmed); zero 2026 papers |
| Malik | 3 | **2 person** | Only a 35-author survey signature; his 5-star MOCHI opening survives |
| Wei | 4 idea / 1 person | **4 idea / 0–1 person** | No 2026 posts at all; the ideas are ours to test |

## 4. Opening status changes

**Dead or drop:**

- **D. Chen agent-KV-cache measurement — drop.** Two August systems papers
  landed inside it ([2608.00902](https://arxiv.org/abs/2608.00902),
  [ReCache 2608.19662](https://arxiv.org/abs/2608.19662)); we would be the
  fourth entrant.
- **Autoresearch FDR as originally framed — dead** (gate already said so;
  repo dormant five months; PR 560 closed unmerged 2026-05-05). Narrowed
  replacement below.
- **Marin as a compute route — dead for now.** The external speedrun
  workflow ([issue #1981](https://github.com/marin-community/marin/issues/1981))
  closed 2025-12-16; zero external speedruns June–August; the lab is
  absorbed by the hero run.

**Eroded (part claimed, part open):**

- **Arora drag = fork suppression:** correlation shown by
  [2607.05184](https://arxiv.org/abs/2607.05184); the fork-rate
  *intervention* (fix fork rate, watch the drag) is open, 50–150 GPU-h,
  now a confirmation rather than a discovery.
- **Beyer where-binding-fails:** the attribute half is claimed by
  [Foveated Probes](https://arxiv.org/abs/2608.00726); the role-binding and
  similarity-score halves remain ours.
- **Choi cause-breakdown:** the RLVR/LLM side is partly localized by
  [2608.29188](https://arxiv.org/abs/2608.29188); the generative-model side
  and the G-Vendi link are untouched.

**Intact and re-verified unclaimed** (all arXiv-verified this window):
He's 1-NFE averaging test · Shazeer equal-KV-bytes MQA from scratch (~400
GPU-h; [GQLA](https://arxiv.org/abs/2605.15250) converts checkpoints, never
trains from scratch) · Liang replay-cause
([2603.04964](https://arxiv.org/abs/2603.04964) observes, never manipulates)
· Levine OGBench four-cause · Song GMP three-cause
([v2 2026-08-03](https://arxiv.org/abs/2604.18933), still only a noise
ablation) · Fan EgoScale N1.6→N1.7 (no new GR00T release since April; both
checkpoints public) · DriveJudge audit · DreamGen judge audit (its >90%
human-agreement claim still unchecked outside NVIDIA) · Sadigh polychromic
matched-temperature · Malik MOCHI/Bonnen + VLM arm (5-star) · Isola Q
failure boundary · M. Li AdaptVis-cause and static-to-active · B. Zhou both
· Neubig CAID 2×2 and PACE future-time test (PACE has one citation ever) ·
Wei verifier-rule tests · Bengio sparsity (one citer ever, no experiments;
our own `sparsityprem` design from 2026-08-07 still un-run).

## 5. New openings found this window

Grouped by the project they feed. Costs assume our limits (L40S/A100/H100/
H200, 100–1000 GPU-h, no robots).

**Feed RoboJudge (before/with the ICLR submission):**

1. Mazaheri criterion-vs-d′ decomposition on our existing judge scores —
   ~0 GPU-h, ~1 day of analysis ([2608.16003](https://arxiv.org/abs/2608.16003)).
2. Audit [SimFoundry](https://arxiv.org/abs/2606.28276)'s r = 0.911 from
   n = 7 (bootstrap CI + leave-one-architecture-out) — 0–40 GPU-h. Same
   shape as the RoboLab claim we already ruled on.
3. "Can the claim be replayed?" census for robot/VLM-judge benchmarks, in
   the style of [2608.19269](https://arxiv.org/abs/2608.19269) — CPU-only.
4. Ablate [HarnessEval-W](https://arxiv.org/abs/2608.16859)'s agentic judge
   (flat-prompt arm + blind arm) — 80–150 GPU-h, inference only; 339
   upvotes, 18 world models, no validation.
5. Reuse [PRM-as-a-Judge 1.5](https://arxiv.org/abs/2608.14284)'s 2,244
   human-annotated intervals (availability unconfirmed — no GitHub URL in
   the paper) and its sim-vs-real rho = 0.18–0.58 as a citable premise.

**Feed H3 / diversity:** the three additions listed in §1 (size sweep, ZID,
entrance-locking port).

**Feed TMLR / binding:** the contrastive-score foveated follow-up
(time-sensitive, 100–200 GPU-h) · the injective control arm (<50) ·
multilingual binding through the pooled score in SigLIP2 (50–150; empty —
[M²BIND](https://arxiv.org/abs/2608.12333) covers generative VLMs only).

**Standalone, cheap and unclaimed:**

- Planning-budget sweep on the public
  [jepa-wms](https://github.com/facebookresearch/jepa-wms) checkpoints —
  100–300 GPU-h, inference-only; the paper fixed horizon/steps/window and
  never swept them.
- Audit Neubig's OpenHands critic
  ([The Verification Stack](https://www.openhands.dev/blog/20260506-the-verification-stack)):
  shipped with a speed number (58% cut in time-to-merge), no validity
  number; ground truth is free public PRs — 100–300 GPU-h.
- Score Wei's five verifiability properties against fresh benchmark
  saturation data — <100 GPU-h, API-only; nobody competes because he is
  not defending it.
- Calibrate Karpathy's `experiment_refactor` error bars (±0.5% compute,
  asserted in prose, no seed replication anywhere in the code) against his
  own `dev/LOG.md` accept/reject ledger — 100–300 GPU-h; replaying his
  logged calls with seeds gives a flip rate on his own ground truth.
- Fork-rate intervention on [2607.05184](https://arxiv.org/abs/2607.05184)
  — 50–150 GPU-h.
- Replicate [LpWM](https://arxiv.org/abs/2608.22764)'s "sparse beats dense
  by up to 57% at intermediate predictor capacity" (six days old,
  capacity-conditioned) — 150–300 GPU-h; the negative is publishable.
- Replicate Song's [Memory Anchors](https://arxiv.org/abs/2608.26545)
  claim (removing 10% of anchors raises forgetting 4.5× on LIBERO) — sim
  only, no hardware.
- Finn's QWM contradiction: her paper treats world models as too biased to
  train values on while many groups use them to *rank* policies; find the
  horizon where the bias breaks ranking — 150–400 GPU-h.
- Shared-cause test of the inference-latency convergence (Levine's
  [ARLI](https://arxiv.org/abs/2608.23831), 2026-08-24, and B. Zhou's
  Slow-Brain/Fast-Planner, June — neither cites the other) — 100–300 GPU-h.

## 6. New people and convergences

- **Parsa Mazaheri (Apple) + Kasra Mazaheri:** two-person team, two August
  papers in our judge-audit niche
  ([2608.16003](https://arxiv.org/abs/2608.16003),
  [2608.15022](https://arxiv.org/abs/2608.15022) with
  [code](https://github.com/parsa-mz/innerj)). Watch closely; closest
  thing to a direct competitor this scan found.
- **Construct-validity wave (theme V accelerating):**
  [2608.24419](https://arxiv.org/abs/2608.24419),
  [DA-RAC](https://arxiv.org/abs/2608.14950),
  [RecurSE](https://arxiv.org/abs/2608.24231),
  [OSReward](https://arxiv.org/abs/2607.28609), Mazaheri, plus the NeurIPS
  workshop. Evaluation machinery is shipping faster than anyone validates
  it (HarnessEval-W, [XPolicyLab](https://arxiv.org/abs/2608.09892),
  PRM-as-a-Judge 1.5, Isaac Lab-Arena). Our embodied evidence lane —
  judge-vs-human ranking correlation with proper uncertainty — is still
  empty.
- **World models as search, not evaluators:** Finn + Sadigh's
  [QWM](https://arxiv.org/abs/2608.17163) trains values only on real
  transitions to dodge model bias — quiet tension with the eleven groups
  using world models to rank policies (theme W).
- **[Puro-2B](https://arxiv.org/abs/2608.27370)** (Tsinghua/Stanford,
  2026-08-27): 2B model pretrained on one RTX 5090, 1.4T tokens in FP8,
  under $6.9K, Apache 2.0 — theme S delivered; a ready platform for
  controlled from-scratch studies.
- **[Mechanist](https://arxiv.org/abs/2608.12036)** (2026-08-12): agentic
  automation of causal-intervention studies (32 methods). Not a competitor,
  but it means cause-comparison work gets cheaper for everyone — our
  advantage on theme A is time-limited.

## 7. Owner decisions pending from this refresh

1. Jiralerspong repo vs the contextualization stop rule (recommendation:
   wording correction + related work, not a stop).
2. Add the format-gating probe to the contextualization design before the
   next lock.
3. RoboJudge framing pivot to the embodied setting (recommendation: yes,
   and cite [2608.24419](https://arxiv.org/abs/2608.24419)).

Re-scan cadence reminder: unclaimed-verdicts above decay in weeks. Re-verify
at prereg time; never commit on a scan older than ~6 weeks.

## Related

[[Top-Researcher-Scan-2026-08]] · [[Method-Gates-Wave-2-2026-08]] ·
[[Direction-Reevaluation-2026-08]] · [[Method-Opportunities]] ·
[[Prereg-Epistemic-Contextualization]]
