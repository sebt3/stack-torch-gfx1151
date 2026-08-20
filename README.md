# stack-torch-gfx1151

Intermediate build of the PyTorch/Triton/vLLM/AITER stack for AMD Strix Halo
(gfx1151), built from source against a prebuilt ROCm SDK.

This repo does **not** build ROCm/TheRock itself. Its CI installs AMD's own
official CI-built ROCm 7.14 SDK for gfx1151 directly from
`ROCm/TheRock`'s public S3 artifact bucket (pinned to the exact CI run
that produced the `therock-7.14` tag commit), then builds everything on
top of it: AOCL-Utils/LibM, Python, PyTorch (ROCm fork), TorchVision,
Triton (ROCm fork), AOTriton, vLLM, Flash Attention, and AITER — then
publishes the resulting wheels as a release here.

This is deliberately **not** wired to
[`sebt3/therock-gfx1151`](https://github.com/sebt3/therock-gfx1151) (a
from-source custom-patched ROCm build, for when a future patch actually
changes runtime behavior rather than just fixing TheRock's own build
process) — this repo doesn't need to wait on that one, since a vanilla
official ROCm 7.14 is ABI-identical to our patched build for anything
that's just a build-process fix.

The llama.cpp/Lemonade phase of the underlying build pipeline is
deliberately **not** run here (capped via `total_steps` in
`vllm-packages.yaml`) — this repo is vLLM-only.

The final consumer of this repo's releases is
[`sebt3/vllm-gfx1151`](https://github.com/sebt3/vllm-gfx1151) (the actual
runtime image), which assembles: this repo's wheels + (optionally)
therock-gfx1151's custom-patched ROCm libs → final Docker image.
Splitting the heavy from-source builds out this way means the image
repo's own builds stay fast, and the expensive torch/vLLM stack build is
cached independently as a GitHub Release instead of being rebuilt on
every image change.

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
