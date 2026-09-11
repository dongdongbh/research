# Do chat VLMs read who does what to whom? A 124-image probe (2026-09-11)

**Answer in one line:** three open chat VLMs read the roles almost perfectly
when a body pose fixes them (ride, push, carry, feed), and fall to chance on
chase scenes, where they default to the common direction ("dog chases cat").
This is the same failure that CLIP-style scorers and image generators show.

A VLM (vision-language model) here means a chat model that looks at an image
and answers in words, trained to predict the next word. It is not a CLIP-style
model, which is trained to match a caption to its image inside a batch. The
question was whether the role-binding failure we study for CLIP-style models
(the [[Prereg-Crop-Consistency-Distillation]] line and the Sony proposal) also
appears in next-word VLMs.

## Data

124 generated images from the Sony motivation-figure work
(`cropdistill/runs/sony_motivation_20260904/`, batches gen2 to gen8; SD 1.5,
SDXL, and FLUX.1 schnell). Each image was made from one of two role-swapped
captions, such as "a photo of a dog chasing a cat" and "a photo of a cat
chasing a dog". The owner labeled by hand what each image actually shows
(`human_labels_20260910.json`). Only images with a clear direction were kept:
71 show the reverse of their prompt, 53 show the prompt as written. Images
labeled "unclear" were dropped. The item list is
`vlm_probe_items_20260911.json` in the same directory.

Two groups of scenes:

- **Pose scenes (71 images):** riding a horse, walking a dog on a leash,
  pushing a wheelchair, carrying, feeding, holding, handing a book, pouring
  tea, photographing. The pose or the object contact fixes who acts.
- **Chase scenes (53 images):** dog and cat, black dog and white dog, cat and
  mouse. Direction must be read from motion in a still frame.

## Protocol

Each model sees the image and the two captions and picks one:

> Which caption describes this photo correctly? (1) caption (2) caption.
> Answer with the number only.

Every image is asked twice with the captions in swapped order. An answer counts
as correct only when both picks name the caption the owner labeled as what the
image shows. This removes a "always pick option 1" habit from the score. By
chance, a model that guesses gets about 25% under this rule. A free-form
question ("who is doing what to whom?") was also logged for reading, not
scored. Greedy decoding, bf16, one L40S or A100 per model, about 2 minutes per
model. Script: `vlm_role_probe.py`; outputs in `vlmprobe_20260911/out/`.

Models (transformers 5.16, all open weights):
[Qwen2.5-VL-7B-Instruct](https://huggingface.co/Qwen/Qwen2.5-VL-7B-Instruct),
[Qwen3-VL-8B-Instruct](https://huggingface.co/Qwen/Qwen3-VL-8B-Instruct)
(the text encoder inside Ideogram 4), and
[InternVL3-8B](https://huggingface.co/OpenGVLab/InternVL3-8B-hf).

## Results

| Model | All 124 | Balanced over the two directions | Same answer in both caption orders | Images showing the prompt (53) | Images showing the reverse (71) | Pose scenes (71) | Chase scenes (53) |
|---|---|---|---|---|---|---|---|
| Qwen2.5-VL-7B | 68% | 71% | 90% | 81% | 58% | 96% | 30% |
| Qwen3-VL-8B | 65% | 67% | 82% | 77% | 55% | 92% | 28% |
| InternVL3-8B | 69% | 74% | 91% | 85% | 58% | 96% | 34% |

Agreement: 75 of 124 images are read correctly by all three models, and 33
are read wrongly by all three. The 33 are almost all chase scenes, and the
wrong answer is nearly always the frequent direction (dog chases cat), whatever
the picture shows.

Per set (InternVL3, sets with at least 3 images): horse 100%, leash 100%,
carry 100%, feed 100%, wheelchair 75 to 100%, cat/mouse 67%, plain dog/cat
chase 11 to 43%, black/white dog chase 0%.

## What this means

1. **Next-word VLMs are not immune.** On two-direction action scenes they
   collapse to the common direction, the "role collapse" that
   [RoleBench](https://arxiv.org/abs/2503.10037) reports for image generators.
2. **The failure is scene-dependent, not universal.** When contact or pose
   fixes the roles, all three models read them. The problem is directional
   actions whose evidence is motion.
3. **Why the text side does not save them.** These models read text with a
   next-word model, which tracks word order well. They see the image through a
   vision encoder pretrained with a CLIP-style matching loss (Qwen3-VL starts
   from SigLIP 2). Whether those image tokens keep "which one is the agent"
   is the same question our dual-encoder work asks.

For the Sony proposal this supports one sentence in the motivation: chat VLMs
show the same directional failure on the same images, so the problem is not
only the contrastive text encoder. It does not add an aim; the proposal stays
scoped to dual encoders.

## Caveats

- 53 chase images from three model families is a pilot, not a benchmark.
- Images are generated, not photographs; a generator's drawing errors can make
  a scene hard for any reader. The owner's labels only kept images with a
  readable direction, which limits this.
- The "images showing the reverse" column is lower for every model partly
  because those images are mostly chase scenes; the two splits are not
  independent.
- 9 to 18% of answers flip with caption order, so a part of the score is
  guessing even on the easy split.

## Evidence

- Items, per-image answers, and summaries:
  `cropdistill/runs/sony_motivation_20260904/vlmprobe_20260911/`
  (`SUMMARY_20260911.json`, `out/<model>.jsonl`, `out/<model>_summary.json`).
- OrangeGrid jobs 1115851.1, 1115852, 1115853 (2026-09-11). Node
  OG-NODE-10-5-174-134 fails every torch job with "No CUDA GPUs are
  available"; it is excluded in the submit files.
