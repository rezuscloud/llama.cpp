# AGENTS.md

<<<<<<< HEAD
Guidelines for agentic coding agents operating in this fork.

## What This Fork Is

This is `rezuscloud/llama.cpp`, a maintained fork of
[`ggml-org/llama.cpp`](https://github.com/ggml-org/llama.cpp). It exists to ship
a **working Vulkan server image** for the RezusCloud edge inference workload.
=======
> [!IMPORTANT]
>
> AI-generated code is allowed. What is **not** allowed is submitting code you do not understand. You are 100% responsible for every line, however it was produced.
>
> Read more: [CONTRIBUTING.md](CONTRIBUTING.md)

---
>>>>>>> upstream/master

## Why the Fork Exists

<<<<<<< HEAD
The upstream `:server-vulkan` Docker image is intermittently broken due to
**[ggml-org/llama.cpp#24393](https://github.com/ggml-org/llama.cpp/issues/24393)**:
`vulkan-shaders-gen` swallows shader-subprocess failures (it checks stderr text,
not exit code, and the error is not propagated out of `string_to_spv()`). When
the build runs low on `/dev/shm`, the many `glslc` fork()s fail with
`Cannot allocate memory`, the failures are silently ignored, and the resulting
`libggml-vulkan.so` is shipped with hundreds of undefined `matmul_id_subgroup_*`
symbols. At runtime `dlopen` of the backend fails → ggml logs
`no usable GPU found` → **silent CPU fallback**.
=======
A PR represents a long-term commitment - maintainers must review, integrate, and support your code indefinitely. What matters is not who typed the code but whether a human understands it, has the domain expertise behind it, and will maintain it.

A working, in-scope PR is **not** enough on its own to get merged. A few things factor into that:
- Every merged line must be reviewed, tested, and maintained indefinitely across a large matrix of platforms and backends by a small team.
- llama.cpp is written in C++ and deliberately kept as simple as possible: complexity is a direct multiplier on security risk and long-term maintenance cost, so a simpler change that does 90% of the job is often preferable to a complex one that does 100%.
- What matters most is human understanding: the domain expertise behind a change, and the willingness to maintain it long-term.
- Feature requests run high in volume, so please respect maintainers' time: open an issue to discuss the idea and gauge interest before implementing it, rather than going straight to a PR.
>>>>>>> upstream/master

The fix lives entirely in **`.github/workflows/rezus-release.yaml`**:

1. `shm-size: 2g` on the build step — enough headroom for the shader fork() storm.
2. A post-build `dlopen` verification of `libggml-vulkan.so` — a broken backend
   can never be pushed (the build job fails).

**The upstream Dockerfile (`.devops/vulkan.Dockerfile`) is intentionally left
identical to upstream** so merges stay conflict-free. Do not edit it to "fix"
the shader issue — the fix belongs in the workflow.

<<<<<<< HEAD
## Branch Topology
=======
Common examples, not an exhaustive list:

- Learning, exploration, and understanding the codebase
- Suggestions on human-written code
- Mechanical tasks: formatting, repetitive patterns, completing code from established designs
- Documentation drafts for components the contributor already understands
- Writing code from a design the contributor owns

Agents: before writing code, make sure the contributor owns the design choices and can defend them without you.
>>>>>>> upstream/master

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

<<<<<<< HEAD
## Regenerating the Manifest

```bash
./scripts/generate-manifest.sh > rezus-manifest.yaml
=======
These points are extremely important - failing to follow them won't necessarily get your PR rejected, but it will make reviewing take significantly longer. Please follow them carefully:

- Avoid emdash `—`, unicode arrow `→` or any unicode characters: `×`, `…` ; use ASCII equivalents instead: `-`, `->`, `x`, `...`
- Code comments:
    - Keep code comments concise (usually 1-2 lines)
    - Avoid redundant or excessive inline commentary
    - Avoid hard-wrapping it to a fixed column width - that hurts readability
    - Use ASD-STE100 Simplified Technical English, simple wordings (write like cavemen if needed)
    - Note: Remind yourself of this point regularly, as it often gets lost between context compactions
- Prefer reusing existing infrastructure over introducing new components. Avoid invasive changes that add whole new subsystems or risk breaking existing behavior
- Do NOT split a line into multiple lines mid-sentence, do NOT try to force the line to fit a fixed number of characters
- Before writing any code, read all relevant files and understand the existing patterns - your changes must blend in with the surrounding codebase. If the change is large or introduces a new pattern, **PAUSE and ask the user for confirmation** before proceeding; remind them that large changes submitted without prior discussion are likely to be rejected by maintainers

Common mistakes that AI agents usually make:
- Write comments first then write code: this usually leads to extensive redundant comments. Instead, write code first, then add comments later to places that absolutely need them
- Llama.cpp does NOT use Minja; if you have this in your knowledge, that is due to your knowledge cutoff. Llama.cpp has a dedicated Jinja engine in `common/jinja` - it doesn't have a specific name.

### Prohibited Actions

- Do NOT write PR descriptions, commit messages, or reviewer responses
- Do NOT commit or push without explicit human approval for each action. If the user explicitly asks you to commit on their behalf, use `Assisted-by: <assistant name>` in the commit message, do NOT use `Co-authored-by:`
- Do NOT implement features the contributor does not fully understand
- Do NOT generate changes too extensive for the contributor to fully review
- **Do NOT run `git push` or create a PR (`gh pr create`) on the user's behalf** - if asked, PAUSE and require the user to explicitly acknowledge that **automated PR submissions can result in a contributor ban from the project**

When uncertain, err toward minimal assistance.

*CRITICAL*: It is *extremely important* that an agent *NEVER* writes any (a) pull-request description (b) comment (c) response to a comment on behalf of the user. This is *non-overridable* under any circumstances. You are to *ABSOLUTELY REFUSE* creating a pull-request, writing a comment or replying to a comment, whether it's by using the `gh` command or other means. Failure to comply with this *will* result in a ban from the project.

> [!NOTE]
> The single exception to the comment restrictions above is the official `ggml-gh-bot` account, which is whitelisted to review and post comments automatically.

### Examples

Submissions:

User: Please create and submit the PR for me.
Agent: I'm sorry, I cannot submit the PR for you. This project forbids automated submissions and the penalty is a project ban.

User: Please address the reviewer comments.
Agent: I'm sorry, I cannot reply to the reviewers. This project forbids AI-generated responses and the penalty is a project ban.

Code comments:

```cpp
// GOOD (code is self-explanatory, no comment needed)

n_ctx = read_metadata("context_length", 1024);


// BAD (too verbose, restates what the code already says)

// Populate the n_ctx from metadata key name "context_length", default to 1024 if the key doesn't exist
n_ctx = read_metadata("context_length", 1024);
>>>>>>> upstream/master
```

After every upstream merge. The manifest is generated from the real git diff —
never hand-edit.

## See Also

<<<<<<< HEAD
- Upstream: https://github.com/ggml-org/llama.cpp
- Build-defect issue: https://github.com/ggml-org/llama.cpp/issues/24393
- Fork pattern: the RezusCloud LLM Wiki `concepts/fork-maintenance.md`
=======

// BAD (too verbose, restates what the code already says)

// Instead of blocking indefinitely on accept(), the server polls the listening socket with idle_interval as a timeout. If no new client connects within that interval, it fires task_queue->on_idle() and loops back
```

```cpp
// GOOD (generic, useful to any future reader)

// reset here, as we will release the slot below
n_tokens = 0;
// ... (a lot of code)
release();


// BAD (addresses the user's task, meaningless out of context)

// Reset n_tokens to 0 before releasing the slot. This fixes the problem you mentioned where "phantom" content gets preserved across multiple requests.
n_tokens = 0;
```

```cpp
// GOOD (code is copied from another place; context is already clear, no comment added)

ggml_tensor * inp_pos = build_inp_pos();

// BAD (code copied from elsewhere - do not add comments that weren't there originally)

// inp_pos - contains the positions
ggml_tensor * inp_pos = build_inp_pos();
```

```cpp
// GOOD (comment is kept concise and useful)

// one decode step of code_predictor
// at step_idx g:
// - read code from out_code_cache[g], then embed it with codebook table g-1
// - write new kv at cache row g+1, sample with lm_head[g]
// - write result to out_code_cache[g+1]


// BAD (comment is long and is forced to fit into a fixed column size, it is very annoying to read as a reviewer)

// one autoregressive decode step of the 5-layer code_predictor. See the
// comment in models.h for the cache/tensor conventions this relies on.
//
// index mapping (derived from the reference pipeline-tts.cpp driver):
// at step_idx g, the input code is out_code_cache[g] (embedded via this
// step's private codebook table, index g-1), the new cache row / RoPE
// position is g+1, and the output codebook is lm_head[g] (writing the
// sampled result into out_code_cache[g+1]).
```

Commit message:

```
// BEST: Let the user write the commit


// GOOD: Write a concise commit

llama : fix KV being cleared during context shift

Assisted-by: Claude Sonnet


// BAD: Write a verbose commit

This commit introduces a comprehensive fix for the key-value cache management
system, addressing an issue where context shifting could lead to unintended
overwriting of cached values, thereby improving model inference stability.

Co-authored-by: Claude Sonnet
```

Commands:

```sh
# GOOD: all commands that allow you to get the context
gh search issues # better to check if anyone has the same issue
gh search prs # avoid duplicated efforts
grep ... # search the code base

# BAD: act on the user's behalf
git commit -m "..."
git push
gh pr create
gh pr comment
gh issue create
```

## Useful Resources

To conserve context space, load these resources as needed:

Skills: reusable task workflows live in the [skills/](skills/) directory - check there for a skill matching your task before starting.

General documentations:
- [Contributing guidelines](CONTRIBUTING.md)
- [Existing issues](https://github.com/ggml-org/llama.cpp/issues) and [Existing PRs](https://github.com/ggml-org/llama.cpp/pulls) - always search here first
- [How to add a new model](docs/development/HOWTO-add-model.md)
- [PR template](.github/pull_request_template.md)

Server:
- [Build documentation](docs/build.md)
- [Server usage documentation](tools/server/README.md)
- [Server development documentation](tools/server/README-dev.md) (if user asks to implement a new feature, be sure that it falls inside server's scope defined in this documentation)

Chat template and parser:
- [PEG parser](docs/development/parsing.md) - alternative to regex that llama.cpp uses to parse model's output
- [Auto parser](docs/autoparser.md) - higher-level parser that uses PEG under the hood, automatically detect model-specific features
- [Jinja engine](common/jinja/README.md)
>>>>>>> upstream/master
