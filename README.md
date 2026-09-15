# AP Pickle Ground — Scoreboard Control + Overlay

Two static pages, no backend/server required:

- **admin.html** — the control panel you use to update scores, names, colors, logos, etc.
- **overlay.html** — the transparent scoreboard you add to OBS as a Browser Source.

## How the sync works

The old version called a `/data` endpoint on a local server that doesn't exist
in a static GitHub Pages site. That's been replaced with a **client-side-only**
sync: `admin.html` writes the current state to `localStorage` and broadcasts
it instantly over the `BroadcastChannel` API; `overlay.html` listens for that
broadcast (plus a `storage` event and a light poll as safety nets) and
re-renders.

**This only works when both pages are loaded from the same origin inside the
same browser engine.** In practice, that means: don't open the admin panel in
your everyday Chrome and expect it to reach an overlay loaded inside OBS —
OBS's browser source runs its own separate embedded browser, with its own
separate storage. See setup below for the correct way to do this.

## 1. Publish to GitHub Pages

1. Create a new GitHub repo and push these two files (`admin.html`,
   `overlay.html`) to it.
2. In the repo: **Settings → Pages → Source → Deploy from branch**, pick
   `main` and `/ (root)`, save.
3. GitHub will give you a URL like:
   `https://yourusername.github.io/your-repo-name/`
4. Your pages will be at:
   - `https://yourusername.github.io/your-repo-name/admin.html`
   - `https://yourusername.github.io/your-repo-name/overlay.html`

## 2. Wire it into OBS (same browser engine = sync works)

**Overlay (Browser Source):**
1. In OBS, add a new **Browser Source**.
2. URL: your `overlay.html` GitHub Pages link.
3. Width/height: e.g. 1920×200 (it has a transparent background).
4. Check **"Refresh browser when scene becomes active"** (optional, harmless).

**Control panel (Custom Browser Dock — this is the key step):**
1. In OBS menu: **View → Docks → Custom Browser Docks…**
2. Give it a name (e.g. "Scoreboard Control") and paste your `admin.html`
   GitHub Pages link.
3. Click Apply/Close — a new dock appears inside OBS that you can drag
   anywhere in the OBS window, or pop out as its own window.

Because both the overlay's Browser Source and the admin dock are rendered by
OBS's own embedded browser (not your regular Chrome), they share the same
`localStorage`/`BroadcastChannel` origin, so updates in the dock hit the
overlay instantly.

If you'd rather run the control panel outside OBS (e.g. on a second monitor
in real Chrome) while the overlay is inside OBS, you'll need a real backend
(Firebase Realtime DB, a small WebSocket relay, etc.) to bridge the two
processes — the localStorage approach can't cross that boundary. Say the
word if you want that version instead.

## 3. Test it without OBS

Just open `admin.html` and `overlay.html` as two tabs in the same regular
browser (e.g. two Chrome tabs pointed at your GitHub Pages URLs) — the sync
works the same way there, so you can sanity-check everything before wiring
it into OBS.

## Notes / limits

- Team logos are stored as base64 in `localStorage`, which has roughly a
  5MB-per-origin budget shared across both team logos and the rest of the
  state — fine for typical small PNG/SVG logos, but avoid huge source images.
- State persists across reloads (the admin dock remembers your last match
  when OBS restarts).
- Fixed a pre-existing bug in the serve-indicator logic (`setService`) where
  a function parameter was accidentally shadowing the shared `state` object,
  which would have silently broken saving the serve indicator.
