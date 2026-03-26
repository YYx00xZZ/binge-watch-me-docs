# Releasing

## Daemon (`binge-watch-me`)

Releases are fully automated via GitHub Actions. To ship a new version:

1. Update the version in `Cargo.toml`:
```toml
[package]
version = "x.y.z"
```

2. Commit and push:
```bash
git add Cargo.toml
git commit -m "chore: bump version to x.y.z"
git push origin main
```

3. Tag and push:
```bash
git tag vx.y.z
git push origin vx.y.z
```

GitHub Actions will:
- Build `BingeWatchMe.app` for Apple Silicon and Intel
- Zip both bundles
- Create a GitHub Release with auto-generated release notes
- Attach both zips as release assets

Build time is approximately 5–10 minutes (Rust cache helps after first run).

## Extension (`binge-watch-me-extension`)

Same process:

1. Update version in `manifest.json`
2. Commit and push
3. Tag and push

GitHub Actions will:
- Run `npm install` and `npm run build`
- Zip the extension
- Create a GitHub Release with the zip attached

## Versioning

Keep versions in sync between the daemon and extension — use the same
version number for both on each release. This makes it easy to know
which versions are compatible.

## How auto-update picks up new releases

The daemon checks the GitHub Releases API for the repo and compares the latest
release tag (semver, `v`-prefix stripped) against `CARGO_PKG_VERSION`. When a
newer version is found, it signals the tray to show the "Update available" menu
item. Users then download directly from:

```
https://github.com/YYx00xZZ/binge-watch-me/releases/download/v{version}/BingeWatchMe-macos-{arch}.zip
```

For the auto-updater to find the new release the zip assets **must** be named
exactly `BingeWatchMe-macos-arm64.zip` and `BingeWatchMe-macos-x86.zip` — CI
already does this. Do not rename them.

The version in `Cargo.toml` must be bumped before tagging. If the binary version
matches the latest release tag the update menu item will never appear.

## Checklist before releasing

- [ ] Test play/pause on Netflix
- [ ] Test next episode (mid-episode and end-of-episode seamless button)
- [ ] Test volume up/down
- [ ] Test phone connection via QR code
- [ ] Test extension setup (Connect to daemon button)
- [ ] Test auto-update: run the previous release, confirm "Update available" appears and the app relaunches correctly
- [ ] Update version in `Cargo.toml` and `manifest.json`
- [ ] Update `SUMMARY.md` in docs if new features were added
