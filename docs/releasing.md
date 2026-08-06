# Releasing Limux

Releases are cut from a version-bumped, green `main` commit. Publishing a
GitHub release triggers Linux package, RPM, and AUR automation.

## Cut a release

1. Bump `workspace.package.version` in `Cargo.toml` and refresh `Cargo.lock`:

   ```bash
   cargo check -p limux-protocol
   ./scripts/check.sh
   ```

2. Merge the version bump through a pull request and wait for the `main`
   `Rust Quality` run to pass.

3. Publish the release at that exact merge commit:

   ```bash
   version=$(sed -n 's/^version = "\([^"]*\)"/\1/p' Cargo.toml | head -1)
   sha=$(gh api repos/am-will/limux/commits/main --jq .sha)
   gh release create "v$version" \
     --repo am-will/limux \
     --target "$sha" \
     --title "Limux v$version" \
     --generate-notes
   ```

4. Wait for `Build Linux Release Packages` and `Build RPM Package`. A
   successful tagged Linux build triggers `Publish to AUR`.

5. Verify release contains tarball, Debian package, AppImage, and RPM. Confirm
   AUR `limux-bin` reports matching version.

## Rebuild release assets

Both package workflows support manual dispatch. Select release tag as workflow
ref and provide matching version:

```bash
gh workflow run release-linux.yml --ref v0.1.22 -f version=0.1.22
gh workflow run release-rpm.yml --ref v0.1.22 -f version=0.1.22
```

`scripts/validate-release-version.sh` rejects malformed versions and mismatches
between workflow input, package version, and checked-out Cargo workspace.
