# Research Wiki — Home

This wiki stores research knowledge that is useful across our projects.
Details that belong to only one project stay in that project's own wiki:
`svib` for the NeurIPS paper and its evidence, and `svib2` for the
distillation work.

These pages were copied from those two wikis on 2026-08-02. This wiki is now
the main source for the shared pages. The old copies in `svib` and `svib2`
will not be updated.

The `wiki/` folder is copied to the GitHub Wiki after each push by
`.github/workflows/wiki-sync.yml`. Results and analysis code belong in the
main repository. See `results/`.

## What to read first

The stars show how strongly we recommend a research direction. Five stars is
the highest rating.

- **[[Unified-Direction-Ranking-2026-08]] — CURRENT RANKING, 2026-08-03.**
  We checked every idea across eight research areas. Three ideas first reached
  the top group: controlling false discoveries in automated research
  (autoresearch FDR) ★★★★★, checking whether AI judges are trustworthy
  ★★★★½, and testing combinations of parallel reinforcement-learning choices
  ★★★★½. We also found many strong ★★★★ ideas. T1 fell to ★★★ because three
  parts of it had already been published at top venues. Ten ideas were ruled
  out with evidence. The page also lists cheap checks to run in week one.
- **[[Method-Gates-2026-08]] — NEW.** This page checks the two candidates that
  meet the owner's definition of a new method. A new method must add a new
  approach, rather than only remove one part or report statistics. Both ideas
  survive. Crop-consistency distillation rates ★★★★. CLIPSelf already uses a
  related idea for dense prediction, but nobody has claimed our
  compositional image-text matching form with a frozen adapter and structured
  teacher. It must pass a week-one check using the CLIPSelf checkpoint.
  Epistemic contextualization also rates ★★★★. Here, “epistemic” means showing
  how certain or well-supported a statement is. No one has built this method
  yet. The key control is a version that adds only tags.
- **Draft pre-registrations waiting for approval:**
  ICLR 2027 — [[Prereg-RoboJudge-Audit]] (a study that diagnoses a problem)
  and [[Prereg-Crop-Consistency-Distillation]] (a new method).
  CVPR 2027 — [[Prereg-1NFE-Diversity]] (diagnosis).
  ICML 2027 — [[Prereg-Epistemic-Contextualization]] (method).
  Old drafts for ideas we no longer chose—an autoresearch accept rule, a
  parallel-RL combination study, and a SigLIP-2 ladder—were removed on
  2026-08-04. Git history can recover them. Their decisions and evidence
  remain in [[Unified-Direction-Ranking-2026-08]] and
  [[Method-Gates-2026-08]].
- [[Top-Researcher-Scan-2026-08]] — profiles of 26 leading researchers. It
  shows where several researchers point to the same problems: weak checking,
  loss of diversity, and competing explanations that no study has tested
  against each other. It also lists research ideas and work habits.
- [[Direction-Reevaluation-2026-08]] — a fresh check of all eight candidate
  directions. It ranks them by how much useful work remains, not by how many
  people are working nearby.
- [[Status-And-Survivors]] — an easy starting point. It explains every idea
  that was still alive, its cost, and its good and bad points. Its star table
  is older than the re-evaluation above. The SVIB section now lives in the
  `svib` repository.

## Earlier surveys and checks

- [[Method-Opportunities]] — possible new methods, including the baselines
  and numbers each method must beat.
- [[Live-Research-Opportunities]] — ideas about testing and evaluation.
- [[Self-Improving-AI-Survey]] — a review of 772 papers. Its old final choice
  was replaced by the re-evaluation, but its map of the field is still useful.
- Other surveys: [[Field-Scouting-Survey]] ·
  [[Math-Grounded-Direction-Survey]] ·
  [[Next-Direction-Literature-Survey]] ·
  [[Calibration-Opportunity-Survey]] · [[Compression-Audit-Direction]]
- Closed checks and pre-registrations: [[Direction-Gate-Results]] ·
  [[LLM-KD-Direction-Gates]] · [[KD-Noise-Floor-Stage1]] ·
  [[KD-Evidence-Audit-Gate]] · [[Calibration-Draw-Preregistration]] ·
  [[Calibration-Prior-Art-Gate]] · [[Temperature-Confound-Preregistration]]

## Cluster setup and resource guides

- [[GPU-Resources-Across-Clusters]] — which GPUs are available and which
  cluster should run each kind of job.
- [[Anvil-Interactive-GPU-Workflow]] — the normal way to run interactive GPU
  work on Anvil.
- [[Delta-Setup-and-Parallel-Workflow]] — how to set up Delta and use Anvil
  and Delta at the same time.
- [[Anvil-vs-Delta]] — hardware, queues, limits, storage, and costs for Anvil,
  Delta, DeltaAI, and Bridges-2.
- [[CUDA-Compatibility-and-vLLM]] — CUDA, driver, and vLLM compatibility.
- [[Data-Transfer-Between-Clusters]] — how to move large files among Anvil,
  Delta, and OrangeGrid.
- [[Data-and-Caches]] — where datasets, feature caches, and other saved files
  live under `/anvil/projects/x-cis261253/`, plus rules for tracking their
  source.
- [[Research-Automation-Tools]] — installed tools and skills.
- [[Anvil-H100-Qwen36-vLLM-Benchmark]] — measured H100 vLLM speed.

One extra hardware fact appears in [[Top-Researcher-Scan-2026-08]]: Isaac Sim
needs ray-tracing (RT) cores. It runs on L40S GPUs, but not on A100, H100, or
H200 GPUs. MuJoCo and SAPIEN run on all of them.

## Rules for choosing research

1. **Check recent work before running an experiment.** Read the method
   sections of the closest papers. Save the exact sentence that proves the
   question is still open. Do not trust a paper's own claim that it is new.
   Repeat any check older than about six weeks. Every search must include the
   newest eight weeks. A July check missed two June papers that had already
   answered its question.
2. **When checking again, find the current competitors.** Do not only repeat
   the old search. Three parts of T1 appeared at top venues while we watched
   the wrong group.
3. **Search the whole research area, not only one person's work.** Wider
   searches disproved four ideas that researcher-by-researcher searches had
   allowed through.
4. **Choose ideas by how much useful work remains.** Do not reject an area
   only because many people work there. If an area looks empty, explain why.
5. **Every study that diagnoses a problem must name the method it could lead
   to before registration.** First prove the problem. Then build the fix.
6. **Release a useful tool with every study that settles a dispute.** This
   may be a test harness, benchmark, or metric. A diagnostic result builds
   trust; a reusable tool helps others and earns citations.
7. **Check that the needed code, data, and checkpoints really work.** A null
   result from our own copy of unreleased code is not strong evidence. Confirm
   that each named benchmark and dataset exists and can be downloaded before
   rating an idea.
8. **Check every quoted number in the original paper or artifact** before
   putting it in a research plan.
9. Before spending compute, ask: **“Who would do something differently if
   this result were true?”**
