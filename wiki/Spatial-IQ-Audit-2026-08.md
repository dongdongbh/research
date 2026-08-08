# Spatial-IQ Audit (2026-08-07) — findings, and the opportunity assessment

The professor pointed us at [Spatial-IQ](https://nvidia.github.io/Spatial-IQ/)
(NVIDIA Research). One afternoon of reading the released artifacts — code,
prompts, and per-item predictions, zero GPU — found three verified defects,
which we confirmed experimentally for $0.63 and reported:
**[NVIDIA/Spatial-IQ#1](https://github.com/NVIDIA/Spatial-IQ/issues/1)**
(filed under dongdongbh; the repo's first issue; their revision is in
flight).

## What we found (short form; full record in tier2gates/spatialiq-study-20260807/)

1. **The 5-choice headline is a prompt bug.** The released 5-choice prompt
   file is missing its `{choices}` placeholder, and Python's string
   formatting drops the options silently — models never saw them. All
   eight released model results sit at exactly chance with zero
   dependence on the answer's rank; restoring the options (200 items,
   their exact model and settings, pre-registered analysis) moves
   accuracy off chance (p=0.017) and makes the rank signature appear
   (p=2.2e-09), matching their own working 4-choice condition. Their
   parser also scores refusals as the answer "E".
2. **Distractor symmetry leaks.** Their distractors are sampled
   near-symmetrically around the correct value, so "answer the median"
   scores 27.9% against a 20% design chance (p≈1e-25) — a leak class
   that bites wherever options are visible.
3. **Statistics:** the consistency metric has an image-blind ceiling
   above the human baseline, and 3 of 7 claimed significant pairs fail
   false-discovery correction on their own released data.

## Is there an opportunity? Three, honestly graded

**1. The method-shaped one (needs a gate before anything): distractor
geometry as a leak class.** MMStar and Cambrian-1 defined blind/leakage
audits of benchmarks, but neither examines HOW DISTRACTORS ARE BUILT —
and Spatial-IQ shows symmetric numeric distractors create a
better-than-chance blind strategy that also interacts with per-model
rank biases. Generalized across N numeric-option benchmarks ("this leak
class exists broadly; here is the construction rule that removes it,
with a fixed re-release"), it qualifies under method route 3 (an old
idea — adversarial option construction — applied to a new problem).
**First cheap step (0 GPU): a census — how many benchmarks release
numeric option sets?** That number is the binding constraint; below ~5,
this is a note, not a paper. Scoop risk moderate (the MMStar lineage is
active); full two-pass gate with adjacent-vocabulary search mandatory
before commitment. Not on the slate; portfolio call at sign-off.

**2. The capability (not a paper — a lab asset).** This is the fourth
time in one week that reading RELEASED ARTIFACTS — code paths, prompt
files, per-item outputs — overturned a published claim in hours at ~zero
cost (MiCA reproduction, the Fisher-z interval audit, the VELOCITI judge
audit, now Spatial-IQ). That is a repeatable audit pipeline and it feeds
the lab's evaluator-validity theme directly: RoboJudge and the
uncertainty audit gain credibility from a public track record of exactly
this kind of finding. Treat "released-artifact audit" as a standing
first move whenever a benchmark or evaluator matters to us.

**3. The relationship.** A well-resourced NVIDIA Research group received
a careful, reproducible, collegially-written defect report while their
revision was in flight, under the owner's name. Low cost, real goodwill;
worth one line in any future conversation with that group.

## Lessons fed back into the gate rules

- Three of the four strongest findings came from code and outputs, not
  the paper.
- Adversarial peer-checking between two workers corrected FOUR confident
  framings (three one way, one the other) before anything reached the
  owner — single confident passes are not trustworthy on this class of
  claim.
- The join hazard: Spatial-IQ's item tuples are non-unique across
  generation batches; keying on them silently cross-matches. Full-path
  keys only.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Top-Researcher-Scan-2026-08]]
(theme V: the checkers are not checked) · tier2gates/spatialiq-study-20260807/
