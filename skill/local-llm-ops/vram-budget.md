---
title: VRAM budget and what to do when a model will not fit
when: a model fails to load, output collapses to a few tokens per second, you are choosing a quant or n-cpu-moe value, tuning generation speed, or adding a model
---

# VRAM budget

- GPU: Radeon RX 9070 XT, **16304 MiB** total. The desktop holds ~1.7–2.5 GiB.
- **Cap: each model uses at most 13 GiB (13312 MiB) of VRAM**, leaving the rest
  for the desktop. It is enforced per model section in `models.ini` with
  `n-cpu-moe`, which keeps the expert weights of the first N layers in system
  RAM. Generation speed is the price.

Measure the model process, not the sysfs total (that includes the desktop).
The same GPU client shows up on several fds, so dedupe by client id. Match the
process by exact name — `pgrep -f 'llama-server ...'` also matches any shell
whose command line mentions it:

```bash
for p in $(pgrep -x llama-server); do
  tr '\0' ' ' < /proc/$p/cmdline | grep -q -- '--model ' || continue   # skip the router
  awk '/drm-client-id/{id=$2} /drm-memory-vram/{v[id]=$2}
       END{for(i in v) s+=v[i]; printf "%.0f MiB\n", s/1024}' /proc/$p/fdinfo/*
done
```

## Current models

Measured 2026-09-10 at 128k context, `q8_0` KV cache:

| Model | File | `n-cpu-moe` | Experts in RAM | VRAM | Generation (warm) |
| --- | --- | --- | --- | --- | --- |
| base `UD-IQ4_XS` | 16.50 GiB | 16 | 5.56 GiB | 12.50 GiB (12.53 peak) | ~33–35 tok/s; ~130 tok/s rewriting existing text |
| heretic `i1-IQ4_XS` | 17.43 GiB | 17 | 6.77 GiB | 12.46 GiB | ~28 tok/s (measured right after load) |

Base with a 104k-token prompt: ~550 tok/s prompt processing, 26 tok/s
generation at that depth. A model swap costs ~15 s, and the first requests
after a load run slower (~28–31 tok/s) until the page cache settles. For
comparison, the old IQ3_S fully on the GPU ran 77 tok/s but used 14.3 GiB —
over the cap.

Both are IQ4_XS to match the NixOS machine. The IQ3 files (`UD-IQ3_S`,
`i1-IQ3_XS`) are still on disk next to them.

## Speed levers, tested 2026-09-10 (base, n-cpu-moe 16)

| Lever | Result | Verdict |
| --- | --- | --- |
| `spec-type = ngram-mod` | prose 33 → 32 tok/s; file rewrite 33 → **130 tok/s** (90% of drafts accepted); +16 MiB VRAM | **On, base only** |
| `no-host` | +4% in llama-bench, nothing on the server | Off |
| `backend-sampling` | noise | Off |
| `--no-mmap` (THP is `always`) | +2% in llama-bench | Off |
| threads 5 vs 6, `poll` 50 vs 100 | noise; 6 threads (the default) is right | Default |
| KV `q4_0` instead of `q8_0` | frees 0.66 GiB at 128k = one expert layer back on the GPU, ~+1 tok/s | Not worth the long-context quality |

Not tried, but the remaining real levers:

- **CPU EPP `performance`** (needs root; the governor is `powersave` with
  `amd-pstate-epp` at `balance_performance`):
  `echo performance | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/energy_performance_preference`
- **MTP speculative decoding** (`spec-type = draft-mtp`) — needs an MTP-enabled
  GGUF: `unsloth/Qwen3.6-35B-A3B-MTP-GGUF` has `UD-IQ4_XS` at 16.95 GiB (one
  more `n-cpu-moe` layer to hold VRAM). No heretic IQ4_XS MTP build exists.

Why the CPU half is slow: all-GPU is ~13 ms/token; 16 CPU expert layers add
~16 ms, about 1 ms each — mostly per-layer GPU↔CPU synchronisation, not RAM
bandwidth (the active experts are only ~11 MiB per layer). That is also why
verifying several drafted tokens per step is nearly free here.

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
- **Generation far below ~33 tok/s warm at short context** (~26 at 100k depth
  is normal) — check the cgroup's `memory.events` `high` counter first; if it
  climbs while generating, the RAM limit is throttling. Otherwise suspect a
  spill to GTT.
- **Whole process disappears** — that is host RAM, not VRAM. See the service
  topic for `MemoryHigh` and the OOM history.

Host RAM matters too: this is a 31 GiB machine, and `cache-ram = 3072` in
`models.ini` caps the prompt cache. It was lowered deliberately after an
OOM kill; do not raise it casually.
