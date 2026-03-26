# Installation

## Requirements

- macOS (Apple Silicon or Intel)
- Brave or Chrome browser

## Download

Go to the [latest release](https://github.com/YYx00xZZ/binge-watch-me/releases/latest)
and download the correct file for your Mac:

- `BingeWatchMe-macos-arm64.zip` — Apple Silicon (M1/M2/M3/M4)
- `BingeWatchMe-macos-x86.zip` — Intel

## Install

1. Unzip the downloaded file
2. Drag `BingeWatchMe.app` to your `/Applications` folder

## First launch

Because the app is not signed with an Apple Developer certificate, macOS will
block it on first launch. To open it:

1. Right-click `BingeWatchMe.app`
2. Hold `Shift` and click `Open`
3. Click `Open` in the dialog

You only need to do this once. After that it launches normally.

When the app starts you will see `BWM` in your menu bar and a setup page
will open in your browser automatically.

## Updates

The app checks for updates automatically in the background. When a new version
is available you will see **Update available vX.Y.Z** in the `BWM` menu bar menu.

1. Click `BWM` in the menu bar
2. Click **Update available vX.Y.Z**
3. Click **Install and Restart** in the confirmation dialog
4. The app downloads the update and restarts automatically

You do not need to visit GitHub or download anything manually.

> **Note:** Because the downloaded bundle is not signed, macOS strips the
> quarantine flag automatically before relaunching. If the updated app is
> blocked on first launch after an update, run `xattr -cr /Applications/BingeWatchMe.app`
> in Terminal.
