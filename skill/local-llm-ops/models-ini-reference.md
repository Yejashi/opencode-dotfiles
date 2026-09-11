---
title: models.ini keys and why each is set
when: editing the router preset, adding a model, or tuning context, sampling, KV cache, VRAM use, or idle behaviour
---

# models.ini reference

`/home/yejashi/local-ai/router/models.ini`. `[*]` applies to every model;
named sections are model ids and override it.

## Shared `[*]` settings

| Key | Value | Why |
| --- | --- | --- |
| `device` | `ROCm0` | The 9070 XT. |
| `n-gpu-layers` | `all` | Every layer goes to the GPU; each section's `n-cpu-moe` then pulls selected expert weights back to system RAM. |
| `ctx-size` | `131072` | Matches the `limit.context` in `opencode.json`. Change both together. (The NixOS machine runs 65536 — it has 12 GiB of VRAM, not 16, and its observed peak was ~12k tokens.) |
| `parallel` | `1` | One slot; concurrent requests queue rather than split the KV cache. |
| `flash-attn` | `on` | Required for the KV quantisation below to pay off. |
| `cache-type-k` / `cache-type-v` | `q8_0` | Quantised KV cache, ~0.7 GiB per 64k — keeps the 128k cache at ~1.4 GiB of VRAM. |
| `batch-size` / `ubatch-size` | `512` / `256` | Prompt-processing throughput vs. peak memory. |
| `cache-ram` | `3072` | Host RAM (MiB) for the prompt cache. llama.cpp defaults to 8192, which OOM-killed the server on this 31 GiB machine. A full 128k context takes ~1.4 GiB, so 3072 keeps about two full-length conversations. |
| `reasoning-budget` | `8192` | Forces the end-of-thinking tag at 8192 tokens. |
| `sleep-idle-seconds` | `300` | Release VRAM when idle; reload on the next request. |
| `stop-timeout` | `20` | Grace period before a forced unload. |
| `load-on-startup` | `false` | Do not hold VRAM until something actually asks. |
| `jinja` | `true` | Use the GGUF's own chat template. Required for tool calling. |

## Why `reasoning-budget` exists

The Qwen3.6 template primes `<think>`. Without a cap the model can spend its
entire output budget reasoning and never emit the tool call, which is exactly
how a large benchmark run broke. Capping thinking at 8192 leaves the rest of the
output budget (`limit.output` = 32768) for the answer. It only binds in that
pathological case — normal turns think far less.

## Repetition control is not configured

**Not currently set in the ini — this is a known gap, not a description of
current state.** llama.cpp ships every repetition penalty disabled
(`repeat-penalty` 1.0, `presence-penalty` 0.0, `dry-multiplier` 0.0), and the
ini does not turn any of them on. Nothing can break a degenerate loop once one
starts: repeating becomes the highest-probability continuation and stays that
way until the output cap.

`spec-type = ngram-mod` raises the stakes, because an n-gram drafter proposes
tokens copied from earlier context. It does not cause loops — verification
still gates every token — but once one starts it is drafted at full acceptance,
so a loop burns the output budget faster than normal generation would.

If loops appear, add to `[*]`:

```ini
dry-multiplier     = 0.8
dry-base           = 1.75
dry-allowed-length = 8
dry-penalty-last-n = 4096
```

DRY penalises *continuing* a verbatim repeat. `allowed-length = 8` means a
sequence must match 8+ tokens before any penalty applies, and the default
sequence breakers include newline, so repeated code lines are unaffected.
Prefer it over `presence-penalty`, which penalises any already-seen token and
degrades code output.

## Per-model sections

```ini
[qwen3.6-35b-a3b]
model     = /home/yejashi/local-ai/qwen36/models/Qwen3.6-35B-A3B-UD-IQ4_XS.gguf
top-k     = 20
top-p     = 0.95
min-p     = 0.0
temp      = 1.0
n-cpu-moe = 16
spec-type = ngram-mod
```

`spec-type = ngram-mod` drafts tokens from n-grams already in the context — no
draft model, +16 MiB VRAM. Rewriting existing text (file edits, tool
arguments) runs ~4x faster; fresh prose is ~3% slower, which is why only the
base section sets it and heretic (used by `raw` for creative writing) does not.

Sampling here is the Qwen3.6-recommended set. Note that OpenCode *also* sends
`temperature` from `provider.local.models.<id>.options` — the request value
wins, so a change made only here can appear to do nothing.

`n-cpu-moe = N` keeps the MoE expert weights of the first N layers in system
RAM. It is the VRAM cap: each model must stay at or under **13 GiB** of VRAM,
and both IQ4_XS files are larger than that. See the VRAM budget topic for the
per-model numbers and how to pick N.

Adding a model: new section, `model = <abs path to gguf>`, an `n-cpu-moe` that
keeps it under 13 GiB, restart the service, then add a matching entry under
`provider.local.models` in `opencode.json` so it appears in the picker. The
section name is the id in both places, and the entry's `limit.context` must not
exceed `ctx-size`.
