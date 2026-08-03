# Method Gates — Wave 3, 2026-08-02 (evening)

This round checked whether five new method ideas were truly new. In plain
language, the ideas ask how to keep detailed image-text relationships without
paying for an expensive comparison every time. They also ask whether a model
should store word position separately from word meaning.

Several paper and model names appear below:

- **TF_Local** compares detailed image patches with text tokens through a small
  learned model.
- **DCLIP and CPRD** teach a cheaper two-encoder model to copy a more expensive
  model that lets image and text interact.
- **NoPE** means no explicit position encoding. The Kimi K3 report calls its
  position-aware attention layers **KDA** and its other attention system
  **MLA**. Its design lets KDA carry position while MLA focuses more on content.

This third round started from [TF_Local (2604.11496)](https://arxiv.org/abs/2604.11496),
[DCLIP (2505.21549)](https://arxiv.org/abs/2505.21549), [CPRD](https://arxiv.org/abs/2407.07479), the
[Kimi K3 NoPE design](https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf), and
[[Researcher-Scan-Raschka]]. The Kimi report's text is saved in
`papers/kimi-k3-tech-report.txt`. It says that KDA layers carry position while
NoPE on MLA separates "where" from "what."

Opus agents tried hard to disprove each idea, then ran a required second
search. One warning applies to the whole round: the WebSearch budget ran out
midway, and arXiv/S2 APIs limited our requests. We therefore **must repeat
forward-citation searches before any pre-registration**. Each check records
this missing step.

## Scoreboard

| # | Candidate | Verdict | ★ | Disposition |
|---|---|---|---|---|
| I1 | Position-tagged late-interaction distillation | **SCOOPED** — [ComAlign (2409.08206)](https://arxiv.org/abs/2409.08206) owns the architecture: role-tagged multi-vectors + per-role MaxSim on frozen CLIP. SaMer/[MetaEmbed (2509.18095, ICLR26 Oral)](https://arxiv.org/abs/2509.18095) own compression. TF_Local's own SGI is already cacheable crop-set MaxSim | — | Stop. Only the training signal changes, so this is at most a workshop paper |
| **Cell A** | **How binding changes as the readout budget changes** ("compositional supervision does not survive multi-vector compression") | **SURVIVES, AMBER** — the compression work was checked and studies retrieval only. [Tübingen theory (2602.24264, ICML26)](https://arxiv.org/abs/2602.24264) predicts NO breakpoint when d≥k, with k≈3–5, so either result teaches us something. [CIE (1911.05248)](https://arxiv.org/abs/1911.05248) gives the per-item framing we should use | **★★★½** | **Possible CVPR method slot** |
| Cell B | Split compositional benchmarks by whether each item depends on position | SURVIVES — unanswered, but its overall design has two clear examples: [temporal twin 2607.12304](https://arxiv.org/abs/2607.12304), 3 weeks old, and CompLearn's split+rerank design. [2503.17349](https://arxiv.org/abs/2503.17349)'s 0.2–2.7% total drops predict a small split | ★★½ | Idea bench; 5 GPU-h pilot decides |
| O1 | Test spectral bands together with knowledge injection to decide between explanations (Raschka/MiCA) | SURVIVES-NARROWED — four studies already answer the learning-rate-artifact question for task adaptation. Only the knowledge-injection setting + TOST + released benchmark remain | ★★ | Idea bench; K1 = 10 GPU-h MiCA reproduction is required before entry |
| RB | Algebraic role-binding embeddings (HRR/TPR in frozen towers) | SURVIVES-NARROWED — [OC-CLIP (2502.14113)](https://arxiv.org/abs/2502.14113) owns parse+non-commutative score+algebraic swap negatives, but its shape is text-conditioned and cross-encoder-like. The HRR×ITM question is empty: 43 papers, 0 CLIP. **Probe update 2026-08-03: kill-arm A CLEARED for spatial roles**: 99.9% linear decodability in frozen ROI features; a parameter-free sign flip gives 100%; pooled embedding and scoring axis stay at chance. See [[Binding-Root-Cause-Analysis]] §6. The verb-argument case stays open until VisMin is unblocked | **★★★ (cond.)** | Promoted; rate again after verb-swap data |

## Cell A — the most important idea (full record)

**Claim:** nobody measures how compositional binding changes with the readout
budget, meaning the number of vectors or bytes stored for each image. We
checked MetaEmbed, SaMer ([2607.04605](https://arxiv.org/abs/2607.04605)), [ColPali](https://arxiv.org/abs/2407.01449), MUVERA,
MM-Matryoshka, MIEB, and LIMIT. All study retrieval only.

**Approved experiment:** compare a ladder of readouts from the same SigLIP2-L
backbone: R0 CLS → R1 truncation → R2 token pooling with K∈{1..256} → R3
crop-set MaxSim with 86 crops and **our SVIB code** → R4 full-patch MaxSim → R5
cross-encoder. Comparing different models has a hidden factor because their
training goals differ. Put that comparison only in a descriptive appendix.

The main measurement is the binding-retention ratio
ρ(b)=ΔBinding/ΔRetrieval. The breakpoint is the largest budget where the
paired-bootstrap confidence interval (CI) for ρ excludes 1. Mark the items that
flip and label each by the type of change, following CIE. Use SCPP++ as the
main test, plus [SugarCrepe](https://arxiv.org/abs/2306.14610), [ARO](https://arxiv.org/abs/2210.01936) as a text-prior control, WhatsUp, VSR,
and MMVP-VLM. Use [Winoground](https://arxiv.org/abs/2204.03162) only for CIs. Cost is about **180–240
GPU-h**. ComAlign has no public code, so cite it but exclude it. MUVERA needs a
two-day FDE reimplementation; otherwise drop it.

**Method this result would make possible, checked as unanswered:** pooling that
preserves binding. Train the pooled representation with a hard-negative MARGIN
distillation loss from full-MaxSim or cross-encoder teachers. This reuses our
crop-distillation machinery. The ★★ crop idea becomes this method arm instead
of a separate paper.

**Results that would end the idea:** (1) ρ≈1 everywhere, which is the most
likely no-breakpoint result and means no paper; (2) no range of change from
R0→R4, so a **1-day pilot is REQUIRED before commitment**; (3) an uncontrolled
difference across the ladder; or (4) an ARO text-prior artifact.

**Main competing group:** Tübingen researchers Uselis, Koishigarina, Dittadi,
and Oh already have three binding-geometry papers in 2026. Testing the
compression axis is an obvious next step for them. Speed matters.

**Venue:** CVPR, around Nov 13. This fills the plan's empty CVPR method slot.

## Cell B — full record

Changes to test: shuffle patches, which gives the cleanest label / shuffle
blocks / shuffle position embeddings (PE) / remove PE / **a noise control
matched for the same performance loss, which is essential**. For each item,
run 20 seeds and compare the Wilson bound with the control. Require agreement
across ≥3/4 backbones. This rule makes the split less tied to one model than the
temporal paper's split. Use BH-FDR for each item and Holm correction for claims
about subsets.

Re-rank the leaderboard with [NegCLIP](https://arxiv.org/abs/2210.01936), [DeGLA](https://arxiv.org/abs/2504.16801), [LABCLIP](https://arxiv.org/abs/2502.03566), and
[DCSM](https://arxiv.org/abs/2503.08723). Cost is about 150 GPU-h. Results that end the idea are a split below
10%, which is likely; shuffling that reduces performance to chance; failure to
repeat with κ<0.4; or τ≥0.9, meaning there is no meaningful re-ranking.

Confirmed warning: [2601.09954](https://arxiv.org/abs/2601.09954) does **not** run this diagnostic. It only swaps
encoders. Never cite it as if it did this test.

## What we learned from round 3

The same pattern held. Ideas copied from visible tools or research plans—I1
from TF_Local+K3 and O1 from Raschka—received ★–★★. The idea from our own
measurements, Cell A, received ★★★½. Cell A also settles what to do with the
crop idea. Instead of a separate ★★ efficiency paper, use the distillation
machinery as Cell A's method arm for CVPR.

Order of work: (1) run Cell A's 1-day pilot and repeat the citation search; (2)
if the pilot shows enough range, write a CVPR pre-registration; (3) run Cell B's
5 GPU-h pilot using the same encoded features.

## Added the same evening: LaSt-ViT — a new Cell-A comparison and a mechanism clue

[**"Vision Transformers Need More Than Registers"** (LaSt-ViT, 2602.22394)](https://arxiv.org/abs/2602.22394),
from HKU/SYSU in Feb 2026, finds **"lazy aggregation."** Vision Transformers
(ViTs) use many BACKGROUND patches to carry the overall meaning of the image,
even though those patches are not about the main objects. The paper describes
"semantically irrelevant background patches as shortcuts... driven by global
attention and coarse-grained semantic supervision." Adding registers does not
fix it. Its fix selects the top K values for each frequency-aware channel and
combines them into CLS.

The method gives very large gains on dense tasks. For example, CLIP ViT-L/14
VOC segmentation rises from 17.1→72.4 mIoU. But none of its 12 benchmarks tests
compositional ITM. [Its code uses the MIT license and releases CLIP-B/16 + DINO checkpoints](https://github.com/ChengShiest/LAST-ViT)
(441★, active).

This is not a separate paper for us. It has three uses:

1. **One Cell A step and one standard comparison (A4 pattern, ~5–10 GPU-h):**
   run the released CLIP-B/16 model through our compositional test set, and add
   selective aggregation as one step in the ladder. Either result is clear.
   Recovery means selection is the binding bottleneck and supports the view
   that execution is the problem. No recovery means **recovering location skill
   does not recover binding skill**, a split nobody has shown.
2. **A mechanism clue for Cell B:** lazy aggregation predicts that CLS should
   barely change when patches are shuffled, because the background set ignores
   order. This explains the tiny 0.2–2.7% drops in [2503.17349](https://arxiv.org/abs/2503.17349). Selective
   readouts should depend more on position. This is a free prediction inside
   Cell B's experiment.
3. **Watch this competitor:** the authors say future work will "apply insights
   to vision-language model alignment beyond CLIP." That is one step from our
   question. Add them to Cell A's watch list beside Tübingen.

## Related

[[Method-Gates-Wave-2-2026-08]] · [[Researcher-Scan-Raschka]] ·
[[Prereg-Crop-Consistency-Distillation]] · [[Top-Researcher-Scan-2026-08]]
