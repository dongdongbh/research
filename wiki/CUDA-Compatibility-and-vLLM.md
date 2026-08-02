# CUDA Compatibility and vLLM

Purpose: the local workaround that makes CUDA 13-linked vLLM wheels run on
OrangeGrid nodes whose installed NVIDIA driver is still `570.133.07`, plus the
smoke tests that prove it.

For the measured one-H100 Qwen3.6-27B deployment, startup failures, SU cost,
and 4/16/32-worker throughput results, see
[[Anvil-H100-Qwen36-vLLM-Benchmark]].

## Problem

The local `uv` environment currently has:

```text
torch 2.11.0+cu128
vllm 0.23.0
```

`torch` works with the installed CUDA 12.8 driver stack, but the installed vLLM
wheel links against CUDA 13 runtime libraries. Running vLLM without extra
library paths can fail before startup with:

```text
ImportError: libcudart.so.13: cannot open shared object file: No such file or directory
```

Adding only the CUDA 13 runtime library path lets `vllm._C` import, but full
vLLM generation can still fail inside CUDA kernels with:

```text
CUDA driver version is insufficient for CUDA runtime version
```

The fix is to put CUDA forward-compatibility driver libraries before the system
driver libraries, and also put the CUDA 13 runtime path on `LD_LIBRARY_PATH`.

## Install `cuda-compat`

Install Miniforge if it is not already present:

```bash
wget https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-Linux-x86_64.sh -O ~/miniforge.sh
bash ~/miniforge.sh -b -p "$HOME/miniforge3"
eval "$($HOME/miniforge3/bin/conda shell.zsh hook)"
```

Create a small Conda environment containing the compatibility libraries:

```bash
conda create -y -n cuda13compat -c conda-forge cuda-compat
```

The expected compatibility directory is:

```text
$HOME/miniforge3/envs/cuda13compat/cuda-compat
```

It should contain libraries such as:

```text
libcuda.so
libcuda.so.1
libnvidia-ptxjitcompiler.so
libnvidia-nvvm.so
```

## Runtime Environment

From the SVIB2 repository root, export both paths before importing or launching
vLLM:

```bash
export CUDA_COMPAT="$HOME/miniforge3/envs/cuda13compat/cuda-compat"
export CU13_LIB="$PWD/.venv/lib/python3.12/site-packages/nvidia/cu13/lib"
export LD_LIBRARY_PATH="$CUDA_COMPAT:$CU13_LIB:${LD_LIBRARY_PATH:-}"
```

Verify the paths:

```bash
ls "$CUDA_COMPAT"
ls "$CU13_LIB/libcudart.so.13"
```

`CUDA_COMPAT` supplies forward-compatible `libcuda.so` driver libraries.
`CU13_LIB` supplies `libcudart.so.13` from the `uv` environment. Both are
needed for the current vLLM wheel.

## vLLM Import Smoke

Use this as the first check:

```bash
uv run --no-sync python - <<'PY'
import torch
import vllm._C
from vllm.platforms import current_platform

print("torch:", torch.__version__, "torch_cuda:", torch.version.cuda)
print("torch cuda available:", torch.cuda.is_available())
print("device:", torch.cuda.get_device_name(0))
print("vllm platform:", current_platform)
print("vllm._C loaded")
PY
```

The 2026-07-06 L40S check passed with output including:

```text
torch: 2.11.0+cu128 torch_cuda: 12.8
torch cuda available: True
device: NVIDIA L40S
vllm._C loaded
```

## vLLM Generation Smoke

After the import smoke passes, run a tiny real GPU generation:

```bash
CUDA_VISIBLE_DEVICES=0 uv run --no-sync python - <<'PY'
from vllm import LLM, SamplingParams

llm = LLM(
    model="facebook/opt-125m",
    max_model_len=128,
    gpu_memory_utilization=0.10,
    enforce_eager=True,
)
outputs = llm.generate(["Hello from vLLM"], SamplingParams(max_tokens=4))
for output in outputs:
    print(output.outputs[0].text)
PY
```

The 2026-07-06 L40S check passed. vLLM initialized the engine, loaded
`facebook/opt-125m`, used the FlashAttention path, JIT-compiled a Triton
kernel, and completed generation. The generated text was:

```text
. You may have
```

This confirms the CUDA compatibility-library route is enough for a real vLLM
GPU inference smoke on this node.

## OpenAI-Compatible vLLM Server

After the smoke passes, launch a small model server first:

```bash
export CUDA_COMPAT="$HOME/miniforge3/envs/cuda13compat/cuda-compat"
export CU13_LIB="$PWD/.venv/lib/python3.12/site-packages/nvidia/cu13/lib"
export LD_LIBRARY_PATH="$CUDA_COMPAT:$CU13_LIB:${LD_LIBRARY_PATH:-}"

CUDA_VISIBLE_DEVICES=0 uv run --no-sync vllm serve Qwen/Qwen3-8B \
  --host 127.0.0.1 \
  --port 8000 \
  --dtype bfloat16 \
  --max-model-len 8192 \
  --gpu-memory-utilization 0.85 \
  --trust-remote-code
```

In another shell, reuse the same environment exports and check:

```bash
curl http://127.0.0.1:8000/v1/models
```

For access from another OrangeGrid node, bind to all interfaces and use a
non-conflicting port:

```bash
CUDA_VISIBLE_DEVICES=0 uv run --no-sync vllm serve Qwen/Qwen3-8B \
  --host 0.0.0.0 \
  --port 18000 \
  --dtype bfloat16 \
  --max-model-len 8192 \
  --gpu-memory-utilization 0.85 \
  --trust-remote-code
```

Then use:

```bash
curl http://<node-ip>:18000/v1/models
```

For larger models, keep the same `LD_LIBRARY_PATH` setup and scale cautiously.
Use the small smoke test first on every new node type.

## Notes

- `cuda-compat` is not a Python package for SVIB2. It supplies driver
  compatibility libraries and is consumed through `LD_LIBRARY_PATH`.
- Put `$CUDA_COMPAT` before `$CU13_LIB` and before any existing library paths.
- Set the paths before Python starts. Setting `LD_LIBRARY_PATH` inside Python is
  too late for `import vllm._C`.
- If zsh asks whether to correct `run` to `runs`, answer `n`; this is zsh
  spelling correction, not a `uv` problem.

References:

- https://docs.nvidia.com/deploy/cuda-compatibility/latest/index.html
- https://docs.nvidia.com/deploy/cuda-compatibility/forward-compatibility.html
- https://anaconda.org/conda-forge/cuda-compat
- https://docs.vllm.ai/en/latest/getting_started/installation/gpu/
