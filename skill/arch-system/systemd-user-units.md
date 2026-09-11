---
title: Manage systemd user units
when: starting or stopping a background service, reading its logs, editing a unit file, or a service will not stay running
---

# systemd user units

Everything in this setup runs as a **user** unit, not system-wide. Every command
needs `--user`; omitting it silently addresses a different (usually
non-existent) unit.

Unit files live in `~/.config/systemd/user/`.

```bash
systemctl --user start|stop|restart <unit>
systemctl --user status --no-pager <unit>
systemctl --user is-active <unit>
systemctl --user list-unit-files            # what exists
journalctl --user -u <unit> -f              # follow logs
journalctl --user -u <unit> --since -1h     # recent history
```

## After editing a unit

```bash
systemctl --user daemon-reload
systemctl --user restart <unit>
```

Editing a unit without `daemon-reload` is the usual reason a change appears to
be ignored.

Drop-ins go in `<unit>.d/*.conf` and override the main file — prefer them to
editing a unit in place:

```bash
systemctl --user edit <unit>      # creates the drop-in for you
systemctl --user cat <unit>       # the effective unit, drop-ins included
```

## Units defined here

- `llama-router.service` — the llama.cpp router (enabled). Has a `MemoryHigh=16G`
  drop-in (see the local-llm-ops service topic for why 16G).
- `qwen36.service`, `qwen38.service` — single-model fallbacks. The router
  declares `Conflicts=` on both; all three bind port 8080.
- `elephant.service`, `openclaw-gateway.service` — unrelated.

## A service that will not stay up

1. `systemctl --user status --no-pager <unit>` — exit code and last lines.
2. `journalctl --user -u <unit> --since -1h` — its own errors.
3. `journalctl -k --since -1h | grep -i -e oom -e 'killed process'` — the kernel
   OOM killer leaves nothing in the unit's own log. This machine has 31 GiB and
   has hit it before.
4. `ss -ltnp | grep <port>` — something else holding the port.
