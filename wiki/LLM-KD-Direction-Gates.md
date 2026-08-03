# LLM/VLM Knowledge Distillation — Survey and Checks

*Updated 2026-08-02 for the general research wiki.*

**Knowledge distillation (KD)** trains a smaller student model to copy a larger
teacher model. LLM means large language model, and VLM means vision-language
model.

Later update: see [[Direction-Reevaluation-2026-08]]. Its T4 review found that
our July search called an area empty even though the paper that had already
filled it had been public for about eight weeks. The area was not empty when we
made the claim. This is separate evidence for the idea-generation rule below.

Status: **Surveyed and gated 2026-07-25.** The on-policy capacity-gap idea is
dead. Two ideas survived. They came from a different way of finding ideas, and
that difference matters.

**Later correction:** the judge-noise survivor below led to
[[KD-Noise-Floor-Stage1]], which was **SUSPENDED on 2026-07-26**. Its numbers
and design were sound, but the field already knew the problem and had no
incentive to change its reporting practice. The two-survivor statement above
is the historical result of the 2026-07-25 gate, not the current go-ahead.

## A better way to find research ideas

Ten ideas had been checked, and all ten failed. Every failed idea began the
same way: **search recent papers for an area that looks empty.** Later checks
found that the area only looked empty because the experiment was hidden in an
appendix, the same idea used different words, or an expert could do the change
in one line.

The two ideas that survived came from a different question: **which exact
published claim does not have enough evidence?** That question also led to this
group's two real findings: the Hugging Face CLIP double-projection bug and the
QuickGELU reference mismatch. Those are SVIB-project findings, with full detail
in the svib repo wiki. This approach works better because an already printed
paper's weak evidence is a fixed fact. An appendix in a later search cannot
erase it.

**Rule from now on: start with weak evidence in a real claim, not with an area
that looks empty.**

## Rejected idea: on-policy distillation and the capacity gap

A **capacity gap** happens when a larger, stronger teacher gives a smaller
student worse results than a somewhat smaller teacher. **On-policy
distillation** teaches from examples produced while the student is running;
**off-policy distillation** teaches from a fixed set made earlier.

This idea has several independent problems. Any one might be fixable, but all
of them together end the direction.

### 1. The experiment is already in print

DistiLLM-2 ([`2503.07067`](https://arxiv.org/abs/2503.07067), ICML 2025) Table 8 uses teachers of
`{1.8B, 7B, 14B}` parameters from Qwen1.5-Chat with one fixed 0.5B student. It compares
[GKD](https://arxiv.org/abs/2306.13649), which is on-policy, with DistiLLM, which is off-policy. The scores are
`64.18 / 71.15 / 72.59` for on-policy and `65.23 / 72.11 / 72.11` for
off-policy. The on-policy scores rise steadily, with **no U-shape**. Its
advantage changes from negative to positive at 14B. This is a weak and
uncontrolled version of the predicted effect, but it is already published.

### 2. Another paper already lists it as future work

The on-policy distillation survey ([`2604.00626`](https://arxiv.org/abs/2604.00626), 89 pages) says that
[Busbridge](https://arxiv.org/abs/2502.08606)'s non-monotonic result came from an "off-policy setting." It suggests
an interference term and says:

> *"Future work could conduct controlled grid searches that independently vary
> N_S, N_T, and D_on under on-policy training."*

### 3. My imitation-learning argument pointed the wrong way

I argued that [DAgger](https://arxiv.org/abs/1011.0686) assumes **realizability**—the student can represent the
target behavior—so on-policy learning should not fix a gap caused by failure of
that assumption. Earlier work says the opposite:

- Foster, Block & Misra ([`2407.15007`](https://arxiv.org/abs/2407.15007), NeurIPS 2024) show that, **when the
  target is realizable**, online imitation learning (IL) cannot improve over
  offline IL.
- Espinosa-Dice et al. ([`2503.13162`](https://arxiv.org/abs/2503.13162), ICLR 2025) show that, **when the model
  cannot exactly represent the target**, interaction is fundamentally needed.
- Zhang et al. ([`2606.30445`](https://arxiv.org/abs/2606.30445), Jun 2026) prove an information-theoretic limit
  for offline IL in that second setting, even when the horizon is 1. A footnote
  says strong-to-weak distillation is exactly this non-realizable setting. Their
  experiments find that supervised fine-tuning (SFT) matches a realizable
  expert and on-policy training adds nothing. They use the same models as
  [`2604.13016`](https://arxiv.org/abs/2604.13016).

So theory predicts that on-policy training helps exactly when the student
cannot fully copy the teacher. The outcome was mostly known before this study.

### 4. The assumed U-shape is probably absent at task scale

Every modern teacher-size ladder found for post-training is monotonic. Besides
DistiLLM-2, Visual-Advantage OPD ([`2605.21924`](https://arxiv.org/abs/2605.21924)) tests Qwen3-VL teachers of
`{4B, 8B, 32B}` with on-policy distillation and says the gain grows
"monotonically along the teacher-size axis." **No real U-shape has been
reported for task distillation within a modern model family.** The experiment
assumes the effect it hopes to explain.

### 5. The larger question is already answered

The ACL 2025 main paper *Towards the Law of Capacity Gap in Distilling Language
Models* gives a rule: the best teacher size grows **linearly with student
size**, tested up to 7B parameters.

The same searches also closed these ideas: forward versus reverse KL
([`2404.02657`](https://arxiv.org/abs/2404.02657), COLING 2025, finds that "neither mode-seeking nor mean-seeking
properties manifest in KD for LLMs"); per-token adaptive temperature
([`2510.11615`](https://arxiv.org/abs/2510.11615)); and compute-matched KD versus SFT (Busbridge, 69 pages).

## Surviving idea 1: the LLM-judge noise floor in KD

A **noise floor** is the amount a score changes from randomness alone. If two
methods differ by less than that, their order may not be trustworthy.

### The published example

DistiLLM (ICML 2024) reports the mean and standard deviation over 5 seeds.
**DistiLLM-2 (ICML 2025) reports zero seeds** (`grep -ic seed` = 0). This page
originally called it a Spotlight; [[KD-Evidence-Audit-Gate]] later corrected
that label to **ICML 2025 Oral (top 1%)**. Its main Table 2 reports the following
for Qwen2-7B distilled into Qwen2-1.5B:

```text
GKD        56.14
DistiLLM   56.35     <- delta = 0.21, single run, no error bar
DistiLLM-2 58.69
```

The table mixes two judges, so the earlier phrase "GPT-4o win rate" was also
wrong: GPT-4o judges AlpacaEval and Evol-Instruct, while GPT-4o-mini judges
UltraFeedback. A downstream claim that GKD is below DistiLLM rests on a
**0.21-point difference from one run with no variance estimate**, in an Oral
paper whose predecessor reported error bars.

### The proposed study

Measure how much variation comes from the training seed, judge model, judge
sampling seed, and decoding temperature. Then place published KD score
differences on top of that noise floor and report how many differences are
smaller. Other KD papers show that multi-seed reporting is possible:
[MiniLLM](https://arxiv.org/abs/2306.08543) uses 5 seeds, [GKD](https://arxiv.org/abs/2306.13649) uses 3, and DistiLLM uses 5 with standard
deviations.

### Cost and nearby work

DistiLLM-2 uses 4x A100-80GB GPUs with LoRA, and all four codebases are public.
It could be reproduced on 8 GPUs.

EasyOPD ([`2607.11012`](https://arxiv.org/abs/2607.11012)) says in its abstract that implementations are
"fragmented" and hard to reproduce and extend. It ships a library, **not an
audit**, so it is both the closest citation and the only visible competitor.
Three surveys do not flag comparability: Xu et al. ([`2402.13116`](https://arxiv.org/abs/2402.13116), 43 pages),
Yang et al. ([`2407.01885`](https://arxiv.org/abs/2407.01885)), and Mansourian et al. ([`2503.12067`](https://arxiv.org/abs/2503.12067), TMLR, 102 pages).

## Surviving idea 2: how contamination passes through distillation

**Question:** if a teacher has seen evaluation examples, does generative
distillation pass that contamination to the student? Does it remain after the
usual cleanup of the data used for distillation?

### Why measure the amount of contamination

The researcher can control the contamination amount, so a closed frontier
teacher is not needed. Add `k` copies of a held-out evaluation slice to the
training of an open 7B model. Distill it into a 1.5B model using [SeqKD](https://arxiv.org/abs/1606.07947),
logit-KD, or on-policy KD. Measure how much scores rise with the contamination
amount and the training objective. Apply both n-gram and embedding-based
cleanup to the distillation data. Then test whether [Min-K%](https://arxiv.org/abs/2310.16789), [Min-K%++](https://arxiv.org/abs/2404.02936), and
[ConStat](https://arxiv.org/abs/2405.16281) detect the contamination in the **student**.

### Four required checks already identified

- DCLLM (ACL ARR [`l5MK6mLjFi`](https://openreview.net/forum?id=l5MK6mLjFi), resubmitted as [`mmbmGjQUQb`](https://openreview.net/forum?id=mmbmGjQUQb), with no
  visible acceptance) measures how teacher contamination raises student scores
  as part of a cleanup method.
- [`2508.07054`](https://arxiv.org/abs/2508.07054) studies memorization with 6 KD methods, 7 tasks, and 3 teacher
  families.
- [`2601.15394`](https://arxiv.org/abs/2601.15394) finds that hard distillation inherits **2.7x** more
  teacher-specific examples than soft distillation.
- [`2604.15559`](https://arxiv.org/abs/2604.15559) shows a student copying an unsafe behavior at 100%, compared
  with a 5% baseline, **even after complete keyword cleanup**.
- [`2510.02386`](https://arxiv.org/abs/2510.02386) (ICLR 2026) shows that brief [GRPO](https://arxiv.org/abs/2402.03300) training hides contamination
  and makes detectors perform near random chance.

This must be presented as a **benchmark-validity** study: can we trust the test
score? It must not be presented as "does memorization transfer?" because
[`2508.07054`](https://arxiv.org/abs/2508.07054) and [`2601.15394`](https://arxiv.org/abs/2601.15394) already answer that broader question.

The strongest combined idea is more specific: is some "distilled reasoning
ability" really inherited contamination? No work found has separated transfer
of reasoning from transfer of answer format on a contamination-controlled,
out-of-distribution set. The setup already exists: [`2502.12143`](https://arxiv.org/abs/2502.12143) finds that
models below 3B do not consistently benefit from long-chain-of-thought
distillation, and [`2606.21704`](https://arxiv.org/abs/2606.21704) provides another relevant setup.

## Claims withdrawn during the survey

An agent correctly withdrew three claims while working:

- Seed variance is almost never reported in LLM KD. This is false:
  [MiniLLM](https://arxiv.org/abs/2306.08543) uses 5 seeds, [GKD](https://arxiv.org/abs/2306.13649) uses 3, and DistiLLM uses 5 with standard
  deviations. The missing report is specific to DistiLLM-2.
- Compute-matched KD versus SFT is open. [Busbridge](https://arxiv.org/abs/2502.08606) already covers it in 69
  pages.
- Teacher size is the live question. The ACL 2025 capacity-gap law already
  closes it.

## Related

[[Direction-Gate-Results]] — earlier checks and the same ways ideas failed.
[[Temperature-Confound-Preregistration]] — the owner-run check that came before
this survey.
