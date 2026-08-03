# Wiki Citation Audit — 2026-08-02

Byproduct of the link-verification pass (38 pages, 101→956 links, every ID
resolved live; diff mechanically verified as link-wrapping only). This page
records the **factual conflicts found**, their status, and the unresolved
references, so cleanup is traceable. Fix opportunistically; items marked
FIXED were corrected same-day.

## Conflicts

1. **FIXED — Scientist AI quote misattributed.** The "specified solely at
   the level of requirements" quote + Def. 3.22 are in
   [2606.29657](https://arxiv.org/abs/2606.29657) (Jun 2026 technical
   paper), NOT the [2502.15657](https://arxiv.org/abs/2502.15657) position
   paper. [[Prereg-Epistemic-Contextualization]] corrected (also
   "fourteen months" → eighteen/one). [[Method-Gates-2026-08]] retains the
   original wording as a historical gate record — read with this note.
2. **FIXED — MMRV attribution.** Introduced by
   [SIMPLER (2405.05941)](https://arxiv.org/abs/2405.05941), not SC3-Eval.
   [[Prereg-RoboJudge-Audit]] corrected.
3. **FIXED — duplicate paper files** (`dcsm-clip-ideal.*`) removed
   (byte-identical to `is-clip-ideal-dcsm.*`).
4. OPEN — [DOR 2410.02681](https://arxiv.org/abs/2410.02681): ICML 2025
   per arXiv, labeled NeurIPS 2024 in [[Calibration-Prior-Art-Gate]]
   (survey page has it right). Same survey lists DAC/OV-Calibration
   ([2402.04655](https://arxiv.org/abs/2402.04655)) twice under two names.
5. OPEN — [[Method-Gates-Wave-2-2026-08]] nits: Representation Surgery is
   ICML 2024 only; MemDecay (2607.10582) is region- not turn-aware;
   MixAttention is Databricks systematizing the Character.AI blog recipe
   (not Character.AI's paper); line ~85 fuses 2509.19476 with Mai
   2603.12433; Terminal-Bench citation is the TB-2.0 report; "Dreamer"
   ambiguous v1/v2/v3.
6. OPEN — [[Next-Direction-Literature-Survey]]: VL-Uncertainty
   (2411.11919) is an unvenued preprint, not CVPR 2025; Conf-OT
   (2505.24693) is conformal prediction; UniPruneBench is not 2511.02650's
   title; TFLocal is a baseline label inside 2604.11496, not a paper.
7. OPEN — [[Unified-Direction-Ranking-2026-08]]: TraceLab (2606.30560) is
   a serving-workload paper — may not support the "killed exclusivity"
   claim about outcome-labeled agent traces; re-verify before relying.
8. OPEN — [[Method-Opportunities]]: PolyPythias "45 runs" vs "Fifty" in
   title; C2LIP mentions unverifiable.
9. NOTE — moved repos (still redirect): awesome-autoresearch →
   webfuse-com; stable-worldmodel → galilai-group; openevolve →
   algorithmicsuperintelligence. `uditgoenka/autoresearch` is a Claude
   Code skill, not the Codex alternative implied. "GR00T N1.7" has no
   public artifact.

## Unresolved references (left unlinked, correctly)

No arXiv exists (linked to OpenReview/GitHub where possible): FineCLIP,
MLLMCLIP, "CLIP Models Generalize Less…", nanochat, Terminal-Bench 1.0,
FunSearch, plus classical/stats citations (Horn 1965, Lenth, Wu & Hamada,
lme4, Gneiting & Ranjan…). Not verifiable (~40 short names in survey
pages, incl. AdaptVis/iMF/pMF tentative matches): see the linkify report
in the coordinator transcript 2026-08-02; verify before first load-bearing
use.

## Standing rule reminder

Links are added at writing time, verified, never guessed
([[project-wikis]] skill; memory `wiki-paper-links`). This audit page
exists because the rule arrived after 38 pages already existed.
