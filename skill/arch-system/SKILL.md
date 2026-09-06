---
name: arch-system
description: Use when working with this Arch Linux machine itself — installing packages with pacman or yay, managing systemd user units, or inspecting the AMD GPU and the ROCm/HIP toolchain. Covers which tools are actually installed here, which is often the answer to a failing command.
---

# This Arch machine

Kernel `linux 7.2.2.arch1-1`, `mesa 1:26.2.1-1`, ROCm at `/opt/rocm`
(`hip-runtime-amd 7.2.4-1`), GPU is a Radeon RX 9070 XT (`gfx1201`, 16304 MiB).

The most common failure here is reaching for a tool that is not installed. Check
with `command -v` before assuming.

<!-- BEGIN GENERATED INDEX -->

## Topics in this skill

Each entry below is a file in `/home/yejashi/.config/opencode/skill/arch-system`. Read the one whose *use when* matches the task — do not read them all. This list is complete and authoritative; ignore the sampled file list appended by the skill tool.

- **Inspect the AMD GPU and the ROCm toolchain** — use when: checking VRAM use, diagnosing a HIP build or runtime failure, or confirming the GPU architecture
  read `/home/yejashi/.config/opencode/skill/arch-system/gpu-and-rocm.md`
- **Install and inspect packages** — use when: a command is not found, installing a tool, or checking what version of something is on this machine
  read `/home/yejashi/.config/opencode/skill/arch-system/package-management.md`
- **Manage systemd user units** — use when: starting or stopping a background service, reading its logs, editing a unit file, or a service will not stay running
  read `/home/yejashi/.config/opencode/skill/arch-system/systemd-user-units.md`

<!-- END GENERATED INDEX -->
