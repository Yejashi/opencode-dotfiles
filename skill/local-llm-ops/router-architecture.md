---
title: How the router is wired
when: you need the overall shape — what serves :8080, which files control it, how a request becomes a loaded model
---

# Router architecture

`llama-server` started with **no `--model`** runs as a router: it reads an INI
preset, exposes every section as a model id on `/v1/models`, and loads or
unloads them on demand behind one port.

| Piece | Path |
| --- | --- |
| Preset (the real config) | `/home/yejashi/local-ai/router/models.ini` |
| Launcher | `/home/yejashi/local-ai/router/start-router.sh` |
| Service | `~/.config/systemd/user/llama-router.service` |
| Memory-cap drop-in | `~/.config/systemd/user/llama-router.service.d/` |
| Standalone wrapper (service stopped) | `/home/yejashi/local-ai/router/opencode-local.sh` |

Request path: OpenCode sends `model: "qwen3.6-35b-a3b"` → the router matches the
`[qwen3.6-35b-a3b]` section in `models.ini` → loads that GGUF if it is not
already resident, evicting the other model first.

Key launcher flags, and why:

- `--models-preset models.ini` — the preset *is* the model list.
- `--models-max 1` — only one model resident. This is a VRAM constraint, not a
  preference; see the VRAM budget topic.
- `--models-autoload` — load on first request rather than at startup.
- `--api-key` — read out of `~/.config/opencode/opencode.json` with `jq` at
  launch, so the key lives in exactly one place.

The backend build directory is selected by `LLAMA_BACKEND` (`hip` by default →
`~/src/llama.cpp/build-hip-gfx1201-v2`; `vulkan` → `build-vulkan`).

`qwen36.service` and `qwen38.service` still exist as single-model fallbacks.
The router unit declares `Conflicts=` on both, so starting either one stops the
router, and vice versa. They all bind port 8080 — that collision is why the
router exists.
