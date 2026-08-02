# LLM/VLM Distillation — Survey and Gates

Status: **Surveyed and gated 2026-07-25.** The on-policy capacity-gap candidate
is killed. Two candidates survive, and they came from a **different generating
process** that is worth naming.

## The generating-process lesson

Ten candidates have now been gated; ten died. Every one that died was generated
the same way: **scan recent literature for an apparently empty area.** The gates
kept finding the area was empty because the experiment sits in an appendix, the
concept was named under other vocabulary, or the move is one line for an
insider.

The two survivors below were generated differently: **find a specific published
claim whose evidence is visibly inadequate.** That is the same process that
produced this group's two real findings — the HF CLIP double-projection bug and
the QuickGELU reference mismatch. It has a far better hit rate because a
documented evidentiary weakness cannot be scooped by an appendix; it is a fact
about a paper that is already in print.

**Rule going forward: generate from inadequate evidence, not from empty space.**

## Killed: on-policy distillation x capacity gap

Three independent problems, any one survivable, together fatal.

**1. Already run, in print.** DistiLLM-2 (`2503.07067`, ICML 2025) Table 8
crosses teachers `{1.8B, 7B, 14B}` (Qwen1.5-Chat) into a fixed 0.5B student,
comparing GKD (on-policy) against DistiLLM (off-policy). Result:
`64.18 / 71.15 / 72.59` on-policy versus `65.23 / 72.11 / 72.11` off-policy —
**monotonically increasing, no U-shape**, with the on-policy advantage crossing
from negative to positive at 14B. A weak, uncontrolled version of the predicted
effect is already published.

**2. Registered as someone's to-do.** The on-policy distillation survey
(`2604.00626`, 89pp) notes Busbridge's non-monotonicity was obtained "under an
off-policy setting," conjectures an interference term, and states verbatim:
*"Future work could conduct controlled grid searches that independently vary
N_S, N_T, and D_on under on-policy training."*

**3. My imitation-learning framing was backwards.** I argued that because DAgger
assumes realizability, on-policy should *not* fix a capacity gap if the gap is a
realizability failure. The literature says the opposite:

- Foster, Block & Misra (`2407.15007`, NeurIPS 2024): **under realizability**,
  online IL *cannot* improve over offline IL.
- Espinosa-Dice et al. (`2503.13162`, ICLR 2025): **under misspecification**,
  interaction is *fundamentally required*.
- Zhang et al. (`2606.30445`, Jun 2026) proves an information-theoretic barrier
  for offline IL under misspecification even at horizon 1, states in a footnote
  that strong-to-weak distillation **is** the non-realizable setting, and
  empirically finds that with a realizable expert SFT matches it while
  on-policy adds nothing. It uses the same models as `2604.13016`.

So non-realizability is precisely where on-policy is *theorized to help*, the
prediction is published with theorems, and the outcome was largely determined in
advance.

**4. The premise is probably false at task scale.** Every teacher ladder found
at post-training scale is monotonic: DistiLLM-2 above, and Visual-Advantage OPD
(`2605.21924`) sweeping Qwen3-VL `{4B, 8B, 32B}` under on-policy reporting the
gain "growing monotonically along the teacher-size axis." **No genuine U-shape
has been reported for task distillation with a modern model family.** The
experiment presupposes the effect it intends to explain.

**5. The question is closed anyway.** ACL 2025 main, *Towards the Law of
Capacity Gap in Distilling Language Models*, inducts a law: optimal teacher
scale is **linear in student scale**, verified to 7B.

Also dead per the same sweeps: forward-versus-reverse KL (`2404.02657`, COLING
2025, finds "neither mode-seeking nor mean-seeking properties manifest in KD for
LLMs"); per-token adaptive temperature (`2510.11615`); compute-matched KD versus
SFT (Busbridge, 69pp).

## Survivor 1: the LLM-judge noise floor in KD, anchored on a documented regression

**The concrete artifact.** DistiLLM (ICML 2024) reported mean and standard
deviation over 5 seeds. **DistiLLM-2 (ICML 2025 Spotlight) reports zero seeds**
(`grep -ic seed` = 0). Its headline Table 2, Qwen2-7B into 1.5B, GPT-4o win
rate:

```
GKD        56.14
DistiLLM   56.35     <- delta = 0.21, single run, no error bar
DistiLLM-2 58.69
```

A GKD-versus-DistiLLM ordering cited downstream as settled rests on **0.21
points of an LLM-judge win rate with no variance estimate**, in a Spotlight,
from a lineage whose own predecessor reported error bars.

**The study.** Decompose the noise floor into its sources — training seed, judge
model, judge sampling seed, decoding temperature — then overlay published deltas
from the KD literature and report what fraction sit below it. Comparable seed
practice elsewhere: MiniLLM 5 seeds, GKD 3 seeds, DistiLLM 5 seeds with std.

**Feasibility.** DistiLLM-2's own setup is 4x A100-80GB with LoRA; all four
codebases are public. Reproducible on 8 GPUs.

**Competition.** EasyOPD (`2607.11012`) states the thesis verbatim in its
abstract — "fragmented implementations that are difficult to reproduce and
extend" — and then **ships a library instead of an audit**. That is the citation
and the only visible competitor. No survey flags comparability: Xu et al.
(`2402.13116`, 43pp), Yang et al. (`2407.01885`), Mansourian et al.
(`2503.12067`, TMLR, 102pp).

## Survivor 2: contamination dose-response through distillation

**The question.** Does eval-set contamination in a teacher transfer to a student
through *generative* distillation, and does it survive standard decontamination
of the distillation corpus?

**Why the dose-response framing matters.** The researcher controls the dose, so
no frontier teacher is needed: contaminate an open 7B model on `k` copies of a
held-out eval slice, distill to 1.5B under SeqKD / logit-KD / on-policy, and
measure lift against dose and objective — **with n-gram and embedding
decontamination applied to the distillation corpus**, plus a test of whether
Min-K%, Min-K%++ and ConStat fire on the *student*.

**Four gates already named.** DCLLM (ACL ARR `l5MK6mLjFi`, resubmitted
`mmbmGjQUQb`, no acceptance visible) measures teacher-contamination inflating
student scores as a route to a decontamination method. The memorization twin is
well executed twice: `2508.07054` runs 6 KD techniques x 7 tasks x 3 teacher
families; `2601.15394` finds hard distillation inherits **2.7x** more
teacher-specific examples than soft. `2604.15559` shows a student inheriting an
unsafe behavior at 100% versus 5% baseline **despite full keyword sanitation**.
`2510.02386` (ICLR 2026) shows brief GRPO training conceals contamination and
makes detectors near-random.

**Framing constraint.** Must be pitched as **benchmark validity**, not as "does
memorization transfer" — the latter is `2508.07054` and `2601.15394` and will be
gated on sight.

**The fusion idea, strongest of the three.** Is "distilled reasoning ability"
partly inherited contamination? Nobody has separated reasoning-transfer from
format-transfer on a contamination-controlled out-of-distribution set. Setup
exists: `2502.12143` (models under 3B do not consistently benefit from long-CoT
distillation) and `2606.21704`.

## Retractions recorded

An agent retracted three of its own claims mid-survey, correctly: that seed
variance is essentially never reported in LLM KD (false — MiniLLM 5, GKD 3,
DistiLLM 5 with std; the failure is specific to DistiLLM-2); that compute-matched
KD versus SFT is open (taken by Busbridge at 69 pages); and that teacher size is
the live question (closed by the ACL 2025 capacity-gap law).

## Related

[[Direction-Gate-Results]] — the earlier gates and the same failure modes.
[[Temperature-Confound-Preregistration]] — the owner-run gate that preceded this.
