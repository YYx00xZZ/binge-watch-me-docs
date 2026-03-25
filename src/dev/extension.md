# Extension development

## Structure
```
binge-watch-me-extension/
├── manifest.json       — extension manifest (MV3)
├── background.js       — service worker, WebSocket to daemon
├── content.js          — injected into streaming sites
├── protocol.js         — shared message types
├── setup.html          — extension popup (connect to daemon)
├── setup.js            — popup logic
├── adapters/
│   ├── index.js        — registers all adapters
│   └── netflix.js      — Netflix-specific implementation
├── icons/              — extension icons
└── dist/               — bundled output (generated, not committed)
```

## Adding a site adapter

Create `adapters/yoursite.js`:
```js
const yourSiteAdapter = {
  matches: "*://*.yoursite.com/*",

  getState() {
    const video = document.querySelector("video");
    if (!video) return null;
    return {
      site:         "yoursite",
      is_playing:   !video.paused,
      title:        document.title,
      current_time: video.currentTime,
      duration:     video.duration,
      volume:       Math.round(video.volume * 100),
    };
  },

  execute(cmd) {
    const video = document.querySelector("video");
    if (!video) return;
    switch (cmd.action) {
      case "play_pause": video.paused ? video.play() : video.pause(); break;
      case "seek_forward":  video.currentTime += cmd.seconds ?? 10; break;
      case "seek_backward": video.currentTime -= cmd.seconds ?? 10; break;
      case "set_volume": video.volume = (cmd.level ?? 100) / 100; break;
    }
  },
};
```

Then:
1. Add it to `adapters/index.js`
2. Add its match pattern to `manifest.json` under `host_permissions` and `content_scripts`
3. Run `npm run build`

## Why content scripts can't use ES modules

Chrome content scripts don't support ES module imports directly. We use
`esbuild` to bundle `content.js` and all its imports into a single
`dist/content.js` file. The background service worker supports modules
natively (via `"type": "module"` in manifest.json).

## Debugging

- **Background script**: `brave://extensions` → binge-watch-me → Service Worker
- **Content script**: Open DevTools on the Netflix tab → Console
- **Popup**: Right-click the extension icon → Inspect popup
