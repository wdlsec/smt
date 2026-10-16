# WDLSS Student Movement Tracker — GitHub Pages Wrapper

This repository is the front door to the tracker. It is a thin page that holds the Google Apps Script web app in a full-screen iframe, so the tracker can be added to a phone's home screen and used like an app.

The tracker itself (data, logic, maintenance mode, the kill switch) lives in Google Apps Script. Nothing in this repository stores or sees student data.

Live address: `https://wdlsec.github.io/smt/`

## Files

| File | Purpose |
|---|---|
| `index.html` | The wrapper page. Loads the Apps Script app in an iframe and talks to it with `postMessage`. |
| `manifest.json` | Makes "Add to Home Screen" install the tracker as a standalone app on Android. |
| `apple-touch-icon.png` | iPhone / iPad home-screen icon (180×180, full bleed; iOS rounds the corners). |
| `favicon-32.png` | Browser-tab icon (32×32). |
| `icon-192.png`, `icon-512.png` | Android / desktop install icons (`purpose: any`, rounded corners). |
| `icon-maskable-512.png` | Android adaptive icon (`purpose: maskable`): full bleed, artwork kept inside the central safe circle so any mask shape only trims background. |
| `icon.png` | Copy of `icon-512.png`, kept so old links and bookmarks to it still work. |
| `icon.svg` | Source artwork for all the PNGs above. |

In the Apps Script project, `index.html` is kept as `Github_index.html` and `manifest.json` as `Github_manifest.json` so all code sits in one place. Rename them when copying here.

The icon shows the loading-screen student (same uniform colours) stepping out of a classroom doorway, past its open door and a desk, under an amber "out" arrow. `icon.svg` is the source artwork, kept in the repo so the PNGs can be re-exported if it ever needs editing.

## What the wrapper does

- **Loads the app** from the Apps Script `/exec` URL (the `baseUrl` constant in `index.html`) with `embed=1` added, which tells the server the request came through this page. Opened without the wrapper, the server returns a bare "Page Unavailable" notice.
- **Carries the admin access token** (`access=`) during maintenance. The token is kept in `sessionStorage` only, so it survives a refresh and disappears when the tab or home-screen app closes. Neither parameter ever shows in the address bar.
- **Answers the wrapper check.** The app pings the wrapper (`WRAPPER_PING`), and the wrapper replies from its own origin (`WRAPPER_PONG`), which another site cannot fake.
- **Keeps the bars one colour.** In a browser tab the status bar and Safari's bars are a fixed `#003380` in both themes (`theme-color`, the page background and the manifest), and the wrapper passes the safe-area inset into the iframe, where it cannot be measured. The app pads its navbar with it, and lifts its form buttons and bulk-action bar by `min(inset, 34px)`: a phone with a notch or Dynamic Island has a 34px home bar. Some iOS 26 home-screen installs give the page one status bar less than the screen (WebKit bug 301994), and anything `position: fixed` stops short of the bottom; the wrapper measures the shortfall, makes the page that much taller and places the iframe in the page instead, so the app runs to the bottom edge (window scrolling is held at the top). Behind it, the home-screen page is the app's own background (`#f7fafc` light, `#0F141A` dark). The loading screen stays navy.
- **Shows a loading screen.** A walking student and "Loading tracker..." show while the iframe stays invisible, until the app reports it has painted (`APP_PAINTED`). After 12 seconds a notice and a Reload button appear; after 25 seconds the wrapper retries once by itself.
- **Checks the app on return.** Coming back to the tab or home-screen app, the wrapper asks the app to answer (`WRAPPER_HEALTH` / `APP_ALIVE`) within 3 seconds, and reloads it behind the loading screen if it does not.
- **Keeps the address bar in step** with the page and tab the user is on (`UPDATE_URL`), so links can be shared.
- **Sends device details** (home-screen vs browser, OS version, phone model) to the app. These appear in lockout and kill-switch alert emails.

## Useful links

| Link | Opens |
|---|---|
| `https://wdlsec.github.io/smt/` | The dashboard |
| `https://wdlsec.github.io/smt/?page=form` | An empty Submit Form |
| `https://wdlsec.github.io/smt/?page=form&reason=...&location=...&teacher=...` | The form with those fields pre-filled |

## Installing on a phone

- **iPhone (Safari):** open the link, tap Share, then Add to Home Screen.
- **Android (Chrome):** open the link, tap the ⋮ menu, then Add to Home screen or Install app.

## Making changes

- **New Apps Script deployment URL:** update `baseUrl` in `index.html`.
- **New GitHub Pages address:** update `Links!B3` in the spreadsheet as well. The app only trusts messages from that origin.
- **Bar colour:** change `#003380` in all four places together: the `theme-color` meta tag and the `html, body` background in `index.html`, `theme_color` / `background_color` in `manifest.json`, and `--band` in the app's `CSS_Themes` (the strip the app paints behind the status bar).
- **Measuring layout on a phone:** open the site in Safari with `?debug=1` on the address to show the viewport measurements over the page.
- **Caching:** GitHub Pages caches files for about 10 minutes. After an edit, wait, or remove and re-add the home-screen icon.

## Changing the icon

1. Replace all six PNGs with the new artwork at the same names and sizes. Keep text out of the design: at tab and home-screen sizes it becomes unreadable stripes.
2. For `icon-maskable-512.png`, fill the whole square and keep everything important inside the central circle (80% of the width), or Android will crop it.
3. Bump `?v=2` to `?v=3` on every icon link and the manifest link in `index.html`, and on each `src` in `manifest.json`, so browsers fetch the new files.
4. Phones keep the old home-screen icon: remove the shortcut and add it again.

## When the tracker is shut down

Maintenance mode and the kill switch are controlled entirely from the spreadsheet and Apps Script. No change to this repository is needed or possible. During a kill-switch lockdown, the wrapper still loads but the app inside shows only "Page Unavailable". See the Google Sheet README for how to lift it.
