# GoBrowserOS

**GoBrowser** is a Chromium-based desktop browser that boots into **GoBrowserOS** — a complete desktop operating system shell built on Electron's Chromium runtime.

```
npm install
npm start        # boot GoBrowserOS (Electron desktop app)
npm run kiosk    # boot straight into full screen (F11 toggles fullscreen any time)
npm run smoke    # automated boot + feature smoke test (Electron)
npm run lint     # syntax-check every source file
npm run test:usb # live USB plug/unplug feature test (uses removable drive if present)
npm run test:settings # settings sections + wallpaper checks (Electron)
npm run test:radios   # Wi-Fi / Bluetooth / Battery / Drivers APIs + settings pages (Electron)
npm run test:password # set password -> lock -> wrong password rejected -> Enter unlocks -> clear (Electron)
npm run native        # (re)build the WinRT/WMI helper native/win32/gosradio.exe

npm run ext          # build the Chrome/Edge extension into extension/dist
npm run ext:smoke    # headless Edge smoke test for the extension build

npm run dist     # build Windows installer + portable exe (release/)
npm run dist:portable   # portable exe only (run it straight from a USB stick)
npm run iso          # release/GoBrowserOS-1.0.0.iso disc image with the portable build

npm run ad           # render the 2:30 promo MP4 (video + soundtrack) -> release/
npm run ad:trailer   # render the 45s cinematic trailer -> release/GoBrowserOS-Trailer.mp4
npm run ad:trailer2  # render the 45s cool & calm trailer -> release/GoBrowserOS-Trailer-2.mp4
npm run ad:stills    # render a few key frames to smoke-output/ad-stills/ for review
npm run yt           # render YouTube avatar/banner/thumbnail -> release/youtube/
npm run site         # serve the website locally -> http://localhost:8080/
```

## What you get

**GoBrowser (the browser)**
- Chromium-rendered tabs with favicons, drag-reorder, loading states
- Omnibox with URL detection, search fallback (DuckDuckGo/Google/Bing) and live suggestions from history + bookmarks
- Bookmarks bar + manager, browsing history, downloads (real files → `Home/Downloads` with progress)
- Find-in-page, zoom, per-site context menus, popups open as tabs, view-source, devtools

**GoBrowserOS (the desktop)**
- BIOS + boot animation + lock/login screen + power states (sleep, restart, shut down, power-off screen)
- Full screen like a real OS: starts maximized, `F11` toggles true fullscreen anywhere (including inside browser tabs), `npm run kiosk` boots straight into it
- Desktop with wallpaper engine (8 built-ins + custom images), icons for apps and `Home/Desktop` files, context menus
- Window manager: drag, 8-way resize, snap (halves/quarters/maximize), z-order, minimize/restore, taskbar sync
- Start menu with app search, quick settings (network, Bluetooth, dark mode, brightness, sounds, battery), notification center, clock
- Taskbar with pinned apps, running windows, show-desktop, power menu
- Real connectivity: Wi-Fi scan/connect/WPA2-WPA3 profiles via `netsh`, Bluetooth discovery/pairing via WinRT, battery status via WMI, and a signed-driver inventory (netsh drivers + `Win32_PnPSignedDriver`) — all behind `gos.wifi` / `gos.bt` / `gos.battery` / `gos.drivers`

**Apps (all functional)**
| App | What it does |
| --- | --- |
| GoBrowser | Full Chromium browsing (above) |
| Files | Real filesystem: drives sidebar, breadcrumbs, rename/delete (Recycle Bin), copy/paste, new folder/file, properties, drag & drop, live refresh |
| Terminal | Built-in commands (`cd`, `ls`, `cat`, `mkdir`, `grep`, `sysinfo`…) plus passthrough to the real system shell with streaming output, history and tab completion |
| Text Editor | Multi-tab editing, line numbers, save/save-as via native dialogs, find, unsaved-changes guard |
| Settings | Wi-Fi (scan, connect, saved profiles, radio), Bluetooth (scan, pair/unpair, radio), Battery (charge, health, runtime), Drivers (adapter + signed-driver inventory, Windows Update), wallpaper, theme, accent, brightness, scale, sounds, USB & portable drives, users + password, app management, browser defaults, shortcuts, about, factory reset |
| Calculator | Full keyboard-driven calculator with history |
| Notes | Auto-saving notes app with search |
| Viewer | Images + audio/video with zoom, next/prev, set-as-wallpaper |
| Task Manager | Live CPU/memory graphs from real Electron process metrics, window list with End task |
| GoBrowse Studios Store | Installs extra apps (Paint, Snake, Clock) |
| Paint | Canvas painter with brush/eraser, open + save PNG |
| Snake | Classic snake with persisted high score |
| Clock | Stopwatch, countdown timer with notification, world clock |
| Web Apps | Launchers for Photopea, Polotno, Spotify, Excalidraw, OpenStreetMap, Wikipedia (embedded, or a real tab when the site forbids framing) |

## GoBrowser as a browser-based OS (Chrome/Edge extension)

The same desktop ships as a **Manifest V3 WebExtension** — every new tab boots
GoBrowserOS inside the browser itself.

```bash
npm run ext         # assembles extension/dist (renderer + bridge + manifest + icons)
```

Then load it:

1. Edge: `edge://extensions` (Chrome: `chrome://extensions`)
2. Enable **Developer mode**
3. **Load unpacked** → select `GoBrowserOS/extension/dist`
4. Open a new tab (or click the GoBrowserOS toolbar button) → the desktop boots

Smoke test it headlessly: `npm run ext:smoke` (drives Edge over CDP, 12 checks + screenshot).

**How it works**

| Electron | Extension |
| --- | --- |
| `preload.js` exposes `window.gos` | `extension/bridge.js` polyfills the exact same surface |
| `main/store.js` → `gos-state.json` | `chrome.storage.local` (same deep-merged defaults) |
| Real filesystem + native dialogs | Virtual Windows-style FS (`C:\GoBrowserOS\Home`) + File System Access pickers with write-through |
| `<webview>` guests (site isolation) | `FrameGuest` iframe adapter (same event/method API: history, zoom, titles) |
| Real shell commands | Built-in mini shell (`dir`, `type`, `cd`, `ver`, `open <url>` …) |
| — | Sites that refuse framing (Google, YouTube, GitHub…) auto-open as **real browser tabs** with an in-OS placeholder |
| — | **Web Apps** launcher: Photopea, Polotno, Spotify, Excalidraw, OpenStreetMap, Wikipedia |

**Known trade-offs of the extension build** (inherent to iframes): cross-origin
pages expose no title/favicon (host name is used), in-page find and context
menus are unavailable inside embedded pages, and `file://` preview items are
hidden in Files/Viewer.

## Portable USB edition (plug in / plug out)

`npm run dist:portable` produces a single self-contained exe
(`release/GoBrowserOS 1.0.0.exe`, ~96 MB) that runs the full OS with **no
installation**. Run it from a USB stick and it becomes portable:

- **Data lives on the stick** — when the exe runs from a removable drive (or
  from `PORTABLE_EXECUTABLE_DIR`, set by the portable launcher), all OS state
  (`gos-state.json`) and the entire `Home` tree are kept in a `GoBrowserOS`
  folder next to the exe. Installed runs still use the normal app-data folder.
- **Plug in** — removable drives are polled every 2.5 s; a toast announces the
  drive (and whether it carries portable GoBrowserOS data), Files refreshes its
  sidebar instantly, and re-plugging the data stick reloads your session from it.
- **Plug out** — if the portable data drive disappears the OS pauses writes,
  saves state and shows a full-screen "USB REMOVED" layer; plug the stick back
  and it resumes (or "Continue without USB" switches to local storage).
- **One-click eject** — the taskbar Quick Settings panel lists every USB drive
  with free/total space and an **Eject** button; a USB tray icon (blinking red
  while unplugged) jumps straight to USB settings. Eject flushes state, asks
  Windows to eject and verifies the drive is actually gone before reporting
  "safe to remove".
- **Wipe GoBrowserOS data** — Settings › USB & Portable deletes **only** the
  `GoBrowserOS` folder on the chosen stick (state + Home files). Every other
  file on the drive is untouched; system drives are hard-rejected by the
  guard rails (`npm run test:usb` live-tests all of this against a real drive,
  `npm run test:usb-ui` checks the panel + tray UI).

## Disc image

`npm run iso` builds `release/GoBrowserOS-1.0.0.iso` (ISO9660 + Joliet + UDF,
volume label `GOBROWSEROS`) containing the portable exe, `autorun.inf` and a
`README.txt`. It is a **data disc** — mount it in Windows or burn it to
CD/DVD to ship the portable app; it is not a bootable OS image.

## Promo video

`npm run ad` builds a **2 minute 30 second** 1080p H.264 + AAC promo
(`release/GoBrowserOS-Ad.mp4`, mirrored to `renderer/media/`). The visuals are a
deterministic canvas animation seeked frame-by-frame by `scripts/make-ad.js`
(30fps, ~10 frames per IPC batch) and piped into ffmpeg's libx264 encoder in
five 30-second segments that are concatenated; the soundtrack is synthesised
pure-Node from the page's `__SFX` timeline (`scripts/ad-audio.js`) and muxed
in as a 160kbps AAC track. Scenes: BIOS boot → desktop tour → GoBrowser tabs →
app suite → portable USB plug/eject → portable-anywhere (extension, ISO, exe)
→ call to action, with UI-style sound effects (powerup, chimes, typing, USB
insert/remove, chord stinger).

Inside the OS the promo is reachable three ways: a `GoBrowserOS-Ad.mp4` icon on
first-boot Desktop (opens in Viewer), a **Watch promo** button in Settings →
About, and a **Watch the GoBrowserOS promo** card on the browser new-tab page.
Local media is served through the `gos://app/local/?p=<path>` protocol route so
video/audio plays in the Viewer (Electron rejects `file://` media from `gos://`
pages). In the extension build these entry points are hidden since the media
file is not bundled.

## Trailer

`npm run ad:trailer` renders a **45-second cinematic trailer**
(`release/GoBrowserOS-Trailer.mp4`, 1080p H.264 + AAC, ~3.6MB). Same pipeline
as the promo (`scripts/ad-trailer.html` → `make-ad.js` → `ad-audio.js` →
`build-ad.js --trailer`) but cut for pace: hard cuts with light flashes,
letterbox bars, kinetic slam titles, and a rapid app montage. Beats: logo cold
open → “FULL DESKTOP. IN YOUR BROWSER.” → desktop → “14 APPS. ZERO INSTALLS.”
→ browser → USB plug/boot → “RUNS ANYWHERE.” → Terminal/Editor/Snake montage
→ logo finale.

`npm run ad:trailer2` renders the **45-second cool & calm trailer**
(`release/GoBrowserOS-Trailer-2.mp4`, 1080p H.264 + AAC, ~4.2MB) from
`scripts/ad-trailer2.html`: same pipeline but slow crossfades instead of hard
cuts, eased text rises instead of slam titles, a sky-blue palette, a higher
ambient pad / quieter SFX mix, and a beat for the Wi-Fi / Bluetooth / Battery /
Drivers support. Beats: logo cold open → “YOUR DESKTOP. IN YOUR BROWSER.” →
desktop → “STAY CONNECTED.” feature cards → USB plug/boot → logo finale with
feature chips and a GoBrowse Studios credit.

## Website

`npm run site` serves the repo root and opens the marketing site at
`http://localhost:8080/` (`website/index.html`, plain static HTML/CSS/JS, no
build step). It covers the feature grid, connectivity (Wi-Fi / Bluetooth /
Battery / Drivers), the 14-app roster, both trailers + the USB walkthrough as
inline video, and download cards that link to the `release/` installer,
portable exe and ISO — the tiny server (`scripts/serve-site.js`, zero
dependencies, HTTP range support for video seeking) exists so those
`../release/` links resolve. The site also works opened straight from disk
(`website/index.html`), except the download links need the files next to it.
Assets (logo, screenshots, trailers, posters) are copied into
`website/assets/`, so the folder is self-contained for deployment.

## Architecture

- **`main/`** — Electron main process: `gos://` custom protocol, hardened `<webview>` guests, filesystem/dialog/shell IPC, downloads, window lifecycle, `usb.js` portable-mode + plug in/out watcher
- **`preload.js`** — minimal `contextBridge` surface (`window.gos`)
- **`renderer/`** — the OS itself (ES modules): window manager, boot/login, taskbar, apps
- **`extension/`** — MV3 WebExtension shell (`manifest.json`, `os.html`, `bridge.js`, `background.js`) that boots the same renderer in a tab; built to `extension/dist` by `scripts/build-extension.js`
- **`scripts/`** — lint, smoke tests, portable/ISO packaging (`build-iso.js` via IMAPI2)
- State persists in `userData/gos-state.json` (or `<usb>/GoBrowserOS/Data/` when portable); user files in `Home/`

Guest web pages run in their own Chromium processes (site isolation), so GoBrowser is a real multi-process Chromium browser.

## Security notes

- `contextIsolation`, `sandbox`, no `nodeIntegration` anywhere; guests are hardened on attach and get a Chrome UA
- Destructive file ops use the OS Recycle Bin (`shell.trashItem`)
- Permission requests from web content are denied except sanitized clipboard writes
