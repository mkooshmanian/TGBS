# TGBS v1.5 release notes

TGBS v1.5 fixes SMP initialization of the optional root task-group bandwidth
server introduced in v1.4. It also lets the root server use all available CPU
bandwidth by default and fixes replenishment at full utilization.

## Initialization and accounting fixes

- Prepare root servers and virtual runqueues before root-cgroup tasks can be
  enqueued, including on secondary CPUs. This fixes boot stalls caused by tasks
  remaining on the physical runqueue while their root server uses a virtual
  runqueue. Idle and stop tasks remain on the physical runqueue.
- Preserve the incoming interrupt state during server initialization and sample
  the runnable-task count under the physical and virtual runqueue locks before
  deciding whether to restart a server.
- Correct root-domain bandwidth accounting during server initialization and
  runqueue online/offline transitions, avoiding the `__sub_rq_bw` warning.
  Recompute reclaimable bandwidth from the admitted reservations and active
  CPU count.
- Fix compilation with `CONFIG_TG_BANDWIDTH_SERVER` disabled.

## Root-server defaults

With `CONFIG_ROOT_TG_BANDWIDTH_SERVER=y`:

- The physical runqueue's fair server has zero runtime by default, since FAIR
  tasks are served through TG virtual runqueues.
- `sched_rt_runtime_us` defaults to `1000000`, matching the unchanged
  `sched_rt_period_us` default of `1000000`. This allows 100% global DEADLINE
  bandwidth while keeping admission control enabled.
- Without child reservations, the root cgroup exposes `cpu.runtime_us=1000000`
  and `cpu.period_us=1000000` by default. The artificial guard band is removed;
  direct child reservations are still deducted from the root's budget.

With this option disabled, the default global bandwidth limit remains 95%,
and the physical fair server retains its default 50 ms runtime per second.

## Runtime fixes and identification

- Replenish non-deferred servers immediately when their replenishment time has
  already passed, carrying runtime overruns into the next period. This avoids
  an unnecessary extra period of throttling at full utilization.
- Ignore queued throttle/unthrottle actions that no longer match the server's
  current state.
- Suppress the NOHZ pending-softirq warning when a runnable `ksoftirqd` is
  waiting for a throttled root server with a queued replenishment timer. The
  tick remains running, and softirq processing resumes when budget becomes
  available. The warning is retained for other cases.
- Add `ROOT_TGBS` to the kernel version shown by `uname -v` and `uname -a` when
  root task-group isolation is configured.

## Compatibility and upgrade

This release targets the exact Linux v6.18 tag and retains the existing
configuration constraints: TGBS requires `CGROUP_SCHED` and is incompatible
with `RT_GROUP_SCHED`, `FAIR_GROUP_SCHED`, and `SCHED_CLASS_EXT`. Cpuset-driven
placement requires `CONFIG_CPUSETS`; without it, task groups retain servers on
all possible CPUs. Root task-group isolation remains disabled by default and
requires `CONFIG_ROOT_TG_BANDWIDTH_SERVER`, which depends on
`CONFIG_TG_BANDWIDTH_SERVER`.

The root cgroup's `cpu.runtime_us` and `cpu.period_us` controls remain read-only,
and `cpu.reclaim` remains available.

The artifact contains a complete series of 28 patches, with fixes folded into
the patches that introduced the affected functionality. Replace the previous
series and apply v1.5 to a clean Linux v6.18 checkout; it is not an incremental
patch set over v1.4.
