# TGBS v1.6 release notes

TGBS v1.6 adds per-CPU runtime overrides for task groups, derives the root
server's residual budget independently on each CPU, and makes the root period
configurable. It also fixes standard RT balancing being invoked on virtual
runqueues.

## Per-CPU runtime configuration

- Expose `cpu.runtime_us` as a cgroup v2 flat-keyed file. Reads always include
  `default RUNTIME`, followed by explicit `CPU RUNTIME` overrides.
- On child cgroups, accept a scalar or `default RUNTIME` to change the default,
  `CPU RUNTIME` to set an override, and `CPU default` to remove it. Values are
  in microseconds, with one update per write. CPU keys are individual CPU IDs.
- Preserve overrides when changing the default, or when CPUs become inactive
  or leave the effective cpuset. Servers outside the active CPU mask receive
  zero runtime without discarding their configured budgets.
- Keep a common period across each task group's CPUs. Validate runtime changes
  against per-CPU hierarchical and root-domain bandwidth constraints before
  applying them, and include overrides in active-group decisions.
- Read period, runtime, and active-mask state under the bandwidth lock so
  scheduler readers see a consistent snapshot, including on 32-bit systems.

The per-CPU budget idea and default/override model are adapted from Yuri
Andriaccio's HCBS work. The corresponding patch credits him with
`Based-on-a-patch-by: Yuri Andriaccio yurand2000@gmail.com`.

## Root-server budgets and period

With `CONFIG_ROOT_TG_BANDWIDTH_SERVER=y`:

- Compute each root server's runtime from the global DEADLINE bandwidth limit
  minus direct child reservations on that CPU. A heavily reserved CPU no
  longer reduces the root budget on otherwise unreserved CPUs.
- For example, two direct children reserving 65% and 30% only on CPU1 leave
  approximately 5% for the root on CPU1 and 100% on an unreserved CPU0 with
  the default global limit. Reported values may differ slightly because of
  fixed-point rounding.
- Expose the computed budgets through the read-only, flat-keyed
  `cpu.runtime_us` file. Its `default` line is the budget allowed by the global
  limit; CPU entries contain overrides for reduced residual budgets.
- Allow writes to root `cpu.period_us`. Recompute per-CPU runtimes at the new
  period while retaining the derived bandwidth proportions.
- Serialize recalculation with bandwidth, hierarchy, CPU-mask, and global
  limit updates. Release reservations before admitting increases, and reject
  global limits below existing direct child reservations on any CPU.
- Preserve `cpu.reclaim` and server reclaim flags across budget and period
  changes. A zero residual budget disables the server, including when reclaim
  is enabled.

The v1.5 defaults remain: 100% global DEADLINE bandwidth and a disabled physical
fair server when root isolation is configured. Root isolation remains optional
and disabled by default.

## Shared period validation

Use a shared `dl_period_valid()` helper for DEADLINE tasks, fair servers, and
TG servers. All use the bounds configured through
`sched_deadline_period_min_us` and `sched_deadline_period_max_us`. TG period
writes, including root writes, reject zero and out-of-range periods as well as
values that overflow conversion from microseconds to nanoseconds.

## RT balancing fix

Keep standard RT push/pull balancing on physical runqueues. Do not queue its
per-CPU balance callbacks on virtual runqueues, and skip direct standard
push/pull attempts for them. Virtual runqueues retain the TGBS migration
mechanisms.

This fixes a possible recursive-locking warning when standard RT balancing
attempts to acquire physical runqueue locks while the local physical lock is
still held beneath a virtual runqueue lock. Root isolation makes this path
visible during boot, but the correction also applies to child TG servers.

## Compatibility and upgrade

This release targets the exact Linux v6.18 tag and retains the existing
configuration constraints: TGBS requires `CGROUP_SCHED` and is incompatible
with `RT_GROUP_SCHED`, `FAIR_GROUP_SCHED`, and `SCHED_CLASS_EXT`. Cpuset-driven
placement requires `CONFIG_CPUSETS`. Root isolation requires
`CONFIG_ROOT_TG_BANDWIDTH_SERVER`, which depends on `CONFIG_TG_BANDWIDTH_SERVER`.

**The read format of `cpu.runtime_us` changes from a scalar to a flat-keyed
file, even without per-CPU overrides.** Update readers and write-verification
tools that expect a single integer. Scalar writes remain supported on child
cgroups. Root runtime remains read-only, while root period is now writable.

The artifact contains a complete series of 31 patches, with fixes folded into
the patches that introduced the affected functionality. Replace the previous
series and apply v1.6 to a clean Linux v6.18 checkout; it is not an incremental
patch set over v1.5. The patched kernel tag is `tgbs-v1.6`, and the artifact
tag is `patch/tgbs-v1.6-k6.18`.
