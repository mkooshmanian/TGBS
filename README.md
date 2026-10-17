# TGBS v1.5 patch artifact

This standalone artifact contains the TGBS v1.5 patch series for Linux
v6.18. Task Group Bandwidth Server (TGBS) is a Linux scheduler extension
that applies runtime/period reservations to CPU cgroups.

## Version mapping

- Kernel base: Linux `v6.18`
  (`7d0a66e4bb9081d75c82ec4957c50034cb0ea449`)
- Patched kernel tag: `tgbs-v1.5`
  (`9f24d207dffe11ccbfc8d2befd28e6219e6d0bd8`)
- Artifact tag: `patch/tgbs-v1.5-k6.18`

[`patches/`](patches/) contains the 28 ordered `git format-patch` files for
TGBS v1.5 on top of Linux v6.18.

This is a complete series with fixes integrated into the patches that introduced
the affected functionality. It replaces the v1.4 series; do not apply it on top
of an already patched TGBS kernel. When updating a build recipe, replace the
previous series in full.

Apply them to a clean Linux v6.18 checkout with:

```sh
git am /path/to/patches/*.patch
```

See [`RELEASE_NOTES.md`](RELEASE_NOTES.md) for the functionality introduced
by this version and [`CITATION.cff`](CITATION.cff) for citation metadata.
The artifact is distributed under GPL-2.0-only; see [`LICENSE`](LICENSE).
