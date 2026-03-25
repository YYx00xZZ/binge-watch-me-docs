# Troubleshooting

## BWM does not appear in the menu bar

Make sure you opened the app using right-click → Shift + Open. If your menu
bar is full some icons may be hidden — try closing other menu bar apps to
make space.

## Extension shows a `!` badge

The extension cannot reach the daemon. Make sure `BingeWatchMe.app` is running,
then click the extension icon and press **Connect to daemon** again.

## Phone shows "Open this page by scanning the QR code"

You opened the remote URL without a token. Use the QR code from the setup page
to connect — it includes the token automatically. Bookmark the URL after
scanning so you don't need to scan again.

## Next episode does not work

Make sure you are on a Netflix watch page (not the browse page) and that the
extension is connected (no `!` badge on the extension icon).

## Setup page does not open on launch

The setup page opens automatically 500ms after the daemon starts. If it does
not appear, open it manually at `http://127.0.0.1:7777/setup`.

## The app says it is damaged

You need to remove the quarantine flag. Run this in Terminal:
```bash
xattr -cr /Applications/BingeWatchMe.app
```

Then try opening it again.
