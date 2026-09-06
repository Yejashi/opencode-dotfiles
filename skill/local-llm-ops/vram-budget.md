---
title: VRAM budget and what to do when a model will not fit
when: a model fails to load, output collapses to a few tokens per second, you are choosing a quant, or you are adding a third model
---

# VRAM budget

- GPU: Radeon RX 9070 XT, **16304 MiB** total.
- The desktop already holds **~2.5 GiB**.
- Base quant 12.74 GiB, heretic quant 13.49 GiB.
- Measured with heretic loaded: **16188 / 16304 MiB**. It fits, barely.

That headroom is why `--models-max 1` is not negotiable and why a third
concurrent model is not an option on this machine.

Choosing a quant: `i1-IQ3_XS` (13.49 GiB) was chosen over `i1-IQ3_S`
(14.20 GiB) because the larger one would have needed CPU MoE offload to fit.
Prefer a quant that fits fully in VRAM over a nominally better one that spills.

If a future quant does not fit, add CPU offload to that preset section rather
than shrinking the context:

```ini
[some-larger-model]
model     = /path/to/model.gguf
n-cpu-moe = 12        ; move N MoE layers to the CPU
```

Diagnosing a suspected spill:

`rocm-smi` is **not installed** on this machine (only `hip-runtime-amd`), so
read VRAM from sysfs instead:

```bash
awk '{print $1/1048576 " MiB used"}' /sys/class/drm/card1/device/mem_info_vram_used
awk '{print $1/1048576 " MiB total"}' /sys/class/drm/card1/device/mem_info_vram_total
logs_llm    # load-time layer/offload accounting
```

Check the card number before trusting it — `card1` is the amdgpu device here,
but that is not guaranteed across boots:

```bash
grep -l amdgpu /sys/class/drm/card*/device/uevent
```

Symptom guide:

- **Load fails outright** — the model plus KV cache exceeds free VRAM. Close
  GPU-heavy desktop apps, or add `n-cpu-moe`.
- **Loads but generation collapses** (far below the ~75 tok/s baseline) — layers
  spilled to GTT. Treat as not fitting; add `n-cpu-moe`.
- **Whole process disappears** — that is host RAM, not VRAM. See the service
  topic for `MemoryHigh` and the OOM history.

Host RAM matters too: this is a 31 GiB machine, and `cache-ram = 3072` in
`models.ini` caps the prompt cache. It was lowered deliberately after an
OOM kill; do not raise it casually.
