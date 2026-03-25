# Known quirks

This page documents the non-obvious decisions and gotchas discovered during
development. Future-you will thank present-you for writing these down.

## macOS tray icon

`tray-icon` crate did not show the icon on macOS 14.5 due to Sonoma's
aggressive menu bar icon hiding. We use `NSStatusBar` directly via `objc2`
instead, which bypasses this behavior.

The tray must run on the main thread (macOS requirement). Tokio runtime runs
on a background thread spawned inside the `tray::run` closure.

Menu item actions using Objective-C selectors (`objc2::sel!`) require a
proper delegate to handle custom actions. Currently "Quit" uses
`terminate:` which works natively. Custom actions (like "Show QR") are not
yet implemented via the menu — the setup page opens automatically on launch
instead.

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

## CoreAudio volume

System volume is read and written via `AudioObjectGetPropertyData` /
`AudioObjectSetPropertyData` with `kAudioHardwareServiceDeviceProperty_VirtualMainVolume`.
Volume is a float `0.0–1.0` internally, converted to `0–100` for the protocol.

## Token generation

We use `DefaultHasher` with time and PID as entropy sources rather than
pulling in a crypto crate. The token controls Netflix playback on a home
network — the stakes don't justify a `ring` or `rand` dependency. This
can be upgraded later if needed.
