# Task Group Bandwidth Server (TGBS)

## Overview

This repository contains a Linux kernel patch named **Task Group Bandwidth Server (TGBS)**.

TGBS extends the Linux scheduler to provide **temporal isolation between task groups** by associating each task group (CPU cgroup) with a **deadline server**. Each server is a **deadline scheduling entity** and follows the **SCHED_DEADLINE** policy.

The objective is to enable **container-level temporal isolation** while supporting **mixed scheduling classes** inside each container.

The default branch (`main`) is documentation-only. The Linux kernel source tree is provided in `master` (upstream reference) and in `tgbs` (kernel tree with TGBS applied). A patch-series form is available in `tgbs-patches`.

## Motivation and Design

Linux control groups allow grouping tasks into containers, but existing CPU controllers provide limited and hard-to-analyze temporal guarantees.

TGBS addresses this limitation by associating each task group with a **budget (Q) and period (P)** and scheduling it as a **SCHED_DEADLINE server**. Temporal isolation between groups is enforced by the **CBS/GRUB semantics** of the deadline scheduling policy.

This is achieved by:
- creating a **full virtual runqueue** per task group,
- allowing `FAIR`, `RT`, and `DEADLINE` tasks to coexist inside the same container,
- accounting all execution against the group reservation.

As a result, TGBS enables **multi-policy hierarchical scheduling** at the container level.

## Relation to Previous Work

TGBS builds directly on the HCBS work originally proposed by **Luca Abeni**
and later updated by **Yuri Andriaccio**. The design of TGBS originates
from the idea of generalizing HCBS beyond real-time workloads to support
multiple scheduling policies. Parts of the implementation reuse and
extend code from the HCBS patches, with appropriate attribution.

Key differences include:
- HCBS applies only to RT workloads.
- TGBS supports **multi-class workloads** inside a single container.
- TGBS virtualizes a full runqueue instead of attaching a server to the RT sub-runqueue only.

References:
- HCBS (original): https://github.com/lucabe72/LinuxPatches/tree/HCBS
- HCBS (updated): https://github.com/Yurand2000/HCBS-patch/tree/rt-cgroups

## Scientific Publications

This work is conducted in the context of a PhD thesis.

The following publication describes the design, analysis, and implementation of TGBS:

- **Towards Multi-Policy Hierarchical Scheduling in Linux for Containerized Space Applications**
  *M. Kooshmanian, J. Ermont, L. Miné, S. Corbin, F. Boniol*
  European Congress on Embedded Real-Time Systems (ECRTS), 2026
  (DOI to be added)

## Repository Organization

### Branches

- `main`
  Documentation only (this README and related material).

- `master`
  Linux mainline kernel used as the upstream reference.

- `tgbs`
  Linux kernel tree with the latest version of the TGBS patch applied.

- `tgbs-patches`
  Same content as `tgbs`, exported as a series of standalone patches.

### Tags

For each released version `X.Y` of TGBS, two tags are provided:

- `tgbs-vX.Y`
  References the kernel tree containing the TGBS implementation
  (based on `tgbs`, possibly rebased).

- `tgbs-vX.Y-patches`
  References the corresponding patch series
  (based on `tgbs-patches`).

Tags suffixed with `-patches` are intended for patch-based workflows.

## License

TGBS is distributed under the **same license as the Linux kernel**.

Please refer to the license information provided in the `master` branch of this repository and in the upstream Linux kernel sources.

## Use of Generative AI Tools

Generative AI tools were used during the development of this work as
assistance for code exploration, refactoring, and documentation.
All design decisions, implementations, and validations were performed
by the author.
