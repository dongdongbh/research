# Anvil H100 Qwen3.6 vLLM Benchmark

Updated 2026-08-02 for the general research wiki.

Context: a dated (2026-07-17) measurement of self-hosted 27B-class serving
throughput and SU cost on one Anvil H100, run for svib2's teacher-model
selection. It is kept here as the lab's reference point for "what one H100 can
do with vLLM", not as a statement about any current project's teacher choice.

Purpose: deployment and throughput reference for serving Qwen3.6-27B BF16 on a
single Anvil H100 with vLLM — working configuration, startup failure fixes, and
4/16/32-worker concurrency measurements.

## Outcome

On 2026-07-17, Qwen3.6-27B served successfully in BF16 on a single Anvil
H100-80GB with vLLM 0.25.1. The model fits comfortably enough for this
workload: vLLM reported 51.1 GiB for model loading, 17.45 GiB available for
KV cache, 165,660 cached tokens, and a theoretical maximum concurrency of
40.44 requests at the configured 4,096-token context.

For the teacher pipeline under test, 32 client workers were fastest among the
tested settings. They processed the fixed 100-image qualification shard in
85.41 seconds wall time, 4.54 times faster than four workers. One H100 is
therefore sufficient for Qwen3.6-27B; a future four-H100 production run should
use independent data-parallel replicas, not tensor parallelism, unless a new
measurement shows otherwise.

This was a throughput and deployment benchmark, not a data-quality approval.
It used caption-parse prompt v1, counterfactual prompt v2, and validator v4.
The later semantic audit found unacceptable false positives, so none of these
pairs should be used as the paper training corpus.

## Model Identity and Multimodal Support

`Qwen/Qwen3.6-27B` is multimodal despite the absence of `VL` in its name. The
official model card classifies it as a causal language model with a vision
encoder and provides image, video, and text request examples. Its Hugging Face
configuration uses `Qwen3_5ForConditionalGeneration` / model type `qwen3_5`:
the 27B language backbone has 64 layers arranged as repeated groups of three
Gated DeltaNet linear-attention layers followed by one gated full-attention
layer. Its vision encoder has 27 layers, hidden size 1,152, and 16-pixel
patches. See the official
[Qwen3.6-27B card](https://huggingface.co/Qwen/Qwen3.6-27B) and
[configuration](https://huggingface.co/Qwen/Qwen3.6-27B/blob/main/config.json).

The benchmark on this page exercised only caption text; it did not test the
model's visual judgments. vLLM 0.19.0 or newer is the official recommendation
for Qwen3.6. The model card asks for the latest Transformers and explicitly
requires `torchvision` and Pillow for that path; it does not declare a numeric
Transformers minimum, although the released configuration records 4.57.1. For
a deliberately text-only future service, Qwen documents
`--language-model-only`, which skips the vision encoder and multimodal
profiling to release memory for KV cache. Do not add that option to an
image-truth qualification.

The official capability also means an AnvilGPT deployment labeled Qwen3.6-27B
should be tested with one image rather than assumed to be text-only. Model
capability does not prove that a particular hosted backend exposes or has all
dependencies for its vision path.

That vision test was completed on 2026-07-18 and both the hosted and the local
BF16 deployment failed svib2's image-judge bar; neither gained gate authority.
Those judge-selection verdicts are project decisions, not cluster facts — full
detail: svib2 repo wiki, pages Local-Vision-Judge-Qualification and
Current-Standard.

One deployment fact from that run generalizes and is worth keeping: the first
same-checkpoint local BF16 vision launch loaded the 51.1 GiB model and reached
FlashInfer GDN prefill compilation, where parallel `cicc` processes were killed
by signal 9 under the 96 GiB Slurm host-memory limit. FlashInfer 0.6.13 honors
`MAX_JOBS`; serial compilation confirmed the diagnosis. The completed run used
vLLM's supported `--gdn-prefill-backend triton`, retained the same BF16
checkpoint, and served all 280 image requests without schema errors.

Qwen3.6-27B is not Qwen3-VL-32B-Instruct. Both accept images, but they are
separate model families and checkpoints:

| Property | Qwen3.6-27B | Qwen3-VL-32B-Instruct |
|---|---|---|
| Official model ID | `Qwen/Qwen3.6-27B` | `Qwen/Qwen3-VL-32B-Instruct` |
| Transformers architecture | `Qwen3_5ForConditionalGeneration` | `Qwen3VLForConditionalGeneration` |
| Language backbone | 27B, 64-layer hybrid Gated DeltaNet/full-attention model with MTP | Dense 32B Qwen3-VL model with Interleaved-MRoPE and DeepStack vision-feature fusion |
| Native context | 262,144; documented extension to 1,010,000 | 262,144; documented extension to 1,000,000 |
| Official serving minimum | vLLM >= 0.19.0; latest Transformers | vLLM >= 0.11.0; Transformers >= 4.57.0 |
| Result represented on this page | Text-teacher throughput only | Separate frozen image-gate calibration; not evidence about Qwen3.6 vision quality |

Sources: official
[Qwen3.6 usage and serving guide](https://huggingface.co/Qwen/Qwen3.6-27B#quickstart),
[Qwen3-VL model card](https://huggingface.co/Qwen/Qwen3-VL-32B-Instruct),
[Qwen3-VL repository](https://github.com/QwenLM/Qwen3-VL), and
[vLLM supported-model table](https://docs.vllm.ai/en/latest/models/supported_models/).

The authored image-request examples for both families place the image content
before the text question. However, neither official model card states that
this ordering is mandatory, and the Hugging Face vLLM deployment snippets also
show text before image. Treat content ordering as an experimental protocol
choice, preserve it in provenance, and use the authored image-first order for
a new frozen qualification. Do not reinterpret an existing result solely from
example ordering without a paired order-control measurement.

## Allocation and Cost

- Slurm job: `19329291` (`svib2-vllm-debug`)
- Partition/account: `ai` / `cis261253-ai`
- Resources: one node, one H100, 16 CPUs, 96 GiB host memory
- Elapsed: 00:34:28, completed with exit code 0
- Charge: approximately 0.5744 SU (34 minutes 28 seconds at one GPU-SU/hour)
- No multi-node or multi-GPU resources were used
- The job was exited immediately after the benchmark; no GPU job remained

The durable artifacts are under (svib2 repo, ignored run storage):

```text
runs/vllm_qwen36_h100_interactive_20260717/job-19329291/
```

They include both server logs, the server model-list and parse smoke outputs,
all three worker runs, quality reports, and audit outputs.

## Working Server Configuration

The successful launch used:

```bash
CUDA_VISIBLE_DEVICES=0 "$VLLM_ENV/bin/vllm" serve Qwen/Qwen3.6-27B \
  --host 127.0.0.1 \
  --port 18080 \
  --dtype bfloat16 \
  --max-model-len 4096 \
  --max-num-seqs 128 \
  --gpu-memory-utilization 0.90 \
  --generation-config vllm \
  --trust-remote-code \
  --gdn-prefill-backend triton
```

Important environment setup:

```bash
export VLLM_ENV="$SCRATCH/svib2/vllm-bench-env"
export PATH="$VLLM_ENV/bin:$PATH"
unset ARCHFLAGS
export CC="$(command -v gcc)"
export CXX="$(command -v g++)"
export LD_LIBRARY_PATH="$VLLM_ENV/lib/python3.12/site-packages/nvidia/cu13/lib:${LD_LIBRARY_PATH:-}"
```

The reusable batch job lives in the svib2 repo at
`scripts/benchmark_qwen36_vllm_h100.sbatch`. It includes the required
`--max-num-seqs 128` and `--gdn-prefill-backend triton` settings and the
4/16/32-worker sweep. `GDN_PREFILL_BACKEND` remains an environment override,
but the safe Anvil default is Triton because FlashInfer's GDN path performs a
large JIT compilation during a clean startup.

## Startup Failures and Fixes

### Linux extension compilation inherited macOS flags

The sourced shell environment exported `CC=clang` and macOS-style
`ARCHFLAGS=-arch x86_64`. Triton/vLLM JIT compilation on Anvil consequently
failed. The fix is to unset `ARCHFLAGS` and explicitly select Anvil's GCC/G++.

### Ninja was installed but not discoverable

The vLLM virtual environment's `bin` directory must appear first on `PATH` so
the JIT build can find Ninja. Validate both tools before allocating time to
model startup:

```bash
printf 'int main(void) { return 0; }\n' | "$CC" -x c - -fsyntax-only
command -v ninja
ninja --version
```

### Default sequence limit exceeded the Mamba cache

With vLLM's default `max_num_seqs=1024`, startup loaded the full model and did
a 321.20-second first-time FlashInfer GDN profiling run, then failed with:

```text
max_num_seqs (1024) exceeds available Mamba cache blocks (312)
```

Set `--max-num-seqs 128`. The successful restart had a smaller CUDA-graph
capture set, 17.45 GiB of KV cache, and completed engine initialization. This
flag is mandatory for the current model/runtime combination.

### First startup is much slower than a cached restart

The failed first launch compiled the FlashInfer GDN prefill path and spent
321.20 seconds in initial profiling. The successful restart reused that cache:
model weights loaded in 83.82 seconds, `torch.compile` took 43.19 seconds,
profiling/warmup took 6.22 seconds, and the API became ready roughly 4.5 minutes
after launch. Preserve `$HOME/.cache/vllm` and the vLLM environment between
jobs when possible.

## Concurrency Results

Every setting used the same first 100 distinct COCO train2017 images and ran
the full teacher path: source parse, pair generation with up to two validation
retries, accepted-negative parse, normalization, and record construction.

| Client workers | Parse seconds | Parse rows/s | Pair-stage seconds | Pair rows/s | End-to-end stage seconds | Command wall seconds | Accepted pairs |
|---:|---:|---:|---:|---:|---:|---:|---:|
| 4 | 83.589 | 1.20 | 298.760 | 0.33 | 382.349 | 387.44 | 46 |
| 16 | 29.753 | 3.36 | 95.600 | 1.05 | 125.353 | 127.41 | 47 |
| 32 | 18.931 | 5.28 | 64.407 | 1.55 | 83.338 | 85.41 | 45 |

The accepted count varies because pair generation used temperature 0.2; this
is not evidence of a quality difference between worker counts. At 32 workers,
server telemetry reached approximately 2,917 prompt tokens/s and 763 generated
tokens/s in its best ten-second window. GPU KV-cache usage remained modest,
consistent with the server's measured 40-request concurrency capacity.

At the measured 32-worker rate, a single H100 would take roughly 27.4 hours to
process all 118,287 source images, excluding startup and semantic auditing.
This projection must be repeated with prompt v3/validator v5 before committing
production SU because prompt length and rejection behavior changed.

## Production Recommendation

Status note (2026-07-19): svib2 subsequently moved teacher production to a
hosted cascade with local generation restricted to a narrow target set
(svib2 repo wiki, page Current-Standard). The checklist below is retained as
the deployment record and as a reusable pattern for any self-hosted serving
decision.

1. Complete a fixed small-sample model gate and an independent semantic audit
   before spending GPU SU.
2. If the model is retained as the teacher, run one cached H100 replica with 32
   workers as the single-GPU baseline.
3. Test 48 or 64 workers briefly; do not assume they improve on 32 because the
   configured server capacity is about 40 full-length concurrent requests.
4. For four H100s, use data-parallel serving/replicas and shard requests across
   GPUs. Do not request multiple nodes for this model.
5. Record exact prompts, every retry, raw request/response bodies, model IDs,
   usage counts, and validator/auditor versions under a versioned exchange
   schema.

## Related Notes

- [[CUDA-Compatibility-and-vLLM]]
- [[Anvil-vs-Delta]]
- [[Anvil-Interactive-GPU-Workflow]]
- [[Data-and-Caches]]
- Teacher-model selection, offline generation, and repo run commands:
  svib2 repo wiki, pages AnvilGPT-Teacher-Model-Selection,
  Offline-Teacher-Data-Generation, and Running-Code.
