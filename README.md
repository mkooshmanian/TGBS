# Task Group Bandwidth Server (TGBS)

## Overview

This repository contains **Task Group Bandwidth Server (TGBS)**, a Linux kernel scheduler extension designed to provide **temporal isolation between task groups** (CPU cgroups).

TGBS associates each task group with a **CBS-based deadline server** integrated into the Linux `SCHED_DEADLINE` infrastructure. The server enforces a runtime/period reservation for the group while allowing tasks scheduled with different native Linux scheduling classes to coexist within the same isolated group.

The main objective is to provide **container-level temporal isolation** together with **multi-policy hierarchical scheduling**.

The default branch (`main`) contains project documentation and metadata only. The Linux kernel source tree is provided through `master` as the upstream reference and `tgbs` as the current kernel tree with TGBS applied. Release-oriented patch artifacts are maintained in `tgbs-patches`.

## Motivation and Design

Linux control groups provide a way to organize tasks into groups and containers, but the standard CPU controllers do not directly provide the hierarchical temporal isolation model targeted by TGBS.

TGBS associates each task group with a **runtime budget `Q`** and a **period `P`**. Execution of the group is controlled by a deadline server using CBS-style bandwidth accounting.

The implementation provides:

* a **full virtual runqueue** for each TGBS-managed task group;
* support for `FAIR`, `RT`, and `DEADLINE` tasks within the same isolated group;
* common runtime accounting against the task-group reservation;
* hierarchical bandwidth admission control;
* multicore placement and balancing of TGBS virtual runqueues;
* cpuset-aware server activation and task placement;
* optional runtime reclaim through the Linux GRUB mechanism;
* optional temporal isolation of workloads running in the root CPU cgroup.

Scheduling inside a TGBS group continues to rely on the native Linux scheduling classes. TGBS therefore isolates bandwidth at the task-group level without requiring one container per scheduling policy.

## Relation to Previous Work

TGBS builds on the **Hierarchical Constant Bandwidth Server (HCBS)** work originally proposed by **Luca Abeni** and later updated by **Yuri Andriaccio** and collaborators.

The TGBS design originates from the idea of extending hierarchical bandwidth-server scheduling beyond RT-only workloads in order to support multiple native Linux scheduling classes within the same task group.

Parts of the implementation reuse and extend code from the HCBS patches, with attribution preserved in the corresponding patch history.

The main differences include:

* HCBS targets real-time workloads;
* TGBS supports mixed `FAIR`, `RT`, and `DEADLINE` workloads in the same task group;
* TGBS virtualizes a full runqueue rather than attaching a server only to the RT sub-runqueue.

References:
- HCBS (original): https://github.com/lucabe72/LinuxPatches/tree/HCBS
- HCBS (updated): https://github.com/Yurand2000/HCBS-patch/tree/rt-cgroups

## Scientific Publications

TGBS is developed in the context of a PhD thesis.

The following publication presents the design, analysis, and implementation of TGBS:

**Towards Multi-Policy Hierarchical Scheduling in Linux for Containerized Space Applications**
*M. Kooshmanian, J. Ermont, L. Miné, S. Corbin, F. Boniol*
Embedded Real Time Systems Conference (ERTS), 2026
DOI: `10.82331/ERTS.2026.34`

## Repository Organization

### Branches

* `main`
  Project documentation, citation metadata, licensing information, and references.

* `master`
  Linux mainline kernel used as the upstream reference.

* `tgbs`
  Current Linux kernel tree with TGBS applied.

* `tgbs-patches`
  Release-oriented TGBS patch artifacts. Each tagged snapshot contains the ordered patch series together with its README, citation metadata, license, and release notes.

Development branches may exist temporarily and are not part of the stable repository interface.

### Tags

Two tag namespaces are used.

#### Kernel-tree tags

```text
tgbs-vX.Y
```

These tags identify historical snapshots of the Linux kernel tree containing a particular TGBS version.

Examples:

```text
tgbs-v1.0
```

These tags are historical references. Because the development branch may be rebased or otherwise rewritten during development, a kernel-tree tag is not necessarily an ancestor of the current `tgbs` branch.

#### Patch artifact tags

```text
patch/tgbs-vX.Y-kA.B
```

These tags identify the standalone TGBS patch artifact for TGBS version `X.Y` targeting Linux kernel version `A.B`.

Examples:

```text
patch/tgbs-v1.0-k6.17
```

Patch artifact tags are used to create GitHub Releases and are archived through Zenodo.

## Releases

Each GitHub Release corresponds to a specific TGBS patch artifact and Linux kernel base.

Current released artifacts include:

| TGBS version | Linux kernel base | Patch artifact tag      |
| ------------ | ----------------- | ----------------------- |
| v1.0         | v6.17             | `patch/tgbs-v1.0-k6.17` |
| v1.1         | v6.17             | `patch/tgbs-v1.1-k6.17` |
| v1.1         | v6.18             | `patch/tgbs-v1.1-k6.18` |
| v1.2         | v6.18             | `patch/tgbs-v1.2-k6.18` |
| v1.3         | v6.18             | `patch/tgbs-v1.3-k6.18` |
| v1.4         | v6.18             | `patch/tgbs-v1.4-k6.18` |

A release archive contains the patch series and the metadata needed to understand and cite the corresponding artifact.

## Citation

Citation metadata for the project is provided in [`CITATION.cff`](CITATION.cff).

The Zenodo DOI representing **all released versions of TGBS** is:

**https://doi.org/10.5281/zenodo.22108568**

This DOI resolves to the latest archived version.

For reproducibility, when referring to experiments performed with a specific TGBS release, citing the DOI corresponding to that exact archived release is recommended.

## License

TGBS is distributed under the **GNU General Public License version 2 only** (`GPL-2.0-only`).

See [`LICENSE`](LICENSE) for the full license text.

## Use of Generative AI Tools

Generative AI tools were used as assistance for code exploration, refactoring, and documentation.

All design decisions, implementation choices, and validation remain the responsibility of the author.
