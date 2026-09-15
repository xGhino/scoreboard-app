# AP Pickle Ground — Scoreboard Control + Overlay

Two static pages, no backend/server required:

- **admin.html** — the control panel you use to update scores, names, colors, logos, etc.
- **overlay.html** — the transparent scoreboard you add to OBS as a Browser Source.

## How the sync works

Two layers, both automatic:

1. **Local layer** (instant): `localStorage` + `BroadcastChannel`. Only
   works when both pages share a browser engine (e.g. both loaded inside
   OBS). Kept because it's free and instant when it applies.
2. **Remote layer** (cross-browser/cross-device): the very first time you
   open `admin.html`, it creates a free, anonymous JSON "board" on
   [jsonblob.com](https://jsonblob.com) and puts that board's id into its
   own URL as `?blob=<id>`. It then shows you two links:
   - **Overlay link** — paste this exact link into OBS's Browser Source.
   - **Admin link** — bookmark/reuse this exact link to control this same
     board from any other browser, tab, or device.

   `overlay.html` reads the `?blob=` id from its own URL and polls that
   board roughly every 1.5 seconds, so it updates no matter which browser
   opened it.

**Important:** the board id lives in the *URL*, not just local storage. If
you open a bare `admin.html` link (no `?blob=`) from a browser that's never
visited this board before, it will create a brand-new, separate board. Always
use the exact links `admin.html` generates for you (bookmark them) rather
than retyping the plain URL.

No account or API key is needed. Since it's a public, anonymous blob store,
technically anyone with your exact `?blob=` link could read or overwrite it —
fine for a casual local match, but don't treat it as private/secure. jsonblob
also removes boards that go unused for 75 days.

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

## 2. First run: pick a court

1. Open `admin.html` (your GitHub Pages link) once, in any browser.
2. Near the top of the page, click **Court 1** (through **Court 4**) to pick
   which scoreboard you're running. Each is a fully independent board.
3. The **Overlay link** and **Admin link** fields update to include that
   court's id (`?board=court1`, etc). Copy both somewhere safe.
4. To run a second/third/fourth court, just click that court's button (or
   open `admin.html?board=court2` directly) — its own links appear the
   same way.

## 3. Wire it into OBS

**Overlay (Browser Source):**
1. In OBS, add a new **Browser Source**.
2. URL: the **Overlay link** from step 2 (not the bare `overlay.html` URL).
3. Width/height: e.g. 1920×200 (it has a transparent background).

**Control panel** — either works, pick what's convenient:
- **From any regular browser, anywhere**: just open the **Admin link** from
  step 2. Since sync goes through jsonblob now, it doesn't need to share a
  browser engine with OBS.
- **Inside OBS as a dock** (optional, if you'd rather keep it all in one
  window): **View → Docks → Custom Browser Docks…**, paste the Admin link.

Avoid having the Admin link open in two places at once controlling the same
board — each save overwrites the board with its own full state, so the last
one to save wins and can clobber the other's edits.

## 4. Test it without OBS

Open the Admin link and Overlay link as two tabs in any regular browser —
same sync path OBS uses, so you can sanity-check everything first. Remote
updates land within ~1.5 seconds (polling), same-browser tabs update
instantly.

## Notes / limits

- Team logos are stored as base64. They ride along in every save to
  jsonblob and in `localStorage` (~5MB/origin budget) — fine for typical
  small PNG/SVG logos, but avoid huge source images or you may hit size
  limits or slow saves.
- The remote board is anonymous and unauthenticated — anyone with your
  exact `?blob=` link can read or write it. Fine for a casual local match;
  don't rely on it for anything sensitive.
- jsonblob removes boards untouched for 75 days.
- If jsonblob is briefly unreachable, the status line under the links says
  so and local/same-browser sync keeps working; it retries on your next change.
- Fixed pre-existing bugs where team name/players, overlay width, opacity,
  and logo size updated the admin's own preview but never actually saved to
  state (so they silently failed to reach the overlay) — and a serve-indicator
  bug where a function parameter shadowed the shared `state` object.
