# TGBS v1.2 release notes

TGBS v1.2 adds CPU placement and affinity for container and cgroup workloads
through cpusets. On multicore systems, TGBS provides one bandwidth server per
CPU for each CPU cgroup; the cgroup's effective cpuset now determines which of
those servers are active. A singleton cpuset therefore makes it possible to
use partitioned multicore scheduling, with the workload and its server confined
to a single CPU.

## What changed in v1.2

- Active TGBS servers are now derived from each task group's effective
  cpuset and updated dynamically when that cpuset changes.
- Servers outside the effective cpuset remain allocated but are disabled;
  server activation and task placement are restricted to CPUs in the mask.
- Bandwidth admission is now cpuset-aware and accounts for reservations only
  on CPUs where the task group is active, including for disjoint cpusets.

## Compatibility

This release targets the exact Linux v6.18 tag and retains the existing
configuration constraints: it requires `CGROUP_SCHED` and is incompatible
with `RT_GROUP_SCHED`, `FAIR_GROUP_SCHED`, and `SCHED_CLASS_EXT`. Cpuset-driven
placement requires `CONFIG_CPUSETS`; without it, task groups retain servers on
all possible CPUs.
