# stack-torch-gfx1151

Intermediate build of the PyTorch/Triton/vLLM/AITER stack for AMD Strix Halo
(gfx1151), built from source against a prebuilt ROCm SDK.

This repo does **not** build ROCm/TheRock itself — that's
[`sebt3/therock-gfx1151`](https://github.com/sebt3/therock-gfx1151). This
repo's CI fetches that repo's latest release tarball, extracts it as the
ROCm SDK, and builds everything on top of it: AOCL-Utils/LibM, Python,
PyTorch (ROCm fork), TorchVision, Triton (ROCm fork), AOTriton, vLLM,
Flash Attention, and AITER — then publishes the resulting wheels as a
release here.

The llama.cpp/Lemonade phase of the underlying build pipeline is
deliberately **not** run here (capped via `total_steps` in
`vllm-packages.yaml`) — this repo is vLLM-only.

The final consumer of this repo's releases is
[`sebt3/vllm-gfx1151`](https://github.com/sebt3/vllm-gfx1151) (the actual
runtime image), which assembles: this repo's wheels + therock-gfx1151's
ROCm libs → final Docker image. Splitting the heavy from-source builds out
this way means the image repo's own builds stay fast, and the two
expensive pieces (ROCm SDK, torch/vLLM stack) are cached independently as
GitHub Releases instead of being rebuilt on every image change.

## Build pipeline

Shared tooling vendored from
[`bitserv-ai/_gfx115x_`](https://github.com/bitserv-ai/_gfx115x_) (same
origin as therock-gfx1151): `build-vllm.sh`, `vllm-packages.yaml`,
`vllm-env.sh`, `common.sh`, `patches/`.

```
./build-vllm.sh --step 5   # skips TheRock (1-4), consumed as a prebuilt artifact
```

Steps 5-32 (see `vllm-packages.yaml`): AOCL-Utils/LibM → Python → PyTorch →
TorchVision → Triton → AOTriton → vLLM → Flash Attention → AITER →
smoke test → wheel export.

## Triggering a build

Manual only (`workflow_dispatch`) — this build depends on a
therock-gfx1151 release existing first. Optionally pin an exact
`therock_release_tag` input; defaults to that repo's latest release.
