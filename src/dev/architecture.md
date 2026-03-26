# Architecture

## Overview
```
Phone browser  ←──── WebSocket (WSS) ────→  Rust daemon (Axum)
                                                     │
                                              WebSocket (localhost)
                                                     │
                                            Browser extension
                                                     │
                                            Netflix content script
                                                     │
                                              Netflix DOM
```

## Components

### Rust daemon (`binge-watch-me`)

Single binary that runs as a macOS menu bar app. Contains:

- **Axum web server** — serves the phone UI and WebSocket endpoints
- **Tray icon** — `NSStatusBar` via `objc2`, runs on main thread
- **Tray delegate** — `TrayDelegate` ObjC class (`define_class!`) handles timer callbacks and menu actions
- **Tokio runtime** — runs on a background thread, owns the async server
- **Update checker** — async Tokio task, checks GitHub Releases every 24h
- **Auth module** — generates and stores a token in the macOS keychain
- **Platform module** — system volume control via CoreAudio
- **Network module** — local IP detection for QR code generation

### Auto-update flow

```
Background Tokio task          Main thread (NSTimer, 5s)
─────────────────────          ─────────────────────────
check GitHub API               read Arc<Mutex<Option<UpdateInfo>>>
  │                              │
  └─ newer version found?        └─ Some(info) → insert menu item
       │
       └─ write UpdateInfo        User clicks menu item
            into Arc               │
                                   └─ NSAlert confirmation
                                        │
                                        └─ spawn std::thread
                                             │
                                             ├─ curl download
                                             ├─ unzip extract
                                             ├─ xattr -cr (strip quarantine)
                                             ├─ write sidecar .sh script
                                             ├─ launch sidecar detached
                                             └─ process::exit(0)
                                                  │
                                             sidecar waits for PID
                                             cp -R new.app → old.app
                                             open new.app
```

The sidecar script is necessary because a running macOS `.app` cannot replace
its own bundle while it is executing. The sidecar is a detached shell process
that waits for the parent PID to disappear, then does the swap and relaunches.

### Browser extension (`binge-watch-me-extension`)

Manifest V3 extension with two scripts:

- **background.js** — service worker, maintains WebSocket to daemon
- **content.js** — injected into Netflix, reads DOM state and executes commands

Site-specific logic lives in `adapters/` — one file per site.

### Phone UI

Static HTML/CSS/JS embedded in the Rust binary via `rust-embed`.
Connects to the daemon via WebSocket, renders `MediaState`, sends `Command`.

## Message protocol

Defined in `src/protocol.rs` (Rust) and `protocol.js` (extension).
```rust
// Commands: phone → daemon → extension
enum Command {
    PlayPause,
    Next,
    SeekForward { seconds: f64 },
    SeekBackward { seconds: f64 },
    VolumeUp,
    VolumeDown,
    SetVolume { level: u8 },
}

// State: extension → daemon → phone
struct MediaState {
    site: String,
    is_playing: bool,
    title: String,
    current_time: f64,
    duration: f64,
    volume: u8,
}
```

## Security

- Token generated on first launch, stored in macOS keychain
- All WebSocket connections require `?token=` query parameter
- `/token` endpoint is unauthenticated but only reachable from localhost
- Phone receives token via QR code on first setup
- Extension fetches token from `/token` on first setup