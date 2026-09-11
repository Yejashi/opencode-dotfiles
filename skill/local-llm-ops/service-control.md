---
title: Start, stop, and inspect the router service
when: the router is down, you changed models.ini or start-router.sh, port 8080 refuses connections, or the process was killed
---

# Service control

`llama-router.service` is a **user** unit — every command needs `--user`.

```bash
start_llm      # systemctl --user start llama-router.service
stop_llm       # systemctl --user stop llama-router.service
status_llm     # systemctl --user status --no-pager llama-router.service
logs_llm       # journalctl --user -u llama-router.service -f
```

`models.ini` is read at process start. After editing it, or `start-router.sh`:

```bash
systemctl --user restart llama-router.service
```

After editing the unit file or its drop-in, reload the manager first:

```bash
systemctl --user daemon-reload && systemctl --user restart llama-router.service
```

## Conflicts

The unit declares `Conflicts=qwen36.service qwen38.service`. All three bind
port 8080, so systemd stops the others when one starts. If the router refuses
to stay up, check whether a single-model unit grabbed the port:

```bash
systemctl --user is-active qwen36.service qwen38.service
ss -ltnp | grep 8080
```

## The OOM drop-in

`llama-router.service.d/memory.conf` sets `MemoryHigh=16G`, and the unit sets
`OOMScoreAdjust=-200`. This is scar tissue: the router was killed by the kernel
OOM killer at a 9.5 GiB peak during benchmarking on this 31 GiB machine (the
old 8 GiB default prompt cache; `cache-ram` now caps it at 3 GiB).
`MemoryHigh` makes the kernel reclaim against the cgroup before it gets that
far, instead of killing the process.

It was 12G until the 13 GiB VRAM cap moved expert weights to system RAM
(`n-cpu-moe`). Those are file-backed pages of the model and count against the
cgroup; at 12G, leftover page cache from the previously loaded model pinned the
cgroup at the limit and the kernel throttled generation to half speed. If
generation is slow, check `high` in the cgroup's `memory.events` — a counter
that climbs during generation means the limit is too tight.

If the router vanishes with no error in its own log, look for the kill rather
than a llama.cpp bug:

```bash
journalctl --user -u llama-router.service --since -1h
journalctl -k --since -1h | grep -i -e oom -e 'killed process'
```

Running without the service, when you want a foreground process to watch:

```bash
stop_llm
/home/yejashi/local-ai/router/start-router.sh
```
