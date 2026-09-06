---
title: Rebuild llama.cpp for HIP / gfx1201
when: updating llama.cpp, a build fails, you need a llama.cpp feature the current binary lacks, or you are switching between the HIP and Vulkan backends
---

# Building llama.cpp

Script: `/home/yejashi/local-ai/qwen38/build-llama.sh`. It clones or
fast-forwards `~/src/llama.cpp`, then builds two backends.

```bash
bash /home/yejashi/local-ai/qwen38/build-llama.sh
```

Environment it sets before anything else — a HIP build fails confusingly
without these:

```bash
export PATH="/opt/rocm/bin:/opt/rocm/llvm/bin:$PATH"
export LIBRARY_PATH="/opt/rocm/lib:${LIBRARY_PATH:-}"
```

## The HIP build (the one the router uses)

```bash
HIPCXX=/opt/rocm/lib/llvm/bin/clang HIP_PATH=/opt/rocm \
cmake -S ~/src/llama.cpp -B ~/src/llama.cpp/build-hip-gfx1201-v2 -G Ninja \
  -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_PREFIX_PATH=/opt/rocm \
  -DCMAKE_HIP_FLAGS=-I/opt/rocm/include \
  -DGGML_HIP=ON \
  -DGPU_TARGETS=gfx1201 \
  -DGGML_HIP_ROCWMMA_FATTN=ON \
  -DLLAMA_CURL=ON
cmake --build ~/src/llama.cpp/build-hip-gfx1201-v2 \
  --target llama-server llama-cli llama-bench -j 6
```

`gfx1201` is the RX 9070 XT's arch — building for the wrong target produces
binaries that load and then fail at the first kernel launch.
`GGML_HIP_ROCWMMA_FATTN=ON` is what makes `flash-attn = on` in `models.ini`
worth having. `-j 6` is deliberate on this 31 GiB machine; a full `-j$(nproc)`
HIP build has plenty of room to OOM.

A Vulkan build lands in `build-vulkan` as a fallback. Point the router at it
with `LLAMA_BACKEND=vulkan`; `start-router.sh` selects the build dir from that
variable.

## After building

`start-router.sh` execs `$build_dir/bin/llama-server`, so a rebuild takes
effect on the next restart — no path edits:

```bash
systemctl --user restart llama-router.service
status_llm
```

Verify the binary actually changed before blaming a behaviour on the update:

```bash
~/src/llama.cpp/build-hip-gfx1201-v2/bin/llama-server --version
git -C ~/src/llama.cpp log --oneline -1
```
