# Next-Direction Literature Survey

Status: **Survey complete 2026-07-25.** This was the first and narrowest survey
of possible next directions. It checked six ideas against work from 2025–2026.
It was written after the Stage-E search for earlier work, in the SVIB repository
wiki, closed the audit-paper path.

**Updated 2026-08-02 for the general research wiki.** This page is a record of
what we checked, not current advice. **Almost every idea below was checked again,
and the new results are on other pages:**

- [[Direction-Gate-Results]] covers the distillation capacity gap, benchmark
  one-dimensionality, succinctness, and C-RASP. The proposed U-shaped
  distillation result was already in an ICML 2025 appendix, so that idea was
  taken.
- [[Temperature-Confound-Preregistration]] covers the distillation-temperature
  follow-up. Its check failed, so do not run that plan.
- [[Calibration-Draw-Preregistration]] covers noise from the chosen calibration
  sample. An ACL 2024 paper already ran the same experiment.
- [[Calibration-Prior-Art-Gate]] closed the calibration/selective-prediction
  line. The gaps truly are unanswered, but a post-hoc calibration change keeps
  the same ordering. It therefore cannot change accuracy, area under the
  risk-coverage curve (AURC), or any risk-coverage curve. A working fix is not
  possible under that plan.
- [[Direction-Reevaluation-2026-08]] and [[Top-Researcher-Scan-2026-08]] give
  the current ranking and opportunity map.

This page shortens project-specific SVIB details such as probe numbers, stage
results, and script names. The full record remains in the SVIB repository wiki.

The table uses three labels. SATURATED means the question is already studied
heavily and we should not enter. ACTIVE means many groups are working on it, so
we need one exact unanswered question. OPEN means this page names such a gap.

## Results for the six directions

| Direction | Verdict | Reason |
|---|---|---|
| Full fine-tuning / better general VLM | **SATURATED, industry-owned** | [VladVA](https://arxiv.org/abs/2412.04378) needs 32×A100. The best models below 4B come from HuggingFace, Alibaba, Google, Apple, or Samsung. [Open-Qwen2VL](https://arxiv.org/abs/2504.00595) is the only academic near miss, and it has ByteDance and NVIDIA authors |
| Visual token selection / pruning | **SATURATED** | About 145 papers per year. [UniPruneBench](https://arxiv.org/abs/2511.02650) finds random pruning competitive and no method always best. ["Are We Solving the Right Problem?"](https://arxiv.org/abs/2502.11501) (ACL Findings 2025) finds that many methods lose to random selection |
| Vision-text tower interaction | **ACTIVE, partly claimed** | [C2LIP](https://arxiv.org/abs/2603.25722) (CVPR 2026) already says "global pooling destroys binding" and uses parameter-free attention pooling at 8×A40. [FLAIR](https://arxiv.org/abs/2412.03561), [FILIP](https://arxiv.org/abs/2111.07783), TFLocal, and [ColPali](https://arxiv.org/abs/2407.01449) exist. Gap: no experiment compares every combination of granularity, interaction, and training signal to separate their effects |
| Gemma/open-VLM as teacher or data generator | **CROWDED, documented failure modes** | [Recap-DataComp-1B](https://arxiv.org/abs/2406.08478) rewrote captions for 1.3B images and gained +3.1% retrieval. [MLLMCLIP](https://openreview.net/forum?id=jZrjHDqTBo), which transfers MLLM features to CLIP, was **withdrawn** from ICLR 2026. ACL 2025 ([2411.05195](https://arxiv.org/abs/2411.05195)) says the generative advantage comes from model design: patch tokens, position embeddings, and prompt weighting. A pooled dual encoder cannot inherit these parts |
| Distillation to smaller models | **Mostly industry; one live niche** | [CompoDistill](https://arxiv.org/abs/2510.12184) (ICLR 2026) transfers *compositional* ability through visual-attention matching: 60.7→66.7, while general knowledge distillation gives 61.5. ["When Better Teachers Don't Make Better Students"](https://arxiv.org/abs/2511.17886) warns that stronger CLIP teachers do not always make better students |
| **Conformal / selective prediction for VL matching** | **ACTIVE field; tested signal closed** | ICLR 2026 already studies risk-coverage for compositional tasks. Our fixed four-model test found no useful gain from dispersion beyond a learned selector that uses only the margin |

**Later results added 2026-08-02.** For row 5, the theory version of the
distillation idea was checked and rejected. See [[Direction-Gate-Results]] and
[[Temperature-Confound-Preregistration]]. Row 6 closed because the proposed fix
cannot change the measurement, not because too many papers exist. See
[[Calibration-Prior-Art-Gate]]. Rows 1–4 were not checked again. Treat their
paper counts from 2026-07 as old information.

## The two specific gaps from the original survey

### Gap 1, revised: does variation across equivalent captions add useful signal for selective prediction?

**Correction recorded 2026-07-25.** The first survey wrongly said that nobody
had tested risk-coverage for compositional matching. **Leveraging Data to Say
No: Memory Augmented Plug-and-Play Selective Prediction** (ICLR 2026,
OpenReview [`wWxdT6LB2D`](https://openreview.net/forum?id=wWxdT6LB2D)) reports AURC and risk-coverage on
[SugarCrepe](https://arxiv.org/abs/2306.14610), [Winoground](https://arxiv.org/abs/2204.03162), [What'sUp](https://arxiv.org/abs/2310.19785), [VL-Checklist](https://arxiv.org/abs/2207.00221), and
[Foil](https://arxiv.org/abs/1705.01359). **Look Again Before You Abstain** ([arXiv `2606.16667`](https://arxiv.org/abs/2606.16667), v4)
also shows that the broader VLM conformal-abstention question is active, not
open. Conformal methods give a statistical promise about error; abstention lets
a model refuse when it is unsure.

The smaller remaining question was tested in Conformal-Probe-Preregistration in
the SVIB repository wiki:

> Does score dispersion across multiple equivalent human captions add
> selective-prediction signal beyond minimum/mean confidence margins and a
> learned margin-only selector?

This was only a check of whether the idea could work, not yet a paper claim.

**Result correction, 2026-07-25.** Conformal-Probe-Results in the SVIB wiki
shows that dispersion beats raw minimum and mean margins. However, on none of
four COCO models does it beat a feature-matched learned margin selector by the
useful amount fixed in advance. The narrow signal test is therefore complete
and negative. The large gap across all equivalent captions remains a useful
description, not a new selection method.

**The whole direction then closed on 2026-07-25.**
[[Calibration-Prior-Art-Gate]] tested the next calibration idea and found a
problem that applies to this whole section. Temperature scaling is monotone: it
changes score values but not their order. For a two-caption choice, it cannot
change which caption wins. AURC and risk-coverage also depend only on the order
of confidence. Both types of measurement stay the same under the only proposed
fix. The decision change is exactly zero.

### Older search record that is no longer current

These full-text arXiv searches returned no results:

- `all:"conformal" AND all:"image-text retrieval"` → **0 hits**
- `abs:"coverage guarantee" AND abs:"image-text matching"` → **0 hits**
- `abs:"learning to defer" AND abs:"vision-language"` → **0 hits**
- `abs:"router" AND abs:"CLIP" AND abs:"uncertainty"` → **0 hits**

The searches were too exact. They missed nearby papers that used "selective
prediction" and benchmark names. Keep them only as a warning about search
process. The three original claims now stand as follows:

1. **Risk-coverage for compositional matching is already claimed.** The ICLR
   2026 paper above reports it directly.
2. **Nobody has made the input-side group of equivalent meanings the unit of
   calibration.** The statistical tools exist but are unused in
   vision-language work: hierarchical exchangeability, where groups and the
   examples within each group can be swapped without changing the statistics
   (Lee/Barber/Willett [2306.06342](https://arxiv.org/abs/2306.06342)); macro-coverage ([2606.28598](https://arxiv.org/abs/2606.28598)); SymmPI
   ([2312.16160](https://arxiv.org/abs/2312.16160)); and equivariant conformal prediction ([2602.03986](https://arxiv.org/abs/2602.03986)). The stronger
   target is **coverage for every wording of the same meaning**:
   `P(correct for every realization of a held-out meaning) >= 1-alpha`.
   Arguably, this is what a compositional claim should promise. No benchmark
   tests it.
3. **The step from description to added prediction signal was not tested in the
   papers we found.** PRSM ([2511.11141](https://arxiv.org/abs/2511.11141), MMM 2026) and LGIP ([2511.13494](https://arxiv.org/abs/2511.13494), Pattern
   Recognition Letters 2026) measure CLIP's instability across paraphrases and
   stop. Our probe asked whether that variation improves selection after
   controlling for the confidence margin.

Nearby but different work to cite clearly: [Conf-OT](https://arxiv.org/abs/2505.24693) studies zero-shot
classification at CVPR 2025; ConfLVLM ([2502.20560](https://arxiv.org/abs/2502.20560)) studies generated claims;
[VL-Uncertainty](https://arxiv.org/abs/2411.11919) studies LVLM hallucinations at CVPR 2025; and probabilistic
embeddings such as [PCME++](https://arxiv.org/abs/2305.18171) and [ProbVLM](https://arxiv.org/abs/2307.00398) produce uncertainty scores but never
give a calibrated refusal rule or risk-coverage curve.

**Data warning.** Captions inside one meaning group must be exchangeable. LLM
paraphrases are *not* exchangeable with human captions because their length and
word choices differ. **COCO has five human captions per image and avoids this
problem.** It is the better data choice. SugarCrepe++ has two positive captions,
which is enough for a pair measurement but weak for finding a separate cutoff
for each group.

**Common reviewer concern to handle early.** The best possible combination of
methods usually looks large but cannot often be achieved. Report this oracle
result beside the result from a real decision rule. Beat both a
confidence-margin baseline and a random baseline at the same coverage. Then
state the refusal rate needed to reach a target risk. An oracle number alone
looks like padding.

### Gap 2, fallback: does effective image detail cause better binding?

One competitor's own numbers show an unexplained reversal. Miranda et al.
([2604.11496](https://arxiv.org/abs/2604.11496), Apr 2026, UPV/EHU) report on BiSCoR-Ctrl that **SGI, which
encodes crops separately without training, scores 24.9**, while **TFLocal, which
uses patch tokens and 13.3M trained parameters, scores 13.2**. On their own test
outside the training distribution, crops beat patch tokens. They do not explain
the result or compare the two routes while changing one part at a time.

Our tests found the same pattern and connected it to image detail. The penalty
from the patch grid becomes about 4× smaller when moving from a 49-token
backbone to a 256-token backbone. Two possible causes are a change in the
contrastive feature space for tight crops, or loss of spatial relationships
during pooling. Nobody has tested them, and our existing tools can. The full
details are in the SVIB wiki page Post-Rebuttal-Measurement-Sprint.

[C2LIP](https://arxiv.org/abs/2603.25722) makes the nearby claim that "final global pooling leads to loss of binding
information." It supports the claim with a model-design argument and attention
maps, not a controlled experiment. That is the exact weak point we could test.

## Important threats before writing a paper

1. **"CLIP Models Generalize Less Than Compositional Benchmarks Suggest"**
   (ICML 2026 CTB workshop) finds a data-overlap problem. In [ARO](https://arxiv.org/abs/2210.01936) VG-A,
   positive captions share COCO attribute-object pairings **79.8%** of the time,
   compared with **41.8%** for swapped negatives. Only 1.2% of examples have no
   pairing seen in COCO. Splits balanced for overlap **change the leaderboard
   and reverse model rankings**. This threatens any paper reporting SCPP++
   changes as small as ours.
2. **Test-Time Matching** ([2510.07632](https://arxiv.org/abs/2510.07632), ICLR 2026) reports that SigLIP-B16
   [Winoground](https://arxiv.org/abs/2204.03162) group score rises `10.25 → 72.50`. It changes the task in two
   ways. GroupMatch has random chance `1/k! = 50%`, while GroupScore has
   `(k-1)!/(2k-1)! = 16.7%`; the paper proves both. TTM also fine-tunes on
   guessed labels from the test set's one-to-one structure. The fair reading is
   `-6.4` above chance → `+22.5` above chance on an easier decision. Reviewers
   will cite it. Keep the chance-level table and the concern about using test
   structure ready in one sentence.
3. **[C2LIP](https://arxiv.org/abs/2603.25722)** ([2603.25722](https://arxiv.org/abs/2603.25722), CVPR 2026, Samsung) directly competes at our
   compute level: 8×A40, CC3M, 5 epochs, with retrieval above the SigLIP
   baseline.
4. **Miranda et al.** ([2604.11496](https://arxiv.org/abs/2604.11496)) is the fair comparison for Gap 2 and is
   actively working on it.

## Why Gap 1 originally fit this group

Three method projects in a row failed because success required beating a
benchmark score. Gap 1 did not have that weakness. Success meant defining a
useful reliability goal and measuring model behavior under it. That matched the
group's careful measurement work. Earlier projects also gave us most of a
deferral study: repair precision and coverage, harm rate, best-possible
headroom, margin-decile firing rates, and a multi-backbone tool that repeats six
outside systems. The full details are in the SVIB wiki page
Post-Rebuttal-Measurement-Sprint.

Compute was not the limit. This work needed only inference and calibration
after training. Calibration-set size, not GPUs, was the limit. **But the later
result above controls the current decision:** [[Calibration-Prior-Art-Gate]]
showed that the proposed fix cannot change the measurements this paper would
report. That is why the direction closed.

## Related

Stage-E-Prior-Art-Audit (SVIB repository wiki) explains why the audit-paper path
closed. Post-Rebuttal-Measurement-Sprint (SVIB repository wiki) records the
Stage A patch-grid and Stage B decision-rule checks behind these gaps.
[[Direction-Gate-Results]] · [[Calibration-Prior-Art-Gate]] ·
[[Calibration-Draw-Preregistration]] ·
[[Temperature-Confound-Preregistration]] record the later outcomes.
[[Field-Scouting-Survey]] · [[Math-Grounded-Direction-Survey]] are the wider
surveys that replaced this page.
