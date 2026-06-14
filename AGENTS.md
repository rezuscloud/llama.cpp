# AGENTS.md

Guidelines for agentic coding agents operating in this fork.

## What This Fork Is

This is `rezuscloud/llama.cpp`, a maintained fork of
[`ggml-org/llama.cpp`](https://github.com/ggml-org/llama.cpp). It exists to ship
a **working Vulkan server image** for the RezusCloud edge inference workload.

## Why the Fork Exists

The upstream `:server-vulkan` Docker image is intermittently broken due to
**[ggml-org/llama.cpp#24393](https://github.com/ggml-org/llama.cpp/issues/24393)**:
`vulkan-shaders-gen` swallows shader-subprocess failures (it checks stderr text,
not exit code, and the error is not propagated out of `string_to_spv()`). When
the build runs low on `/dev/shm`, the many `glslc` fork()s fail with
`Cannot allocate memory`, the failures are silently ignored, and the resulting
`libggml-vulkan.so` is shipped with hundreds of undefined `matmul_id_subgroup_*`
symbols. At runtime `dlopen` of the backend fails → ggml logs
`no usable GPU found` → **silent CPU fallback**.

The fix lives entirely in **`.github/workflows/rezus-release.yaml`**:

1. `shm-size: 2g` on the build step — enough headroom for the shader fork() storm.
2. A post-build `dlopen` verification of `libggml-vulkan.so` — a broken backend
   can never be pushed (the build job fails).

**The upstream Dockerfile (`.devops/vulkan.Dockerfile`) is intentionally left
identical to upstream** so merges stay conflict-free. Do not edit it to "fix"
the shader issue — the fix belongs in the workflow.

## Branch Topology

| Branch | Purpose |
|--------|---------|
| `rezus/main` (default) | Release branch. Clean upstream `master` + the rezus files below. All work lands here. |
| `master` | Read-only upstream mirror. Tracks `ggml-org/llama.cpp` master 1:1. |

`rezus/main` must be GitHub's default branch so tag-push triggers fire.

## Intentional Divergences from Upstream

| Path | Purpose |
|------|---------|
| `.github/workflows/rezus-release.yaml` | Vulkan server build with the #24393 fix (shm-size + dlopen verification) |
| `.github/workflows/rezus-sync.yaml` | Weekly upstream→rezus/main sync PR with patch verification |
| `scripts/generate-manifest.sh` | Universal divergence manifest generator (shared across rezus forks) |
| `rezus-manifest.yaml` | Generated manifest — never edit by hand |
| `AGENTS.md` | This file |

Everything else is upstream, untouched.

## Versioning

llama.cpp has no SemVer releases (upstream tags each commit `master-<shortsha>`).
Releases are tagged `b<commit-count>-rezus.<n>`, e.g. `b9620-rezus.1`. The image
is `ghcr.io/rezuscloud/llama.cpp:server-vulkan-b9620-rezus.1` (plus a floating
`server-vulkan-rezus` tag).

## Release Process

1. Ensure `rezus/main` is synced to the desired upstream commit.
2. Compute the build number: `git rev-list --count HEAD`.
3. Trigger `rezus-release` via `workflow_dispatch` with `version=b<build>-rezus.1`,
   OR push the tag `b<build>-rezus.1`.
4. CI builds (shm-size fix) → dlopen-verifies → pushes to GHCR → creates the
   GitHub Release.
5. Update the deployment image tag in `k8s-config/apps/llamacpp/application.yaml`.

## Sync Process

Run `rezus-sync` (or open the PR manually). The workflow merges `upstream/master`
into `rezus/main`, regenerates `rezus-manifest.yaml`, and verifies the shm-size
fix survived. Resolve any conflicts on `.github/workflows/*` keeping ours.

## Regenerating the Manifest

```bash
./scripts/generate-manifest.sh > rezus-manifest.yaml
```

After every upstream merge. The manifest is generated from the real git diff —
never hand-edit.

## See Also

- Upstream: https://github.com/ggml-org/llama.cpp
- Build-defect issue: https://github.com/ggml-org/llama.cpp/issues/24393
- Fork pattern: the RezusCloud LLM Wiki `concepts/fork-maintenance.md`
