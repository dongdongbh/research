# Weight Space Learning — scout report, 2026-08-09

**What this page is.** A first look at a field the owner asked about:
**Weight Space Learning (WSL)**. WSL means treating the numbers inside a
trained neural network — its **weights** — as *data*. Instead of feeding a
network pictures or text, you feed it *another network's weights*, and ask
questions like "how accurate is this model?", "what was it trained on?", or
"give me a new set of weights that does task X".

**Status: PRE-GATE. Every star on this page is a scout star.** Our own standing
lesson says scout stars fall by 1.5–2 stars once a real gate runs
([[Unified-Direction-Ranking-2026-08]], Part 5, lesson 6). Nothing here is a
recommendation to start work. Everything here needs the full two-pass gate,
an OpenReview sweep, and a verified cheap first step before it can enter a
pre-registration.

Work notes and reading records: `code/tier2gates/wsl-scout-20260808/`.

---

## 0. Words you need, defined once

- **Weights / parameters.** The numbers a network learns during training. A
  small image classifier has thousands; a large language model has billions.
- **Checkpoint.** One saved copy of all of a model's weights at one moment.
- **Model zoo.** A big collection of trained models, saved so somebody can
  study them as a *population* — the way a biologist studies many beetles
  rather than one.
- **LoRA (low-rank adaptation).** A cheap way to fine-tune a big model. You
  freeze the big model and train two small matrices `A` and `B`; their product
  `B·A` is added to a frozen weight matrix. A LoRA is a few megabytes where a
  full model is many gigabytes. Hugging Face hosts hundreds of thousands.
- **Adapter.** Any small add-on module like a LoRA. Used interchangeably here.
- **Hypernetwork.** A network whose *output is the weights of another network*.
- **Permutation symmetry.** Shuffle the hidden units in a layer, shuffle the
  matching rows and columns, and the network computes exactly the same
  function. Two very different-looking weight files can be the same model.
  This is the central nuisance in WSL.
- **GL symmetry of a LoRA.** For any invertible matrix `R`, the pair
  `(R⁻¹A, BR)` gives the same product `B·A`, so the same model. There are
  infinitely many ways to write the same adapter.
- **Equivariant / invariant architecture.** A network built so a harmless
  change to its input does not change (invariant) or changes in a matching way
  (equivariant) its output. Used so the learner is not fooled by the symmetries
  above.
- **Canonicalization.** Rewriting a weight file into one agreed standard form
  first, so all the equivalent versions collapse to the same thing.
- **Probe.** Two meanings, both used below. (1) A *linear probe* is a small
  classifier trained on frozen features to test what they carry — our house
  meaning. (2) A *probe input* is a fixed test input fed to an unknown model,
  whose output becomes a fingerprint.
- **Spectrum / singular values.** Every matrix can be summarised by a list of
  numbers describing how much it stretches space in each direction. Summarising
  a weight matrix this way is cheap and does not care about permutations.
- **Task vector.** The difference `θ_finetuned − θ_base`. Adding and
  subtracting task vectors is called *task arithmetic* or *model merging*.
- **AUC.** Area under the ROC curve: 1.0 means a detector separates two groups
  perfectly, 0.5 means it is guessing, and 0.0 means it is perfectly *backwards*.

Three words this lab uses that are worth spelling out, since they appear below:

- **Gate.** Our formal check on an idea before any work starts: read the source
  papers, pull their citations, search other communities, and name a cheap
  experiment that could kill the idea. An idea that has not passed a gate is
  "pre-gate".
- **Scoop risk.** The chance that somebody else has already published the idea,
  or will publish it before we finish. I give it as LOW / MEDIUM / HIGH and name
  the closest papers so the reader can check my judgement.
- **Cell.** A small patch of research territory — one question, attacked one
  way. Saying a cell is "occupied" means a named group is already working there.

---

## 1. The field map: who does what

WSL is small enough that about six groups cover most of it. Its own map is
[the March 2026 survey, arXiv 2603.10090](https://arxiv.org/abs/2603.10090),
whose author list *is* the map: Xiaolong Han, Zehong Wang, Bo Zhao, Binchi
Zhang, Jundong Li, **Damian Borth**, Rose Yu, **Haggai Maron**, Yanfang Ye, Lu
Yin, Ferrante Neri (Surrey, Notre Dame, UC San Diego, Virginia, St. Gallen,
Technion, NVIDIA). The community index the owner sent,
[Awesome-Weight-Space-Learning](https://github.com/Zehong-Wang/Awesome-Weight-Space-Learning),
comes from the same group and uses the same three-part split:

1. **Understanding** — the geometry of weight space: symmetries, mode
   connectivity, quotient spaces.
2. **Representation** — turn weights into a vector you can compare, search, and
   predict from.
3. **Generation** — produce new weights with a hypernetwork, diffusion model, or
   flow.

### 1.1 Damian Borth ([University of St. Gallen, HSG-AIML](https://ics.unisg.ch/chairs/damian-borth-artificial-intelligence-and-machine-learning/)) — hyper-representations and the zoos

A note on the chair page the owner sent. It lists four research lines:
representation learning, multimodal analysis, remote sensing, and financial
data. It **never mentions weight-space learning or hyper-representations at
all**. The weight-space work is visible in only two places: the group's papers
and its [GitHub organisation](https://github.com/HSG-AIML), and the current
team — Léo Meynent, Damian Falk, Konstantinos Tzevelekakis, Aron Asefaw — who
are the authors on the papers below. Konstantin Schürholt, the line's main
first author, is no longer listed on the team page.

Borth's group invented the "train an autoencoder on model weights" line and,
just as importantly, **built most of the model zoos the whole field trains on.**

- [Self-supervised representation learning on neural network weights (NeurIPS 2021)](https://github.com/HSG-AIML/NeurIPS_2021-Weight_Space_Learning)
  and [Hyper-Representations as Generative Models (NeurIPS 2022)](https://github.com/HSG-AIML/NeurIPS_2022-Generative_Hyper_Representations).
- [SANE, ICML 2024, arXiv 2406.09997](https://arxiv.org/abs/2406.09997)
  (Schürholt, Mahoney, Borth) — the scaling paper. It chops a big model's
  weights into a *sequence* of small windows so a transformer can read them.
  **Its ceiling is ResNet-18, about 12 million parameters, 140 models per zoo.**
  Code: [HSG-AIML/SANE](https://github.com/HSG-AIML/SANE).
- Zoos, all April 2025: [A Model Zoo of Vision Transformers (arXiv 2504.10231)](https://arxiv.org/abs/2504.10231)
  — 250 ViTs; [A Model Zoo on Phase Transitions (arXiv 2504.18072)](https://arxiv.org/abs/2504.18072)
  — 12 populations; [The Impact of Model Zoo Size and Composition on Weight
  Space Learning (arXiv 2504.10141)](https://arxiv.org/abs/2504.10141).
- **The scaling move that matters:** [Learning Model Representations Using
  Publicly Available Model Hubs (arXiv 2510.02096)](https://arxiv.org/abs/2510.02096)
  (Falk, Schürholt, Tzevelekakis, Meynent, Borth, Oct 2025). They stop building
  zoos. Instead they train the weight encoder on real Hugging Face models:
  **2,000 training + 200 validation Hugging Face models, 171 billion parameters
  in total, individual models up to 1.3B parameters**. Of those models, 42% are
  transformers and 31.4% are undocumented. Their own backbones are 456M and
  900M parameters. This setup beats zoo-trained backbones. **It is the single
  most direct occupant of "classic WSL meets the real model hub".**
- Newest: [WeightCLIP (ICML 2026, arXiv 2607.03551)](https://arxiv.org/abs/2607.03551)
  ([ICML poster page](https://icml.cc/virtual/2026/poster/61346))
  — one encoder for weights, one for data samples, aligned contrastively so you
  can ask for "a model for this dataset". Code:
  [HSG-AIML/weightCLIP](https://github.com/HSG-AIML/weightCLIP). **Its zoos are
  10,000 CNNs of about 1.1M parameters and 1,000 half-width ResNet-18s.** An
  ICML 2026 paper on 1-million-parameter CNNs: the small-zoo tradition has not
  gone away.
- [GeoSANE (CVPR 2026, arXiv 2603.23408)](https://arxiv.org/abs/2603.23408)
  (Hanna, Falk, Yu, Borth) — the same idea on Earth-observation models: **103
  models, ~38 billion parameters total, tokenized into ~165M weight tokens**.

### 1.2 [Haggai Maron](https://haggaim.github.io/) (Technion + NVIDIA) — equivariant weight-space networks

Maron's line is the mathematics: build architectures that respect the
symmetries so the learner is not fooled by them.

- [Equivariant Architectures for Learning in Deep Weight Spaces (DWSNet),
  ICML 2023, arXiv 2301.12780](https://arxiv.org/abs/2301.12780) (Navon,
  Shamsian, Achituve, Fetaya, Chechik, Maron). Gives a *full characterisation*
  of every affine layer that respects the permutation symmetry of an MLP's
  weights. This is the founding architecture paper. Its own framing is "a first
  step"; it works on small MLPs and implicit representations.
- [Graph Metanetworks for Processing Diverse Neural Architectures
  (arXiv 2312.04501)](https://arxiv.org/abs/2312.04501) (Lim, Maron, Law,
  Lorraine, Lucas) — turn the network into a graph, run a graph network on it,
  so one model reads many architectures.
- **The LoRA move, and the paper to read before proposing anything about
  reading adapters:** [Learning on LoRAs: GL-Equivariant Processing of Low-Rank
  Weight Spaces for Large Finetuned Models (arXiv 2410.04207)](https://arxiv.org/abs/2410.04207)
  (Putterman, Lim, Gelberg, Jegelka, Maron; Oct 2024; the OpenReview version
  adds Bronstein). It defines the GL symmetry and builds invariant encoders. It
  already applies them to four tasks: **performance prediction, fine-tuning
  attribute detection, membership inference, and downstream accuracy
  estimation**. It runs those tasks on text-to-image diffusion LoRAs and on
  language-model LoRAs.
- 2026: [On the Expressive Power of Permutation-Equivariant Weight-Space
  Networks (arXiv 2602.01083)](https://arxiv.org/abs/2602.01083) (Dayan, Eitan,
  Maron), **ICML 2026 spotlight** — shows the main equivariant weight-space
  architectures are equally expressive; the redesign that follows gains 34%.
- [SHINE (arXiv 2602.06358)](https://arxiv.org/abs/2602.06358) (Liu, Wang, Mao,
  Gelberg, Maron, Zhang) — an in-context hypernetwork that turns a context into
  a LoRA in one forward pass, on **Qwen3-8B, trained on 6 billion tokens**.

### 1.3 [Kai Wang](https://kaiwang960112.github.io/) (Tencent Hunyuan; before that NUS) and Zhangyang Wang (UT Austin) — weight generation

- [Neural Network Diffusion ("p-diff"), arXiv 2402.13144](https://arxiv.org/abs/2402.13144)
  — train a diffusion model on saved checkpoints, then sample new weights.
- [Recurrent Diffusion for Large-Scale Parameter Generation (RPG),
  arXiv 2501.11587](https://arxiv.org/abs/2501.11587) — about 200 million
  parameters generated on one GPU; NeurIPS 2025.
- [Drag-and-Drop LLMs: Zero-Shot Prompt-to-Weights, arXiv 2506.16406](https://arxiv.org/abs/2506.16406)
  (Liang et al., with Schürholt, Borth, Bronstein, You, Zhangyang Wang, Kai
  Wang; NeurIPS 2025) — a text encoder reads unlabeled prompts and a cascaded
  decoder emits a whole LoRA in 0.11–0.73 seconds. Bases: Qwen2.5-0.5B / 1.5B /
  7B and Qwen2.5-VL-3B.
- [Position: Weight Space Should Be a First-Class Generative AI Modality,
  arXiv 2605.18632](https://arxiv.org/abs/2605.18632) (Zhangyang Wang, Peihao
  Wang, Kai Wang; ICML 2026). The manifesto. It frames all weight generation as
  sampling `p(W | A, C, R)` — weights given **A**rchitecture, **C**ondition, and
  training **R**ecipe. Note: **zero citations as of this sweep**, which is
  normal for a two-month-old position paper but worth re-checking.

### 1.4 Yedid Hoshen and Eliahu Horwitz (Hebrew University) — model search, model atlas, lineage

This group owns "there are a million models on Hugging Face and nobody knows
what they do".

- [Deep Linear Probe Generators for Weight Space Learning (ProbeGen),
  ICLR 2025, arXiv 2410.10811](https://arxiv.org/abs/2410.10811) (Kahana,
  Horwitz, Shuval, Hoshen) and
  [Learning on Model Weights using Tree Experts (ProbeX), CVPR 2025,
  arXiv 2410.13569](https://arxiv.org/abs/2410.13569) (Horwitz, Cavia, Kahana,
  Hoshen). ProbeX's Model-J zoo is **14,000 models**: 4,000 discriminative
  fine-tunes and **10,000 Stable Diffusion LoRA personalizations** — all
  self-trained.
- [Can this Model Also Recognize Dogs? Zero-Shot Model Search from Weights
  (ProbeLog), arXiv 2502.09619](https://arxiv.org/abs/2502.09619). Describes
  each *output logit* by how a fixed set of probe inputs activates it. Scale:
  1,500 classifiers → 85,000+ logits, **but only 71 real Hugging Face models**.
- [We Should Chart an Atlas of All the World's Models, arXiv 2503.10633](https://arxiv.org/abs/2503.10633).
  The numbers that define the problem: Hugging Face hosts **over 1.5 million
  models, ~100,000 added monthly; over 60% have no documentation; fewer than 30%
  name a parent model.** They build an atlas of 63,000 documented models and
  analyse 314,000. Project page: [horwitz.ai/model-atlas](https://horwitz.ai/model-atlas).
- [Unsupervised Model Tree Heritage Recovery (MoTHer), ICLR 2025,
  arXiv 2405.18432](https://arxiv.org/abs/2405.18432) — recovers who was
  fine-tuned from whom, from weights. Benchmark of 500+ models; they scanned
  over 800,000 Hugging Face model cards and found ~470,000 uninformative.
- [Dataset Size Recovery from LoRA Weights (DSiRe), arXiv 2406.19395](https://arxiv.org/abs/2406.19395)
  — predicts *how many images* a LoRA was fine-tuned on, from the **norm and
  spectrum** of its matrices. Released the LoRA-WiSE benchmark: over 25,000
  weight snapshots from more than 2,000 fine-tuned LoRA models. **Read this
  before proposing anything that reads LoRA spectra to recover a training
  fact — that idea is established.**
- [Discovering Hidden Gems in Model Repositories, arXiv 2601.22157](https://arxiv.org/abs/2601.22157)
  (Jan 2026) — finds under-downloaded fine-tunes that beat popular ones across
  2,000+ real Hub models. Behavioural, not weight-space: a notable pivot away
  from weights by the group that owns weight-space search.

### 1.5 The survey's own group (Notre Dame / Surrey) — and the LoRA-reading paper

[W2T: LoRA Weights Already Know What They Can Do, arXiv 2603.15990](https://arxiv.org/abs/2603.15990)
(Xiaolong Han, Ferrante Neri, Zijian Jiang, Fang Wu, Yanfang Ye, Lu Yin, Zehong
Wang; Mar 2026) — same first and senior authors as the survey. It **fixes the
GL ambiguity by canonicalizing each LoRA update with a QR decomposition
followed by an SVD**, then tokenizes the canonical form for a transformer.
Tasks: attribute classification, performance prediction, adapter retrieval,
without ever running the base model. **This is the largest LoRA zoo I found:
CelebA-LoRA 10,177 adapters; CUB-LoRA 11,788; GoEmotions-LoRA 20,000;
ARC-Easy-LoRA 10,000; retrieval pool 1,296 gallery + 600 query. Bases:
Llama-3.2-3B and Stable Diffusion v1.4. All self-trained.** Its abstract does
not discuss robustness to adversarial or function-preserving transforms, or
generalisation across fine-tuning recipes.

### 1.6 Others the sweep surfaced

- **Frederic Sala (Wisconsin)** — [WARP: Weight-Space Analysis for Recovering
  Training Data Portfolios, arXiv 2607.01686](https://arxiv.org/abs/2607.01686)
  (Jul 2026, ICML 2026 Weight-Space Symmetries workshop). It recovers *what
  mixture of data sources* a model was fine-tuned on, using only the released
  weights. The trick: interpolate from base to fine-tuned to fake a training
  trajectory, then read that trajectory's geometry. Models: BERT and GPT-2.
  Mean absolute error: 0.046 and 0.104.
- **Giorgos Bouritsas / Yannis Panagakis (Athens)** — scale-equivariant graph
  metanetworks (NeurIPS 2024);
  [Metanetworks as Regulatory Operators (arXiv 2512.15469)](https://arxiv.org/abs/2512.15469).
- **Tan M. Nguyen (NUS)** — [Quasi-Equivariant Metanetworks (arXiv 2604.23720)](https://arxiv.org/abs/2604.23720),
  ICLR 2026.
- **Sakana AI (Robert Lange)** — [Text-to-LoRA (arXiv 2506.06105)](https://arxiv.org/abs/2506.06105),
  ICML 2025, and [Doc-to-LoRA (arXiv 2602.15902)](https://arxiv.org/abs/2602.15902),
  Feb 2026.
- **Zhuang Liu / Boya Zeng (Princeton)** — the memorization critique, §2.4.
- **Jaejun Yoo (UNIST)** — [What Linear Probes Miss: Multi-View Probing for
  Weight-Space Learning (MVProbe), arXiv 2605.23410](https://arxiv.org/abs/2605.23410),
  ICML 2026. Adds Gram-matrix interaction views to probing and beats ProbeX,
  including on Stable Diffusion LoRAs. **It says plainly that "processing
  full-scale weights is computationally prohibitive" — that is why they probe.**
- **Yaoqing Yang (Dartmouth) + Michael Mahoney (Berkeley/ICSI) + Shiwei Liu** —
  the cheap-statistics school. They never touch raw tensors. They read the
  *spectrum* — the singular-value distribution — of each weight matrix. That is
  data-free, permutation-blind, and cheap enough for a CPU.
  [Spectral Signatures of Large Language Models (arXiv 2607.03377, KDD 2026)](https://arxiv.org/abs/2607.03377)
  does lineage tracking, clustering, and performance prediction across **499
  models from 19M to 70B parameters**.
  [AlphaPruning (NeurIPS 2024, arXiv 2410.10912)](https://arxiv.org/abs/2410.10912)
  and [FARMS (ICML 2025, arXiv 2506.06280)](https://arxiv.org/abs/2506.06280)
  come from the same school. **FARMS matters for us specifically.** It shows
  that a weight matrix's *aspect ratio* — its shape — biases heavy-tail
  spectral estimates. It fixes that by subsampling submatrices with a fixed
  aspect ratio. So any cross-layer or cross-model spectral comparison that
  ignores shape is measuring shape as much as substance. This group also
  bridges to Borth's: they co-authored the
  [phase-transition model zoo](https://arxiv.org/abs/2504.18072).
- **Charles Martin (WeightWatcher)** — still active; his
  [anti-grokking detector (arXiv 2602.02859)](https://arxiv.org/abs/2602.02859)
  reports data-free failure signals in weights, including in GPT-OSS 20B/120B.

### 1.7 Where the field meets, and a warning about venues

- **[ICLR 2025 Workshop "Neural Network Weights as a New Data Modality"](https://weight-space-learning.github.io/)**
  — 27 April 2025, Singapore; 45 accepted papers, 5 spotlights. Organisers:
  Schürholt, Bouritsas, Horwitz, Lim, Gelberg, Bo Zhao, Allan Zhou, Borth,
  Jegelka. Steering: Bronstein, Chechik, Stella Yu, Maron, Hoshen. This one page
  is the best single snapshot of the field.
- **There was no 2026 sequel.** Verified against the full workshop lists for
  [ICLR 2026](https://iclr.cc/virtual/2026/events/workshop) (40 workshops, none
  on weight space) and [NeurIPS 2025](https://neurips.cc/virtual/2025/events/workshop)
  (56 workshops, none); NeurIPS 2026 workshops are not announced.
- **The 2026 venue is narrower:** the
  [ICML 2026 Workshop on Weight-Space Symmetries](https://weightsymmetry.com/)
  ([ICML page](https://icml.cc/virtual/2026/workshop/54080)), 10 July 2026,
  Seoul — Yani Ioannou, Boris Knyazev, Ekaterina Lobacheva, Adnan Mohammed,
  Antonio Orvieto, Alexander Theus. Speakers included Borth, Sidak Pal Singh,
  Gintare Karolina Dziugaite. The framing moved from "weights as data" to
  "symmetries and merging".

**Read that history before committing.** The broad "weights as data" community
lost its dedicated workshop after one year, and the surviving venue belongs to a
different, more theory-flavoured crowd. A paper from us would go to a main
conference and be reviewed by the people in §1.

---

## 2. What changed in 2025–2026

### 2.1 The field named itself

Three artifacts in six months: the
[survey (Mar 2026)](https://arxiv.org/abs/2603.10090), the
[position paper (May 2026)](https://arxiv.org/abs/2605.18632), and the
[awesome-list](https://github.com/Zehong-Wang/Awesome-Weight-Space-Learning).
When a field writes its survey, the easy ideas are taken and the hard ones are
written down as "open questions". Both documents state theirs, and I quote them
in §2.5 because they are the honest map of what is left.

### 2.1.1 How crowded is it — the numbers

Counts of arXiv papers whose **abstract** contains each phrase (measured
2026-08-08; these are lower bounds because abstracts do not always name the
method):

| Phrase | 2024 | 2025 | 2026 (Jan–Aug 1) | 2026 rate |
|---|---|---|---|---|
| "model merging" | 132 | 245 | 182 | **~26 / month** |
| "task vector" | 37 | 90 | 63 | ~9 / month |
| "weight space" (any use) | 127 | 166 | 204 | ~29 / month |
| **"weight space learning"** (the field's own name) | 2 | 7 | 8 | **~1.1 / month** |
| "heavy-tailed self-regularization" | 3 | 6 | 6 | ~0.9 / month |
| "model zoo" | 17 | 17 | 5 | ~0.7 / month |
| "model atlas" | 0 | 4 | 2 | ~0.3 / month |

Read this carefully, because it cuts both ways. **Weight-space learning as a
named field is genuinely small — about one paper a month.** The stampede is in
the rooms next door: model merging at 26 papers a month, and LoRA serving and
ecosystem mining beyond that. Our own kill ledger already lost one idea per week
to fast-moving neighbourhoods, so the relevant question is never "is this field
crowded?" but "which neighbouring room will publish my idea first?"

### 2.2 Generation reached LLM scale — this half of the gap is CLOSED

The owner's intuition was that classic WSL never caught up with the LLM/LoRA
explosion. On the **generation** side that stopped being true during 2025–2026.
All verified:

| Work | Scale it operates at |
|---|---|
| [Text-to-LoRA (ICML 2025)](https://arxiv.org/abs/2506.06105) | LoRA from a task description, trained on 9 adapters |
| [Doc-to-LoRA (Feb 2026)](https://arxiv.org/abs/2602.15902) | Gemma-2-2B, Mistral-7B, Qwen3-4B |
| [SHINE (arXiv 2602.06358)](https://arxiv.org/abs/2602.06358) | Qwen3-8B, 6B pretraining tokens |
| [Drag-and-Drop LLMs (NeurIPS 2025)](https://arxiv.org/abs/2506.16406) | Qwen2.5 up to 7B, plus a 3B VLM |
| [ORAL (arXiv 2503.24354)](https://arxiv.org/abs/2503.24354) | Mistral-7B; hundreds of millions of LoRA parameters |
| [ICM-LoRA (IJCAI 2025, arXiv 2501.17635)](https://arxiv.org/abs/2501.17635) | Llama-3-8B |
| [RPG (arXiv 2501.11587)](https://arxiv.org/abs/2501.11587) | ~200M parameters generated in one pass |
| HY-WU (Tencent, 2026, named in the position paper §4.5) | 80B multimodal backbone, 13B active, 8.11B generator emitting 0.72B LoRA parameters |
| [LoRAGen (ICLR 2026 poster)](https://openreview.net/forum?id=mrafO7aTYj) | FLAN-T5-large, Gemma-2-2B-Instruct. **OpenReview only — it is not on arXiv or Semantic Scholar; do not cite an arXiv ID for it** |
| [WIZARD (arXiv 2606.07217)](https://arxiv.org/abs/2606.07217) | LoRAs for vision-language-action robot policies |
| [MoEGen (arXiv 2608.03275)](https://arxiv.org/abs/2608.03275) | 4 August 2026 — instance-adaptive LoRA generation |

The position paper's own summary: *"today's strongest evidence is not
unrestricted full-checkpoint generation at frontier scale, but large-backbone,
adapter-scale, and conditional weight generation."* Two industry labs (Tencent,
Sakana) and several academic groups publish here monthly. **Treat LLM/LoRA-scale
weight generation as a crowded, well-funded cell. We should not enter it.**

### 2.3 The representation side: where the gap really is

Here the owner's intuition holds, but the shape of the gap is not what it looks
like from outside.

**Evidence the gap is real.** The survey's benchmark table (§6, Table 4) lists
the zoos the field trains on. They are: 3.8 million small 3-layer CNNs; 1.4
million SIREN implicit-representation MLPs; 1.7 million sparsified 3-layer
CNNs; and 60K–161K LoRAs for *image* diffusion models. **The entire
"Transformers" subsection of that table contains one entry** — Falk et al.
2025, with ten ViT-S pretraining seeds plus 240 fine-tuned heads. **No large
language model zoo appears in the table at all.** The survey says so itself in
§7.2. Current WSL is *"validated mostly on small or medium-sized networks"*,
and transformer-scale attempts *"typically explore partial weight spaces or
undertrained checkpoints."* As late as ICML 2026, WeightCLIP's zoos are
1.1-million-parameter CNNs.

**But three different escapes are already in use, and each one is somebody's
territory.**

*Escape 1 — don't read the weights at all.* ProbeLog and MVProbe describe a
model by its answers to fixed probe inputs, so model size is irrelevant.
MVProbe states outright that full-scale weights are computationally
prohibitive. Hoshen owns this route.

*Escape 2 — read a cheap summary of the weights instead of the tensors.* This
is where real LLM scale already exists.
[AWM (arXiv 2510.06738, ICLR 2026)](https://arxiv.org/abs/2510.06738) first
solves a linear assignment problem to undo neuron permutation, then compares
models with CKA. It runs on **150 model pairs from 1.3B to 70B parameters**. It
stays robust through supervised fine-tuning, 5.5 trillion tokens of continued
pretraining, reinforcement learning, pruning, and upcycling.
[Spectral Signatures of Large Language Models (arXiv 2607.03377, KDD 2026)](https://arxiv.org/abs/2607.03377)
uses the shape of each weight matrix's singular-value distribution as a
data-free signature across **499 leaderboard models from 19 million to 70
billion parameters**.
[Scaling Linear Mode Connectivity and Merging to Billion Parameter Pretrained
Transformers (arXiv 2606.23607)](https://arxiv.org/abs/2606.23607) runs the
symmetry machinery at billion scale.

*Escape 3 — read adapters, not full models.* This is now the field's default,
and the base model is almost always 3B–8B. Adapter zoos of 10,000–20,000 are
normal: [W2T](https://arxiv.org/abs/2603.15990) (10K–20K adapters on
Llama-3.2-3B), [ProbeX/Tree Experts](https://arxiv.org/abs/2410.13569) (10,000
Stable Diffusion LoRAs), [Learning on LoRAs](https://arxiv.org/abs/2410.04207)
(thousands).

**What is genuinely still missing.** No paper trains an encoder that ingests
the *raw weight tensors* of a 7B+ model. Everything at that size uses a
summary. That is the true remaining gap — and it is the survey's own flagship
open question (§7.2), which means every group in §1 is pointed at it with more
compute than we have.

**Verdict on the owner's intuition.** Half right, and the useful half is not
where it looks. Yes, classic WSL was built on small CNN zoos and has not caught
up on raw LLM weights. But the field noticed eighteen months ago, three escape
routes are already in production, and the groups holding them are exactly the
groups that could out-spend us. **The honest opening is not "scale WSL to
LLMs". It is the thing nobody in §1 is checking: whether these readouts survive
contact with an adversary, with a change of fine-tuning recipe, or with real
wild models instead of self-trained zoos.**

### 2.4 A critique wave hit the generation line — and the obvious audit is taken

[Generative Modeling of Weights: Generalization or Memorization?
(arXiv 2506.07998)](https://arxiv.org/abs/2506.07998) — Boya Zeng, Yida Yin,
Zhiqiu Xu, Zhuang Liu; **CVPR 2026 Highlight**; code at
[boyazeng/weight_memorization](https://github.com/boyazeng/weight_memorization).
It audits four flagship weight generators (Hyper-Representations, G.pt,
HyperDiffusion, p-diff) and finds they *"synthesize weights largely by
memorization: they produce either replicas, or, at best, simple interpolations
of the training checkpoints"*, losing to two trivial baselines — adding noise to
a checkpoint, and averaging checkpoints.

Two facts about it matter to us. It is an **inside job**: its first two authors
and its senior author are also authors on p-diff. And **there is no rebuttal** —
p-diff's last revision is December 2024, its abstract still claims the opposite,
and the community answered by scaling up rather than by replying.

**The lesson is a warning, not an opening: the obvious audit of this field has
already been done, well, by insiders, and it landed as a CVPR Highlight.**

### 2.5 The field's own list of what is open

From the survey §7, in plain words:

- **§7.1** No architecture-agnostic weight reader exists. The survey suggests
  LoRA as the universal format.
- **§7.2** Scaling is blocked by input size, symmetry, and inter-layer
  dependence. Suggested routes: shared weight templates plus layer tokens;
  generate *fine-tuning modules* not full tensors; compress into a
  symmetry-reduced subspace.
- **§7.3 Robustness and safety** — the shortest section, with the fewest
  citations. Verbatim: *"Most existing works prioritize performance and
  efficiency, leaving security largely unexplored."* Two named sub-problems:
  (a) controllable, auditable weight generation; (b) **"adversarial risk
  detection and defense in weight space… developing anomaly metrics and
  embedding-based probes to identify suspicious weight patterns"**. Its only
  citation for (b) is one workshop paper (Shor et al., §3.2).

The position paper's §5(iv) names the same hole from the other side: a
**"model-supply-chain problem: provenance, lineage tracking, memorization
tests, safety audits, and quarantine of untrusted checkpoints."** Its §5(iii)
adds the sentence that candidate D1 below is built on: checkpoints differ by
optimizer, objective, data mixture and fine-tuning recipe, so *"treating all
checkpoints as samples from one unconditioned density is unlikely to succeed…
such metadata should be used for filtering, stratification, or conditioning,
rather than ignored."*

**Both flagship 2026 documents point at the same corner: safety, and robustness
of weight-space readouts.** But the corner is no longer empty — see §2.6, which
is the most important section of this page.

### 2.6 The safety-from-weights wave: twelve months old, and already contested

This is where the field moved while its own survey was in press, and it changes
every recommendation below. Reading models' weights to decide whether they are
*safe* barely existed a year ago. Six of the eight relevant papers are from
2026, and the newest landed **six days before this page was written**.

**What is being claimed:**

| Paper | Date | What it reads | Headline |
|---|---|---|---|
| [Weight-space Detection of Backdoors in LoRA Adapters (arXiv 2602.15195)](https://arxiv.org/abs/2602.15195) | Feb 2026 | 5 spectral statistics × Q,K,V,O = a 20-number signature | **100% accuracy** on unseen adapters, Llama-3.2-3B / Qwen2.5-3B / Gemma-2-2B, no inference, trigger-agnostic |
| [Spectral Geometry of LoRA Adapters (arXiv 2604.08844)](https://arxiv.org/abs/2604.08844) | Apr 2026 | per-projection spectral drift | **AUC 1.00** within preference training; harm rank-correlation 0.72 |
| [Has This Checkpoint Been Abliterated? (arXiv 2607.01854)](https://arxiv.org/abs/2607.01854) | Jul 2026 | activation refusal-gap **plus** weight-recovery energy of `θ_candidate − θ_base` | **AUROC 0.95** over a 273-checkpoint registry (57 stripped-safety models vs 37 benign) across Qwen, DeepSeek-distilled Qwen, Llama, Gemma |
| [Detecting CSAM Text-to-Image LoRAs From Weights (arXiv 2607.25750)](https://arxiv.org/abs/2607.25750) | Jul 2026 | the single top singular vector of the LoRA update | inference-free fingerprint of training content, robust to noise, rescaling and precision reduction |
| [Evaluation without Generation (arXiv 2604.25119)](https://arxiv.org/abs/2604.25119) | Apr 2026 | internal activations under Gaussian probes | separates benign from harmful specialization without generating |
| **[PEFTGuard (arXiv 2411.17453)](https://arxiv.org/abs/2411.17453), IEEE S&P 2025 — added 2026-08-09, see §3.6** | Nov 2024 | raw query and value LoRA updates, stacked across layers, read by a convolutional meta-classifier | near-perfect in-domain; **releases PADBench, 13,300 LoRA adapters** over 5 base models, 5 datasets, 6 attacks, 5 PEFT methods, ranks 8–2048. **This is by far the largest registry in this table and it is public** |
| **[Z-PEFT (arXiv 2608.02271)](https://arxiv.org/abs/2608.02271) — see the failure list below and §3.6** | Aug 2026 | 16 spectral numbers per attention **head**, per projection, per layer | beats both above on every out-of-distribution axis it tests; **inverts to AUROC .2628 on held-out AdaLoRA and does not fix it** |

**And what is being found to fail — this is the load-bearing part:**

- [Z-PEFT (arXiv 2608.02271, 3 August 2026)](https://arxiv.org/abs/2608.02271)
  (Pitzalis, Shenaj, Cignoni, Cossu, Bacciu, Carta — Pisa). Verbatim from its
  abstract: *"strong performance in the closed-world setting does not
  necessarily translate to high accuracy in zero-shot backdoor detection."* It
  evaluates on **previously unseen attacks and datasets**. It then proposes a
  lightweight meta-classifier built only on layer-wise spectral measures, and
  that classifier degrades least. **Corrected 2026-08-09 after reading the full
  text (§3.6): the abstract undersells it. It also tests unseen adapter ranks
  and unseen PEFT methods, on the public 13,300-adapter PADBench. And its own
  detector inverts to AUROC .2628 on held-out AdaLoRA — a second published
  inversion, also unfixed.** One thing it does *not* vary is the training
  objective. Every PADBench adapter is supervised fine-tuning on poisoned data.
- [Spectral Geometry (arXiv 2604.08844)](https://arxiv.org/abs/2604.08844):
  a detector trained on preference-training adapters scores **every**
  activation-steering adapter as safer than **every** preference adapter —
  **AUC ≈ 0.00**, a clean inversion, not mere failure.
- [Has This Checkpoint Been Abliterated? (arXiv 2607.01854)](https://arxiv.org/abs/2607.01854)
  names two zero-cost ways to evade its own detector. The first is a **spoofed
  reference**: declare a different base model, so the weight difference is zero
  by construction. That defeats both of the detector's axes with no training at
  all. The second is a white-box owner who simply trains past the threshold.
  The author's own summary: *"effective triage, not tamper-proofing."*
- [On Trojan Signatures in Large Language Models of Code (arXiv 2402.16896)](https://arxiv.org/abs/2402.16896)
  tested the original weights-only trojan detector on code language models and
  concluded that *"detecting trojans only from the weights in such models is a
  hard problem."*
- [Reading the Finetuning Prior (arXiv 2605.25902)](https://arxiv.org/abs/2605.25902)
  is direct evidence against the premise: a grey-box method that sees only
  output probabilities recovers implanted facts across 1B–32B models,
  *outperforming* a white-box internal-access baseline while running about 170
  times faster.
- The behavioural baseline any weights method must beat is strong and comes
  from serious people:
  [Detecting Adversarial Fine-tuning with Auditing Agents (arXiv 2510.16255)](https://arxiv.org/abs/2510.16255)
  (Egler, **John Schulman**, **Nicholas Carlini**) reaches 56.2% detection at a
  1% false-positive rate over 1,400+ audits.
- The authoritative accounting of what weight analysis can and cannot do is
  [the TrojAI Final Report (arXiv 2602.07152)](https://arxiv.org/abs/2602.07152)
  — a multi-year IARPA program summarised by roughly sixty authors including
  Koushanfar, Mahoney, Xiangyu Zhang and XiaoFeng Wang. **Read this before
  proposing anything in this corner.**

**Two things to carry forward.**

*First, a warning.* Four independent groups converged on essentially the same
trick — a spectral summary of the adapter's update — between February and
August 2026. That is roughly one paper a month into a cell that was empty a
year ago, and our own standing lesson is that zero-citation papers under eight
weeks old are now the main source of scoops. **Any plan here has a short
clock and must be re-checked the week it is gated.**

*Second, an opening that is real but narrower than it looks.* Every headline
number above comes from a **small, self-manufactured registry**: 38 adapters,
273 checkpoints, 15 Hub models, adapter counts sometimes unstated. Several of
these papers are short single-author or few-author preprints with no stated
venue. Treat the *problem framings* as solid and the *effect sizes* as
unreplicated.

---

## 3. Candidate directions for us

Each candidate gives: the claim; why our assets help; the admission route under
the owner's method definition (new mechanism / improvement on a current method /
old method on a new problem — diagnosis, ablation, and statistics alone are
**not** methods); honest scoop risk with the closest papers linked; a cost
estimate; and the cheapest step that decides whether to continue.

**All stars are pre-gate scout stars and should be read as 1.5–2 stars too
high.**

**One framing note before the list.** D1, D2 and D3 are three arms of the same
underlying project: *the 2026 safety-from-weights wave published six headline
numbers on small self-made registries, and nobody has stress-tested them.* They
could be gated separately or run as one paper — audit plus the readout that
survives. I list them separately so each can be killed separately.

### D1. Make weight-only adapter screening survive a change of fine-tuning recipe ★★½ pre-gate (★★★½ → ★★★ → ★★½, all on 2026-08-09; the reading check in §3.6 re-scopes this candidate and everything below must be read against it)

**The claim.** People are starting to screen LoRA adapters for safety by reading
their weights, without running the model. Two 2026 papers do exactly this, and
they disagree in a very informative way.

- [Weight-space Detection of Backdoors in LoRA Adapters (arXiv 2602.15195)](https://arxiv.org/abs/2602.15195)
  (Puertolas Merenciano, Vasyagina, Zhu, Ferrando, Chaudhary; Feb 2026, v3 Apr
  2026) takes five spectral statistics from each attention projection — Q, K, V,
  and O. That gives a 20-number signature per adapter. It fits a logistic
  regression on those numbers. It reports **100% accuracy** separating poisoned
  adapters from clean ones on Llama-3.2-3B, Qwen2.5-3B and Gemma-2-2B. The
  method is trigger-agnostic and never runs the model. Its abstract never
  states how many adapters were used.
- [Spectral Geometry of LoRA Adapters Encodes Training Objective and Predicts
  Harmful Compliance (arXiv 2604.08844)](https://arxiv.org/abs/2604.08844) (Roi
  Paul, single author, **pre-registered**; Apr 2026) builds 38 adapters on
  Llama-3.2-3B-Instruct, in four families: healthy supervised fine-tuning;
  preference training on inverted harmlessness; preference training on inverted
  helpfulness; and adapters derived from activation steering. It then trains a
  detector on the preference-training families. That detector **assigns every
  steering adapter a lower drift score than every preference adapter, AUC ≈
  0.00**. That is not "it fails". It is *perfectly backwards*. The author's own
  conclusion is that *"cross-method monitoring requires per-method
  calibration."*

Put together: **the signal these detectors read is largely an artifact of *how*
the adapter was trained, not of *what* it does.** Change the recipe and the
detector inverts. A screening tool that inverts when the attacker switches from
preference training to activation steering is not a screening tool — an attacker
picks the recipe.

The method to build: a readout for adapter screening whose feature is invariant
to (or explicitly conditioned on) the fine-tuning recipe, so that it transfers
across training methods it has never seen. The position paper already tells us
the missing variable has a name — the `R` in `p(W | A, C, R)` — and says
metadata about it *"should be used for filtering, stratification, or
conditioning, rather than ignored"*. Nobody has done that on the discriminative
side — **but read the scoop-risk paragraph below before believing that sentence**,
because a paper posted on 3 August 2026 does something adjacent.

**A concrete mechanism hypothesis, worth stating up front so it can be
falsified.** Both detectors above read *spectral* features — statistics of the
singular values of the adapter's update. Two known facts make those features
suspect as a measure of harm.

1. **Shape leaks into the spectrum.** [FARMS (ICML 2025, arXiv 2506.06280)](https://arxiv.org/abs/2506.06280)
   shows that a weight matrix's aspect ratio biases heavy-tail spectral
   estimates, and fixes it by subsampling fixed-aspect-ratio submatrices. The
   backdoor detector takes its features from the Q, K, V and O projections —
   and in the exact base models it uses (Llama-3.2-3B, Qwen2.5-3B, Gemma-2-2B),
   grouped-query attention makes K and V far narrower than Q. So part of that
   20-number signature may be reading layer geometry, not poisoning.
2. **Magnitude leaks in from the recipe.** Preference training and activation
   steering push the weights different distances for reasons that have nothing
   to do with harm. A drift-magnitude feature will therefore rank one family
   above the other regardless of content — which is precisely the *sign
   inversion*, not mere failure, that the pre-registered study observed.

If that is right, the fix is a feature that is normalised for both shape and
recipe-induced scale — for example FARMS-corrected spectra plus a per-recipe
scale normaliser estimated from *benign* adapters only, so no labelled harmful
example of a new recipe is ever needed. That is a mechanism, not a
recalibration, and it is exactly the weakness in the existing answer
("per-method calibration", which needs labelled examples of every future
recipe).

**Why our assets help.**
- *Evaluator-audit credibility.* A 100%-accuracy claim with an unstated sample
  size and an AUC-0.00 claim at n=38 are exactly the two shapes our Fisher-z
  work is built for. Our house rules (paired bootstrap intervals,
  validation-locked selection) are the right instrument, and we have a track
  record of publishing the honest version.
- *Probe/readout expertise.* Designing a readout that separates "what the
  adapter can do" from "how it was trained" is a readout-design problem, which
  is the thing we do.
- *The free OrangeGrid fleet.* This study needs several hundred short,
  completely independent LoRA fine-tunes of a 3B model — the same job shape as
  the sparsity-premise sweep. OrangeGrid has 19 free L40S, 9 free A100-80GB, 15
  free A40, no wall-clock limit, and 20–60 second queue latency
  ([[GPU-Resources-Across-Clusters]]). This is the single best asset match in
  the whole report: the field's bottleneck is adapter zoos, and we have a free
  adapter factory.
- *Sparsity-premise design experience.* The 300-model factorial design
  (`code/sparsityprem/DESIGN.md`) is the template: widths × arms × seeds, six
  checkpoints each. Swap the axes to recipes × harm-condition × seeds.

**Route.** Route 2 (improvement on a current method) if we fix an existing
detector; Route 1 (new mechanism) if the recipe-invariant feature is genuinely
new. Note carefully: **the reproduction and the cross-recipe measurement alone
are diagnosis and would not qualify.** The paper must ship the readout that
transfers.

**Honest scoop risk — MEDIUM-HIGH, and it got worse while this page was being
written. The closest work, with links:**
- **[Z-PEFT (arXiv 2608.02271, 3 August 2026)](https://arxiv.org/abs/2608.02271)
  is a partial scoop and must be read in full before anything else happens.**
  It publishes the same *shape* of contribution as ours. Its message: these
  detectors look great in a closed world and do not transfer, so here is a
  lightweight layer-wise-spectral meta-classifier that degrades least. What its
  abstract does *not* claim is transfer across **fine-tuning methods**. Its axis
  is unseen *attacks and datasets*. Our axis is the recipe axis — supervised
  fine-tuning versus preference training versus activation steering versus
  merging. That is the axis where
  [arXiv 2604.08844](https://arxiv.org/abs/2604.08844) measured the inversion,
  and Z-PEFT does not appear to cover it. That distinction is our whole
  remaining claim. So verify it against Z-PEFT's full PDF, not its abstract. If
  Z-PEFT already covers recipes, **D1 is dead** and should be recorded as such.
  Note also that "degrades least" is a weaker deliverable than a real
  invariance. That leaves us room, but not much.
- [arXiv 2604.08844](https://arxiv.org/abs/2604.08844) names the problem but
  does not fix it; n=38, one base model, one author. Its recommendation
  ("per-method calibration") is the *weak* answer — it needs labelled examples
  of every future recipe.
- [Detecting Adversarial Fine-tuning with Auditing Agents (arXiv 2510.16255)](https://arxiv.org/abs/2510.16255)
  (Egler, Schulman, Carlini) is the behavioural baseline. Any weights-only
  method that cannot beat 56.2% detection at 1% false positives, at comparable
  cost, has no story.
- [The TrojAI Final Report (arXiv 2602.07152)](https://arxiv.org/abs/2602.07152)
  is the authority on what weight-analysis detectors can do, and
  [On Trojan Signatures in LLMs of Code (arXiv 2402.16896)](https://arxiv.org/abs/2402.16896)
  is the clearest published failure of the approach at language-model scale.
- ~~[arXiv 2602.15195](https://arxiv.org/abs/2602.15195) is the detector to
  beat.~~ **Struck 2026-08-09 (§3.6). It has been beaten, and its 100% was a
  closed-world number: out of distribution it scores .4792 and .4683, at or
  below guessing. The detector to beat is
  [Z-PEFT](https://arxiv.org/abs/2608.02271); the baseline of record is
  [PEFTGuard (IEEE S&P 2025)](https://arxiv.org/abs/2411.17453), whose
  **public 13,300-adapter PADBench** removes most of the cost of the first
  experiment.**
- [Learning on LoRAs (arXiv 2410.04207)](https://arxiv.org/abs/2410.04207) —
  Maron's group already does attribute detection and membership inference from
  LoRA weights with GL-invariant encoders. **They are the group most likely to
  do this next.** Their invariance is to the *factorization*, not to the recipe.
- [W2T (arXiv 2603.15990)](https://arxiv.org/abs/2603.15990) — canonicalizes
  LoRAs by QR-then-SVD and predicts attributes and performance at 10K–20K
  adapter scale. Its abstract does not address cross-recipe transfer or
  robustness. That absence is the opening, and also the risk: they could add it
  in a revision.
- [Evaluation without Generation (arXiv 2604.25119)](https://arxiv.org/abs/2604.25119)
  (Suriyakumar, Sekhari, Stempfle, Wang, Simpson, Portnoff, Ghassemi, Wilson;
  Apr 2026) audits harmful specialization of LoRA adapters non-generatively, but
  through **internal activations** rather than weights, and reports robustness to
  weight rescaling. A strong MIT/Ghassemi group is adjacent.
- [DSiRe (arXiv 2406.19395)](https://arxiv.org/abs/2406.19395) already reads a
  LoRA's norm and spectrum to recover a training fact (dataset size), with a
  25,000-snapshot benchmark. So "spectra of adapters encode training facts" is
  established prior art, not our discovery. Our claim has to be about *transfer*.
- [Spectral Signatures of LLMs (KDD 2026, arXiv 2607.03377)](https://arxiv.org/abs/2607.03377)
  is the landmine for any spectra-based proposal: as of July 2026 it does
  data-free lineage, clustering and quality prediction across 499 LLMs. It does
  **not** do safety screening or cross-recipe transfer, but Yaoqing Yang's group
  could add either.
- **Not yet checked and mandatory before any gate:** the harmful-fine-tuning
  defence literature (a different vocabulary: "safety alignment robustness",
  "tamper-resistant weights", "fine-tuning attack detection") — for example
  [AntiDote (arXiv 2509.08000)](https://arxiv.org/abs/2509.08000). House lesson
  9 says emptiness checks must key on mechanism shape, not vocabulary.

**Cost.** Low in credits, moderate in wall time, and **zero Anvil credits**. A
zoo of ~400 adapters on a 3B base at short training lengths is a few hundred
free GPU-hours spread across ~30 OrangeGrid cards running concurrently. Storage
is the real constraint: OrangeGrid disk is at 13.5 of 14.7 TB, but 400 LoRAs at
~50–100 MB each is under 40 GB, which is fine.

**Cheapest decisive step — two stages, and stage 0 costs nothing.**

*Stage 0 (0 GPU-hours, one afternoon).* Read
[Z-PEFT](https://arxiv.org/abs/2608.02271) in full, plus
[the TrojAI Final Report](https://arxiv.org/abs/2602.07152) sections on
weight-analysis detectors. **Kill rule: if Z-PEFT's experiments already vary the
fine-tuning method, or if TrojAI already settled cross-method transfer, D1 is
dead and gets a row in the kill ledger.**

*Stage 1 (about 6–10 free GPU-hours, no credits, only if stage 0 clears.)* Do
not build the full zoo. Train **two** small adapter sets on one 3B base — say 20
harmful and 20 benign under supervised fine-tuning, and 20 / 20 under preference
training — then fit the published 20-number spectral signature on one recipe and
test on the other. **Pre-stated kill rule, written before the run: if the
cross-recipe AUC stays above 0.75, the premise is wrong and the direction dies
that week. If it lands near 0.5, the signal is recipe-specific noise. Only a
result near 0.0 — a clean inversion, replicating the pre-registered finding at a
second harm condition — justifies going further.** Report a paired bootstrap
interval either way; the negative is worth a paragraph regardless.

### D2. Can a model owner disguise weights for free and walk past the auditors? ★★½ pre-gate (downgraded: a zero-cost evasion is already published)

**The claim.** Weight-space tools are now used to decide things a model's owner
may want hidden: what data it was trained on
([WARP](https://arxiv.org/abs/2607.01686)), which model it was copied from
([MoTHer](https://arxiv.org/abs/2405.18432),
[AWM](https://arxiv.org/abs/2510.06738)), what it was fine-tuned to do
([Learning on LoRAs](https://arxiv.org/abs/2410.04207),
[W2T](https://arxiv.org/abs/2603.15990)), whether it is backdoored
([arXiv 2602.15195](https://arxiv.org/abs/2602.15195)). These are auditing
tools, and auditing tools have adversaries.

Weight space has a property no other data type has: **there are transformations
that change every number in the file while changing nothing the model does.**
Shuffle a layer's hidden units and permute the matching rows and columns. Scale
one layer by `c` and the next by `1/c`. Rewrite a LoRA's `(A, B)` as
`(R⁻¹A, BR)`. Pad the rank with zeros. Merge the adapter into the base and
re-extract a different low-rank approximation of the same update. Each is free,
instant, needs no data, and leaves behaviour identical.

The claim to test: **the accurate readouts are not invariant to the full set of
these moves, so an uploader evades them at zero cost, and the invariant readouts
pay an accuracy price nobody has measured.** The deliverable is the readout that
keeps the accuracy and closes the gap.

**Why our assets help.** Same audit muscle as D1, and nearly free: the transforms
are a few lines of tensor manipulation, the detectors and zoos are public
(SANE, ProbeX, the Parameter-Space-Attack-Suite, weight_memorization), and the
models are small.

**Route.** Route 3 for the attack side (function-preserving symmetry transforms
are textbook; nobody has aimed them at weight-space *auditors* as an evasion
channel). Route 1 or 2 for the fix. **The audit alone is diagnosis and does not
qualify.**

**Honest scoop risk — MEDIUM-HIGH, and the field has moved since I first drafted
this.** Three separate defences already exist, which shrinks the opening a lot:
- [AWM (ICLR 2026)](https://arxiv.org/abs/2510.06738) explicitly undoes neuron
  permutation with a linear assignment problem before comparing, at up to 70B.
  So permutation alone is *already* handled by the best fingerprinting method.
- [W2T](https://arxiv.org/abs/2603.15990) canonicalizes the LoRA GL ambiguity by
  QR-then-SVD.
- [Learning on LoRAs](https://arxiv.org/abs/2410.04207) builds GL-invariant
  encoders directly.
- [Adversarial Attacks in Weight-Space Classifiers (arXiv 2502.20314)](https://arxiv.org/abs/2502.20314)
  (Shor, Fetaya, Baskin, Bronstein; v1 Feb 2025, v3 Mar 2026;
  [code](https://github.com/tamirshor7/Parameter-Space-Attack-Suite)) is the
  closest paper, and I read its PDF. It studies **implicit-representation**
  weight classifiers. It attacks them by *perturbing* weights with white-box
  gradient methods. It finds surprising robustness, and it attributes that
  robustness to **gradient obfuscation**. In the adversarial-images literature,
  gradient obfuscation is the classic sign that a robustness claim will not
  survive an adaptive attack. Its §5 says the robustness *"weakens and
  potentially fails under the presence of gradient-free attacks"*, and it
  **defers black-box attacks to future work**. It never tests
  function-preserving symmetry transforms — which are exactly a gradient-free,
  zero-distortion attack.
- The spoofing community is awake but is doing something different:
  [GhostPrint (arXiv 2606.16100)](https://arxiv.org/abs/2606.16100) evades
  fingerprinting by *fine-tuning a weak model to imitate a strong one*, not by
  symmetry.
- **A published zero-cost evasion already exists**, and it is not a symmetry
  transform: [Has This Checkpoint Been Abliterated? (arXiv 2607.01854)](https://arxiv.org/abs/2607.01854)
  shows that simply **declaring the wrong base model** makes the weight
  difference zero by construction and defeats both of its detector's axes with
  no training at all. That paper has therefore already planted the flag on
  "these audits can be evaded for free". Our angle would have to be a
  *different* evasion channel (the symmetry transforms) and a fix.

So the residual opening is narrow but real: **the deployed, best-accuracy
readouts (the spectral backdoor detector, MVProbe, SANE-style encoders) are not
the canonicalized ones, and nobody has measured the accuracy-versus-evadability
trade-off across them.** Whether that is a paper or a section of D1 is a
judgement call — my honest read is that it is **a strong second arm of D1, not a
standalone paper.**

**Cost.** Very low: 5–20 free GPU-hours for the decisive experiment.

**Cheapest decisive step (0 GPU-hours, then ~4).** (1) One afternoon in the
model-security vocabulary — "model scanning", "model file provenance", "neural
network watermark removal by permutation". If symmetry-based evasion of model
provenance is already published there, stop. (2) If clear: take one public
weight-space property predictor with released weights, feed it a zoo, apply
plain neuron permutation, and measure the drop. **Pre-stated kill rule: if the
drop under plain permutation is under 10 points, the premise is wrong and the
direction dies that afternoon.**

### D3. Do Hub-scale screening claims survive real, wild adapters? ★★★ pre-gate — **survived its reading check unchanged; now the top-ranked of the three. See §3.6.**

**The claim.** Almost every "we can search or screen the model hub" result is
validated on **self-trained** zoos. ProbeX's Model-J is 14,000 models the
authors trained themselves. MVProbe's Model Jungle is the same construction.
W2T's 10K–20K adapters are self-trained. ProbeLog's only real-Hub evaluation is
**71 models**. Meanwhile the Hub has over 1.5 million models, more than 60% of
them undocumented, and its adapters were trained by strangers with unknown
recipes, unknown data, and unknown ranks.

The claim: **the reported accuracies are an artifact of homogeneous,
self-trained populations, and they fall apart on wild adapters.** There is
already independent support for this shape:
[The Appeal and Reality of Recycling LoRAs with Adaptive Merging
(arXiv 2602.12323)](https://arxiv.org/abs/2602.12323) (Liu, Je, Ciccone, Xu,
Raffel) assembled a pool of nearly **1,000 user-contributed LoRAs pulled from
the Hub, all from Llama-3.1-8B**. They found that *which* LoRAs you merge barely
matters. Randomly-initialized LoRAs perform about as well. So the apparent
benefit looks like regularisation, not transfer. That is a wild-population
result, and it contradicts what self-trained populations suggest.

The deliverable is the readout that works on wild adapters. Most likely it has
two properties. It is lineage-blind: we never test it on a descendant of a
model it trained on. And it encodes the *task vector* rather than the raw
weights, so the encoder sees what the fine-tuning did rather than which family
the model came from.

**Why our assets help.** Contamination-controlled splits are house discipline
(the `run-provenance` rules), and we have caught leakage before. Downloading and
featurising a thousand adapters is mostly CPU work. Labelling them means running
them once, which is exactly a judge-style evaluation campaign — our RoboJudge
infrastructure shape, on free L40S cards that fit a 7–8B model comfortably.

**Route.** Route 3 (old method on a new problem) if the contribution is the
readout that survives wild data plus the lineage-blind protocol. **The benchmark
alone is not a method** — the owner's definition excludes it.

The same objection applies with more force to the safety detectors in §2.6.
Their registries are **38 adapters** (spectral geometry), **273 checkpoints**
(abliteration audit), **15 Hub models** (modelDNA), and an unstated count (the
backdoor detector). Every harmful example in them was manufactured by the
authors. Whether a screening tool trained on adapters somebody made *on purpose
to be caught* works on adapters strangers uploaded is simply unknown.

One encouraging fact: the sweep found **no paper that characterises the
geometry or statistics of a large scraped population of published LoRAs.** The
largest real scraped pool in the literature is Raffel's ~1,000. Everyone else
self-trains. That is unusual for a field this visible.

The hard part, stated honestly: **wild adapters have no safety labels.** You get
them by running the adapters once. That is fine — the claim is still "weights
predict it without running" — but it makes the labelling pass the real cost, and
it makes label quality the thing a reviewer will attack. This is the same
problem the RoboJudge audit already solved once, which is the reason to think we
could solve it again.

**Honest scoop risk — MEDIUM-HIGH.**
[The Impact of Model Zoo Size and Composition (arXiv 2504.10141)](https://arxiv.org/abs/2504.10141)
already studies in- and out-of-distribution generalisation of zoo-trained
encoders. [Learning Model Representations Using Publicly Available Model Hubs
(arXiv 2510.02096)](https://arxiv.org/abs/2510.02096) already trains on 2,000
wild Hugging Face models and reports it beats lab zoos — which is the *opposite*
sign to our premise and must be read in full before this is gated.
[Hidden Gems (arXiv 2601.22157)](https://arxiv.org/abs/2601.22157) works over
2,000+ wild Hub models behaviourally. Hoshen's and Borth's groups are both
better placed than us to notice and fix this.

**Cost.** Low-to-medium: mostly download bandwidth and CPU, plus one labelling
pass. 50–150 free GPU-hours. Watch the OrangeGrid disk ceiling.

**Cheapest decisive step (0 GPU-hours).** Read
[arXiv 2510.02096](https://arxiv.org/abs/2510.02096) in full and extract its
in-distribution versus out-of-distribution gap. **If Borth's group already shows
wild-model training closes the gap, this candidate is dead on arrival.** Only if
their evaluation is itself in-distribution does the question survive.

### D4. A model zoo whose safety label is exact, riding almost free on the sparsity study ★★★ pre-gate — **confirmed, but it is NOT free: the design saves no weights at all. See §3.6 and the drafted addendum.**

**The claim.** Every existing model zoo labels its models with things that are
easy to measure: accuracy, learning rate, seed, task. No zoo carries a label for
a *formally defined safety property*, because in real settings nobody can
compute one. Our designed-but-unauthorised sparsity-premise study
(`code/sparsityprem/DESIGN.md`) works in a small synthetic world where the
property **can be computed exactly**. The study trains 300 models: 4 widths ×
3 training arms × 25 seeds. It measures each model at six checkpoints, so about
1,800 checkpoints in total. Every one of them carries an exact "dangerous / not
dangerous" label, taken from
[Definition 5.22 of Bengio's LawZero paper (arXiv 2606.29657)](https://arxiv.org/abs/2606.29657).

The WSL question riding on top: **can a weight-space readout detect the
dangerous models?** This is exactly the survey's §7.3 request for *"anomaly
metrics and embedding-based probes to identify suspicious weight patterns"*, and
it is the only setting I found where the answer is checkable against ground
truth rather than against a proxy.

**Why our assets help.** The zoo is already designed, costed (~9.7 GPU-hours for
the training block, ~70 GPU-hours all-in), and pointed at OrangeGrid. The
marginal cost of the weight-space arm is a few GPU-hours. Our probe expertise is
the exact skill needed.

**Route.** Route 3 — an old method (weight-space property prediction) on a new
problem (a formally defined, exactly computable safety property). Honestly weak
as a standalone paper; strongest as a second arm of the sparsity paper, or as a
workshop paper at the ICML symmetries workshop.

**Honest scoop risk — LOW, because the label exists nowhere else.** The nearest
neighbours are [WARP](https://arxiv.org/abs/2607.01686) (recovers a training
data mixture, not a safety property) and Shor et al. The risk is *value*, not
novelty: a synthetic world is a synthetic world, and reviewers will ask what it
predicts about real models. The design document already flags this.

**Cost.** About 2–5 GPU-hours on top of a study that must run anyway.

**Cheapest decisive step (0 GPU-hours, do this at the sparsity design review).**
Check that the design keeps **full weights** for all 1,800 checkpoints, not just
summary statistics, and add "release the zoo" to its artifact list. If it only
keeps summaries, this candidate costs a design change rather than nothing.

### D5. Frozen-tower readout zoo — the cheap zoo only we can make ★★ pre-gate, listed for completeness

**The claim.** The field's bottleneck is labelled zoos. We can make an unusually
large one almost free: a frozen SigLIP2 tower (our backbone of record,
`ViT-B-16-SigLIP2-256/webli` via open_clip) plus thousands of small readout
heads, each labelled with an exactly measured score on our own binding and
retrieval batteries. The features are already cached — 193 GB on OrangeGrid — so
each head costs seconds.

**Why our assets help.** The cache, the backbone, the powered role-decodability
corpora, and 84 idle RTX 6000 cards that are useless for bf16 judges but fine
for training linear heads.

**Route and honest assessment.** Weak. A zoo of linear heads is one layer with
no depth and almost none of the symmetry structure that makes weight space
interesting, and building a zoo is a service contribution rather than a method.
**I include it because the cost is near zero, not because I think it is a
paper.**

**Cheapest decisive step.** Answer one reviewer-shaped question in a paragraph:
what claim would this zoo support that a table of scores would not? If there is
no answer, drop it.

### Ranking of the five, pre-gate

**D1 ≈ D4 > D3 > D2 > D5.** ← **superseded on 2026-08-09. The reading checks in
§3.6 revise this to D3 ≈ D1 > D4 > D2 > D5.** The three caveats below still
stand and caveat 1 turned out to understate the problem.

Three honest caveats on this ranking.

1. **D1 lost half a star while this page was being written**, to a paper posted
   six days earlier. Its remaining claim is one axis narrower than I first
   wrote, and that axis has to be confirmed from Z-PEFT's full text before
   anything else. This is the third time this month a direction has been
   narrowed by a paper under eight weeks old.
2. **D4 is the only candidate whose novelty does not depend on a race**, because
   its label — a formally defined, exactly computed safety property — exists
   nowhere else and cannot be scraped. It is also nearly free. It should be
   folded into the sparsity design review whatever else happens.
3. **D2 is best read as a second arm of D1**, not a paper. The abliteration
   audit already published a zero-cost evasion of its own detector.

Do the three zero-GPU-hour reading checks first — Z-PEFT's full text (D1),
[arXiv 2510.02096](https://arxiv.org/abs/2510.02096)'s in- versus
out-of-distribution gap (D3), and the sparsity design's checkpoint format (D4).
Between them they can kill two candidates and confirm one, in one day, for
nothing. D5 should not be gated at all.

**The three checks were run on 2026-08-09. Their verdicts are §3.6, and they
change the ranking above. Read §3.6 before acting on anything in §3.**

### 3.6 Reading-check verdicts — 2026-08-09

All three checks are done. Zero GPU-hours, zero credits, one day, as planned.
Working notes, the cached PDFs, and the drafted design addendum are in
`code/tier2gates/wsl-checks-20260809/`. **Nothing here is a decision to start
work.** Every candidate below still needs the full two-pass gate.

**The one-line summary. D1 survives, but only after being re-scoped: the axis it
was built on turned out to be four-fifths covered, and what is left is a
different, cheaper, sharper claim. D3 survives untouched and is now the
strongest of the three. D4 is confirmed, but it is not free — the sparsity
design saves no weights at all, so it costs a design change.**

**Revised ranking: D3 ≈ D1 > D4 > D2 > D5.** D1 and D3 are close for different
reasons — D1 is now almost free to test and has a sharp falsifiable mechanism,
D3 is the least raced. Run D1's new free stage first, because it decides fastest.

---

#### Verdict 1 — D1: **ALIVE but re-scoped, ★★★ → ★★½, on a short clock**

I read [Z-PEFT (arXiv 2608.02271)](https://arxiv.org/abs/2608.02271) in full.
It is much wider than its abstract admits, and it changes three things.

**First: the axis list. Z-PEFT tests five kinds of shift, not two.** The
abstract only advertises "unseen attacks and datasets". The full text also
tests unseen adapter ranks and unseen PEFT methods. All numbers are AUROC —
area under the ROC curve, where 1.0 is perfect, 0.5 is guessing, and below 0.5
means the detector is *backwards*.

| Axis | How it is tested | Z-PEFT | PEFTGuard | WSD |
|---|---|---|---|---|
| Unseen **attack**, same dataset | train on 1 of 4 attacks, test on the other 3 (12 pairs) | **.8718** | .8153 | **.4792** |
| Unseen **attack**, multi-attack training | leave-one-attack-out on AG News | **.9433** | .8395 | **.4683** |
| Unseen **dataset** | pool of 4 datasets, 6 attacks, 5 PEFT methods, ranks 8–2048 | .878–.998, but **Alpaca .635** | — | — |
| Unseen **rank** | leave-one-rank-out, 8 to 2048 | ≥ .9808 up to rank 1024; **.9600** at 2048 | — | — |
| Unseen **PEFT method** | leave-one-adapter-out | DoRA 1.000, LoRA+ 1.000, LoRA .9956, QLoRA .9924, **AdaLoRA .2628** | — | — |
| Architecture | Flan-T5-XL, Qwen1.5-7B, Llama-2-7B/13B, RoBERTa | .9908–1.0000 — but the paper says these are **in-domain**, not transfer | — | — |
| In-domain | held-out samples from seen configurations | **.9986** pooled | — | — |

**Second: it kills a claim we were relying on.** Our page called
[arXiv 2602.15195](https://arxiv.org/abs/2602.15195) ("WSD" here) **"the
detector to beat"**, on the strength of its 100%-accuracy headline. Z-PEFT
beat it, and worse: WSD's off-diagonal AUROC is **.4792 and .4683 — at or
below guessing**. WSD's 100% was a closed-world number. **Correct that line
wherever it appears.**

**Third: our page is missing the paper that actually owns this cell.** Z-PEFT
does not build a zoo. It uses **PADBench**, released with
[PEFTGuard (arXiv 2411.17453)](https://arxiv.org/abs/2411.17453) (Sun, Cong,
Liu, Lin, He, Chen, Han, Huang), published at **IEEE Symposium on Security and
Privacy 2025**. PADBench is **13,300 LoRA adapters**. It covers five base
models — Llama-2-7B, Llama-2-13B, Qwen1.5-7B, Flan-T5-XL, RoBERTa-base — plus
five datasets, six attacks, five PEFT methods (LoRA, AdaLoRA, DoRA, LoRA+,
QLoRA), and ranks from 8 to 2048. It ships only the query and value projection
weights. **Neither
PEFTGuard nor PADBench appears anywhere on this page. That was a real gap:
PEFTGuard is the security-venue baseline of record, and PADBench is far larger
than every registry in §2.6 put together.**

**So does D1's axis survive?** This needs care, because the kill rule as
written and the candidate as written do not use the word "recipe" the same way.

- The **kill rule** said: *"if Z-PEFT's experiments already vary the
  fine-tuning method, D1 is dead."* Z-PEFT does vary the fine-tuning method, in
  the sense of the **adapter parameterization** — LoRA versus AdaLoRA versus
  DoRA versus LoRA+ versus QLoRA. On a literal reading, D1 dies.
- The **candidate** spelled out its own axis as *"supervised fine-tuning versus
  preference training versus activation steering versus merging"* — the
  training **objective**, not the parameterization. Every one of PADBench's
  13,300 adapters is supervised fine-tuning on poisoned data. There is no
  preference training, no activation steering, no merging, no continued
  pretraining anywhere in it. On that reading the axis is untouched.

**Both readings are honest, and the honest conclusion is that the kill rule was
written against the wrong word.** Four of the five shifts D1 imagined are now
done by somebody else, on a bigger and public benchmark. The training-objective
shift is genuinely untouched, but it is one slice, not the whole idea.

**What makes D1 worth keeping is not the leftover axis. It is what Z-PEFT
found and did not fix.**

Held-out AdaLoRA scores **AUROC .2628, accuracy .2880**. Their own words: *"The
below-chance AUROC indicates that the learned scores largely reverse the
ordering of clean and backdoored AdaLoRA adapters."* That is a **second
independent inversion** of a spectral adapter detector, on a public
13,300-adapter benchmark, four months after
[arXiv 2604.08844](https://arxiv.org/abs/2604.08844) reported AUC ≈ 0.00 on the
objective axis with 38 adapters. Two different shifts, two inversions, two
groups, **and nobody has repaired either one.**

And the inversion has a named cause that matches our own mechanism hypothesis
almost exactly. Z-PEFT's SHAP analysis reports that **stable rank dominates the
AdaLoRA cohort** — and AdaLoRA is precisely the one method in the list that
*reallocates rank by construction*. So the feature that inverts is a
rank-family feature, and it is reading the parameterization rather than the
backdoor. That is the same shape as the "magnitude leaks in from the recipe"
argument in the D1 section above, one level down.

**The re-scoped D1, in one sentence: a spectral adapter-screening readout whose
rank-family and scale features are normalised against a benign reference bank
*of the same parameterization and the same objective*, so that it does not
invert — with the pre-stated target of lifting held-out AdaLoRA from .2628 to
above .5 without losing the other four methods.** That is Route 2, an
improvement on a current method, and it is a fix rather than a diagnosis.

**Three things got much better.**

1. **The cost collapsed.** PADBench is public, and Z-PEFT's whole feature
   extraction runs on **CPU** — 26.8 minutes for 850 adapters on 56 cores. The
   first decisive experiment no longer needs us to build 400 adapters. It needs
   a download and a laptop. Building adapters is only required later, for the
   objective axis that PADBench lacks.
2. **The premise is now confirmed for free.** D1's old stage 1 was "fit the
   published 20-number signature on one recipe, test on another, kill if AUC
   stays above 0.75". Z-PEFT already ran that on the attack axis and got
   .4792. We do not have to spend 6–10 GPU-hours to learn it.
3. **The mechanism claim is falsifiable without training anything.** If the
   inversion is a rank-family artifact, then removing or renormalising stable
   rank, effective rank, spectral entropy and concentration should move
   held-out AdaLoRA above 0.5. If it does not, the mechanism hypothesis is
   wrong and D1 dies that afternoon.

**Three things got worse.**

1. **The competitor is strong and close.** Pisa published this six days ago and
   says code comes "upon acceptance". They can see their own AdaLoRA hole. The
   obvious camera-ready revision is exactly our re-scoped claim.
   **This is a weeks-long clock, not a months-long one.**
2. **A security venue owns the cell.** PEFTGuard is IEEE S&P. That is a
   different reviewer pool with different standards from the weight-space
   crowd in §1.
3. **What is left is narrower than the section above describes.** Everything in
   the D1 text before this verdict was written against a five-axis opening.
   Four of those axes are gone.

**What Z-PEFT still does not do, verified from the full text**, and any of these
could carry a contribution: the training-objective axis (no DPO, no steering,
no merging); any harm label other than triggered backdoor versus clean;
projections beyond query and value, because PADBench ships only those; an
adaptive adversary that optimises an adapter to look benign, which its
limitations name as future work and which is **D2's premise stated by the
competitor**; naturally occurring backdoors, which its limitations also name
and which is **D3's premise stated by the competitor**; deployment-time
threshold calibration, since its accuracy numbers use the known number of
positives; and **any statistics at all — there is not one confidence interval,
seed, or bootstrap in the paper, and thirteen of its twenty-three in-domain
cells are exactly 1.000000.** That last one is our instrument.

**Citation pull, both ways.** Z-PEFT has **0 forward citations** (six days
old); so do
[2604.08844](https://arxiv.org/abs/2604.08844),
[2602.15195](https://arxiv.org/abs/2602.15195) and
[2607.01854](https://arxiv.org/abs/2607.01854). The productive pull was
backwards: PEFTGuard has **36 forward citations**, all screened. Four are new
to this page, and none of them is a scoop.

- [DFBScanner (arXiv 2605.18907)](https://arxiv.org/abs/2605.18907) — static
  inspection of the **final classification layer** of vision networks. It is
  attack-agnostic, and it covers over 5,000 backdoored models, 12
  architectures, and 20 attack types. It is the classic-vision analogue of this
  cell, and a baseline shape we should know about.
- [Fine-Tuning Integrity (arXiv 2604.04738)](https://arxiv.org/abs/2604.04738) —
  zero-knowledge cryptographic proofs that a released model's drift lies in a
  declared class of norm-bounded, low-rank, or sparse updates. It is a
  completely different answer to the same supply-chain worry. Cite it rather
  than compete with it.
- [LoRA as Oracle (arXiv 2601.11207)](https://arxiv.org/abs/2601.11207) —
  attaches a probe adapter and reads its optimisation dynamics. So it needs
  data and training, and it is not weights-only.
- [A fine-tuning-security survey (arXiv 2605.25073)](https://arxiv.org/abs/2605.25073)
  — its own re-evaluation finds that cross-lingual backdoor transfer *"reported
  as near-perfect at larger scales, fails entirely on tested 1B–4B models"*.
  That is a second, independent case of this literature's headline numbers not
  surviving a change of setting.

Ten mechanism-shape searches in adjacent vocabulary returned **nothing new**.
The ten: fine-tuning recipe, preference optimization, training objective,
harmful + spectral, AdaLoRA + detection, aspect ratio + singular value,
third-party adapters + supply chain, community-contributed adapters, model hub
+ safety + weights, and recipe + invariant.

**New cheapest decisive step, replacing the old two-stage plan. Stage 0.5,
about zero GPU-hours and a few CPU-hours.** Download PADBench. Reproduce
Z-PEFT's leave-one-adapter-out AdaLoRA cell and confirm the .2628. Then drop or
renormalise the rank-family features and re-fit. **Pre-stated kill rule, to be
written down before the run: if held-out AdaLoRA does not clear 0.55 under any
of the three normalisations we name in advance, the mechanism hypothesis is
wrong and D1 dies that week.** Report a paired bootstrap interval on the
difference — nobody in this literature reports one, so that alone is a
contribution to the write-up. Re-check for a Z-PEFT camera-ready or code
release the same week, and again the week of any gate.

---

#### Verdict 2 — D3: **SURVIVES CLEANLY, ★★★ held, and now the strongest of the three**

I read
[Learning Model Representations Using Publicly Available Model Hubs
(arXiv 2510.02096)](https://arxiv.org/abs/2510.02096) in full. The kill rule
was: *"if Borth's group already shows wild-model training closes the gap, this
candidate is dead on arrival. Only if their evaluation is itself
in-distribution does the question survive."* **Their evaluation is entirely on
curated targets. The question survives.**

**What the paper actually is.** They collect Hugging Face models under four
vision tags, which gives 22,055 candidates. They keep only the models that load
through the auto-classes without remote code and that tokenize successfully.
They stop at **2,000 training plus 200 validation** models, **171 billion
parameters** in total, with individual models up to 1.3 B. The mix is 42.0%
transformer, 21.8% convnet, 5% hybrid, and **31.4% whose architecture cannot
even be told from the name**. They then adapt the SANE encoder-decoder with
three changes: per-token loss normalisation at runtime, dense tokenization, and
sinusoidal position encodings. They train one backbone for everything, at 450 M
or 900 M parameters.

**What tasks it demonstrates.** The main body is **entirely weight
generation**: produce ResNet-18 weights for five image datasets; produce 25
different timm architectures for ImageNet-1K, scored after one to five epochs
of fine-tuning; produce a GPT-2 initialisation for OpenWebText. The
discriminative side is **one appendix table** predicting three ordinary
properties: **test accuracy, generalization gap, and training epoch**.

**Safety or property screening: none.** The words safety, backdoor, harm,
poison and malicious do not occur in the paper.

**Does it report an in-distribution versus out-of-distribution gap? Not the one
D3 asks about — and the one it does report points against its own headline.**

The wild-versus-lab contrast is about the **training** set. Every **evaluation**
target is curated: the Schürholt ResNet-18 zoo, five fixed image datasets, 25
timm architectures, GPT-2. **They never evaluate any downstream task on wild
Hugging Face models**, for the obvious reason that wild models carry no labels.
That is exactly D3's gap, and the paper walks straight past it.

Their Table 9 is the closest thing to an OOD number. It is the explained
variance (R², where 1.0 is perfect) of a linear probe on the *lab* ResNet-18
zoo:

| Backbone trained on | Test accuracy (C10 / C100 / TIN) | Generalization gap | Epoch |
|---|---|---|---|
| CIFAR10 zoo | 91.69 / 95.60 / 94.99 | 75.93 / 91.94 / 88.81 | 99.67 / 99.34 / 99.11 |
| CIFAR100 zoo | 92.70 / 96.22 / 95.73 | 77.56 / 92.21 / 88.90 | 99.65 / 99.54 / 99.30 |
| **Hugging Face (wild)** | **69.44** / 91.58 / 90.29 | **53.75** / 87.39 / 85.05 | 94.78 / 96.86 / 90.39 |

The wild-trained backbone is **worse in all nine cells** — about 4–5 points on
CIFAR100 and TinyImageNet, but **about 23 points on CIFAR10 accuracy and 24 on
CIFAR10 generalization gap**. Their own sentence: *"We observe a performance
drop compared to previous work in all cases."* So **"beats zoo-trained
backbones" is a claim about weight generation, not about reading properties out
of weights** — and even on generation they lose on CIFAR10 in every comparison,
and generated weights are *"slightly above or at random guessing"* until
fine-tuned.

**Our exact delta, in one sentence.** They vary the **training** population and
evaluate on **curated** targets, on **whole vision models**, for **generation**
plus three benign properties. D3 holds the readout fixed and varies the
**evaluation** population, on **adapters**, for **screening** claims. Those are
orthogonal questions and this paper answers only theirs.

**Three things that strengthen D3.**

1. **The competitor states our premise as its own open problem.** Z-PEFT's
   limitations say PADBench *"cannot capture the full variability of adapters
   encountered in practice… performance may differ for **naturally occurring
   backdoors**."* Six days old. That is the best motivation citation we could
   ask for — and simultaneously a warning that Pisa may do it next.
2. **Their download protocol is a reusable template**, and their
   selection-bias limitation is a caveat we can quote rather than invent:
   *"we have not applied manual curation beyond basic feasibility checks…
   This raises the possibility of selection bias."* Note how severe the filter
   is: 22,055 candidates down to 2,200 kept. Even the "wild" collection in the
   literature is heavily screened, which cuts both ways.
3. **A leakage question worth one sentence in our write-up.** Table 9 says
   *"100 models are split into train/test/validation with proportions 70/15/15
   using checkpoints from epochs 1, 3, 5, 10, 15, 20, and 25"* and never says
   whether the split is over models or over checkpoints. If over checkpoints,
   the same model sits in train and test — the pattern our run-provenance rules
   exist to catch. There are also no intervals on that table.

**Citation pull.** 2510.02096 has **4 forward citations**: WeightCLIP, GeoSANE
and Hidden Gems, all already on this page, plus one new one —
[ModelLens (arXiv 2605.07075)](https://arxiv.org/abs/2605.07075), which learns
to rank unseen models on unseen datasets from **1.62 million public leaderboard
records over 47,000 models and 9,600 datasets**. It never reads a weight. It is
not a scoop of D3, but it is the largest in-the-wild model-selection result we
have found and it is a competing answer to "how do you know what a stranger's
model can do". **Any D3 write-up must say why reading weights beats reading
leaderboard records** — which for safety screening has a good answer, since a
malicious uploader will not publish an honest evaluation, but the answer has to
be written down.

**Unchanged from the section above:** the deliverable must be the readout, not
the benchmark; the labelling pass is the real cost; and Hoshen's and Borth's
groups are both better placed than we are. The decisive step is unchanged too,
except that it should now use adapters rather than whole models, since that is
where the screening claims live.

---

#### Verdict 3 — D4: **CONFIRMED, but it costs a design change, not nothing. ★★★ held.**

The check was: *"check that the design keeps full weights for all 1,800
checkpoints, not just summary statistics."*

**It does not. The sparsity sweep currently saves no weights at all.**

Searching the whole `sparsityprem` repository for `torch.save`, `safetensors`
and `state_dict` returns **zero hits**. The word "checkpoint" in the design
document means "a step at which we measure something", not "a saved file".

Here is what the code does. In `src/sparsityprem/harness.py`, `train_one`
probes at six steps. At each step it appends a **dictionary of scalars** to a
trace: loss, marginal miss rate, the harm-coordination scores, and the
dangerous flag. It then hands back the live model object. The driver keeps that
object in a Python list, and writes only a JSON file. Every model is destroyed
when the process exits. The band-sampling routine does the same. All nineteen
recorded run directories together total **14 MB** of JSON, with no binary
artifacts.

So D4 is real, its label is real, and it is still the one candidate whose
novelty does not depend on winning a race — **but the scout page's claim that
it is nearly free was wrong by one code path.** It is free in GPU-hours and
cheap in disk; it is not free in design approval.

**The addendum is written and ready for the owner.** Full text:
`code/tier2gates/wsl-checks-20260809/sparsity-design-addendum.md`, drafted to
drop into `sparsityprem/DESIGN.md` between §7 and §8. Its substance:

- **Save `.safetensors` in float32**, one per model per probe step, plus a JSON
  sidecar carrying the full architecture config, the arm, the seed, the step,
  the loss, the whole hazard-world definition, and the git commit. Float32
  rather than bfloat16 because weight-space methods read singular values, and
  halving the precision changes them.
- **Save the raw per-query scores, not the verdict.** The dangerous/not-dangerous
  flag depends on four settings the design deliberately leaves free. Saving the
  model's log P(HAZARD) on each of the 256 critical queries — **1 KB** — makes
  every label at every setting recomputable offline, forever, on a laptop. Save
  the query pool and the pattern index sets once, at the top level.
- **Do not save optimizer state.** AdamW's moments would triple the disk cost
  and nothing needs them.
- **Save the SGLD band samples at the two smallest widths only.** This is the
  genuinely new artifact. Every published zoo is a collection of *converged*
  models; a population drawn from *inside one loss band*, each with an exactly
  computed harm score, does not exist anywhere. 20 chains × 100 samples at
  widths 64 and 128 is 4,000 weight vectors for 3.8 GB. At widths 256 and 512
  the same rate would cost about 250 GB, so do not.

**Disk cost, computed from the parameter counts already recorded in
`runs/20260806-scale-*`, at 4 bytes per parameter:**

| width / layers | parameters | MB each | ×75 models, finals | ×75, all six steps |
|---|---|---|---|---|
| 64 / 2 | 104,867 | 0.40 | 0.03 GB | 0.18 GB |
| 128 / 2 | 406,307 | 1.55 | 0.11 GB | 0.68 GB |
| 256 / 4 | 3,178,531 | 12.13 | 0.89 GB | 5.33 GB |
| 512 / 8 | 25,258,019 | 96.35 | 7.06 GB | 42.34 GB |
| **total** | 28,947,724 | | **8.09 GB** | **48.53 GB** |

Sizes here are binary — 1 MB is 2²⁰ bytes. In the decimal units disk quotas are
usually quoted in, the same two totals are 8.68 GB and 52.11 GB.

Plus 3.81 GB of band samples. **About 52 GB for everything; about 8 GB for
final weights only.** OrangeGrid is at 13.5 of a 14.7 TB soft quota, so roughly
1.2 TB is free and 52 GB is about 4% of the headroom. If it gets tight, drop the
four intermediate steps at width 512 first — that alone saves 28 GB and leaves
full trajectories at the three smaller widths.

**Effect on the ~70 GPU-hour budget: none.** Writing weights is disk work off
the GPU critical path; 48.5 GB spread over the 9.7 GPU-hour training block
averages about 1.4 MB per second. The only thing to get right is HTCondor
output staging — write to node-local scratch and stage once at job end, or a
width-512 job ships up to 578 MB back per model. Any residual sits inside the
35 GPU-hour contingency already in §7.

**One risk the addendum raises that the scout page did not.** The zoo may be
**label-degenerate**: in the smoke run, 0 of 8 models were dangerous at
`m_H = 8`. If almost every model lands on one side of the line there is nothing
to classify. The fix is already implied by the design — make the **continuous**
harm-coordination score the primary weight-space target, not the binary flag,
so it is a regression problem that needs no rare positives; and rely on Arms B
and C, which manufacture danger on purpose, as the positive controls. The
addendum proposes this as a new sanity invariant.

**The argument for approving it, in one sentence:** keeping the weights costs
about 52 GB and no GPU time, and recovering them later would cost the whole
9.7 GPU-hour training block plus the SGLD block over again.

**Unchanged:** the claim would be the exactly computed safety label, never the
collection. Our own §5 item 4 says building a zoo is a service, not a claim,
and 75 models per architecture is small by this field's standards — Borth's
vision-transformer zoo is 250, SANE trains on 140 per zoo, WeightCLIP's are
1,000 to 10,000. The band samples are what make it a population worth reading.

---

#### What these three checks changed on this page

1. **§2.6 and D1: [arXiv 2602.15195](https://arxiv.org/abs/2602.15195) is no
   longer "the detector to beat."** Its 100% accuracy was closed-world; out of
   distribution it scores .4792 and .4683, at or below guessing. The detector to
   beat is Z-PEFT, and the baseline of record is PEFTGuard.
2. **§2.6 was missing its biggest entry.**
   [PEFTGuard (arXiv 2411.17453, IEEE S&P 2025)](https://arxiv.org/abs/2411.17453)
   and its **13,300-adapter public PADBench** belong in that table. Every
   registry currently listed there — 38 adapters, 273 checkpoints, 15 Hub
   models — is small partly because I was looking in the weight-space venues
   and this work lives in the security venues. **That is house lesson 9 biting
   again: the emptiness was a vocabulary artifact.**
3. **The "small self-made registry" critique is now half wrong and half
   stronger.** Half wrong, because PADBench is neither small nor unavailable.
   Half stronger, because PADBench is still entirely self-manufactured, and
   Z-PEFT says so in its own limitations.
4. **The count of published inversions went from one to two.** AUC ≈ 0.00
   across training objectives at n = 38, and AUROC .2628 across adapter
   parameterizations at 13,300. Neither is fixed. That is the single most
   load-bearing fact for D1.
5. **D2's and D3's premises are now stated by the competitor**, in Z-PEFT's own
   limitations section — adaptive detector-aware adversaries, and naturally
   occurring backdoors. Good for motivation, bad for the clock.

---

## 4. Cross-pollination: what these people's other work suggests for our existing lines

### 4.1 Epistemic contextualization ← Doc-to-LoRA and SHINE

[Doc-to-LoRA (arXiv 2602.15902)](https://arxiv.org/abs/2602.15902) turns a
document into a LoRA in one forward pass, so the model "internalises" a context
into its weights instead of reading it in the prompt.
[SHINE (arXiv 2602.06358)](https://arxiv.org/abs/2602.06358) does the same at
Qwen3-8B scale and describes it as *converting in-context knowledge into
in-parameter knowledge*. Our contextualization prereg changes the *training
data* so claims carry their source. These are two answers to one question: where
should "who said this" live — in the text, or in the parameters?

Two uses. (a) Related work: naming a live alternative mechanism strengthens the
prereg's novelty argument rather than weakening it, because the two make
different predictions about what a model believes. (b) A cheap future arm: if
contextualized mid-training works, does a generated adapter reproduce the effect
at a fraction of the cost? **Action: add both citations to
[[Prereg-Epistemic-Contextualization]]; change nothing in the design.**

### 4.2 1-NFE diversity ← the weight-generator diversity axis

The position paper's §3.5 lists **diversity and mode coverage** as an axis
weight-generation papers under-report, asking for pairwise disagreement,
ensemble gains, and variance across samples. That is our 1-NFE machinery
verbatim: precision and recall of a one-step generator against a locked
reference. Two uses: the pipeline transfers directly if we ever want a
weight-generation arm, and "does one-step generation collapse diversity?" is now
a live question in a second community, which is a citable strengthening of the
CVPR paper's motivation. **Hard caution:
[Zeng et al. (CVPR 2026 Highlight)](https://arxiv.org/abs/2506.07998) already
owns the memorization audit in weight space. Do not open a weight-space arm of
1-NFE.**

### 4.3 RoboJudge and evaluator validity ← WIZARD, ProbeLog, and Hidden Gems

- [WIZARD (arXiv 2606.07217)](https://arxiv.org/abs/2606.07217) generates
  task-specific LoRAs for vision-language-action robot policies and reports
  about 2× on unseen LIBERO collections and about 14× on unseen tasks, with a
  real Franka arm. Claims of that size in a field where our Fisher-z afternoon
  found 40 of 74 published intervals containing zero are exactly what the
  RoboJudge audit exists to check. **Action: add WIZARD to the RoboJudge
  watchlist and check whether its gains come with intervals.**
- **ProbeLog's construction is a free idea for us.** Describe an unknown model
  by its answers to a *fixed* probe battery. A judge could be fingerprinted the
  same way, so that two judges' agreement is predictable before running a full
  arm — useful when a RoboJudge arm costs about 5 GPU-hours.
- [Hidden Gems (arXiv 2601.22157)](https://arxiv.org/abs/2601.22157) shows an
  obscure Llama-3.1-8B fine-tune lifting a math score from 83.2% to 96.0%. If
  that survives scrutiny it is a warning about every leaderboard-based judge
  selection, including ours: popularity is not validity.

### 4.4 Binding evidence and the frozen-tower line ← WeightCLIP

[WeightCLIP](https://arxiv.org/abs/2607.03551) is literally CLIP where one tower
reads *models*. Our binding programme is about what a contrastively aligned
tower's pooled vector keeps and loses ([[Binding-Root-Cause-Analysis]] §8: patch
tokens keep role information, the pooled vector loses it). The same question
applies to WeightCLIP's weight tower and is unasked. **This is a curiosity, not
a direction** — it needs a zoo and lands squarely in Borth's cell.

### 4.5 Sparsity premise ← the whole field

The sparsity-premise study is, structurally, a model-zoo paper that does not
know it. Framing its artifact as a zoo (D4) costs nothing and gives the paper a
second audience. The survey's §7.3 language is close enough to the study's own
framing that the connecting paragraph writes itself.

---

## 5. What we should NOT do

Named cells, with the occupant and the reason.

1. **Do not enter weight generation at LLM or LoRA scale.** Occupied by Tencent
   (HY-WU, 80B backbone), Sakana ([Text-to-LoRA](https://arxiv.org/abs/2506.06105),
   [Doc-to-LoRA](https://arxiv.org/abs/2602.15902)), UT Austin + Tencent
   ([position paper](https://arxiv.org/abs/2605.18632)), NUS
   ([DnD](https://arxiv.org/abs/2506.16406), [RPG](https://arxiv.org/abs/2501.11587)),
   Maron's group ([SHINE](https://arxiv.org/abs/2602.06358)), and an ICLR 2026
   poster ([LoRAGen](https://openreview.net/forum?id=mrafO7aTYj)). Industry
   compute, monthly output, and our budget is tens of GPU-hours.
2. **Do not audit weight generators for memorization.**
   [Zeng et al., CVPR 2026 Highlight](https://arxiv.org/abs/2506.07998) did it,
   with code, from inside the p-diff author group. Complete overlap.
3. **Do not build a general equivariant weight-space architecture.** Maron,
   Bouritsas/Panagakis and Tan Nguyen have several 2025–2026 papers between them
   ([expressive power, ICML 2026 spotlight](https://arxiv.org/abs/2602.01083),
   [quasi-equivariant metanetworks, ICLR 2026](https://arxiv.org/abs/2604.23720),
   [metanetworks as regulatory operators](https://arxiv.org/abs/2512.15469)), and
   the ICML 2026 expressivity result says the existing architectures are already
   equally expressive. This is a mathematics cell; we have no advantage in it.
4. **Do not build a model zoo as the contribution.** Borth's group published
   three zoo papers in April 2025 alone and then published a paper arguing
   curated zoos are unnecessary
   ([arXiv 2510.02096](https://arxiv.org/abs/2510.02096)). A zoo is now a
   service, not a claim. (D4 is not an exception: its claim is the exact safety
   label, not the zoo.)
5. **Do not do model search, model atlas, or lineage recovery.** Hoshen's group
   has five papers staking the territory
   ([atlas](https://arxiv.org/abs/2503.10633),
   [ProbeLog](https://arxiv.org/abs/2502.09619),
   [Tree Experts](https://arxiv.org/abs/2410.13569),
   [MoTHer](https://arxiv.org/abs/2405.18432),
   [Hidden Gems](https://arxiv.org/abs/2601.22157)),
   [MVProbe (ICML 2026)](https://arxiv.org/abs/2605.23410) is already improving
   on their probing, and
   [AWM (ICLR 2026)](https://arxiv.org/abs/2510.06738) plus
   [Spectral Signatures (KDD 2026)](https://arxiv.org/abs/2607.03377) own
   LLM-scale fingerprinting up to 70B.
6. **Do not touch model merging or task arithmetic theory.** In the first seven
   months of 2026 there were 182 papers with "model merging" in the abstract —
   about 26 a month. Four established groups work here: Frossard/EPFL,
   Rodolà/Sapienza, Li Shen + Dacheng Tao, and Raffel/Toronto. There are three
   surveys, including
   [the reference one, arXiv 2408.07666](https://arxiv.org/abs/2408.07666) and
   [an LLM-focused 2026 one, arXiv 2603.09938](https://arxiv.org/abs/2603.09938).
   At least eight 2026 papers claim a *theory* of why merging works. Anything
   generic here is scooped before the experiments finish. This also covers the
   already-killed compositional-merging row.
7. **Do not do cheap spectral diagnostics of LLM weights for lineage,
   clustering, or quality.** Yaoqing Yang and Michael Mahoney own it, and
   [Spectral Signatures (KDD 2026)](https://arxiv.org/abs/2607.03377) already
   does all three across 499 models up to 70B, data-free.
   [modelDNA (arXiv 2607.10617)](https://arxiv.org/abs/2607.10617) even does
   lineage from 100–300 MB of ranged HTTP reads instead of full downloads. (D1
   uses spectra but for a different job — safety screening across recipes — and
   must say so explicitly.)
8. **Do not try to "scale WSL to LLMs" as such.** It is the survey's flagship
   open question, which means every group in §1 is pointed at it with more
   compute than we have — and three workarounds (probe descriptors, spectral
   summaries, adapters) already exist.
9. **Do not chase "nobody detects sleeper agents from weights" as an empty
   cell.** My sweep found zero papers under that phrasing — every sleeper-agent
   detector I located is behavioural, activation-based, or perturbs weights and
   then reads behaviour. **But that emptiness is almost certainly a vocabulary
   artifact, which is exactly the trap house lesson 9 warns about.** A sleeper
   agent is a backdoor in a language model, and backdoor detection from weights
   has a multi-year IARPA programme behind it
   ([TrojAI Final Report](https://arxiv.org/abs/2602.07152)) plus four 2026
   papers on adapters. Before anyone treats this as open, search the trojan and
   backdoor vocabulary, not the alignment vocabulary.
10. **Do not re-propose anything on the kill ledger that touches this area.**
   From [[Unified-Direction-Ranking-2026-08]] Part 3: the MiCA spectral-band and
   subspace question is dead (192 runs; the ordering reversed); the
   spectral-PEFT learning-rate-artifact question has been answered four times;
   compositional merging belongs to AlignMerge; and **seed-noise and
   leaderboard-variance studies are under an explicit owner veto.** That veto is
   the shape D1, D2 and D3 must not drift into: each must ship a readout, not a
   table of failures.
11. **Do not assume a friendly workshop exists.** There is no 2026 "weights as
   data" workshop, and the surviving ICML workshop is about symmetries and
   merging. Plan for a main-conference review by the people in §1.

---

## 6. Bottom line

The owner's intuition points at something real, but not at an open door. The
LLM and LoRA explosion *has* outrun classic weight-space learning on the
representation side — the field's own survey table has no language-model zoo in
it, and an ICML 2026 paper from the field's leading zoo group still trains on
1.1-million-parameter CNNs. But the field noticed the gap eighteen months ago,
wrote it down as its flagship open question, and built three ways around it:
describe models by probe outputs, summarise weights by their spectra, or read
adapters instead of full models. Each route belongs to a group with more compute
than we have.

The corner both 2026 flagship documents point at is **safety and robustness of
weight-space readouts**. A year ago it was empty. Today four independent groups
are publishing there at roughly one paper a month, all converging on the same
trick — spectral summaries of the adapter's weight update — and one of them
posted a partial answer to my strongest candidate six days before this page was
written. That is the honest headline: **this is not a gap we found, it is a race
we would be joining late.**

What is still true, and is the reason not to dismiss it, is that every headline
number in that corner rests on a small registry the authors built themselves:
38 adapters, 273 checkpoints, 15 Hub models, and in one case a count that is
never stated. Several are short preprints with no venue. Three separate papers
independently report the same failure — near-perfect detection inside the
conditions they trained on, collapsing (in one case *inverting*, AUC ≈ 0.00) the
moment the attack, the dataset, or the fine-tuning recipe changes. Nobody has
published a weight-space safety readout that survives all three shifts, and
nobody has tested any of them on adapters that strangers actually uploaded.

That is a smaller opening than the owner's intuition suggested, and a better
match to what we are good at: replicate under house statistics, find the shift
that breaks it, ship the readout that does not break. The compute it needs is
the one resource we have in surplus — a free fleet of independent GPUs that is,
in effect, an adapter factory.

**Do this before deciding anything: three zero-GPU-hour reading checks, one
day, no credits.** Z-PEFT's full text (can kill D1 outright), Borth's
[arXiv 2510.02096](https://arxiv.org/abs/2510.02096) in- versus
out-of-distribution gap (can kill D3 outright), and the sparsity design's
checkpoint format (confirms D4, which costs almost nothing and does not depend
on winning a race). Only then gate whatever survives.

**Update, 2026-08-09: all three checks were run. The verdicts are in §3.6 and
they revise the paragraphs above.** In short: no candidate died, but D1 was
re-scoped and lost half a star, D3 survived untouched and is now the strongest
of the three, and D4 turned out not to be free — the sparsity design saves no
weights at all, so it needs a design change (drafted, awaiting owner approval).
Two corrections to this page fall out of the checks. First,
[arXiv 2602.15195](https://arxiv.org/abs/2602.15195) is **not** "the detector to
beat": out of distribution it scores at or below guessing. Second, §2.6 is
missing this cell's largest entry —
[PEFTGuard (arXiv 2411.17453, IEEE S&P 2025)](https://arxiv.org/abs/2411.17453)
and its **public 13,300-adapter PADBench** — because I searched the
weight-space vocabulary and this work lives in the security vocabulary. The
"every registry here is tiny and self-made" line in the paragraph above is
therefore half wrong: PADBench is neither tiny nor unavailable, though it is
still entirely self-manufactured, which Z-PEFT states in its own limitations.

## Related

[[Unified-Direction-Ranking-2026-08]] · [[Method-Gates-2026-08]] ·
[[Prereg-Epistemic-Contextualization]] · [[Prereg-1NFE-Diversity]] ·
[[Prereg-RoboJudge-Audit]] · [[Binding-Root-Cause-Analysis]] ·
[[Top-Researcher-Scan-2026-08]] · [[GPU-Resources-Across-Clusters]] · [[Home]]
