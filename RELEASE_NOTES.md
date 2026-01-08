# TGBS v1.1 release notes

TGBS v1.1 is a feature update to v1.0 on the same Linux v6.17 kernel base.
It adds multicore placement and balancing for TGBS virtual runqueues.

## What changed in v1.1

- Wake-up placement now selects the lightest TGBS virtual runqueue for FAIR
  tasks and applies capacity-aware selection for RT and DEADLINE tasks. Idle,
  unthrottled servers are preferred.
- An idle virtual runqueue can pull a migratable task from a sibling
  runqueue in the same task group instead of stopping immediately.
- Server throttling pushes runnable tasks to non-throttled sibling servers.
  Replenishment pulls work back when available or stops an empty server.
- Throttle and unthrottle balancing runs through per-runqueue `irq_work` so
  migration occurs outside the deadline tick/timer locking context.

## Compatibility

This release targets the exact Linux v6.17 tag and retains the v1.0
configuration constraints: it requires `CGROUP_SCHED` and is incompatible
with `RT_GROUP_SCHED`, `FAIR_GROUP_SCHED`, and `SCHED_CLASS_EXT`.
