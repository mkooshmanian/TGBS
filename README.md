# TGBS v1.4 patch artifact

This standalone artifact contains the TGBS v1.4 patch series for Linux
v6.18 LTS. Task Group Bandwidth Server (TGBS) is a Linux scheduler extension
that applies runtime/period reservations to CPU cgroups.

## Version mapping

- Kernel base: Linux `v6.18`
  (`7d0a66e4bb9081d75c82ec4957c50034cb0ea449`)
- Patched kernel tag: `tgbs-v1.4`
  (`713c3ae9db374d9882922037afc8a89bb1ce2cd8`)
- Artifact tag: `patch/tgbs-v1.4-k6.18`

[`patches/`](patches/) contains the 28 ordered `git format-patch` files for
TGBS v1.4 on top of Linux v6.18.

Apply them to a clean Linux v6.18 checkout with:

```sh
git am /path/to/patches/*.patch
```

See [`RELEASE_NOTES.md`](RELEASE_NOTES.md) for the functionality introduced
by this version and [`CITATION.cff`](CITATION.cff) for citation metadata.
The artifact is distributed under GPL-2.0-only; see [`LICENSE`](LICENSE).
