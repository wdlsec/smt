# WDLSS Student Movement Tracker — GitHub Pages Wrapper

This repository is the front door to the tracker. It is a thin page that holds the Google Apps Script web app in a full-screen iframe, so the tracker can be added to a phone's home screen and used like an app.

The tracker itself (data, logic, staff sign-in, maintenance mode, the kill switch) lives in Google Apps Script. Nothing in this repository stores or sees student data.

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

## Staff sign-in

Staff sign in with their ICON email: the tracker emails a 6-digit One-Time Password (OTP), and the OTP is typed on the sign-in page. There are no passwords. The sign-in pages are served by Apps Script inside the iframe; the wrapper only keeps what the browser needs between visits, in `localStorage`:

| Key | Holds |
|---|---|
| `smt-token` | The sign-in token the server issued. Renewed by the app once a day; removed on sign-out. |
| `smt-device` | This browser's device ID. Kept after signing out, as it gives no access on its own. |
| `smt-signin` | An OTP request in progress (request ID, email, when the OTP was asked for, whether the person has gone Back to the email screen, and the page asked for), for 10 minutes. Switching to the Mail app and back, or iOS reloading the app, returns to the OTP screen (or to the email screen after Back) with the resend countdown where it was, and Send OTP for the same email within the minute goes back to the OTP already sent instead of asking for another. |

None of these ever shows in the address bar. Every message carrying them is exchanged only with the Apps Script frame, and the app only ever sends them to this page's own origin (`Links` tab in the spreadsheet), never to `*`, so a copy of this page at another address receives nothing.

How long a sign-in lasts, how many devices each person may have, who may sign in and who is an admin are all set in the spreadsheet (Admin and Staff List tabs). Admins are the spreadsheet's editors and owner.

## What the wrapper does

- **Loads the app** from the Apps Script `/exec` URL (the `baseUrl` constant in `index.html`) with `embed=1` and the stored token and device ID added. Opened without the wrapper, the server returns a bare "Page Unavailable" notice.
- **Carries the sign-in hand-offs.** The app tells the wrapper when an OTP was requested (`SIGNIN_REQUESTED`, with when), when the person went Back to the email screen (`SIGNIN_BACK`, the request is kept), when a request can no longer be used (`SIGNIN_CANCELLED`), completed (`SIGNIN_DONE`, with the token and device ID), renewed (`TOKEN_RENEWED`) or ended (`SIGNED_OUT`), and the wrapper stores or drops what it keeps and reloads the frame where needed. `LOCKDOWN` (kill switch) drops the token; `RELOAD` reloads the frame on the same page.
- **Answers the wrapper check.** The app pings the wrapper (`WRAPPER_PING`), and the wrapper replies from its own origin (`WRAPPER_PONG`), which another site cannot fake.
- **Keeps the bars one colour.** In a browser tab the status bar and Safari's bars are a fixed `#003380` in both themes (`theme-color`, the page background and the manifest), and the wrapper passes the safe-area inset into the iframe, where it cannot be measured. The app pads its navbar with it, and lifts its form buttons and bulk-action bar by `min(inset, 34px)`. Some iOS 26 home-screen installs give the page one status bar less than the screen (WebKit bug 301994); the wrapper measures the shortfall, makes the page that much taller and places the iframe in the page instead, so the app runs to the bottom edge. Behind it, the home-screen page is the app's own background (`#f7fafc` light, `#0F141A` dark). The loading screen stays navy.
- **Shows a loading screen.** A walking student and "Loading tracker..." show while the iframe stays invisible, until the app reports it has painted (`APP_PAINTED`). After 12 seconds a notice and a Reload button appear; after 25 seconds the wrapper retries once by itself.
- **Checks the app on return.** Coming back to the tab or home-screen app, the wrapper asks the app to answer (`WRAPPER_HEALTH` / `APP_ALIVE`) within 3 seconds, and reloads it behind the loading screen if it does not.
- **Keeps the address bar in step** with the page and tab the user is on (`UPDATE_URL`), so links can be shared. Sign-in parameters and old ones (`access`, `bypass`, `teacher` …) are stripped from the address bar and never passed on.

No device details (browser, OS, phone model) are read or sent.

## Useful links

| Link | Opens |
|---|---|
| `https://wdlsec.github.io/smt/` | The dashboard |
| `https://wdlsec.github.io/smt/?page=form` | An empty Submit Form |
| `https://wdlsec.github.io/smt/?page=form&reason=...&location=...` | The form with Reason and Location pre-filled |

Reported By always starts as the signed-in person's name, so it is no longer pre-filled from the link (`teacher=` is ignored).

## Installing on a phone

- **iPhone (Safari):** open the link, tap Share, then Add to Home Screen.
- **Android (Chrome):** open the link, tap the ⋮ menu, then Add to Home screen or Install app.

A home-screen app and the browser keep separate storage, so each signs in on its own and counts as its own device.

## Making changes

- **New Apps Script deployment URL:** update `baseUrl` in `index.html`.
- **New GitHub Pages address:** update the "Direct to current dashboard" row of the `Links` tab in the spreadsheet as well. The app only trusts that origin, and no sign-in OTP is sent for any other. Everyone has to sign in again at the new address.
- **Bar colour:** change `#003380` in all four places together: the `theme-color` meta tag and the `html, body` background in `index.html`, `theme_color` / `background_color` in `manifest.json`, and `--band` in the app's `CSS_Themes`.
- **Measuring layout on a phone:** open the site in Safari with `?debug=1` on the address to show the viewport measurements over the page.
- **Caching:** GitHub Pages caches files for about 10 minutes. After an edit, wait, or remove and re-add the home-screen icon.

## Changing the icon

1. Replace all six PNGs with the new artwork at the same names and sizes. Keep text out of the design: at tab and home-screen sizes it becomes unreadable stripes.
2. For `icon-maskable-512.png`, fill the whole square and keep everything important inside the central circle (80% of the width), or Android will crop it.
3. Bump `?v=2` to `?v=3` on every icon link and the manifest link in `index.html`, and on each `src` in `manifest.json`, so browsers fetch the new files.
4. Phones keep the old home-screen icon: remove the shortcut and add it again.

## When the tracker is shut down

Maintenance mode and the kill switch are controlled entirely from the spreadsheet and Apps Script. No change to this repository is needed or possible. During maintenance, staff see a notice while admins keep working. During a kill-switch lockdown every device is signed out and the app inside shows only "Page Unavailable". See READ_ME in the Apps Script project for how to lift it.
