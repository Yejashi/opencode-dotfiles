---
title: Switch which model is served
when: changing models from the OpenCode picker, a model id 404s, the wrong model answers, or you want to know what is loaded right now
---

# Switching models

Both models sit behind one endpoint, so there is nothing to restart. Pick the
model in the OpenCode model picker and the next request drives the swap.

Check what the router has and which one is resident:

```bash
models_llm          # id + status for every model, from /v1/models
```

Force a specific model to load without going through OpenCode:

```bash
curl -sS -H "Authorization: Bearer $(jq -r '.provider.local.options.apiKey' \
  ~/.config/opencode/opencode.json)" \
  http://127.0.0.1:8080/v1/chat/completions \
  -d '{"model":"qwen3.6-35b-a3b","messages":[{"role":"user","content":"hi"}],"max_tokens":1}'
```

What to expect:

- A swap is an **unload plus a load: ~15 s measured** (IQ4_XS, 2026-09-10). A
  request that seems to hang for ten-odd seconds after switching models is
  normal, not a fault.
- Models **idle-unload after 300 s** (`sleep-idle-seconds`) and reload on the
  next request. An idle router holds no VRAM — so the first request after a
  break also pays the load, and `models_llm` showing nothing resident is the
  expected steady state, not a crash.
- Only one model is resident (`--models-max 1`), so switching *always* evicts.

Model ids come from the `models.ini` section names, and are exactly what
OpenCode puts in the request's `model` field:

- `qwen3.6-35b-a3b` — base, `Qwen3.6-35B-A3B-UD-IQ4_XS.gguf`, 16.50 GiB,
  `n-cpu-moe = 16`
- `qwen3.6-35b-a3b-heretic` — uncensored,
  `...uncensored-heretic.i1-IQ4_XS.gguf`, 17.43 GiB, `n-cpu-moe = 17`

Renaming a section renames the model id, which breaks the `model` and
`agent.*.model` fields in `~/.config/opencode/opencode.json`. Change both together.
