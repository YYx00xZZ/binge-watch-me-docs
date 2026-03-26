# Known quirks

This page documents the non-obvious decisions and gotchas discovered during
development. Future-you will thank present-you for writing these down.

## macOS tray icon

`tray-icon` crate did not show the icon on macOS 14.5 due to Sonoma's
aggressive menu bar icon hiding. We use `NSStatusBar` directly via `objc2`
instead, which bypasses this behavior.

The tray must run on the main thread (macOS requirement). Tokio runtime runs
on a background thread spawned inside the `tray::run` closure.

## objc2 custom menu actions

Menu item actions require an Objective-C delegate object to receive the selector.
We use `objc2::define_class!` (renamed from `declare_class!` in objc2 0.6.x — do
not use the old name) to define `TrayDelegate`, a `MainThreadOnly` ObjC class.
Each custom `NSMenuItem` has its `target` set to the delegate instance and its
`action` set to a selector the delegate implements.

Inside `define_class!` methods on a `MainThreadOnly` type, call `self.mtm()` to
get a `MainThreadMarker` — it is guaranteed safe there without using `unsafe`.

State shared between the background Tokio task and the main-thread delegate uses
`Arc<Mutex<Option<UpdateInfo>>>` (safe to send across threads) plus
`thread_local!` `RefCell`s for `Retained<NSMenu>` and `Retained<NSStatusItem>`
(main-thread-only ObjC objects that cannot implement `Send`).

## Netflix seek

Setting `video.currentTime` directly triggers Netflix error **M7375**
(DRM violation). Netflix detects programmatic seeks and kills the stream.

Seek must be done via Netflix's own player controls (`[data-uia="control-forward10"]`
and `[data-uia="control-back10"]`). These buttons only exist in the DOM
when the controls are visible.

Current workaround: pause → play sequence causes Netflix to show controls
briefly, giving a window to click the seek buttons. This is jarring UX so
seek buttons are currently removed from the phone UI pending a better solution.

Keyboard events (`KeyboardEvent`) dispatched from JS always have
`isTrusted: false` and Netflix ignores them.

## Netflix next episode

Same DOM visibility issue as seek. Solution: pause → play → click
`[data-uia="control-next"]` within 300ms window.

## Netflix title disappears

Netflix removes the title element from the DOM during playback and only
shows it when you hover. We cache the last known title in the adapter
so it persists after the element is removed.

## Content Security Policy in extension popup

Chrome extensions block inline event handlers (`onclick="..."`) due to CSP.
All event listeners must be attached via `addEventListener` in a separate JS file.

## Extension context invalidated

When the extension is reloaded during development, content scripts running
in open tabs throw `"Extension context invalidated"`. This is caught with
a try/catch that clears the polling interval. Normal users never see this.

## macOS Gatekeeper

Unsigned `.app` bundles downloaded from the internet are quarantined by
macOS. Users must right-click → Shift + Open on first launch. The
`xattr -cr` command removes the quarantine flag if needed.

The auto-updater runs `xattr -cr` on the newly extracted bundle before
launching the sidecar script, so the relaunched app is already trusted.

## self_update stores the wrong download URL

`self_update`'s GitHub backend maps `ReleaseAsset.download_url` to
`asset["url"]` from the GitHub API — an endpoint that returns JSON unless
the request includes `Accept: application/octet-stream`. Passing this URL
to any downloader without that header produces a JSON file, not a zip.

We ignore the field entirely and construct the `browser_download_url`
directly from known parts:
```
https://github.com/{owner}/{repo}/releases/download/v{version}/{asset}
```
This is a stable GitHub pattern that serves the binary with no special headers.

## self_update zip extraction fails for .app bundles

`self_update::Extract::extract_into` iterates every zip entry and calls
`fs::File::create` on each path unconditionally. Directory entries (names
ending with `/`) and symlinks both cause this to fail on macOS with
`IoError: No such file or directory`.

macOS `.app` bundles inside zips contain both. Use `unzip -o file.zip -d dest`
instead — it is always present on macOS and handles all entry types correctly.

## self_update Download TLS fails on GitHub CDN

`self_update::Download` builds a `reqwest::blocking::Client` with
`use_rustls_tls()`, which uses bundled WebPKI roots rather than the macOS
system keychain. GitHub release downloads redirect through
`objects.githubusercontent.com`; the rustls TLS handshake against that CDN
fails intermittently even when the URL opens fine in a browser.

Use `curl -fsSL -o dest url` instead. macOS curl uses Apple's native TLS
stack, reads from the system keychain, and follows redirects correctly.

## CoreAudio volume

System volume is read and written via `AudioObjectGetPropertyData` /
`AudioObjectSetPropertyData` with `kAudioHardwareServiceDeviceProperty_VirtualMainVolume`.
Volume is a float `0.0–1.0` internally, converted to `0–100` for the protocol.

## Token generation

We use `DefaultHasher` with time and PID as entropy sources rather than
pulling in a crypto crate. The token controls Netflix playback on a home
network — the stakes don't justify a `ring` or `rand` dependency. This
can be upgraded later if needed.
