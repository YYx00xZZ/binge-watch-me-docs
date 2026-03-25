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

## Checklist before releasing

- [ ] Test play/pause on Netflix
- [ ] Test next episode (mid-episode and end-of-episode seamless button)
- [ ] Test volume up/down
- [ ] Test phone connection via QR code
- [ ] Test extension setup (Connect to daemon button)
- [ ] Update version in `Cargo.toml` and `manifest.json`
- [ ] Update `SUMMARY.md` in docs if new features were added
