# Unified Research-Direction Ranking — 2026-08-03

## Direction re-evaluation, version 2

**Status: THIS IS THE CURRENT RANKING.** It replaces the older ranking in
[[Direction-Reevaluation-2026-08]] and the opportunity lists in
[[Top-Researcher-Scan-2026-08]]. We keep those pages as a record.

We checked ideas in eight research areas. For every idea, we:

- searched the whole research area, not only one person's papers;
- saved quotes that show whether the idea has already been claimed;
- checked that the needed code, data, and model files actually exist;
- checked whether the work fits our computers and time;
- wrote a specific experiment plan before seeing results;
- named the new method that a diagnostic result could lead to; and
- checked whether the work could meet the ICLR 2027 abstract deadline on
  September 18.

Eight agents searched the research areas in parallel. One teammate checked
the code, data, and model files.

**Search limit:** WebSearch and OpenAlex ran out of budget, while arXiv and
Semantic Scholar limited our request rate. The results therefore lean heavily
on arXiv and Hugging Face. We did not search the native proceedings for CoRL,
RSS, GECCO, and ICSE as fully. Each research area records its own missing
coverage. Before choosing an abstract, run one more focused search.

## Current ranking

### Top group

The stars show our strength of recommendation. Five stars is the highest.
“Method unlocked” means the new method we could build if the study finds the
expected problem.

| # | Direction | ★ | Cost and cluster | ICLR? | Method unlocked |
|---|---|---|---|---|---|
| 1 | **Autoresearch false-discovery-rate (FDR) study plus a sound accept rule.** FDR is the share of accepted findings that are actually false. **The owner declined this as a paper on 2026-08-03** because a statistics-only contribution would look like engineering. The removed draft is in git history. Keep this only as an internal tool. | ~~★★★★★~~ | — | — | — |
| 2 | **Check whether robot-policy judges and evaluators are valid.** Main target: [RoboReward](https://arxiv.org/abs/2601.00675). Also change judges on [DreamGen](https://arxiv.org/abs/2505.12705), test ready-made VLM judges, and compare them with human rankings from [RoboArena](https://arxiv.org/abs/2506.18123). | **★★★★½** | 250–500 GPU-h; inference only; OrangeGrid | **Yes** | Rank-calibrated evaluator plus an “evaluator report card” |
| 3 | **Abbeel parallel-RL combination experiment.** Test combinations of the choices behind the disagreement between BRC and TD overfitting, using the authors' published setup. | **★★★★½** | 400–650 GPU-h; OrangeGrid; MuJoCo only | **Yes** | Minimal [FastTD3](https://arxiv.org/abs/2505.22642) recipe |

### Strong group: ★★★★

In this table, “direct comparison” means an experiment built to decide among
competing explanations.

| Direction | Cost | ICLR? | Note |
|---|---|---|---|
| Direct comparison of replay explanations (Liang [2603.04964](https://arxiv.org/abs/2603.04964)); test five possible causes, including a test that detects loss spikes | 250–400; OrangeGrid | Yes | Could lead to a stage-change schedule that does not use replay. Use Marin in two ways: help with public infrastructure, but keep our hypotheses private. |
| Tension between RLVR and self-consistency calibration (D. Zhou); DCPO misses all three needed parts | 100–250; OrangeGrid + Anvil | Yes | Could lead to calibrated agreement aggregation. OLMo-3 model families are free to use. |
| Test whether a critic transfers across agent systems, with a future-time [PACE](https://arxiv.org/abs/2606.08106) test as the validity section | 100–500 plus API cost | Yes | Release 8B critic weights. Our traces with known outcomes are an advantage; [TraceLab](https://arxiv.org/abs/2606.30560) means they are not unique. |
| One-step (1-NFE) generative diversity: is collapse caused by averaging, or is it built into the method? Drifting can disprove the claim, and recall can be computed now but has not been reported. | 150–350 | Likely | This belongs to a different venue and community from LLM collapse. Keep them as separate papers. |
| Measure KV-cache use in agent workloads; the footprint metric has never been tested there, and the gap was named publicly on July 9 | 200–400; OrangeGrid; evaluation only | Yes | Could lead to eviction that understands turn boundaries. PruLong checkpoints are missing, so drop it or retrain it. |
| Test Bengio's sparsity assumption from Requirement 5.23 using about 300 small prediction models | about 250; OrangeGrid | Yes | Decides whether to build the full contextualization method, M4. |
| Add the missing VLM test to [MOCHI](https://arxiv.org/abs/2409.05862); human reaction times are already released, and no citing paper claims this test | 20–60; OrangeGrid L40S | Yes | Could provide a training signal that matches human perception. |
| Bundle A, “the readout, not the representation”: test Isola's shared-Q idea on compositional tasks, compare map types, join COCO nuisance labels, and include binding location as one cited step | under 100; cached features | Yes, if gate passes | Could produce an alignment map that keeps compositional information. **First run the zero-GPU DataComp 7×7 Jaccard check.** |
| Track where Sutskever's RL environments came from | 350–600 plus heavy environment authoring | No; target ICML | **Decide within two weeks.** Microsoft Echoverse already owns nearby tools. |
| Compare three explanations for GMP (Song) | 100–250 | Yes | **First read the PDF and check whether the paper already removes each part itself.** |
| Levine offline-RL comparison, reframed as a test of the three failure modes in [2601.00831](https://arxiv.org/abs/2601.00831) | about 840, previously thought to be 200–500 | Tight | **Expires around September 13.** An independent group ran our method on OGBench on July 29. |
| Rank the useful parts of SigLIP-2 at different scales | 1,000–1,400 H100-h; **Delta 8×H200-long** | No; target ICML or CVPR | Three of five parts must be rebuilt in OpenCLIP because big_vision is frozen. |
| Arora drag-versus-fork direct comparison, as a small add-on | 50–200 | Yes | Test an intervention that keeps forks while changing context. |

### Ideas that moved down

| Direction | Old rating | New rating | Why |
|---|---|---|---|
| **T1: frozen parts × training goal × training stage, as first designed** | ★★★★½, rank 1 | **★★★** | Three of its four combinations appeared at CVPR, ICLR, or ICML 2026: CoVFT; **PIVOT, titled “RL makes MLLMs see better than SFT”**; and From Seeing to Thinking. We watched the wrong group as the publication risk. Our cost estimate was also about five times too low compared with PIVOT's 144 H100-h per recipe. A smaller idea still rates ★★★★: test the interaction plus {tuned}×{extra training goal}, add a GRPO arm at 1.5–3B, use PIVOT's harness, spend 150–250 GPU-h, and target a mid-level venue. |
| KV-cache combination study from scratch, M1 | ★★★★ in the researcher scan | **★★★** | Two later checks lowered it. [Cost-Optimal GQA (EMNLP 2025)](https://arxiv.org/abs/2503.09579) already tested the number of KV heads from scratch, including `n_kv=1`. [CLA](https://arxiv.org/abs/2405.12981) tested heads with layer sharing at equal memory. [MixAttention](https://arxiv.org/abs/2409.15012) tested layer sharing with attention windows. Only the three-way interaction remains. Run a pilot first. The LCKV repository already implements all three choices, so this is a port-and-scale study, not a new invention. |
| Choi hivemind breakdown | ★★★★ in the researcher scan | ★★★½ | The “first breakdown” claim is no longer true. Three papers from April–May 2026 give competing causes, but nobody has tested them against each other. What remains is the between-model axis plus a direct test of all three accounts. **Blocker: the Infinity-Chat Hugging Face dataset returns 404.** |
| Wei verifier-rule Q1 | ★★★★ in the researcher scan | ★★½ | An ICML 2026 study with 37 authors already owns the dataset and attention control. It does not test verifiability, but the missing part is easy for that group to add. |
| CAID cost matching as a paper | Tier 1 in the scan | Internal only | Five papers answered the main question in eight months, ending with **Nature Machine Intelligence on 2026-07-24**. The honest cost is $2,000–$5,000. Run only the smaller edit-isolation check for internal use. |
| Polychromic set-RL check | ★★★★ in the scan | ★★½ | The hidden factor is real, but none of the three targets has public code. A null result from our own reimplementation would not be convincing. |
| M5 diversity-preserving method | Method direction | Benchmark only | At least 15 methods already exist. The useful opening is a fair test that compares them at the same pass@1. |

### Ideas ruled out in this sweep

- **Hyperball combination study:** [2607.22444](https://arxiv.org/abs/2607.22444)
  already did it; our design was also wrong, and the source group had already
  disproved it.
- **M7 verifier hardening:** two papers claimed it three days apart in June
  2026.
- **Safety-aware KV allocation:** [2606.09864](https://arxiv.org/abs/2606.09864)
  ran the exact study on June 1 and found “no universal safe bit-width.” It
  also published diverge. AnchorKV published the allocator on June 16. Both
  papers appeared before our July check incorrectly called the idea open.
- **Binding-location comparison as first planned:** LABCLIP from February
  2025 and [DCSM, ICCV 2025](https://arxiv.org/abs/2503.08723) already cover it.
- **VPBench grid as proposed:** it mixes unlike categories. The COCO
  annotation join can still be used.
- **LPT token-matched control:** ACL 2026 already published the conclusion,
  and the original target is outdated.
- **PULSE replication:** there is no path to real use.
- **DriveJudge audit for now:** labels are not released. Put it on a watchlist
  and check again in six weeks.
- **BPP dose-response:** the public artifact called LIBERO-Gen does not exist.
- **Cosmos Policy audit as planned:** Cosmos 3 replaced the target. A useful
  piece remains: build and release the procedural-shift generator.

## Suggested group of projects

### Start now for the September 18 ICLR deadline

Draft pre-registrations already exist for
[[Prereg-Crop-Consistency-Distillation]] and [[Prereg-RoboJudge-Audit]].

1. **Crop-consistency distillation.** This is the main new-method project.
   The owner chose it on 2026-08-04 after applying the rule that a method must
   be more than a diagnostic test. See [[Method-Gates-2026-08]]. In week one,
   test the CLIPSelf checkpoint on our task set and test the aggregation-fix
   version.
2. **First paper in the judge-audit program.** It reuses our VLM evaluation
   tools, needs only inference, and does not compete with project 1 for the
   same resources. In week one, check the RoboArena download size.
3. **Run cheap add-ons at the same time:** [MOCHI](https://arxiv.org/abs/2409.05862)
   for 20–60 GPU-h, the calibration-tension test for 100–250 GPU-h, and Bundle
   A for under 100 GPU-h if its Jaccard check passes.

### Chosen for the next cycle

- One-step diversity → CVPR 2027; see [[Prereg-1NFE-Diversity]].
- Epistemic contextualization → ICML 2027; see
  [[Prereg-Epistemic-Contextualization]]. Start building the pipeline now.
  The updated design trains OLMo-2-1B in the middle of training and costs
  about 300–450 GPU-h. Training from scratch happens only if an earlier check
  says it is needed.

Keep these on the bench. Their older designs are in git history or the
research-area reports: the parallel-RL combination study, SigLIP-2 ingredient
ladder, replay comparison, and calibration-tension test. The parallel-RL idea
will expire as the source group continues testing its choices one by one.

Other reasonable next-cycle choices are the Abbeel combination study, the
SigLIP-2 ingredient ranking, the replay study, and the environment-source
study. The professor could replace the judge audit with Abbeel's project if a
more method-like top project is preferred. Start OpenCLIP engineering now for
SigLIP-2. Replay also fits ICLR. Decide on the environment-source study within
two weeks.

### Cheap checks to run this week

- DataComp 7×7 Jaccard: 0 GPU.
- RoboArena download-size check.
- Read the GMP PDF.
- Find or resolve the Infinity-Chat dataset.
- Pre-register the autoresearch noise-floor test and run 30 repeats: about
  10 GPU-h.

## Decisions shared across research areas

- Remove the collapse project's statistical stopping-rule track. It makes the
  same claim for the same reviewers as former flagship 1.
- Fit the 60,000-trajectory agent-variance study
  ([2602.07150](https://arxiv.org/abs/2602.07150)) once. Use it as a shared
  variance and power module for former flagships 1 and 2 and the critic paper.
- Put the predictive-validity idea
  ([2606.19704](https://arxiv.org/abs/2606.19704)) in the critic/PACE paper,
  not the FDR paper.
- Keep LLM collapse and one-step diversity as separate papers. They may share
  one measurement tool that compares coverage at the same quality.

## Lessons from this sweep

1. **A fresh check must find today's competitors, not only repeat the old
   claim search.** Three parts of T1 appeared at top venues while we watched
   the wrong group.
2. **Search the whole research area, not only one person's work.** Wide
   searches disproved four scan ideas: CAID, hivemind decomposition, M7, and
   safety-aware KV allocation.
3. **Every search must include the newest eight weeks.** Our July KV search
   found nothing but missed two June papers that had already closed the idea.
4. **Check that code, data, and checkpoints truly exist and work.** A null
   result using our copy of unreleased code is weak evidence, as in the
   polychromic idea. A named benchmark may not exist, as with LIBERO-Gen.
5. **Check quoted numbers in the original artifact.** We could not confirm the
   “7-run 0.016 CORE spread” in our notes, so we replaced it with evidence
   copied from the source.

## Related

[[Direction-Reevaluation-2026-08]] (older ranking, kept as a record) ·
[[Top-Researcher-Scan-2026-08]] · [[Self-Improving-AI-Survey]] ·
[[Method-Opportunities]] · [[Live-Research-Opportunities]] · [[Home]]
