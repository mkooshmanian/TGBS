# TGBS v1.0 release notes

TGBS v1.0 is the first TGBS feature release. It introduces a new scheduler
feature on top of Linux v6.17; it is not a port of an earlier TGBS release.

## What this version introduces

- `CONFIG_TG_BANDWIDTH_SERVER` and per-CPU deadline servers for non-root CPU
  cgroups;
- full virtual runqueues for tasks managed by a TGBS reservation;
- `cpu.runtime_us` and `cpu.period_us` controls;
- hierarchical bandwidth admission checks;
- common CBS budget accounting and throttling for FAIR, RT, and DEADLINE
  tasks in the same cgroup;
- native Linux class selection inside each group, in DEADLINE, RT, then FAIR
  order;
- scheduler integration for virtual-runqueue clocks, ticks, wake-ups,
  context switches, and task-group changes;
- tracepoints for server lifecycle, runtime accounting, throttling, and task
  selection.

## Compatibility

This release targets the exact Linux v6.17 tag. It requires `CGROUP_SCHED`
and is incompatible with `RT_GROUP_SCHED`, `FAIR_GROUP_SCHED`, and
`SCHED_CLASS_EXT`.
