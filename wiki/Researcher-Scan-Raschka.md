# Researcher Scan — Sebastian Raschka (2026-08-02)

This is a one-person addition to [[Top-Researcher-Scan-2026-08]]. It was made
with the `researcher-scan` skill. A prompt-injection check found nothing unsafe
on his pages.

## What he is actually working on now

**He has no lab and no students.** "His lab" is a one-person company called
"Raschka AI Research (RAIR) Lab LLC," as shown in his newsletter footer. PyCon
DE 2026 lists him as independent and says he has "no corporate affiliation."
He used to be a professor at UW-Madison and resigned around 2022. He also used
to work at Lightning AI, but we could not confirm when he left.

His current work has three parts:

1. He explains and reverse-engineers model architectures. He wrote about 30
   posts in 2026 and has 200k subscribers. One example is his
   [Kimi K3 notes](https://sebastianraschka.com/blog/2026/kimi-k3-architecture-notes.html).
2. He writes books. *Build a Reasoning Model (From Scratch)* shipped on
   2026-06-30.
3. In the past five years, he has exactly one research paper as the last
   author: [MiCA (arXiv 2604.01694)](https://arxiv.org/abs/2604.01694), posted
   in April 2026. It has two authors and zero citations.

**He has done no VLM or compositional work. There is no opening for that
research area here.**

## His useful tools and data

- **[rasbt/LLMs-from-scratch](https://github.com/rasbt/LLMs-from-scratch)**
  (100k★, Apache-2.0, last pushed 2026-07-29): `ch04/` contains matched,
  CI-tested versions of nearly every important 2026
  efficiency method—KV cache, GQA, MLA, SWA, MoE, DeltaNet, DeepSeek sparse
  attention, and KV-sharing. They all use one codebase and one style. This
  removes the hidden differences caused by comparing unrelated implementations.
  The code is meant for teaching. Use it as a **trusted reference and source of
  unit tests, not as a training system**.
- **[rasbt/llm-architecture-gallery](https://github.com/rasbt/llm-architecture-gallery)**
  (last pushed 2026-08-02): a list of 93 models. For each model it gives
  KV-bytes/token, attention type, configuration links, and a comparison tool.
  It is a **structured list of vendor claims that have not been carefully
  tested against each other**. It is maintained weekly, costs us nothing, and
  can feed claim-mining work.
- [rasbt/reasoning-from-scratch](https://github.com/rasbt/reasoning-from-scratch)
  (4.9k★): GRPO, evaluation, and distillation for Qwen3-0.6B/1.7B. This is a
  cluster-S code base.
- `local-coding-agent-evals`: five tasks, one run per task, no random seeds,
  and **no license**. Treat this as a warning about weak evidence, not as a
  code base to build on.

## Claims with weaker evidence than they need

1. **MiCA does not cite or compare with MiLoRA**
   ([NAACL 2024, about 96 citations](https://arxiv.org/abs/2406.09044)), a PEFT
   method that uses the smaller singular-vector components. **Gate correction
   from a deep read on 2026-08-02:** two other complaints in the first scan were
   wrong. MiCA's LoRA comparison IS budget-matched: rank, learning rate (LR),
   and epochs were searched separately for both methods. Only full-FT is
   uneven. MiCA also cites [2602.04998](https://arxiv.org/abs/2602.04998), and
   §6 already includes a random-subspace control. With everything matched, the
   scores are Minor 75.63 > Random 73.75 > Major 74.21. The real problems are:
   no MiLoRA or PiSSA comparison, choosing from test curves, probes with only
   300/102 items, standard errors (SE) of 2.8–4.8 percentage points for claimed
   gains of 1.8–3 points, and data that was not released.
2. **Reasoning-effort control currently mixes four recipes that cannot be
   compared fairly.** Raschka admitted this himself in July 2026. The four
   recipe families are prompt conditioning, RL length penalties, mixed-mode
   SFT, and distillation from specialist models. K3 trains 9 specialists.
   ThinkDial covers part of the question, but no study compares several effort
   levels across all four families.
3. **NoPE everywhere as the normal frontier design has no independent test.**
   The open claim is conditional: NoPE may work only when linear/KDA layers
   carry position information. Expected effects are as small as those for
   the S-M1 class, so our budget would not give enough statistical power.
4. The 2026 area for mHC and attention-residual ablations is **already studied
   heavily. Do not enter.**
5. His experimental posts use `n=5`, one run each, and no seeds. The person who
   wrote the standard tutorial on model selection does not use that level of
   care in his LLM-era evaluations. In one person, this shows the main problem
   behind cluster V.

## Research openings, checked for earlier claims

| # | Study | Cost | Decision | ★ |
|---|---|---|---|---|
| O1 | **Careful test of spectral band × knowledge-injection method** — GATED 2026-08-02 and lowered. Four papers answer the learning-rate-artifact question for task adaptation: [2602.04998](https://arxiv.org/abs/2602.04998) covers PiSSA + MiLoRA; [2602.03493](https://arxiv.org/abs/2602.03493) tests bands and forgetting; [LoRAFactory 2601.22708](https://arxiv.org/abs/2601.22708); and [batch-size bias 2602.09492](https://arxiv.org/abs/2602.09492). The remaining experiment is band × knowledge injection, with TOST equivalence testing and a released benchmark checked for contamination. The benchmark has three levels: real post-cutoff facts / synthetic biographies / paired-regression forgetting. Use a kill-arm ladder: K1 is a 10 GPU-hour MiCA reproduction test. MiCA has no code or data, so failure ends the project. Possible venues are ICML/COLM; ICLR is impossible. The sharing advantage is ONE-SIDED because Raschka is unlikely to promote a paper that disproves his own paper. | 400–600 GPU-h (150–200 with a smaller scope) | SURVIVES-NARROWED | ★★ |
| O2 | **Careful comparison of all four reasoning-effort-control families** using his reasoning-from-scratch code. Release a recipe and checkpoints that can change effort level. The test must include ThinkDial. | 300–600 GPU-h | SURVIVES-NARROWED | ★★½ |
| O3 | Use the `ch04` implementations as trusted comparison references and the Gallery as the claims list. | ~0 | infrastructure, not a paper | — |
| O4 | Separate test-harness variation from model variation on his task pack. | 50–120 | crowded by OctoBench, AgentSpec, and ReliabilityBench 2026; at most run a 5 GPU-hour stop test | ★½ |

Sharing note: he has 200k subscribers and writes about other researchers' work
every week. A careful test of a claim he discussed could become visible very
quickly.

## Overall star rating

**★★★** — 2 as a researcher to watch because he has no lab, poses little risk
of publishing first, and has one preprint; **4 as a source of claims and useful
code**. He does the claim-finding step for us, releases matching implementations
that can test the claims, and is unlikely to run the careful comparison himself.

## How he connects to the scan's groups

**S — strong:** he supplies the small-scale area with trusted references, but
not a training system. **A:** he is the best public *index* of claims that need
careful comparison. **V:** he is an example and a target, not a contributor.
**D:** he does not work in this area; expect nothing.

## Limits of this scan

The WebSearch budget ran out early, so Google Scholar was never tried. Semantic
Scholar (S2) stayed rate-limited. The arXiv Atom API returned nothing through curl;
OpenAlex and WebFetch were used instead. The Lightning departure date is an
inference, not a confirmed fact.

## Related

[[Top-Researcher-Scan-2026-08]] · [[Method-Gates-Wave-2-2026-08]] ·
[[Direction-Reevaluation-2026-08]]
