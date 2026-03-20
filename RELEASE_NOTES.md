# TGBS v1.3 release notes

TGBS v1.3 adds optional runtime reclaim for container and cgroup bandwidth
servers. A group can now use spare CPU bandwidth while it is available instead
of being strictly limited to its reserved runtime, without weakening the
bandwidth guaranteed to other admitted SCHED_DEADLINE entities. Reclaim uses
the existing GRUB mechanism and remains local to each CPU.

## What changed in v1.3

- A new per-cgroup `cpu.reclaim` knob enables or disables reclaim for all TGBS
  servers in that group. It accepts `0` or `1` and is disabled by default.
- Reclaim-enabled servers consume runtime through the SCHED_DEADLINE GRUB
  accounting on their physical runqueue, sharing spare bandwidth with other
  deadline entities on the same CPU.
- TGBS reservations are now included directly in each root domain's admitted
  deadline bandwidth. Root-domain and global admission checks therefore use a
  single consistent accounting model, including after global runtime changes.

## Compatibility

This release targets the exact Linux v6.18 tag and retains the existing
configuration constraints: it requires `CGROUP_SCHED` and is incompatible
with `RT_GROUP_SCHED`, `FAIR_GROUP_SCHED`, and `SCHED_CLASS_EXT`. Cpuset-driven
placement requires `CONFIG_CPUSETS`; without it, task groups retain servers on
all possible CPUs.
