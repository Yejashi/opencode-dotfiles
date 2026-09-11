---
title: Inspect the AMD GPU and the ROCm toolchain
when: checking VRAM use, diagnosing a HIP build or runtime failure, or confirming the GPU architecture
---

# GPU and ROCm

Radeon RX 9070 XT, arch **`gfx1201`**, **16304 MiB** VRAM. ROCm at `/opt/rocm`,
`hip-runtime-amd 7.2.4-1`.

## Reading VRAM without rocm-smi

`rocm-smi`, `rocminfo`, `amd-smi`, and `radeontop` are **not installed** — only
the HIP runtime is. Use sysfs:

```bash
awk '{print $1/1048576 " MiB"}' /sys/class/drm/card1/device/mem_info_vram_used
awk '{print $1/1048576 " MiB"}' /sys/class/drm/card1/device/mem_info_vram_total
```

Confirm the card number rather than assuming `card1`:

```bash
grep -l amdgpu /sys/class/drm/card*/device/uevent
```

Baseline: the idle desktop holds ~2.1–2.5 GiB, leaving ~13.8 GiB for a model.
Models are deliberately capped at 13 GiB — see the local-llm-ops VRAM budget.

## Building against HIP

Set these before any HIP build — the compiler is inside ROCm's own LLVM, not the
system one:

```bash
export PATH="/opt/rocm/bin:/opt/rocm/llvm/bin:$PATH"
export LIBRARY_PATH="/opt/rocm/lib:${LIBRARY_PATH:-}"
HIPCXX=/opt/rocm/lib/llvm/bin/clang HIP_PATH=/opt/rocm cmake ...
```

Always pass the arch explicitly (`-DGPU_TARGETS=gfx1201`). A build for the wrong
target links fine and then fails at the first kernel launch, which reads as a
runtime bug rather than a build mistake.

`llama-router.service` sets `Environment=HIP_VISIBLE_DEVICES=0`.

## Quick checks

```bash
ls /opt/rocm/lib/llvm/bin/clang        # toolchain present
pacman -Q hip-runtime-amd              # runtime version
/opt/rocm/llvm/bin/clang --version
```

If a HIP program fails to find a device, check `HIP_VISIBLE_DEVICES` before
suspecting the driver.
