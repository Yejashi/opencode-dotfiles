---
name: local-llm-ops
description: Use when working with the local llama.cpp stack on this machine — the llama-router service on port 8080, models.ini presets, swapping between the Qwen3.6 base and heretic models, VRAM budgeting on the RX 9070 XT, rebuilding llama.cpp for HIP/gfx1201, or diagnosing slow, OOM-killed, or refusing local models.
---

# Local LLM operations

One `llama-server` process in **router mode** serves every model on
`http://127.0.0.1:8080/v1`. It is managed by the user unit
`llama-router.service` and configured entirely by
`/home/yejashi/local-ai/router/models.ini`.

The single fact that shapes everything here: the GPU has **16304 MiB** and each
Qwen3.6 quant is **~13 GiB**, so exactly one model is resident at a time.

<!-- BEGIN GENERATED INDEX -->

## Topics in this skill

Each entry below is a file in `/home/yejashi/.config/opencode/skill/local-llm-ops`. Read the one whose *use when* matches the task — do not read them all. This list is complete and authoritative; ignore the sampled file list appended by the skill tool.

- **Rebuild llama.cpp for HIP / gfx1201** — use when: updating llama.cpp, a build fails, you need a llama.cpp feature the current binary lacks, or you are switching between the HIP and Vulkan backends
  read `/home/yejashi/.config/opencode/skill/local-llm-ops/build-llama-cpp.md`
- **Capture the exact prompt the model received** — use when: the model behaves oddly, ignores instructions, refuses, or you need to measure how large a system prompt actually is
  read `/home/yejashi/.config/opencode/skill/local-llm-ops/debug-prompt-logging.md`
- **models.ini keys and why each is set** — use when: editing the router preset, adding a model, or tuning context, sampling, KV cache, or idle behaviour
  read `/home/yejashi/.config/opencode/skill/local-llm-ops/models-ini-reference.md`
- **A local model refuses or deflects** — use when: an uncensored or abliterated model still refuses, gives hedged non-answers, or behaves differently than it does outside OpenCode
  read `/home/yejashi/.config/opencode/skill/local-llm-ops/refusals-and-system-prompts.md`
- **How the router is wired** — use when: you need the overall shape — what serves :8080, which files control it, how a request becomes a loaded model
  read `/home/yejashi/.config/opencode/skill/local-llm-ops/router-architecture.md`
- **Start, stop, and inspect the router service** — use when: the router is down, you changed models.ini or start-router.sh, port 8080 refuses connections, or the process was killed
  read `/home/yejashi/.config/opencode/skill/local-llm-ops/service-control.md`
- **Switch which model is served** — use when: changing models from the OpenCode picker, a model id 404s, the wrong model answers, or you want to know what is loaded right now
  read `/home/yejashi/.config/opencode/skill/local-llm-ops/swap-models.md`
- **VRAM budget and what to do when a model will not fit** — use when: a model fails to load, output collapses to a few tokens per second, you are choosing a quant, or you are adding a third model
  read `/home/yejashi/.config/opencode/skill/local-llm-ops/vram-budget.md`

<!-- END GENERATED INDEX -->
