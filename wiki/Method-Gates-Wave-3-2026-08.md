# Method Gates — Wave 3, 2026-08-02 (evening)

Third round, seeded by [TF_Local (2604.11496)](https://arxiv.org/abs/2604.11496),
[DCLIP (2505.21549)](https://arxiv.org/abs/2505.21549), [CPRD](https://arxiv.org/abs/2407.07479), the
[Kimi K3 NoPE design](https://github.com/MoonshotAI/Kimi-K3/blob/main/k3_tech_report.pdf)
(extracted: `papers/kimi-k3-tech-report.txt` — position carried by KDA
layers, NoPE on MLA = "where"/"what" separation), and the [[Researcher-Scan-Raschka]].
Standard protocol: adversarial Opus gates with mandatory second sweep.
Session-wide caveat: WebSearch budget exhausted mid-wave and arXiv/S2 APIs
rate-limited — **forward-citation sweeps must re-run before any prereg**
(gap flagged per gate).

## Scoreboard

| # | Candidate | Verdict | ★ | Disposition |
|---|---|---|---|---|
| I1 | Position-tagged late-interaction distillation | **SCOOPED** — [ComAlign (2409.08206)](https://arxiv.org/abs/2409.08206) owns the architecture (role-tagged multi-vectors + per-role MaxSim on frozen CLIP); SaMer/[MetaEmbed (2509.18095, ICLR26 Oral)](https://arxiv.org/abs/2509.18095) own compression; TF_Local's own SGI is already cacheable crop-set MaxSim | — | dead; residual = training-signal swap, workshop at most |
| **Cell A** | **Readout-budget vs binding frontier** ("compositional supervision does not survive multi-vector compression") | **SURVIVES, AMBER** — compression line verified retrieval-only; [Tübingen theory (2602.24264, ICML26)](https://arxiv.org/abs/2602.24264) predicts NO breakpoint (d≥k, k≈3–5) so either outcome informs; [CIE (1911.05248)](https://arxiv.org/abs/1911.05248) is the per-item framing precedent to adopt | **★★★½** | **CVPR method-slot candidate** |
| Cell B | Per-item position-dependence split of compositional benchmarks | SURVIVES — unoccupied but doubly-precedented in shape ([temporal twin 2607.12304](https://arxiv.org/abs/2607.12304) 3 weeks old; CompLearn owns split+rerank); [2503.17349](https://arxiv.org/abs/2503.17349)'s 0.2–2.7% aggregate drops predict a small split | ★★½ | bench; 5 GPU-h pilot decides |
| O1 | Spectral-band × knowledge-injection arbitration (Raschka/MiCA) | SURVIVES-NARROWED — LR-artifact question answered 4× for task adaptation; only the injection regime + TOST + released benchmark survive | ★★ | bench; K1 = 10 GPU-h MiCA repro gate is the entry ticket |

## Cell A — the one that matters (full record)

**Claim:** nobody measures compositional binding as a function of readout
budget (vectors/bytes-per-image). Verified against MetaEmbed, SaMer
([2607.04605](https://arxiv.org/abs/2607.04605)), [ColPali](https://arxiv.org/abs/2407.01449), MUVERA, MM-Matryoshka, MIEB, LIMIT — all
retrieval-only.

**Design (gate-approved):** within-backbone readout ladder on SigLIP2-L —
R0 CLS → R1 truncation → R2 token-pooling K∈{1..256} → R3 crop-set MaxSim
(86 crops, **our SVIB code**) → R4 full-patch MaxSim → R5 cross-encoder.
Cross-model ladder is CONFOUNDED (objectives differ) — descriptive appendix
only. Primary quantity: binding-retention ratio ρ(b)=ΔBinding/ΔRetrieval,
breakpoint = largest budget where paired-bootstrap CI on ρ excludes 1.
Per-item flip sets annotated by perturbation type (CIE methodology).
Battery: SCPP++ primary, +[SugarCrepe](https://arxiv.org/abs/2306.14610)/[ARO](https://arxiv.org/abs/2210.01936)(text-prior control)/WhatsUp/VSR/
MMVP-VLM; [Winoground](https://arxiv.org/abs/2204.03162) CI-only. Cost ≈ **180–240 GPU-h**; ComAlign has no
public code (excluded, cited); MUVERA needs 2-day FDE reimplementation or
drop.

**Method unlocked (verified unoccupied):** binding-preserving pooling —
hard-negative MARGIN distillation from full-MaxSim/cross-encoder teachers
onto the pooled budget. This is our crop-distillation machinery repurposed
— the ★★ crop reframe folds in as the method arm instead of standing alone.

**Kill-arms:** (1) ρ≈1 everywhere (most likely null — no breakpoint, no
paper); (2) no dynamic range R0→R4 → **1-day pilot NON-NEGOTIABLE before
commitment**; (3) ladder confound; (4) ARO text-prior artifact.
**Competitor:** Tübingen (Uselis/Koishigarina/Dittadi/Oh) hold 3 binding-
geometry papers in 2026 — the compression axis is their obvious next
empirical move. Speed matters.

**Venue:** CVPR (~Nov 13) — fills the slate's empty CVPR method slot.

## Cell B — record

Operators: patch-shuffle (the clean labeler) / block-shuffle / PE-shuffle /
PE-zero / **degradation-matched noise null (load-bearing)**. Per-item:
20 seeds, Wilson bound vs null, cross-model agreement ≥3/4 backbones (the
objectivity differentiator vs the temporal paper's model-specific
partition); BH-FDR per item, Holm on subset claims. Leaderboard rerank arm
([NegCLIP](https://arxiv.org/abs/2210.01936)/[DeGLA](https://arxiv.org/abs/2504.16801)/[LABCLIP](https://arxiv.org/abs/2502.03566)/[DCSM](https://arxiv.org/abs/2503.08723)). ~150 GPU-h. Kill-arms: <10% split (likely),
shuffle-to-chance, κ<0.4 replication failure, τ≥0.9 no-rerank.
Fabrication warning confirmed: [2601.09954](https://arxiv.org/abs/2601.09954)
does NOT do this diagnostic — encoder-swap only; never cite it for this.

## Wave-3 synthesis

The day's rule held: artifact/agenda-derived ideas (I1 from TF_Local+K3,
O1 from Raschka) gate at ★–★★; the measurement-derived cell (A) gates at
★★★½. Cell A additionally resolves the crop-reframe dilemma: instead of a
standalone ★★ efficiency paper, the distillation machinery becomes Cell A's
method arm at CVPR. Ordering: (1) Cell A 1-day pilot + citation re-sweep;
(2) if pilot shows dynamic range → prereg for CVPR; (3) Cell B 5 GPU-h
pilot piggybacks on the same encode passes.

## Addendum (same evening): LaSt-ViT — new Cell-A rung + mechanism citation

[**"Vision Transformers Need More Than Registers"** (LaSt-ViT, 2602.22394)](https://arxiv.org/abs/2602.22394),
HKU/SYSU, Feb 2026. Finds **"lazy aggregation"**: ViTs encode global
semantics through abundant BACKGROUND patches ("semantically irrelevant
background patches as shortcuts... driven by global attention and
coarse-grained semantic supervision"); registers do not fix it. Fix:
frequency-aware per-channel top-K selective aggregation into CLS. Huge
dense-task gains (CLIP ViT-L/14 VOC segmentation 17.1→72.4 mIoU) but
**zero compositional-ITM evaluation** across its 12 benchmarks.
[Code MIT, CLIP-B/16 + DINO checkpoints released](https://github.com/ChengShiest/LAST-ViT)
(441★, active).

Disposition — not a standalone paper for us; three uses:
1. **Cell A rung + battery baseline (A4-pattern, ~5–10 GPU-h):** run their
   released CLIP-B/16 through our compositional battery + add selective
   aggregation as a ladder rung. Sharp either way: recovery → selection is
   the binding bottleneck (mechanistic support for the execution camp);
   no recovery → **localization-recovery ≠ binding-recovery**, a
   dissociation nobody has shown.
2. **Cell B mechanism link:** lazy aggregation predicts CLS
   shuffle-INsensitivity (background multiset is permutation-invariant) —
   explains [2503.17349](https://arxiv.org/abs/2503.17349)'s tiny 0.2–2.7% drops; selective readouts should
   increase position-dependence. Free prediction inside Cell B's design.
3. **Competitor watch:** their stated future work is "apply insights to
   vision-language model alignment beyond CLIP" — one step from our lane;
   add to Cell A's scoop-watch list alongside Tübingen.

## Related

[[Method-Gates-Wave-2-2026-08]] · [[Researcher-Scan-Raschka]] ·
[[Prereg-Crop-Consistency-Distillation]] · [[Top-Researcher-Scan-2026-08]]
