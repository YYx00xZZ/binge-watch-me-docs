# Building from source

## Requirements

- Rust stable (`rustup install stable`)
- `cargo-bundle` (`cargo install cargo-bundle`)
- Node.js 20+ (for the extension)

## Daemon
```bash
# Clone
git clone https://github.com/YYx00xZZ/binge-watch-me
cd binge-watch-me

# Development build (fast, unoptimized)
cargo build

# Run in development
cargo run

# Release build
cargo build --release

# Build .app bundle
cargo bundle --release
# Output: target/release/bundle/osx/BingeWatchMe.app
```

## Extension
```bash
# Clone
git clone https://github.com/YYx00xZZ/binge-watch-me-extension
cd binge-watch-me-extension

# Install dependencies
npm install

# Build once
npm run build

# Watch mode — rebuilds on save
npm run watch
```

After building, load the extension in `brave://extensions` → Developer mode →
Load unpacked → select the extension folder.

## Running both together

1. `cargo run` in the daemon repo — server starts on port 7777
2. Reload the extension in `brave://extensions`
3. Click the extension icon → Connect to daemon
4. Open `http://127.0.0.1:7777/setup` to see the QR code
