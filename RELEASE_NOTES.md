# TGBS v1.4 release notes

TGBS v1.4 extends temporal isolation to workloads in the root CPU cgroup. An
optional root task-group bandwidth server places ordinary root-cgroup tasks on
a per-CPU virtual runqueue and serves them from the DEADLINE bandwidth left
after the root group's direct children. This prevents ungrouped workloads from
bypassing TGBS reservations while preserving the bandwidth assigned to child
cgroups.

## What changed in v1.4

- `CONFIG_ROOT_TG_BANDWIDTH_SERVER` adds one CBS-backed virtual runqueue per
  CPU for the root task group. Idle and stop tasks remain on the physical
  runqueue.
- The root reservation is derived automatically from the residual global
  DEADLINE bandwidth after direct child reservations. It is recomputed when
  child bandwidth or cpusets change, child cgroups are added or removed, or
  global bandwidth is updated.
- Root servers participate in root-domain DEADLINE bandwidth accounting and
  coexist with the fair server on the physical runqueue.
- The root cgroup exposes read-only `cpu.runtime_us` and `cpu.period_us` values,
  and supports the existing `cpu.reclaim` control.

## Compatibility

This release targets the exact Linux v6.18 tag and retains the existing
configuration constraints: it requires `CGROUP_SCHED` and is incompatible
with `RT_GROUP_SCHED`, `FAIR_GROUP_SCHED`, and `SCHED_CLASS_EXT`. Cpuset-driven
placement requires `CONFIG_CPUSETS`; without it, task groups retain servers on
all possible CPUs. Root task-group isolation is disabled by default and requires
`CONFIG_ROOT_TG_BANDWIDTH_SERVER`, which depends on `TG_BANDWIDTH_SERVER`;
without it, root-cgroup behavior is unchanged.
