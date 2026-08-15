# RetailMesh Desktop (Tauri)

A thin Windows desktop shell around https://retailmesh.co.za that adds:

- **System tray** icon (Open / Quit), close-to-tray so it keeps running
- **Native Windows toast notifications** when a ticket is assigned to you —
  even when no browser is open
- **Auto-start on login** (launches minimised to the tray)
- Single-instance: relaunching focuses the existing window

The web app detects it is running inside this shell (`window.__TAURI__`) and
mirrors every in-app notification as a native toast (see
`resources/js/stores/notifications.js` → `notifyDesktop`).

## Building the Windows installer (.msi)

Must be built **on Windows** (Tauri compiles per-platform).

One-time setup on the build machine:

1. Install [Rust](https://rustup.rs) (stable, MSVC toolchain)
2. Install [Node.js](https://nodejs.org) 18+
3. Install the Visual Studio Build Tools (Desktop development with C++)
4. WebView2 runtime (preinstalled on Windows 10/11)

Then:

```powershell
cd desktop
npm install
# One-time: generate icons from a 1024x1024 source PNG
npx tauri icon path\to\retailmesh-icon.png
npm run build
```

The installer lands in `desktop/src-tauri/target/release/bundle/msi/`.

## Rollout

- The build is **unsigned** for the internal pilot: Windows SmartScreen shows
  an "unknown publisher" prompt once — click "More info → Run anyway".
- For a wider rollout, buy a code-signing certificate and configure
  `bundle.windows.certificateThumbprint` in `tauri.conf.json`.
- Install per staff PC; the app auto-starts on login into the tray.
- Staff must log in once inside the app window; the session persists.

## How notifications flow

ticket assigned (auto-routing / manual)
→ `TicketAssignedNotification` (broadcast via Pusher)
→ web app receives it on the user's private channel
→ `notifyDesktop()` sees `window.__TAURI__` and fires a native toast.
