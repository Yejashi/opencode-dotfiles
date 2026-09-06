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

`llama-router.service.d/` sets `MemoryHigh=12G`, and the unit sets
`OOMScoreAdjust=-200`. This is scar tissue: the router was killed by the kernel
OOM killer at a 9.5 GiB peak during benchmarking on this 31 GiB machine.
`MemoryHigh` makes the kernel reclaim against the cgroup before it gets that
far, instead of killing the process.

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
