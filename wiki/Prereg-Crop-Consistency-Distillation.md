# Pre-registration: Crop-Consistency Distillation — Region Information at Single-Pass Cost

**Plain-language summary:** detailed image-text models compare many image
patches with many words. They can understand relationships well, but they are
slow because they repeat that work for every image-caption pair. This project
asks whether a small training adapter can teach a cheap model to keep most of
that detailed information. The cheap model would still save one reusable
vector for each image and run at about 1.1× normal cost.

Terms used below:

- A **frozen backbone** is a base model whose weights do not change.
- **ROI pooling** combines features from one region of interest in an image.
- A **dual encoder**, also called a **bi-encoder**, encodes image and text
  separately, so their vectors can be saved and reused.
- A **cross encoder** lets image and text interact during every comparison. It
  is usually stronger but slower.
- **Image-text matching (ITM)** asks whether an image and a caption belong
  together.
- **Distillation** trains a smaller or cheaper student to copy a stronger
  teacher.

Status: **LOCK HOLD, 2026-08-02.** The final literature check found
**[arXiv 2604.11496](https://arxiv.org/abs/2604.11496)** from Apr 2026, with a 3.5/4-part match. Its TF_Local
method already publishes the main insight: fine-grained alignment of FROZEN
patch/token features gives large gains in compositional understanding while
keeping retrieval quality. [SugarCrepe](https://arxiv.org/abs/2306.14610) rises from 73.0→86.3. Its §3 test is
also our crop teacher.

Our remaining difference is **distillation into ROI-pooled patch features that
keeps about 1.1× DUAL-ENCODER inference and allows embeddings to be cached**.
TF_Local is a per-pair cross-encoder. Also, **[FineCLIP](https://openreview.net/forum?id=nExI4FuKWD) (NeurIPS 2024)**
shows that region self-distillation improves compositional skill when the
backbone is trained. We must therefore limit the A4 conclusion to
"[CLIPSelf](https://arxiv.org/abs/2310.01403)'s released checkpoint, frozen ITM."

**New check on 2026-08-02: SURVIVES-NARROWED, level 2/5, ★★.** Nobody has run
the exact combination of a crop/cross-encoder teacher, an ROI-pooled student on
a frozen backbone, compositional benchmarks, and an efficiency comparison.
However, others own every piece. DCLIP ([2505.21549](https://arxiv.org/abs/2505.21549)) makes the efficiency claim
word for word in a retrieval-only setting: "without requiring region processing
at inference." [CPRD](https://arxiv.org/abs/2407.07479) (CVPR 2024) already distills a cross-encoder VLM
into a bi-encoder. CLIPSelf and [DeCLIP](https://arxiv.org/abs/2505.04410) own the crop-teacher→ROI mechanism.
[CLIC](https://arxiv.org/abs/2505.24424) owns retrieval preservation. A reviewer may therefore describe this as
"DCLIP + [SugarCrepe](https://arxiv.org/abs/2306.14610)."

TF_Local's code and BiSCoR-Ctrl are UNRELEASED. The repository is only a
"Coming soon" page and the paper has 0 citations, so we must reimplement the
best-case reference. Paper 2604.11496 never discusses cost or cacheability. The
gap is real, but it looks like an obvious next step, and competing NeurIPS'26 or
ICLR'27 submissions are hidden from us by design. Raise the rating to ★★½ only
if we show a ≥10× cost advantage while keeping ≥70% of the gain, paired CIs on
≥2 benchmarks, and end-to-end latency rather than FLOPs. Lower it to ★½ if our
TF_Local reimplementation does not reproduce the result.

**Decision needed at sign-off:** continue with the narrower ★★ paper or move
the method to the idea bench. If benched, A4/A5 results become part of the SVIB
paper's story. The older status was DRAFT v1 for professor sign-off. The week-1
deciding checks in §8 are complete. Target venue remains **ICLR 2027**, with
abstracts due Sep 18. Changing the framing needs new wording and one extra
baseline, but no new experiment for the abstract.

Paper type: **METHOD**. Under the owner's definition, this is a new training
procedure that challenges the belief that patch tokens already contain region
information. Check record: [[Method-Gates-2026-08]]. Related diagnostic:
[[Prereg-RoboJudge-Audit]].

---

## 1. The problem

Contrastive vision-language models (VLMs) often fail when meaning depends on
how parts are combined. Our SVIB post-mortem found where the fix works and how
much it costs. Re-encoding **image regions at full resolution** with a 20-crop
grid and one self-attention layer gives +2.66 on [SugarCrepe++](https://arxiv.org/abs/2406.11171) with corrected
CLIP, but costs about **8× more at inference**. Reusing the Vision Transformer's
(ViT's) own ROI-pooled patch tokens costs only 1.06×, but **loses 1.32 points**,
with paired 95% CI [−2.51, −0.12]. The patch tokens do not contain the needed
region information. Our method tries to put that information there during
training so inference needs only one pass.

## 2. What recent work has shown (checked 2026-08-03)

- **[CLIPSelf](https://arxiv.org/abs/2310.01403) (ICLR 2024)** is the earlier mechanism. It aligns region
  representations from a dense feature map with the image-level embedding of
  the matching crop. It uses this only for **open-vocabulary dense prediction**,
  fully fine-tunes the ViT, and has **no compositional or image-text-matching
  (ITM) evaluation**. None of its 100 citing papers applies the method there.
  The repository is public with 207★ and has been unchanged since Feb 2024, but
  it has **no license**. We may evaluate its checkpoint, but must reimplement
  the mechanism.
- **DeGLA ([2504.16801](https://arxiv.org/abs/2504.16801))** improves compositional skill at about 1× cost with a
  *different* mechanism: a global exponential-moving-average (EMA) teacher
  preserves knowledge while LLM-written hard negatives improve training. It
  gains +3.5% on average across [VALSE](https://arxiv.org/abs/2112.07566), SugarCrepe, and [ARO](https://arxiv.org/abs/2210.01936). It is a
  **required baseline**, not earlier work on our mechanism.
- **Work on aggregation**, including LABCLIP, [DCSM (ICCV 2025)](https://arxiv.org/abs/2503.08723), and
  ["Similarity Is Not Logic"](https://arxiv.org/abs/2607.23052) (ICML 2026), says binding fails because of how
  features are used, not because information is absent from them. Our −1.32
  result concerns region information behind the grid+attention gain, not only
  binding. Reviewers may still mix these issues. We therefore include an
  aggregation fix as a pre-registered **experimental arm and rule that can end
  the project**, not merely as a citation.
- **Added after the wave-2 check on 2026-08-02
  ([[Method-Gates-Wave-2-2026-08]]):** "LABCLIP" is **[arXiv 2502.03566](https://arxiv.org/abs/2502.03566)
  (ICLR 2026)**. It uses a D×D text-side matrix with *frozen* encoders, about
  590K parameters, shuffled-negative training, ARO, and [SugarCrepe](https://arxiv.org/abs/2306.14610). LABCLIP
  and DCSM are published versions of A5. Use them instead of making our own
  aggregation fix, and list both as named baseline rows. This search also
  supports our framing: every 1× competitor—DCSM, LABCLIP, [ABE-CLIP](https://arxiv.org/abs/2512.17178), and
  TF-Local—reads features already present in the frozen model. None teaches the
  patch path with a stronger signal.
- [SILC](https://arxiv.org/abs/2310.13355) and [SigLIP-2](https://arxiv.org/abs/2502.14786) use local-to-global self-distillation during
  pretraining on the pipeline's own crops. Our work instead adds post-hoc
  crop→patch distillation to a frozen model. The direction, setting, and goal
  all differ.

## 3. The method

**Teacher, frozen and used only during training:** send the 20-view grid of
full-resolution crops through the frozen backbone and our *trained
grid+self-attention head*. This teacher understands several regions together;
it is not a set of independent crop embeddings. That is the difference from
[CLIPSelf](https://arxiv.org/abs/2310.01403)'s teacher.

**Student:** ROI-pool the patch tokens from the same frozen backbone, then send
them through a **small adapter**. The main design puts the adapter after the
frozen ViT. A second design uses LoRA on the last N blocks.

**Training losses:** match each region's teacher feature with cosine
similarity. Also match the teacher head's pattern of attention between regions.
This second, relational loss transfers the dense routing pattern that our
earlier tests found was essential.

**Data:** images only from COCO/VG and our cached crops. This is self-supervised
distillation, with no captions or labels.

**At inference:** one backbone pass plus the adapter, about **1.1×** cost. Test
it with our corrected ITM procedure, with every choice fixed on validation data.

## 4. Exact experiment plan

**Experimental arms** with CLIP ViT-B/32 as the main model and SigLIP2 B/16 as
the second model:

- A0: raw frozen backbone with corrected baselines.
- A1: patch-ROI without distillation, the lower reference at −1.32.
- A2: grid+self-attention at 8×, the upper reference at +2.66.
- A3: **our** distilled adapter at about 1.1×.
- A4: the released [CLIPSelf](https://arxiv.org/abs/2310.01403) checkpoint with ROI pooling and the same
  procedure. This is the deciding week-1 check.
- A5: an aggregation fix over raw patch tokens at 1×. This arm can end the idea.
- A6: DeGLA using published numbers, or its checkpoint if released.
- Also test whether A3 keeps COCO retrieval in both directions.

**Predictions, fixed before the run:**

- **H1:** on [SugarCrepe++](https://arxiv.org/abs/2406.11171), A3 recovers ≥⅔ of the A2−A1 gap under
  choices fixed on validation data. Predict TRUE.
- **H2:** A3 − A0 ≥ +1.0 SCPP++, with paired CI excluding 0. Predict TRUE.
- **H3:** A4 does NOT close the gap. Predict TRUE because CLIPSelf trains for
  dense prediction, not ITM.
- **H4:** A5 alone does NOT close the gap. Predict TRUE. If FALSE, the features
  already had enough information and our method is unnecessary; follow the rule
  below that ends the idea.
- **H5:** A3 keeps COCO retrieval within 0.5 R@1 in both directions.
- **H6:** A3's measured inference overhead is ≤1.2× under the same L40S setup
  used for SVIB.

**Decision rules:** use 3 seeds and our existing paired-bootstrap CIs. Choose
α and configurations only on validation data. Use Holm correction across
H1–H6.

**Honest limit on tuning:** try **6 training configurations total**, covering
loss weights × adapter size. This limit is fixed now. Do not search for a good
setting after seeing test results, and report every attempt.

**Rules that end the standalone project:** (i) if A4 closes ≥⅔ of the gap in
week 1, this becomes only a study of how CLIPSelf transfers; move it to the idea
bench. (ii) If A5 closes ≥⅔, the problem is how information is read out, not
whether it exists. Move our evidence into the readout-ladder paper. (iii) If A3
cannot beat A1 beyond random-seed variation within the six attempts, publish it
inside SVIB's honest-negative section, not as its own paper.

**Claims we will not make:** state of the art compositional skill against models
fine-tuned with hard negatives, because that is a different setting; claims
about generative or decoder VLMs; or claims about models beyond the two tested.
Our claim is limited to a frozen backbone with no annotation needed at
inference.

## 5. What each possible result means

- **Main result, with H1–H4 as predicted:** preserve compositional gains at the
  cost of one pass. This turns SVIB's honest negative into a usable fix. The new
  part of the mechanism is the loss that transfers relations between regions.
- **If A5 wins:** the field learns that the original representation had enough
  information. Combine the result with the readout study. It can still be
  published there.
- Release the adapter weights, training recipe, and test harness either way, on
  top of the already released SVIB evaluation tools.

## 6. Resources and schedule

**Cost:** 200–400 GPU-h. Adapter training is small, and teacher features for
CLIP are already cached. Use OrangeGrid A100 first and Anvil for SigLIP2.

Week 1: A4 and A5 checks, then lock. Weeks 2–3: A3 training within the fixed
attempt limit. Week 4: second backbone and retrieval. Week 5: random seeds and
CIs. Week 6: analysis. Week 7: writing. The abstract needs the A3-versus-A1
result on the main backbone.

## 7. Risks and competing work to watch

- The main risk is that an aggregation group moves from readout to training.
  Watch citations of LABCLIP, DCSM, and SNL. Run the standard search every 6
  weeks, and search within 48h after any hit for "crop distillation
  compositional."
- [CLIPSelf](https://arxiv.org/abs/2310.01403) has no license. Evaluate its checkpoint, but never fork it. Build
  our implementation from the paper.
- The teacher's gain is only +2.66. Effects are small, so use paired CIs and the
  ≥⅔-gap target instead of claiming absolute state of the art.

## 8. Checklist before locking

1. Professor approval of §4, especially the six-attempt limit and rules that
   end the project.
2. ~~Finish the week-1 A4/A5 checks.~~ **DONE 2026-08-02. Both matched our
   predictions, and neither rule ended the project.** Results are in the
   cropdistill repository under `results/a4` and `results/a5`, with manifests
   and same-day wiki records:
   - **A4/H3 TRUE; rule (i) cleared clearly:** ROI-pooled [CLIPSelf](https://arxiv.org/abs/2310.01403) scores 8–10
     points BELOW the base patch arm for every seed. Locked A4−A1 = −12.8,
     CI [−14.1, −11.5]. Its global endpoint, meaning main measurement, falls to
     42.7 versus 68.6. The dense-prediction goal harms compositional ITM instead
     of adding region information.
   - **A5/H4 TRUE; rule (ii) cleared:** LABCLIP closes a negative share of the
     A2−A1 gap on all three seeds: −0.97/−0.32/−0.18. The same matrices repeat
     the paper's +5.2 [SugarCrepe](https://arxiv.org/abs/2306.14610) gain, so the reimplementation is valid and the
     negative result is specific to strict SCPP++. The α=1 reproduction check
     is 5.1e-7.
3. **DECIDE BEFORE LOCK, based on A4/A5:** (a) choose whether the A5 ratio uses
   A0 or A1 as its reference. Both are calculated; select one. (b) Describe A4
   as "A4 ≪ A1," not with a closure ratio. [EVA02](https://arxiv.org/abs/2303.11331)'s native crop-versus-patch
   difference is small and noisy, and one seed is exactly 0. (c) Record one
   change from the published A5 recipe: 19,006 Karpathy-train rows were removed
   to prevent data leakage.
4. Repeat the literature search with the most recent 8 weeks named directly,
   including OpenReview. The tool was installed on 2026-08-02.
5. Mark the page LOCKED and record the git hash. Log any later changes below
   this line.

## Related

[[Method-Gates-2026-08]] · [[Unified-Direction-Ranking-2026-08]] ·
[[Prereg-RoboJudge-Audit]] · [[Status-And-Survivors]]
