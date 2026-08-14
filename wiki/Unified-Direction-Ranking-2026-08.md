# Unified Research-Direction Ranking — version 4, 2026-08-07

**Status: this is the current ranking. Every row now has a formal verdict,
including the checks run on Aug 7.**

## Words used on this page

Read this list once. Every word below is used the same way everywhere on this
page.

- **Direction** — a research idea we might turn into a paper.
- **Gate** — a check with rules written down before we run it. The gate decides
  whether a direction lives or dies.
- **Level** — how much of an idea is already published. A lower number means
  more overlap. Level 1 means the idea is already fully published by someone
  else. To survive, a direction must score better than Level 2.
- **Kill** — the gate found a reason to stop. A killed direction must not be
  proposed again.
- **Scooped** — someone else already published the idea.
- **Cell** — a small patch of research territory: one question, attacked one
  way. Saying a cell is "occupied" means a named group already works there.
  ("Cell A" and "Cell B" below are project names, not this sense of the word.)
- **Stars (★)** — our rating of how promising a direction is, out of five. A
  star given before a deep check is only a first impression.
- **GPU-h** — one hour of one graphics card. It is how we measure compute cost.
- **Pre-registration (prereg)** — a plan we write and freeze before running the
  experiment, so we cannot change the prediction afterwards.
- **Kendall τ** (say "tau") — a number from −1 to +1 that says how well two
  rankings agree. +1 is identical order, 0 is no relation, −1 is reversed.
- **CI (confidence interval)** — the range of values the data still allow. If
  the interval does not contain zero, chance alone is an unlikely explanation.
- **Contamination** — a model was trained on some of the very data we test it
  on, so its score there is not a fair test.
- **First step** — the cheapest action that decides whether to continue.

## What changed most recently

**Two more checks ran on Aug 7, at the owner's request.**

The first check asked whether merging crop-consistency distillation with the
role-decodability evidence would rescue both as one CVPR method paper. The
answer was no, at Level 2. Three pieces are already published by others: the
crop teacher is a standalone published method
([2506.09691](https://arxiv.org/abs/2506.09691)); the caption-free image-side
axis is taken by [ATAS, ICCV 2025](https://arxiv.org/abs/2506.08678); and
frozen-tower relational alignment belongs to
[ComAlign](https://arxiv.org/abs/2409.08206).

The second check killed the Abbeel parallel-RL factorial, because its premise
was wrong. Details are in its row in Part 2.

**The live slate is now exactly six items.**

| Direction | Where it stands |
|---|---|
| RoboJudge | ICLR, ready to lock |
| 1-NFE diversity | CVPR, week-1 passed, ready to lock |
| Epistemic contextualization | ICML, pipeline ready, needs sign-off |
| SVIB→TMLR | **Owner decision Aug 14: withdraw from NeurIPS now, submit to TMLR.** Drafting started; target draft-complete ~Aug 28, submit on withdrawal |
| Sparsity-premise test | Design ready, runs on OrangeGrid, a standalone safety-empirics paper |
| Robot-evaluator uncertainty audit | Headline proven; write the boundary note, then design |

**Owner decision, Aug 7:** the role-decodability evidence JOINS the SVIB→TMLR
paper as its analysis backbone. One question is still on the meeting agenda:
whether the DisCoCLIP audit note becomes a subsection of that paper or a
separate note.

**What happened on Aug 6.** We ran a full adjudication day on one held 2-GPU
node. Thirteen directions were killed, either by their own pre-stated rules or
by verified prior work. One direction survived its check: the sparsity-premise
test. Two committed pre-registrations moved forward: 1-NFE passed its week-1
check, and the contextualization pipeline was built and smoke-tested. Two big
results landed: the verb-role probe replication, and the MOCHI cross-task
contrast. Total compute for all of it: about 20 GPU-hours.

Every kill below names its evidence, so that the idea is not proposed again.
The version-2 page from Aug 3 is in git history.

**The one lesson to carry.** Across 18 formally checked ideas this month,
first-look star ratings dropped by 1–2 stars on average once a deep check ran.
The two worst kills were papers less than eight weeks old with zero citations.
So nothing enters a pre-registration without three things: the two-pass check,
an OpenReview sweep, and a verified cheap first step.

## Part 1 — Committed work

These projects are already chosen, so they are not ranked against each other.

### ICLR (Sep 18) — RoboJudge audit ★★★★½

**The question:** do robot-policy evaluators recover the human ranking?

**RESULTS FINAL (Aug 10).** H1 is true: 4 of 6 evaluators name the wrong best
policy. SigLIP2 is significantly ANTI-correlated with human judgment, at τ
−0.714. H5 holds in its strongest form. Contamination worth +0.10 τ was caught
by the double-reporting rule. Every judge has an axis-named noise floor.

**State:** the campaign is closed and archived. The project is in its WRITING
PHASE, with the abstract due Sep 18.

**Full record:** [[Prereg-RoboJudge-Audit]]

### ICLR (Sep 18) — ~~Crop-consistency distillation~~ — CLOSED Aug 7

**What the idea was:** distil a crop-consistency teacher into a dual encoder,
as one ICLR paper. It was benched on Aug 6. The owner then asked whether
merging it with the role-decodability evidence would rescue it as one CVPR
method paper.

**Why it is dead:** the merge gate returned Level 2. The crop teacher is
already a published standalone method
([2506.09691](https://arxiv.org/abs/2506.09691)); it is training-free, and its
own ablation shows that the crops carry the gain.
[ATAS](https://arxiv.org/abs/2506.08678) owns caption-free image-side
self-distillation. [ComAlign](https://arxiv.org/abs/2409.08206) owns
frozen-tower relational alignment. Only one loss-function detail is literally
unclaimed — "distill the teacher's inter-region attention pattern into a dual
encoder" — and a loss-function detail is not a paper.

**Where the evidence lives:** tier2gates `runs/bindfix-gate-20260807/`

### CVPR (~Nov 13) — 1-NFE diversity

**RESULTS FINAL (Aug 10).** H1 is confirmed in Shortcut — 13 of 13 matched
contrasts, +0.17 at 50k samples — but NOT in iMF. So the effect depends on the
model and is not a property of the objective. H2 is true, at +0.215 with a CI
that excludes zero, but the mechanism we proposed for it is unsupported.

**The central finding:** the training recipe dominates, and the objective
family predicts nothing.

**How it was run:** 40 cells, paired intervals, and 4 self-corrected errors
written into the ledger. The H3 launch spec is delivered, with the knob
confound declared in advance; H3 needs its own compute window.

**State:** WRITING PHASE.

**Full record:** [[Prereg-1NFE-Diversity]]

### CVPR (~Nov 13) — ~~Readout-budget vs binding frontier (Cell A)~~ — KILLED Aug 6

**What the idea was:** measure a frontier between how much readout budget a
model spends and how well it binds words to objects.

**Why it is dead:** its own pre-stated pilot rule fired. The best untrained
readout minus R0 is **−21 points** on SugarCrepe++, on both backbones, with CIs
that exclude zero in the wrong direction. The readout ladder is also flat: only
a 2.6-point spread from K=1 to K=256. So there is no frontier to measure. We
bug-hunted before accepting this: the retrieval control is healthy, the wiring
checks are exact, and the reruns are bit-identical.

**Where the evidence lives:** cropdistill `runs/cellA_pilot/20260806-report/`

### ICML (~Jan 28) — Epistemic contextualization

**STOPPED BY ITS OWN PRE-REGISTERED GATE (Aug 10).** The treatment rewriter
preserves facts on 91.2% of items, against a 97% threshold that was frozen
before any judging. The same model's placebo paraphrase scores 97.1%. So the
attribution mechanism itself INVENTS sources when metadata is thin. That is a
mechanism-level finding, and §5 of the prereg anticipated a publishable
negative result.

**State:** all infrastructure is banked and resumable. The learning rate is
locked. DELTA_LAUNCH.md is ready, behind a blocking notice.

**Owner decides between two branches:** an honest-note paper, or an amended
method with a stronger rewriter or retrieval-grounded sources. Either way, the
prereg still requires the 50-item blinded human spot-check.

**Full record:** [[Prereg-Epistemic-Contextualization]]

### TMLR (no deadline) — SVIB audit paper

**Decided Aug 4:** make it one paper. It contains the audit, the locked
protocol, the positive grid-and-attention result, and the released suite.

**State:** a writing task of about 2–3 weeks.

### Running — Role-decodability program

**COMPLETE at power on both strata (Aug 6).** For spatial items, accuracy is
99.5% in the patches but 50.1% at the score. For verbs, with n=279, accuracy is
74% in the patches, 52–57% pooled, and at chance at the score; the pilot's
below-chance 29.8% was small-n noise. The region-readout inversion was
independently replicated, at 33–39% on unseen items. Both strata land in the
suppression-at-readout cell. See [[Binding-Root-Cause-Analysis]] §8.

**⚠ Aug-6 scoop alert for the planned method paper.**
[2608.00726](https://arxiv.org/abs/2608.00726), posted Aug 1, publishes the core
patch-versus-global finding. The method side is occupied by DisCoCLIP; see the
killed algebraic-binding row in Part 2. Four things remain uniquely ours: the
powered two-strata corpora, the inversion result, the Cell A dissociation, and
the MOCHI specificity contrast. The plan to turn the probe into a method paper
needs an owner decision at sign-off.

## Part 2 — The all-in-one ranking of available directions

Stars are post-check where a check ran, and those rows are marked ✓. All other
stars are pre-check.

### ★★★★½–★★★★ — strongest available

These directions are still open.

| Direction | ★ | Cost (GPU-h) | First step | Gate note |
|---|---|---|---|---|
| Bengio sparsity-premise test ✓ (Aug 6) | ★★★★ **SURVIVES — design ready for sign-off** | **~70** (measured, not the old ~250 guess) | Approve `code/sparsityprem/DESIGN.md`, then run it on **OrangeGrid** as 300 short independent jobs, using the HTCondor profile. Measured: H100 concurrency buys exactly 1.00×, so it does not help. | Level 3, against a bar of ≤2. The only paper citing the source (Oberman/LawZero, zero experiments) names our exact hypothesis as untested. That is expiry risk, so move fast. |
| [MOCHI](https://arxiv.org/abs/2409.05862) VLM arm | ★★★★ | 20–60 | **DONE Aug 6**, for 0.7 GPU-h | Results below. |

**Two corrections to the sparsity-premise row, found by reading the source.**
First, the row's label mixed up two separate premises. In
[2606.29657](https://arxiv.org/abs/2606.29657), sparsity is Definition 5.22,
the R-shell, while Requirement 5.23 is no-enrichment. The design measures both,
plus loss-visibility. Second, the link to the contextualization project was
overstated: a result here buys that paper one paragraph, not a whole
experimental arm. So judge this as a standalone safety-empirics paper.

**MOCHI results (Aug 6).** The sanity gate passed twice, with r>0.988 against
the published per-trial data. SigLIP2-L **ties the best published model**:
DINOv2-giant scores 0.443 and SigLIP2-L scores 0.442. Base-size SigLIP2 beats
every published CLIP. At a fixed architecture, the pretraining recipe changes
the score by 2.5× — compare OpenAI against LAION on B/32. Humans, at 0.782, are
still far ahead.

**The MOCHI cause test found NO suppression and NO inversion anywhere.** So
view-consistency information is never built in the first place; it is not
destroyed at the readout. The explicit mismatch with the role-binding cell is
the finding: readout-repair methods should NOT be pointed at 3D view tasks.
Report: cropdistill `runs/mochi/20260806/REPORT.md`. Next decision, an owner
call at sign-off: is this a paper arm, or a completed side result?

#### ~~Abbeel parallel-RL factorial~~ — KILLED Aug 7 (premise wrong, plus Level 1)

**What the idea was:** run a capacity × batch × UTD factorial experiment on
parallel reinforcement learning, motivated by a claimed contradiction between
two papers.

**Why it is dead.** The row's headline "BRC-vs-TD-overfitting contradiction"
does not exist. BRC is a multi-task paper. [Compute-Optimal Scaling (2508.14881,
NeurIPS 2025)](https://arxiv.org/abs/2508.14881) is a single-task scaling
paper. They share three authors, and they agree with each other. Worse, that
same paper already ran our proposed capacity × batch × UTD grid on our own
evaluation suite, and fitted a law to it. Every one of the five "ingredients"
already has a published one-at-a-time curve: FastTD3 §2.1,
[FastSAC](https://arxiv.org/abs/2512.01996), [FlashSAC
RSS26](https://arxiv.org/abs/2604.04539), and
[2605.10236](https://arxiv.org/abs/2605.10236). Only the interaction terms are
left, and the method definition excludes those — the same reason T1 died. The
real cost was about 850 GPU-h, not the 400–650 the row claimed.

**One zero-cost keeper.** FastTD3's shipped batch size of 32,768 sits 1.5–58×
above the published compute-optimal prescription, while its successors moved
inside that range. That is one citable paragraph, not a paper.

**Where the evidence lives:** tier2gates `runs/parallelrl-gate-20260807/`

#### ~~Replay-mechanism arbitration~~ — KILLED Aug 6 (false premise, plus Level 1)

**What the idea was:** arbitrate between competing mechanisms that explain why
replay helps during pretraining.

**Why it is dead.** This was an embarrassing double failure, caught at the gate
for 0 GPU-h.

1. The row misstated its own source.
   [2603.04964](https://arxiv.org/abs/2603.04964) is about replay improving
   FINE-TUNING data efficiency, by 1.9–2.1×. It is not about pretraining
   stability. Loss spikes appear only in its Appendix B.1, where the authors
   call the spike explanation "one relatively vague hypothesis" and have not
   shown that spikes are harmful. The "five candidate causes with cheap
   falsifiers" appear in no wiki record at all, so that claim was unsourced.
2. The corrected question is already fully published.
   [2605.26097](https://arxiv.org/abs/2605.26097) runs the
   capacity-versus-optimization-versus-replay arbitration.
   [2607.00634](https://arxiv.org/abs/2607.00634) owns the transition-transient
   arm, which our own wave-2 record had already flagged.
   [2506.04805](https://arxiv.org/abs/2506.04805) publishes the spike-detection
   instrument.

**One thing to remember for next time:** the fast-moving group here is
Raghunathan/Springer plus Wilson, with four mechanism papers between March and
May. It is not the source authors, who moved to synthetic data in March.

**Where the evidence lives:** replayarb `runs/20260806-stage1/stage1_verdict.json`

#### ~~RLVR vs self-consistency calibration~~ — DIED at its pilot, Aug 6

**What the idea was:** test whether RLVR training breaks the calibration of
self-consistency agreement, and then build a calibrated-aggregation method to
fix it.

**Why it is dead.** The gate PASSED at Level 3; six OLMo-3 pairs isolate the
RLVR stage, and the gate record is [[RLVR-Calibration-Gate]]. Then the
pre-registered pilot answered the question honestly, and the answer was no.
RLVR does NOT break that calibration. The difference in ECE before and after
excludes zero on neither dataset, with Holm-corrected p-values of 0.77 and
0.15. The agreement signal is already well calibrated on BOTH checkpoints, at
ECE 0.06–0.13. The named secondary measurement even moved the other way: after
RLVR, agreement became MORE informative on MMLU-Pro, with a slope of +0.95
[+0.23, +1.74]. A calibrated-aggregation method has nothing left to fix, so the
row closes.

**What survives:** a clean, well-powered negative result, worth one paragraph
in any future aggregation paper. Cost: about 12 GPU-h.

**Where the evidence lives:** cropdistill `runs/rlvr_calib/20260806/`

#### ~~KV-cache footprint under agentic workloads~~ — SCOOPED Aug 6 (Level 1)

**What the idea was:** diagnose how the KV cache grows under agent workloads,
and then design an agent-aware eviction policy.

**Why it is dead.** The gate found BOTH halves already published.
[MemDecay (2607.10582)](https://arxiv.org/abs/2607.10582) fits attention
half-lives on agent traces, with CIs, and ships a turn-aware and region-aware
eviction policy. [IntentKV (2606.09916)](https://arxiv.org/abs/2606.09916)
publishes our exact diagnosis plus a cross-turn policy at 8–14B.
[TraceLab (2606.30560)](https://arxiv.org/abs/2606.30560) released 4,300 real
agent sessions and names agent-aware eviction as its own next step. An SGLang
RFC is building it in production. Worse for any revival: MemDecay's data
suggests our premise was backwards. Accumulated-attention retention, as in H2O,
STRENGTHENS with scale; only the recency family is mistuned. This neighborhood
publishes about one agent-KV paper per week.

**Retire it. Do not propose it again** — this is the third death in the KV
family this cycle. Cost: 0 GPU-h.

**Where the evidence lives:** kvagent `runs/20260806-stage1/stage1_verdict.json`

#### ~~GMP three-mechanism arbitration~~ — KILLED Aug 6 (Level 2, and infeasible)

**What the idea was:** arbitrate between three mechanisms behind GMP. GMP is
**Gated Memory Policy**, from [2604.18933](https://arxiv.org/abs/2604.18933),
by the Song group. The acronym had never been written out anywhere in our
records until this gate ran.

**Why it is dead.** The "read the PDF first" warning was right twice over.
First, the paper partly ablates itself: it fully ablates the noise mechanism,
and the authors admit that the gate is NOT what produces the MIKASA headline.
Second, [Tedrake-group 2606.16447](https://arxiv.org/abs/2606.16447), from
June, already runs the credit-reassignment study on the sibling method. That
includes the encoder-freezing confound we would have found, and it shows that
"naive context scaling is not as brittle as advertised."
[RMBench](https://arxiv.org/abs/2603.01229) and
[μVLA](https://arxiv.org/abs/2606.12497) isolate the rest.

**It is also infeasible for us.** The paper's own cost table implies a floor of
about 220 H100-h. Two of the tasks need real robots. MIKASA needs Vulkan.
Cost spent: 0 GPU-h.

**Where the evidence lives:** `code/gmparb/`

### ★★★½–★★★ — real but narrowed (all ✓ checked)

Two directions here are still open.

| Direction | ★ | Cost (GPU-h) | First step |
|---|---|---|---|
| **Robot-evaluator uncertainty audit** ✓✓ (headline CONFIRMED Aug 6) | ★★★ **proceeds to design** | 200–400 | Write the boundary-versus-RoboJudge note, then design the study |
| Choi hivemind decomposition | ★★★½ | low | Resolve the 404'd dataset. Blocked on data, unchanged. |

**What the robot-evaluator check found.** The Fisher-z afternoon ran, and
**the field's headline correlations are statistically empty.** WorldEval's r =
0.942 rests on n=4, and its interval is [−0.20, +0.999]. Of 74 published
intervals, 40 contain zero. A claimed improvement from 0.700 to 0.875 has
p=0.63. One paper's interval halves once its 7×3 pseudoreplication is unwound.

**Correction to an earlier claim:** RoboDojo
([2607.04434](https://arxiv.org/abs/2607.04434)) publishes NO correlations. It
is the best *target* for our audit, with 30 policies, which is 4× anyone else.
It is not a source of the problem. Its own tables show task-level real-robot
noise, with standard deviation up to 23 points — twice the spread of the entire
leaderboard. Full table: decisive0806 `runs/partB_fisherz/`

#### Two new candidates that have not been gated yet

**Weight-space learning, 5 rows.** Stars run hot before a gate, so treat them
as first impressions. The first step is three reading checks that cost zero
GPU. See [[Weight-Space-Learning-Scout-2026-08]], the owner-requested scout
from Aug 9. Best-graded row: D4, the sparsity-premise zoo dual-use. Our 1,800
planned checkpoints carry an exactly computed safety label, which makes a
weight-space dataset nobody else can build, at about 2–5 GPU-h of extra cost.
It is also the only candidate in the group that is not in a race. Rows D1 and
D3, on the robustness of weight-only safety screening, are real but narrowed by
[Z-PEFT](https://arxiv.org/abs/2608.02271), which was posted six days before
the scout. Generation at LLM scale is closed. Representation-at-scale is a race
against named groups. The scout lists eleven do-not-do cells. Three reading
checks decide whether any gate runs at all: the Z-PEFT full text, Borth's OOD
gap, and the sparsity checkpoint format.

**Distractor geometry as a benchmark leak class.** Unrated so far. The first
step costs 0 GPU: run a census of how many benchmarks release numeric option
sets. Below about 5, this is a note rather than a paper. The idea was born from
the [[Spatial-IQ-Audit-2026-08]], which verified that symmetric numeric
distractors let a blind median-picker score 27.9% against a chance level of
20%, and that this interacts with each model's own rank biases. If it
generalizes, method route 3 opens. MMStar and Cambrian-1 audit blindness but
never audit how distractors are constructed. A mandatory full gate must run
before we commit anything.

#### ~~Video role-direction test~~ — KILLED Aug 6, by its own pre-stated rule

**What the idea was:** test role directions in video, using an automatic judge
to score the construct.

**Why it is dead.** The judge audit ran on VELOCITI's Agent Binding test, using
the official gated release; the owner accepted the license, and we used an
order-robust A∧B protocol. The best of four judges scored 67.3% [59.5, 74.3],
against a bar of 85%. That is five standard errors short. Humans score 92.9%.
So no automatic judge exists for this construct, and the 300–600 GPU-h
direction cannot be built. A clean kill for 0.4 GPU-h.

**Where the evidence lives:** decisive0806 `runs/partA_velociti/`

#### ~~Algebraic role-binding embeddings~~ — SCOOPED Aug 6 (Level 1, full overlap)

**What the idea was:** a frozen CLIP tower plus a trained algebraic composition
head, giving one vector per side, scored by cosine similarity, and evaluated on
subject–object swaps.

**Why it is dead.** The pre-design re-check found
[DisCoCLIP (2509.21287)](https://arxiv.org/abs/2509.21287), from Sept 2025. It
is a frozen CLIP tower, plus a small trained tensor-product composition head,
plus one vector per side scored by plain cosine, evaluated on subject–object
swaps with a commutative control. That is our claim on all four axes, in the
same cacheable dual-encoder regime. A second blocker:
[2605.31503](https://arxiv.org/abs/2605.31503), ICML 2026, publishes our
diagnosis AND the multiplicative-mechanism thesis, with evidence against the
frozen-CLIP route. A third: [2608.00726](https://arxiv.org/abs/2608.00726), Aug
1, publishes the probe program's core finding, that patch tokens retain binding
while the global embedding is the bottleneck.

**Why our earlier check missed it.** The comforting result "43 HRR papers, 0 on
CLIP" was keyed to VOCABULARY. DisCoCLIP does the same algebra under the name
"tensor network" and never says HRR.

**Honest residue, diagnostic rather than method:** DisCoCLIP's 93.68% is
measured on 95 pairs with no text-prior control, and our probe says the pooled
vector it scores is role-blind. Auditing that number is cheap, and it would be
a real correction. An owner decision is still needed on the plan to turn the
probe into a method paper.

**Where the evidence lives:** cropdistill
`.orchestrator/tasks/rb-design-20260806-01/stage1_verdict.json`

#### ~~Anthropic VLA-supervision replication~~ — KILLED Aug 6 (false premise)

**What the idea was:** replicate a result from Anthropic's robotics post about
supervising a VLA model.

**Why it is dead.** The [post](https://www.anthropic.com/research/claude-plays-robotics)
finds the OPPOSITE of what our row claimed: "every tested model still performs
substantially worse than MolmoAct does on its own." Supervision *hurts*. The
headline cell has n=200, not 36; the n=36 cell is the one place where the
authors chose to report confidence intervals. Separately, a replication is not
a method under our definition. The missing repo is a corrected fact, not the
reason for the kill.

**Residue:** a binomial-interval comment that costs zero.

#### ~~T1 narrowed interaction study~~ — KILLED Aug 6

**What the idea was:** a narrowed study of interactions between freezing and RL
algorithm choice.

**Why it is dead.** [PIVOT](https://arxiv.org/abs/2510.16333), accepted at ICLR
2026 — our records missed the acceptance — itself publishes the GRPO arm our row
claimed as its own contribution, in §5, plus PPO and MPO. Its own Table B shows
that every {freeze, update} × {GRPO, PPO} cell already has a running repo. The
only unpublished piece is an interaction term, and interaction terms are
statistics, which the method definition excludes. Its true cost would be about
860 H100-h, from the paper's own figure of 18 h × 8 H100 per recipe. That is
3.5–6× the row's budget.

#### ~~KV three-way interaction~~ — KILLED Aug 6 (unaffordable, not unoriginal)

**What the idea was:** a three-way heads × tying × window factorial at matched
KV bytes.

**Why it is dead.** The idea is genuinely unclaimed; we verified that across
three vocabularies, and the agentic-KV blockers do NOT cover it. But it means
something only at about 1,000 H100-h, which we cannot afford. And
[2606.15378](https://arxiv.org/abs/2606.15378) shows that small-scale versions
measure the *speed* of emergence, not converged quality. The Tsinghua group
behind Cost-Optimal GQA is systematically walking these axes anyway.

**Revival test, at 0 GPU:** check whether their convergence curves leave any
three-way interaction at the converged point.

### ★★½ and below — bench these; run only as cheap side arms

| Direction | ★ | First step |
|---|---|---|
| Accept-rule run-level residual | ★★ | A seed-variance check, about 20 GPU-h. PACE is the mandatory baseline. |
| V2A secondary analysis (salvage) | note-scale | Days of work: bootstrap CIs and power curves on [SynthSync's released 306K ratings](https://arxiv.org/abs/2607.09091). This is a methods note, not a paper. |
| Wei verifier-rule Q1 · polychromic audit · Arora drag-fork addon · SigLIP-2 ladder (next-cycle, Delta) · environment-provenance study | ★★–★★★ | As in version 2. The environment-provenance decision window has lapsed — revisit or drop it at sign-off. |

#### Video coverage re-measurement ✓✓ — pilot DONE Aug 6, 1 GPU-h — ★★½, audit remnant only

**What we found.** The ρ>0.8 kill bar was NOT met: ρ=0.62 [0.48, 0.73]. But a
structural finding decides more than that number does. Off-the-shelf STREAM-D
is a two-sample recall statistic, so it needs a real-video reference. Against
any reference we can get, the generated set and the real set are disjoint:
precision 0.004, coverage 0.001. So the metric reads out the generated set's
own spread, not its coverage of anything.

**What this settles.** The question "coverage of what?" now has a measured
answer: no runnable reference exists for text-to-video. The AUDIT remnant
survives concretely. The metric route is closed.

**Where the evidence lives:** cropdistill `runs/stream_check/report/`

#### ~~Fork-preserving context construction~~ — KILLED Aug 6, at its own 48-h re-check

**What the idea was:** build long-horizon context that preserves forks in the
agent's trajectory.

**Why it is dead.** The long-horizon home that the wave-2 record reserved is
now occupied. [PivoARL (2607.03702)](https://arxiv.org/abs/2607.03702) retries
from the pivotal wrong turn and reuses the correct prefix; its code is
released, and it beats full retry — which was our falsifier — with 42% fewer
turns. [CWL (2606.11213)](https://arxiv.org/abs/2606.11213) ships the eviction
rule that keeps exploration and sheds persisted actions. This area moves at
≥1.7 papers per month, not the ~1 we assumed. What is left for us is two
missing ablations inside someone else's shipped method.

#### ~~Cell B: per-item position split~~ — KILLED Aug 6

**What the idea was:** split benchmark items by whether their answer depends on
position, and treat that as a property of the items.

**Why it is dead.** The pilot ran, for 0.2 GPU-h. 17% of items flag as
position-dependent, which clears the 10% bar. But an equal 17% flag in the
MIRROR direction, and cross-backbone agreement is κ=0.11 against a pre-stated
bar of 0.40. So the per-item split measures model-specific fragility, not a
property of the items.

**One survivor worth keeping.** At the SUBSET level, relation items and
object-swap items reliably lose more to patch shuffle than to accuracy-matched
noise, on both backbones; attribute items go the other way. That is real
signal. But SugarCrepe++'s existing labels already carry it, so there is no new
paper in it.

**Also refuted:** LaSt-ViT's lazy-aggregation prediction. Full shuffle costs
19.8 points, not the ~0 it predicts.

**Where the evidence lives:** cropdistill `runs/cellB_pilot/20260806-report/`

#### ~~Spectral-band × knowledge injection (MiCA)~~ — KILLED Aug 6

**What the idea was:** inject knowledge into a specific spectral band of the
weights, following MiCA's claim about minor subspaces.

**Why it is dead.** The K1 reproduction gate ran — 1.8 GPU-h, 192 training runs
— and FAILED. With an exact reimplementation, whose trainable-parameter count
matches the paper, the minor-subspace arm never beats the random-subspace
control at any setting. The result is stronger than a null: on the most direct
absorption measure the ordering REVERSES, and the major subspace absorbs new
text better than the minor one on 8 of 8 seeds. No subspace arm acquires any
retrievable knowledge, despite near-zero training loss. The paper's own
standard errors exceed its claimed gains, and its corpus, questions, and
ablation settings were never released. O1 is dead.

**Where the evidence lives:** cropdistill `runs/mica_k1/2026-08-06/RESULT.md`

## Part 3 — Killed, vetoed, or expired (do not propose these again)

**Aug-6 pilot kills.** Two pilots killed their own directions.

- Cell A readout-budget frontier: killed by its own pre-stated rule, at −21
  points, with the CI excluding zero in the wrong direction on both backbones.
  Details are in Part 1.
- Cell B per-item position split: κ=0.11 against the 0.40 bar, with equal flags
  in both directions. The surviving subset-level effect already lives in
  SugarCrepe++'s own labels. Details are in the Part 2 row.

Two Cell A by-products survive, and they feed Cell B. First, patches help
retrieval monotonically but never help binding — a clean dissociation on 4,757
items. Second, the apparent sign inversion at battery scale is a MaxSim
caption-length artifact; the real inversion lives only in unprojected token
space, which matches [[Binding-Root-Cause-Analysis]] §8.

**Aug-5 checks.** Three directions closed.

- Language-necessity index: a Level-1 kill.
  [2606.04233](https://arxiv.org/abs/2606.04233) published the cross-benchmark
  study in June, and LIBERO-Plus ran real policies blind.
- V2A metric validation: killed by
  [SynthSync 2607.09091](https://arxiv.org/abs/2607.09091), with 306K
  annotations, plus Omni-Judge, PEAVS, and AVBench.
- Seed-noise and variance of agent leaderboards: **OWNER VETO.** The owner's
  reasons: not significant, no barrier, and not novel. The veto includes the
  design-layer arm.

**Waves 1–3 and earlier.** See [[Method-Gates-Wave-2-2026-08]],
[[Method-Gates-Wave-3-2026-08]], and version 2 in git history. The list:
autoresearch accept rule (PACE) · cross-scaffold critic (the source group
published it) · active-view spatial (World2VLM/SIMS-V) · compositional merging
(AlignMerge) · frozen-VLM binding heads (DCSM/Q-Former) · KV from-scratch
factorial (CLA/MixAttention) · late-interaction distillation (ComAlign) ·
spectral-PEFT LR-artifact question (answered 4×) · verifier hardening ·
safety-aware KV · Hyperball · CAID · multi-agent budget-matching · LPT control ·
PULSE · BPP (LIBERO-Gen does not exist) · DriveJudge (labels unreleased; on the
watchlist) · weather · novelty forensics (the biology side owns it) · T4
anneal-window · B1 diversity attribution.

## Part 4 — The cheap decisive steps, gathered in one list

1. ~~Fisher-z afternoon~~ DONE Aug 6 — the headline problem EXISTS, so the audit
   proceeds to design. Details are in the Part 2 row.
2. ~~VELOCITI judge audit~~ DONE Aug 6 — no judge clears 85%, so the video role
   direction is killed.
3. ~~Cell A dynamic-range pilot~~ DONE Aug 6 — kill-arm 2 fired, so Cell A is
   dead.
4. MOCHI — 20–60 GPU-h — RUNNING as of Aug 6, on GPU 1 of the held allocation.
5. ~~STREAM-D vs dispersion~~ DONE Aug 6, for 1 GPU-h — the metric route is
   closed, and the audit remnant survives. Details are in the Part 2 row.
6. ~~MiCA repro gate~~ DONE Aug 6, for 1.8 GPU-h — FAILED, so O1 is dead. The
   subspace ordering reverses on absorption. Details are in the Part 2 row.
7. ~~Owner: verb labeling sessions~~ DONE Aug 6 — 401/401 items labeled. The
   powered rerun corrected the below-chance score claim, which turned out to be
   noise, and confirmed the region-readout inversion
   ([[Binding-Root-Cause-Analysis]] §8).

## Part 5 — Standing lessons

All of version 2's lessons still stand. These were added later.

6. **Scout ratings are provisional by about 1.5–2 stars** (Aug 5). Never cite a
   scout star in a decision document.
7. **Zero-citation papers under eight weeks old are the main source of scoops
   now** (Aug 5). Recency-weighted search matters more than breadth.
8. **Check our own wiki first** (Aug 5). Two ideas this month died to papers
   that our own earlier sweeps had already surfaced, for a different purpose.
9. **Emptiness checks must key on the SHAPE of a mechanism, not on one
   community's vocabulary** (Aug 6). The "43 HRR papers, 0 on CLIP" search gave
   us eleven months of false confidence, while DisCoCLIP did the same algebra
   under the name "tensor network."
10. **Read the source paper before rating any row that cites one** (Aug 6). A
    row reached ★★★★ and a funded task while misstating what its source paper
    claims. Ten minutes with the abstract would have caught it.
11. **Pull the source paper's citing papers as a required gate pass** (Aug 6).
    Twice in one day, the citation graph found the blocking work when keyword
    search did not — those papers were indexed under "forgetting" and
    "overtraining," not "replay."
12. **Read the released artifacts first** (Aug 7). Four times in one week — for
    MiCA, Fisher-z, VELOCITI, and Spatial-IQ — the decisive finding came from
    code, prompt files, or per-item outputs rather than from the paper, in
    hours and at near-zero cost. And single confident passes are not
    trustworthy: adversarial peer-checking corrected four framings before they
    reached the owner.

## Related

[[Direction-Scouts-2026-08-05]] · [[Method-Gates-Wave-3-2026-08]] ·
[[Compositional-VLM-Survey]] · [[Binding-Root-Cause-Analysis]] ·
[[Prereg-RoboJudge-Audit]] · [[Prereg-Crop-Consistency-Distillation]] ·
[[Prereg-1NFE-Diversity]] · [[Prereg-Epistemic-Contextualization]] ·
[[Top-Researcher-Scan-2026-08]] · [[Home]]
