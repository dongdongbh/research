# Method Gates — 2026-08-03

This page checks whether recent work already did either of our two real method
ideas. Here, a **method** means a new model design, training procedure,
representation, assumption, data source, or workflow. A diagnosis, a test that
removes or changes one part, or statistics alone does not count as a method.

This check was run directly because no subagent budget remained. The evidence
came from S2, Hugging Face (HF), and arXiv APIs, followed by full-page reading.
One warning: this was a single pass and leaned heavily on arXiv and HF. Before
we lock a pre-registration, we must run the usual confirmation pass and search
the most recent 8 weeks on purpose.

## Gate 1 — Crop-consistency distillation → **SURVIVES, ★★★★ conditional**

**The basic mechanism exists, but nobody has used it for our target.**

- **[CLIPSelf](https://arxiv.org/abs/2310.01403) ([arXiv 2310.01403](https://arxiv.org/abs/2310.01403), ICLR 2024)** is the earlier method. It "aligns a
  region representation extracted from its dense feature map with the
  image-level representation of the corresponding image crop." In other
  words, it uses crop→dense self-distillation: one part of the model teaches
  another part of the same model. However, it only studies open-vocabulary
  **dense prediction** (making a prediction for each image region). It fully
  fine-tunes the ViT and has **no compositional or image-text-matching (ITM)
  evaluation**. We checked 100 papers that cite it. None uses this mechanism
  for compositional matching between pairs of captions.
- The closest compositional competitor, **DeGLA ([2504.16801](https://arxiv.org/abs/2504.16801))**, uses a different
  mechanism. During hard-negative fine-tuning, it uses a global EMA teacher to
  *preserve* knowledge, plus text-side IGC/TGC losses. It gains +3.5% on average
  across [VALSE](https://arxiv.org/abs/2112.07566), [SugarCrepe](https://arxiv.org/abs/2306.14610), and [ARO](https://arxiv.org/abs/2210.01936). It is therefore a **required
  baseline**, not earlier work that already did our method.
- No 2024–2026 search found crop-re-encoding→patch distillation on a frozen
  backbone. No search found a structured teacher that uses several regions.

**What is still new, in one sentence:** teach ROI-pooled patch features from a
**frozen** backbone to copy both full-resolution crop re-encodings and our
*trained grid+self-attention multi-region teacher*. Use only a light adapter.
The goal is to keep our measured compositional gain (+2.66 teacher / −1.32
patch gap, from our own paired-CI numbers) at about 1.1× inference cost while
preserving retrieval quality.

**What must happen for the idea to succeed:** the student must close ≥⅔ of the
1.32-point gap. Ideally, it gets close to the teacher that costs 8× more. Two
named risks remain. First, the student may never reach the teacher. Second,
there is an **aggregation-objection kill-arm**, meaning a comparison that can
end the project. "Similarity Is Not Logic" (ICML 2026) says binding fails
because of how the representation is used, not because the representation is
missing information. We therefore pre-register this rule: if an aggregation
fix over raw patch tokens closes the gap at 1×, our training method is not
needed. That comparison is an experimental arm and a rule for ending the idea,
not just a possible problem.

**Systems we must compare against:** raw frozen backbone · grid+self-attn @8×
(best-case reference) · patch-ROI @1.06× (lower reference) · CLIPSelf
checkpoint with ROI pooling (repo [`wusize/CLIPSelf`](https://github.com/wusize/CLIPSelf), 207★, frozen since
2024-02, **no license** — evaluate it, do not fork it) · DeGLA · aggregation
fix @1×.

**Cost and venue:** 200–400 GPU-h for adapter training and evaluation with our
cached features, on OrangeGrid/Anvil. **ICLR 2027 is feasible.**

**Why the star rating is conditional:** the deciding week-1 check is to run
CLIPSelf's released checkpoints through our compositional test set with ROI
pooling. If CLIPSelf *already* closes the compositional gap, our paper becomes
a study that measures their method. The rating falls to ★★½, with the backup
result "what crop-distillation does and does not transfer." If it does not
close the gap, as expected because its goal is dense-prediction alignment
rather than ITM, the question remains open.

## Gate 2 — Epistemic contextualization (first implementation) → **SURVIVES, ★★★★**

**No working implementation exists.** We checked again, including the most
recent weeks. There is no paper and no LawZero code. Its repository still
returns 404, and its hiring page still lists the pipeline jobs. The Scientist
AI paper itself says: "contextualization is specified solely at the level of
requirements (i.e., we do not provide a completed algorithm)."

**The closest competing idea is tag conditioning, but it is different.** Tag
conditioning adds labels or codes to the input:

- **Source-aware training ([2404.01019](https://arxiv.org/abs/2404.01019), Khalifa et al.)** connects "unique
  source document identifiers with the knowledge in each document," then
  instruction-tunes the model to cite sources. It uses **ID tags, synthetic
  data, and attribution-only results**. It does not rewrite text or measure
  calibration, sycophancy, or goal adoption. The paper also stresses that
  "pretraining data augmentation" is needed. That supports the idea that
  changing the data is the part doing the useful work.
- [CTRL-style control codes](https://arxiv.org/abs/1909.05858) and conditional pretraining add tags at the
  beginning. They do not reorganize claims by how we know them. GenProve
  ([2601.04932](https://arxiv.org/abs/2601.04932)) tracks where generated text came from. None rewrites statements into
  records of who made each claim, as in Bengio Definition 3.22.

**What is still new, in one sentence:** build the first pipeline that
*rewrites* pretraining text into records that show the limits of each claim
("X is true" versus "S claimed X at T in V"). Include a
**tag-only-conditioning arm** to separate the effect of rewriting from the
effect of metadata tags. This directly answers the likely objection that we
merely rediscovered metadata conditioning.

**Evaluation tools checked directly:** ConflictBank ([2408.12076](https://arxiv.org/abs/2408.12076)) for conflicts in
knowledge; "How LLMs Balance Internal Knowledge with User and Document
Assertions" ([2604.22193](https://arxiv.org/abs/2604.22193)), which is a nearly exact fit; sycophancy test sets;
and a standard ability test set to measure any cost. A placebo rewrite is a
standard control ([2603.24826](https://arxiv.org/abs/2603.24826)). We add the tag-only arm as the key scientific
control.

**What must happen for the idea to succeed:** contextualized models must beat
raw text, tag-only text, AND placebo rewrites on calibration during conflicts
and on sycophancy, while losing little general ability. The rewriter must also
pass a pre-registered sample check for fact accuracy. The main technical risk
is that rewriting changes facts. Nearby work suggests this can be managed, but
we measure it first.

**Cost and venue:** 900–1,400 GPU-h. Rewriter inference is the largest part at
about 400+ H100-h; the pretraining arms use 0.5–1.5B models. Run on Anvil H100 /
Delta. **ICML 2027.** LawZero may have a 6–12 month lead and NVIDIA support, so
check again at every milestone.

## Final decision

Both method ideas pass their checks. Write pre-registrations for both. Make
crop-consistency distillation the **main ICLR method project**. The parallel-RL
experiment that compared combinations moves to the idea bench. Make epistemic
contextualization the **main ICML method project**. The [SigLIP-2](https://arxiv.org/abs/2502.14786) ladder moves to
the bench. The judge audit for ICLR and the 1-NFE diagnostic for CVPR do not
change.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Prereg-RoboJudge-Audit]] ·
[[Prereg-1NFE-Diversity]] · [[Status-And-Survivors]] ·
[[Method-Gates-Wave-2-2026-08]] (wave 2, 2026-08-02: 15 ideas → 7 live
gates → 1 narrowed survivor; both main projects stand)
