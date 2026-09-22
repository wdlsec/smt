# WDLSS Student Movement Tracker — GitHub Pages Wrapper

This repository is the front door to the tracker. It is a thin page that holds the Google Apps Script web app in a full-screen iframe, so the tracker can be added to a phone's home screen and used like an app.

The tracker itself (data, logic, maintenance mode, the kill switch) lives in Google Apps Script. Nothing in this repository stores or sees student data.

Live address: `https://wdlsec.github.io/smt/`

## Files

| File | Purpose |
|---|---|
| `index.html` | The wrapper page. Loads the Apps Script app in an iframe and talks to it with `postMessage`. |
| `manifest.json` | Makes "Add to Home Screen" install the tracker as a standalone app on Android. |
| `icon.png` | Home-screen and browser-tab icon (192×192 and 512×512). |

In the Apps Script project, `index.html` is kept as `Github_index.html` and `manifest.json` as `Github_manifest.json` so all code sits in one place. Rename them when copying here.

## What the wrapper does

- **Loads the app** from the Apps Script `/exec` URL (the `baseUrl` constant in `index.html`) with `embed=1` added, which tells the server the request came through this page. Opened without the wrapper, the server returns a bare "Page Unavailable" notice.
- **Carries the admin access token** (`access=`) during maintenance. The token is kept in `sessionStorage` only, so it survives a refresh and disappears when the tab or home-screen app closes. Neither parameter ever shows in the address bar.
- **Answers the wrapper check.** The app pings the wrapper (`WRAPPER_PING`), and the wrapper replies from its own origin (`WRAPPER_PONG`), which another site cannot fake.
- **Paints the status-bar strip** on iOS to match the app's theme, and passes the safe-area inset into the iframe, where it cannot be measured.
- **Hides the reload flash.** The iframe stays transparent until the app reports it is full size (`APP_PAINTED`), with a 4-second fallback.
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
- **Status-bar colour:** set `STATUS_BAR.bandColour` in `index.html`. It must match `--primary-dark` / `--band` in `CSS_Themes`.
- **Caching:** GitHub Pages caches files for about 10 minutes. After an edit, wait, or remove and re-add the home-screen icon.

## When the tracker is shut down

Maintenance mode and the kill switch are controlled entirely from the spreadsheet and Apps Script. No change to this repository is needed or possible. During a kill-switch lockdown, the wrapper still loads but the app inside shows only "Page Unavailable". See the Google Sheet README for how to lift it.
