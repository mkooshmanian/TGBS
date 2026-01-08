# TGBS v1.1 patch artifact

This standalone artifact contains the TGBS v1.1 patch series for Linux
v6.17. Task Group Bandwidth Server (TGBS) is a Linux scheduler extension
that applies runtime/period reservations to CPU cgroups.

## Version mapping

- Kernel base: Linux `v6.17`
  (`e5f0a698b34ed76002dc5cff3804a61c80233a7a`)
- Patched kernel tag: `tgbs-v1.1`
  (`c7bd0401bcfae8c607f33e6ce6fe315ee145b823`)
- Artifact tag: `patch/tgbs-v1.1-k6.17`

[`patches/`](patches/) contains the 22 ordered `git format-patch` files for
the range `v6.17..tgbs-v1.1`.

Apply them to a clean Linux v6.17 checkout with:

```sh
git am /path/to/patches/*.patch
```

See [`RELEASE_NOTES.md`](RELEASE_NOTES.md) for the functionality introduced
by this version and [`CITATION.cff`](CITATION.cff) for citation metadata.
The artifact is distributed under GPL-2.0-only; see [`LICENSE`](LICENSE).
