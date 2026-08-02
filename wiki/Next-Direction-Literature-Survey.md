# Next-Direction Literature Survey

Status: **Survey complete 2026-07-25.** The earliest and narrowest of the
direction surveys: six candidates assessed against the 2025-2026 literature.
Written after the Stage-E prior-art audit (svib repo wiki) closed the
audit-paper path.

**Updated 2026-08-02 for the general research wiki.** Read this as a record of
what was checked, not as live guidance — **almost every candidate below was
subsequently gated, and the outcomes live elsewhere**:

- [[Direction-Gate-Results]] — the distillation capacity gap (scooped: the
  U-shape theorem was already published, ICML 2025 appendix), plus the
  benchmark-unidimensionality, succinctness and C-RASP gates.
- [[Temperature-Confound-Preregistration]] — the distillation temperature
  follow-up; gate failed, protocol must not be run.
- [[Calibration-Draw-Preregistration]] — the calibration-draw noise-floor
  protocol; scooped by an ACL 2024 paper that ran the identical design.
- [[Calibration-Prior-Art-Gate]] — closed the calibration/selective-prediction
  line: both gaps are literally unclaimed, but post-hoc recalibration is
  monotone and therefore cannot change accuracy, AURC or any risk-coverage
  curve, so the "fix" step is structurally unavailable.
- [[Direction-Reevaluation-2026-08]] and [[Top-Researcher-Scan-2026-08]] —
  current direction ranking and current opportunity map.

Project-specific evidence (SVIB probe numbers, stage results, script names) has
been condensed here; full detail: svib repo wiki.

Verdicts are SATURATED (do not enter), ACTIVE (crowded, entry needs a sharp
angle), or OPEN (specific unclaimed gap named).

## Verdicts on the six candidate directions

| Direction | Verdict | Reason |
|---|---|---|
| Full fine-tuning / better general VLM | **SATURATED, industry-owned** | VladVA needs 32×A100. Sub-4B SOTA is HuggingFace/Alibaba/Google/Apple/Samsung. Open-Qwen2VL is the only academic near-miss and had ByteDance + NVIDIA co-authors |
| Visual token selection / pruning | **SATURATED** | ~145 papers/yr run rate. UniPruneBench: random pruning is competitive, no method consistently wins. "Are We Solving the Right Problem?" (ACL Findings 2025): many methods underperform random selection |
| Vision-text tower interaction | **ACTIVE, partly claimed** | C2LIP (CVPR 2026) already asserts "global pooling destroys binding" with parameter-free attention pooling at 8×A40. FLAIR, FILIP, TFLocal, ColPali exist. Gap: no factorial ablation isolating granularity vs interaction vs training signal |
| Gemma/open-VLM as teacher or data generator | **CROWDED, documented failure modes** | Recap-DataComp-1B recaptioned 1.3B images for +3.1% retrieval. MLLMCLIP (feature-level MLLM→CLIP) was **withdrawn** from ICLR 2026. ACL 2025 (2411.05195) shows the generative advantage is architectural — patch tokens, position embeddings, prompt weighting — so a pooled dual encoder structurally cannot inherit it |
| Distillation to smaller models | **Mostly industry; one live niche** | CompoDistill (ICLR 2026) distills *compositional* ability via visual-attention alignment: 60.7→66.7 where general KD gives 61.5. Note "When Better Teachers Don't Make Better Students" (2511.17886): stronger CLIP teachers do not reliably give better students |
| **Conformal / selective prediction for VL matching** | **ACTIVE field; tested signal closed** | ICLR 2026 already claims compositional risk-coverage, and the locked four-model probe finds no practically meaningful dispersion gain beyond a learned margin-only selector |

**Outcome notes 2026-08-02.** Row 5 (distillation): the theory-side reframing
this survey later attracted was gated and killed — see
[[Direction-Gate-Results]] and [[Temperature-Confound-Preregistration]]. Row 6
(conformal/selective prediction): closed for a structural reason rather than a
crowding reason — [[Calibration-Prior-Art-Gate]]. Rows 1–4 were not re-gated;
treat their 2026-07 crowding counts as stale.

## The two open gaps

### Gap 1 (revised): equivalence-class dispersion as an incremental selective-prediction signal

**Correction recorded 2026-07-25.** The survey's original claim that no
risk-coverage protocol existed for compositional matching was false.
**Leveraging Data to Say No: Memory Augmented Plug-and-Play Selective
Prediction** (ICLR 2026, OpenReview `wWxdT6LB2D`) reports AURC and
risk-coverage results on SugarCrepe, Winoground, What'sUp, VL-Checklist, and
Foil. **Look Again Before You Abstain** (arXiv `2606.16667`, v4) also makes the
broad VLM conformal-abstention space active rather than open.

The surviving narrow gap was tested in Conformal-Probe-Preregistration
(svib repo wiki):

> Does score dispersion across multiple equivalent human captions add
> selective-prediction signal beyond minimum/mean confidence margins and a
> learned margin-only selector?

This is a feasibility question, not yet a paper claim.

**Outcome correction 2026-07-25.** Conformal-Probe-Results (svib repo wiki)
finds that dispersion improves over raw minimum/mean margins but not by the
locked practical threshold over a feature-matched learned margin selector on any
of four COCO models. The narrow signal gap is therefore tested and negative. The
large all-realizations gap survives as a reporting observation, not as a new
selective-prediction method.

**And then the whole line closed, 2026-07-25.** [[Calibration-Prior-Art-Gate]]
ran on the successor calibration hypothesis and killed it for a reason that
applies to everything in this section: temperature scaling is monotone, so on a
two-way caption decision it cannot change which caption wins, and AURC and
risk-coverage curves depend only on the *ranking* of confidences. Both metric
families used by this literature are invariant to the only fix such a paper
would offer — decision delta exactly zero.

### Superseded search record

Emptiness evidence (arXiv full-text queries):

- `all:"conformal" AND all:"image-text retrieval"` → **0 hits**
- `abs:"coverage guarantee" AND abs:"image-text matching"` → **0 hits**
- `abs:"learning to defer" AND abs:"vision-language"` → **0 hits**
- `abs:"router" AND abs:"CLIP" AND abs:"uncertainty"` → **0 hits**

These zero-result queries were too narrow and are retained only as a process
warning: exact phrase queries did not recover adjacent work that used
"selective prediction" and benchmark names. Of the original three claims,
only the within-equivalence-class calibration signal remains plausibly open:

1. **Risk-coverage on compositional matching is claimed.** The ICLR 2026 work
   above directly reports it.
2. **Nobody has made the input-side semantic equivalence class the calibration
   unit.** The machinery exists and is unused in vision-language: hierarchical
   exchangeability (Lee/Barber/Willett 2306.06342 — groups exchangeable,
   observations within group exchangeable), macro-coverage (2606.28598),
   SymmPI (2312.16160), equivariant CP (2602.03986). The strong target is
   **all-realizations coverage**: `P(correct for every realization of a
   held-out meaning) >= 1-alpha`. That is arguably what a compositionality
   claim *should* assert, and no benchmark tests it.
3. **The specific descriptive → incremental-signal step remains untested in
   the located work.** PRSM (2511.11141, MMM 2026) and LGIP (2511.13494,
   Pattern Recognition Letters 2026) both measure CLIP paraphrase instability
   and stop. The new probe asks whether that dispersion improves selection
   after controlling for margin.

Adjacent-but-not-it, to cite and distinguish: Conf-OT (CVPR 2025, zero-shot
classification), ConfLVLM (2502.20560, generative claim-level), VL-Uncertainty
(CVPR 2025, LVLM hallucination), the probabilistic-embedding line (PCME++,
ProbVLM — produce uncertainty scores, never a calibrated abstain rule or a
risk-coverage curve).

**Data caution.** Within-class exchangeability must hold. LLM-generated
paraphrases are *not* exchangeable with human captions (different length and
lexical statistics). **COCO's five human captions per image sidesteps this
entirely** and is the stronger experimental choice; SugarCrepe++'s two
positives support a pair statistic but are thin for per-class quantiles.

**Reviewer trap to pre-empt.** Oracle complementarity gaps are always large and
usually unrealizable. Report oracle *and* realized gate side by side, beat a
confidence-margin baseline and a random-coverage baseline at matched coverage,
and convert to a calibrated statement (deferral rate needed to hit a target
risk). Bare oracle numbers read as padding.

### Gap 2 (fallback): effective resolution / receptive-field granularity as a causal variable for binding

The competitor's own numbers contain an unexplained inversion. Miranda et al.
(2604.11496, Apr 2026, UPV/EHU) report on BiSCoR-Ctrl: **SGI (separately
encoded crops, training-free) = 24.9** versus **TFLocal (patch tokens, 13.3M
trained params) = 13.2**. The crop route beats the patch-token route on their
own OOD diagnostic and they offer no explanation and no head-to-head ablation.

Our own probes showed the same phenomenon with a granularity signal attached:
the patch-grid penalty shrinks by roughly 4× going from a 49-token to a
256-token backbone. Candidate mechanisms — contrastive-manifold drift for tight
crops versus spatial-correlation collapse under pooling — are unstudied and
directly testable on existing infrastructure. Full detail: svib repo wiki,
page Post-Rebuttal-Measurement-Sprint.

C2LIP asserts the adjacent claim ("final global pooling leads to loss of
binding information") but supports it with architectural argument and attention
maps, not a controlled experiment. That is the exploitable soft spot.

## Threats to know before writing anything

1. **"CLIP Models Generalize Less Than Compositional Benchmarks Suggest"**
   (ICML 2026 CTB workshop). On ARO VG-A, positive captions overlap COCO
   attribute-object bindings **79.8%** of the time versus **41.8%** for
   swapped negatives; only 1.2% of samples have no COCO-overlapping bindings.
   Overlap-balanced splits **reorder the leaderboard and flip model ranks**.
   This threatens any paper reporting SCPP++ deltas of our magnitude.
2. **Test-Time Matching** (2510.07632, ICLR 2026) reports SigLIP-B16 Winoground
   group `10.25 → 72.50`. This is a **changed task**, twice: GroupMatch's
   chance level is `1/k! = 50%` versus GroupScore's `(k-1)!/(2k-1)! = 16.7%`
   (both proven in their own paper), and TTM fine-tunes on pseudo-labels
   derived from the test set's bijective structure. Corrected reading:
   `-6.4` above chance → `+22.5` above chance on a strictly easier decision.
   Reviewers will cite it; have the chance-level table and the transductivity
   objection ready in one sentence.
3. **C2LIP** (2603.25722, CVPR 2026, Samsung) is a direct competitor at our
   exact compute scale (8×A40, CC3M, 5 epochs) with retrieval *above* the
   SigLIP baseline.
4. **Miranda et al.** (2604.11496) is the fair competitor for Gap 2 and is
   actively working it.

## Why Gap 1 fits this group specifically

The generalizable version of the argument: three consecutive method campaigns
failed on the same dependency — success required beating a benchmark number.
Gap 1 does not. Success is "we defined a meaningful reliability target and
characterized how models behave under it," which removes that failure mode and
plays to the measurement discipline. The prior campaigns had also already
produced most of a deferral analysis (repair precision and coverage, harm rate,
oracle headroom, margin-decile firing rates) plus a multi-backbone harness
reproducing six external systems, so the assets mapped directly. Full detail:
svib repo wiki, page Post-Rebuttal-Measurement-Sprint.

Compute is not the constraint: this direction is inference plus post-hoc
calibration. The binding constraint is calibration-set size, not GPUs. **But
see the note above** — [[Calibration-Prior-Art-Gate]] later showed that the
metrics this direction would report are invariant to the fix it would propose,
which is what actually closed it.

## Related

Stage-E-Prior-Art-Audit (svib repo wiki) — why the audit-paper path closed.
Post-Rebuttal-Measurement-Sprint (svib repo wiki) — the Stage A patch-grid and
Stage B gate audit that feed the gaps above.
[[Direction-Gate-Results]] · [[Calibration-Prior-Art-Gate]] ·
[[Calibration-Draw-Preregistration]] ·
[[Temperature-Confound-Preregistration]] — where the outcomes live.
[[Field-Scouting-Survey]] · [[Math-Grounded-Direction-Survey]] — the later,
broader surveys that replaced this one.
