# Gate — does RLVR change the calibration of the self-consistency agreement signal?

Task `rlvr-calib-20260806-01`, run 2026-08-06. This page records Stage 1: the
formal check that had to pass before any GPU time was spent. It carries out the
first step named in the [[Unified-Direction-Ranking-2026-08]] row "RLVR vs
self-consistency calibration" (★★★★, 100–250 GPU-h, "verify OLMo-3 checkpoint
availability"). The idea came from the D. Zhou row of
[[Top-Researcher-Scan-2026-08]].

**Verdict: BOTH HALVES PASS. Level 3 — Medium Overlap on the scoop check
(the kill bar was Level ≤2), and a checkpoint pair exists that isolates the
RLVR stage.** The pilot was cleared to run.

## The idea in plain words

Self-consistency means: ask a model the same question k times, then keep the
answer that appears most often. The **agreement fraction** is the share of
those k answers that match the winner. People treat that share as a confidence
number.

The share is **calibrated** if it means what it looks like. A share of 0.8
should mean the winner is right about 80% of the time.

**RLVR** stands for reinforcement learning from verifiable rewards. It is a
post-training step where a checker (for example, a math answer checker) says
whether an answer is right, and the model is trained to produce more right
answers. RLVR is known to make a model's outputs less varied.

The question: does RLVR move the line between agreement and correctness? If it
does, then every pipeline that votes over an RLVR-trained model is using a
confidence number that no longer means what it used to.

## Half 1 — scoop check (two adversarial passes)

A **scoop check** asks whether someone already published the idea. We ran two
passes with different wording, plus a sweep of OpenReview (the site where
conference submissions are public before acceptance).

**Verdict: Level 3 — Medium Overlap.** In our scale, Level 5 means no
overlapping work and Level 1 means fully scooped. The bar for killing this row
was Level 2 or worse. Level 3 clears it.

Three nearby literatures exist. None of them makes our claim:

- **(a) RLVR makes outputs less varied.** This is about how spread out the
  answers are, not about whether the agreement share predicts correctness.
- **(b) Calibration of *stated* confidence.** Here the model writes a number
  like "I am 80% sure", or the confidence comes from token probabilities.
  Different signal.
- **(c) Better ways to combine sampled answers.** These papers improve voting
  but never compare a model from before RLVR with the same model after RLVR.

### The papers that came closest

| Paper | Date | What it actually does | Why it is not the same claim |
|---|---|---|---|
| [Post-Training Shifts Confidence: A Three-Stage Analysis of How SFT, RL, and OPD Shape CoT Calibration (2607.13753)](https://arxiv.org/abs/2607.13753) | 2026-07-15, ~0 citations | Trains one Qwen2.5-7B-Instruct backbone into SFT, RL and distilled versions, then asks which confidence signal helps at three points of the reasoning trace, including answer aggregation | **The closest paper.** Its confidence is the average token probability of a trace, not the agreement share. It never draws a reliability curve or computes ECE on the vote share. Its own Limitations section names "consistency across samples" as a signal it did **not** study |
| [How Uncertainty Estimation Scales with Sampling in Reasoning Models (2603.19118)](https://arxiv.org/abs/2603.19118) | 2026-03-19 | Uses the agreement share as a confidence signal in three RLVR-trained reasoning models over 17 tasks | Section 2.3 explicitly **refuses** to compute ECE on the agreement share, and reports only AUROC (a ranking measure). It also has no pre-RLVR checkpoint — every model it uses is already RL-trained |
| [Decoupling Reasoning and Confidence: Resurrecting Calibration in RLVR (2603.09117)](https://arxiv.org/abs/2603.09117) | 2026-03-10, ICML 2026 | Shows RLVR makes models overconfident, with reliability diagrams and ECE above 0.3, then proposes a fix (DCPO) | The confidence is a number the model writes out in words. Bucket (b) |
| [Certified Self-Consistency (2510.17472)](https://arxiv.org/abs/2510.17472) | 2025-10-20 | Proves majority voting identifies the most likely answer under the model's own distribution, and that RL sharpening reduces the samples needed | Certifying the *most likely answer* is not certifying the *correct* answer. Bucket (a)+(c) |
| [Calibration Collapse Under Sycophancy Fine-Tuning (2604.10585)](https://arxiv.org/abs/2604.10585) | 2026-04-12 | Base / SFT / RL three-way ECE comparison on MMLU | Closest **method template** to ours, but the signal is stated confidence and the reward is sycophancy, not a verifier. Its effect was not significant (p = 0.41) |
| [Uncalibrated Reasoning: GRPO Induces Overconfidence for Stochastic Outcomes (2508.11800)](https://arxiv.org/abs/2508.11800) | 2025-08-15 | GRPO's group standard normalization makes stated probabilities overconfident; PPO and RLOO stay calibrated | Bucket (b), and about probability predictions for random outcomes, not agreement. Useful because it names a **mechanism** we could borrow |

### The delta sentence

> Unlike [2607.13753](https://arxiv.org/abs/2607.13753), which measures how
> post-training reshapes the *token-probability* confidence of a reasoning
> trace and uses it to filter traces before voting, and unlike
> [2603.19118](https://arxiv.org/abs/2603.19118), which measures the *ranking*
> quality (AUROC) of the agreement share inside models that are already
> RL-trained and deliberately declines to compute its calibration error, this
> work measures the agreement-share-to-accuracy mapping itself — a reliability
> curve and an ECE — across a matched pre-RLVR / post-RLVR checkpoint pair,
> which is the only evidence that tells a pipeline builder whether an 8-out-of-16
> agreement threshold still means 80%.

### Honest limits of this check

- The second pass could not use WebSearch; that call budget was already spent
  for the session. The arXiv API sweeps (about 70 distinct queries across both
  passes) and the OpenReview sweep both ran in full, and the first pass also
  went through Semantic Scholar, OpenAlex, Crossref and DBLP. So the check is
  multi-channel, but one channel is missing and must be repeated before any
  submission.
- OpenReview's `/notes` endpoint now returns a 403 challenge, so we could
  search submissions but could not list a venue's full submission set.
- [2607.13753](https://arxiv.org/abs/2607.13753) is three weeks old with about
  zero citations. Under the standing lesson that recent zero-citation papers
  are the main scoop source, **re-check it before any pre-registration**.

## Half 2 — do checkpoints exist that isolate the RLVR stage?

Yes. Ai2's OLMo-3 family releases every post-training stage. Every OLMo-3
post-training card carries the same table with rows **Base Model / SFT / DPO /
Final Models (RLVR)**, plus a section reading "Stage 3: RLVR — reinforcement
learning from verifiable rewards on the Dolci-…-RL dataset". Each checkpoint's
`base_model` field confirms the chain link by link, so the DPO → final step is
RLVR and nothing else.

| Family | Size | Pre-RLVR checkpoint | Post-RLVR checkpoint | What the step isolates |
|---|---|---|---|---|
| OLMo-3 Instruct | 7B | [Olmo-3-7B-Instruct-DPO](https://huggingface.co/allenai/Olmo-3-7B-Instruct-DPO) | [Olmo-3-7B-Instruct](https://huggingface.co/allenai/Olmo-3-7B-Instruct) | RLVR only (short answers — cheapest usable pair) |
| OLMo-3 Think | 7B | [Olmo-3-7B-Think-DPO](https://huggingface.co/allenai/Olmo-3-7B-Think-DPO) | [Olmo-3-7B-Think](https://huggingface.co/allenai/Olmo-3-7B-Think) | RLVR only (long chain-of-thought) |
| OLMo-3 RL-Zero | 7B | [Olmo-3-1025-7B](https://huggingface.co/allenai/Olmo-3-1025-7B) (base) | [Olmo-3-7B-RL-Zero-Math](https://huggingface.co/allenai/Olmo-3-7B-RL-Zero-Math) | RLVR applied straight to the base, no SFT or DPO at all |
| OLMo-3 Think | 32B | [Olmo-3-32B-Think-DPO](https://huggingface.co/allenai/Olmo-3-32B-Think-DPO) | [Olmo-3-32B-Think](https://huggingface.co/allenai/Olmo-3-32B-Think) and [Olmo-3.1-32B-Think](https://huggingface.co/allenai/Olmo-3.1-32B-Think) | RLVR only; two different RL runs from one input |
| Tülu-3 | 8B | [Llama-3.1-Tulu-3-8B-DPO](https://huggingface.co/allenai/Llama-3.1-Tulu-3-8B-DPO) | [Llama-3.1-Tulu-3-8B](https://huggingface.co/allenai/Llama-3.1-Tulu-3-8B) (PPO) and [Llama-3.1-Tulu-3.1-8B](https://huggingface.co/allenai/Llama-3.1-Tulu-3.1-8B) (GRPO) | RLVR only; also a free comparison of two RL algorithms |
| OLMo-2 | 1B | [OLMo-2-0425-1B-DPO](https://huggingface.co/allenai/OLMo-2-0425-1B-DPO) | [OLMo-2-0425-1B-Instruct](https://huggingface.co/allenai/OLMo-2-0425-1B-Instruct) | RLVR only; smallest RLVR pair anywhere in the Ai2 lineup |

Three facts worth keeping:

- **There is no OLMo-3 at 1B.** The family has only 7B and 32B. So the smallest
  OLMo-3 pilot is 7B.
- **The whole RL run is public, not just its endpoints.**
  `Olmo-3-7B-Think` has 55 `step_*` git branches and
  `Olmo-3-7B-RL-Zero-Math` has 19. A follow-up study can sample the middle of
  training, not only before and after.
- **Both sides of an OLMo-3 pair recommend the same sampling settings**
  (temperature 0.6, top_p 0.95), so neither checkpoint is favoured by using its
  own recommended setting. The RL-Zero variants state nothing and their base
  has no chat template, so that pair needs prompt formatting matched by hand.

## What the pilot then did

The pilot design, the pre-stated verdict rule and the answer-extraction rules
were fixed in writing before any samples were generated:
`cropdistill/rlvr/PREREGISTERED.md`. Results live in
`cropdistill/runs/rlvr_calib/20260806/`.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Top-Researcher-Scan-2026-08]] ·
[[Method-Gates-Wave-3-2026-08]] · [[Calibration-Prior-Art-Gate]] (a different
calibration idea, on image-text matching) · [[Home]]
