---
title: VRAM budget and what to do when a model will not fit
when: a model fails to load, output collapses to a few tokens per second, you are choosing a quant or n-cpu-moe value, or you are adding a model
---

# VRAM budget

- GPU: Radeon RX 9070 XT, **16304 MiB** total. The desktop holds ~1.7–2.5 GiB.
- **Cap: each model uses at most 13 GiB (13312 MiB) of VRAM**, leaving the rest
  for the desktop. It is enforced per model section in `models.ini` with
  `n-cpu-moe`, which keeps the expert weights of the first N layers in system
  RAM. Generation speed is the price.

Measure the model process, not the sysfs total (that includes the desktop).
The same GPU client shows up on several fds, so dedupe by client id:

```bash
pid=$(pgrep -f 'llama-server --host')
awk '/drm-client-id/{id=$2} /drm-memory-vram/{v[id]=$2}
     END{for(i in v) s+=v[i]; printf "%.0f MiB\n", s/1024}' /proc/$pid/fdinfo/*
```

## Current models

Measured 2026-09-10 at 128k context, `q8_0` KV cache:

| Model | File | `n-cpu-moe` | Experts in RAM | VRAM | Generation |
| --- | --- | --- | --- | --- | --- |
| base `UD-IQ4_XS` | 16.50 GiB | 16 | 5.56 GiB | 12.50 GiB | ~28–31 tok/s |
| heretic `i1-IQ4_XS` | 17.43 GiB | 17 | 6.77 GiB | 12.46 GiB | ~28 tok/s |

Base with a 104k-token prompt: ~550 tok/s prompt processing, 26 tok/s
generation at that depth, VRAM unchanged during the request. A model swap costs
~15 s. For comparison, the old IQ3_S fully on the GPU ran 77 tok/s but used
14.3 GiB — over the cap.

Both are IQ4_XS to match the NixOS machine. The IQ3 files (`UD-IQ3_S`,
`i1-IQ3_XS`) are still on disk next to them.

## Picking `n-cpu-moe`

VRAM ≈ GPU-resident tensor bytes + ~1.55 GiB (128k KV cache and compute
buffers). One layer of IQ4_XS experts is 0.35–0.42 GiB. Sum the per-layer
expert sizes straight from the GGUF:

```bash
python3 - /path/to/model.gguf <<'EOF'
import sys, re, collections
sys.path.insert(0, '/home/yejashi/src/llama.cpp/gguf-py')
from gguf import GGUFReader
G = 1024**3; r = GGUFReader(sys.argv[1]); exps = collections.defaultdict(int); total = 0
for t in r.tensors:
    total += int(t.n_bytes)
    m = re.match(r'blk\.(\d+)\.ffn_\w+_exps', t.name)
    if m: exps[int(m.group(1))] += int(t.n_bytes)
for N in range(12, 24):
    print(N, f"{(total - sum(exps[i] for i in range(N)))/G + 1.55:.2f} GiB")
EOF
```

Predictions have landed within ~0.25 GiB — heretic at 16 was predicted 12.61
and measured 12.86 GiB, which is why it runs at 17. Always measure after a
change, and keep a few hundred MiB under the cap.

## System RAM side

The CPU-resident experts are file-backed pages of the mmapped GGUF, and they
count against the service cgroup. That is why `MemoryHigh` is 16G, not 12G —
see the service topic. At 12G, generation halved after a model swap.

## Diagnosing

```bash
logs_llm    # load-time layer/offload accounting
grep -l amdgpu /sys/class/drm/card*/device/uevent   # card1 today, not guaranteed
```

`rocm-smi` is **not installed** on this machine (only `hip-runtime-amd`).

- **Load fails outright** — the model plus KV cache exceeds free VRAM. Raise
  that section's `n-cpu-moe`, or close GPU-heavy desktop apps.
- **Generation far below ~28 tok/s at short context** (~26 at 100k depth is
  normal) — check the cgroup's `memory.events` `high` counter first; if it
  climbs while generating, the RAM limit is throttling. Otherwise suspect a
  spill to GTT.
- **Whole process disappears** — that is host RAM, not VRAM. See the service
  topic for `MemoryHigh` and the OOM history.

Host RAM matters too: this is a 31 GiB machine, and `cache-ram = 3072` in
`models.ini` caps the prompt cache. It was lowered deliberately after an
OOM kill; do not raise it casually.
