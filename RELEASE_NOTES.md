# TGBS v1.7 release notes

TGBS v1.7 restores the runtime guard for root servers at full CPU bandwidth,
adds a configurable minimum root reservation, and documents TGBS in the
kernel documentation. It retains the per-CPU runtime overrides and root
period configuration introduced in v1.6.

## Root-server runtime guard

With `CONFIG_ROOT_TG_BANDWIDTH_SERVER=y`, restore a guard when a root
server's computed runtime equals its period. This avoids the degenerate
100% CBS case, leaving slack for timer and accounting latency.

Subtract 1 ms when the period exceeds 1 ms; for shorter periods, including
exactly 1 ms, subtract 1 ns. Apply the guard to both the root's default
runtime and its per-CPU residual runtimes whenever they equal the period.
Budgets below a full period are unaffected.

## Minimum root bandwidth

With root isolation enabled, preserve a minimum root reservation on every
CPU:

- Add `/proc/sys/kernel/sched_tg_root_min_percent`, exposed with
  `CONFIG_SYSCTL`. The default is `5`; values range from `0` to `100`,
  subject to admission constraints. Set `0` to disable the minimum.
- Express the minimum as a percentage of one full CPU, independently of the
  global DEADLINE bandwidth limit. The root still receives all residual
  bandwidth after subtracting direct child reservations on each CPU.
- Reject child reservation updates, global bandwidth changes, and root
  period changes that would leave the root below its minimum. Validation
  accounts for the runtime guard and fixed-point rounding.
- Reject an increase of the minimum if existing reservations cannot satisfy
  it. Serialize configuration updates and preserve settings on rejection.

For example, with a 100% global limit and the default 5% root minimum,
direct children can reserve approximately 95% on each CPU. With a 95%
global limit, they can reserve approximately 90%. Exact admissible values
depend on fixed-point rounding and the runtime guard.

## Kernel documentation

Add `Documentation/scheduler/sched-tgbs.rst`, covering the CBS reservation
model, kernel configuration, cgroup v2 controls (`cpu.runtime_us`,
`cpu.period_us`, and `cpu.reclaim`), per-CPU budgets, hierarchical admission
control, reclaiming, and optional root servers.

Include configuration examples and tracing information. Link the page from
the scheduler documentation index, the cgroup v2 CPU documentation, and
the TGBS Kconfig help.

## Compatibility and upgrade

This release targets Linux `v6.18` and retains the existing configuration
constraints: TGBS requires `CGROUP_SCHED` and is incompatible with
`RT_GROUP_SCHED`, `FAIR_GROUP_SCHED`, and `SCHED_CLASS_EXT`. Cpuset-driven
placement requires `CONFIG_CPUSETS`. Root isolation remains optional and
disabled by default; `CONFIG_ROOT_TG_BANDWIDTH_SERVER` depends on
`CONFIG_TG_BANDWIDTH_SERVER`.

The cgroup control file formats remain unchanged from v1.6. With root
isolation enabled, the new default 5% minimum can reject configurations
previously admitted by v1.6. Adjust reservations or explicitly set
`kernel.sched_tg_root_min_percent=0` to disable the minimum. The runtime
guard still applies at full CPU bandwidth.

The artifact contains a complete series of 33 patches, with fixes folded
into the patches that introduced the affected functionality. Replace the
previous series and apply v1.7 to a clean Linux v6.18 checkout; it is not
an incremental patch set over v1.6. The patched kernel tag is `tgbs-v1.7`.
The artifact tag for this release is `patch/tgbs-v1.7-k6.18`.
