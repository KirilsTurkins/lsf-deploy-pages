# Source-verified dependency repairs

This action keeps the pinned runtime dependency graph and backports two fixes:

- `glob` 10.5.0: the four return-after-abort changes and the `onResume`
  change from upstream commit `4682e8c9babee1b385202ede36389634e12076d7`
  for GHSA-3g3m-2fjr-6pw4.
- `minimatch` 9.0.9: the two optimization limits from upstream commit
  `77a0f5c4b80bdc641c3c930a1d9edf5ad2db57f8` for
  GHSA-g3hv-75wg-m249.

The local `deepmerge` 4.3.1 build dependency retains the existing bounded
prototype-key repair for GHSA-gcqx-w48m-w2h4. This is a local repair, not an
upstream fixed release.

`npm ci` runs `prepare`, which first verifies the exact root manifest and lock,
reconstructs all three repaired archives from the pinned npm originals and
recipes, and checks every installed member and the relevant Node resolution
paths. Only the six expected `glob` and `minimatch` files may be changed in
place. All checks finish before any such write. The installed `deepmerge`
package must already equal its repaired local archive. The process is
idempotent and rejects unexpected files or bytes. `npm test` repeats the
checks; run `npm run test:security-source` for focused regression controls.

The normal lock entries for `glob` and `minimatch` still name the original
registry versions, so a version-only advisory scanner will continue to report
them. Any scanner recognition of these repairs requires a separate exact
source and bundle verification; an unknown version, changed advisory or drift
must remain reportable. No exception or scanner change is included here.

The runtime consumer `archiver-utils.file.expand` uses this `glob` copy.
`archiver.glob` uses a different `readdir-glob` dependency; the focused tests
exercise both paths. The tests do not establish a successful GitHub Pages
deployment or every supported platform.
