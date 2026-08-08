# CHI CINEMA

A Netflix-inspired movie & TV streaming front-end, built with vanilla HTML5, CSS3, and ES6 JavaScript modules — no frameworks, no build step. Powered by [TMDB](https://www.themoviedb.org/) for all movie/show data.

---

## 1. Installation

This is a static site — there is no `npm install` or build step required.

1. Download or clone this project folder (`CHI-CINEMA/`).
2. Open the folder in **Visual Studio Code**.
3. Install the **Live Server** extension (by Ritwick Dey) if you don't already have it.
4. Right-click `index.html` → **"Open with Live Server"**.
5. The site opens at `http://127.0.0.1:5500/index.html` (or similar) and is fully functional immediately — hero banner, content rows, search, login/register, favorites, and the player all work out of the box using the bundled TMDB API key.

No `.env` file, no server, no database. Everything runs client-side, using `localStorage`/`sessionStorage` for accounts, favorites, and history.

> **Why Live Server specifically?** The site uses ES6 `<script type="module">` imports, which browsers block on the `file://` protocol for security reasons. It must be served over `http://`, even locally — Live Server is the simplest way to do that.

---

## 2. Folder Structure

```
CHI-CINEMA/
├── index.html            Home page (hero + content rows)
├── movie.html             Movie/TV details page
├── player.html            Video player page
├── search.html            Search page
├── login.html             Login page
├── register.html          Registration page
├── favorite.html          Saved favorites page
├── profile.html           Account profile page
├── manifest.json          PWA manifest
├── robots.txt
├── sitemap.xml
├── README.md
│
├── assets/
│   ├── backgrounds/       Auth page backdrop images
│   ├── images/            Placeholder/fallback images (poster, backdrop, avatar SVGs), OG cover
│   ├── posters/            (unused by default — TMDB posters are loaded live; use this if you add local artwork)
│   ├── logos/               Brand mark SVG(s)
│   ├── loading/              Loading screen assets, if you build a custom one
│   ├── icons/                 Favicons + PWA icons
│   ├── avatars/                (unused by default — avatars are stored as data URLs; use this for stock avatar options)
│   ├── fonts/                   (unused by default — fonts load from Google Fonts CDN; put local font files here to self-host)
│   ├── trailers/                  (unused by default)
│   └── movies/                     Local MP4 files go here — see "Adding MP4 Movies" below
│
├── scripts/                (all ES6 modules, see architecture layers below)
│   ├── config.js           Layer 0 — TMDB key, endpoints, feature flags
│   ├── utils.js             Layer 1 — pure helpers (formatting, DOM, debounce, hashing)
│   ├── storage.js            Layer 1 — the ONLY file touching localStorage/sessionStorage
│   ├── api.js                  Layer 2 — the ONLY file touching the TMDB network
│   ├── animation.js             Layer 2 — Web Animations API + Intersection/ResizeObserver helpers
│   ├── router.js                 Layer 2 — page detection, URL building, auth guard, navbar wiring
│   ├── auth.js                     Layer 3 — register/login/logout/session
│   ├── favorite.js                  Layer 3 — favorites/continue-watching/history + change events
│   ├── home.js                        Layer 4 — index.html controller
│   ├── movie.js                        Layer 4 — movie.html controller
│   ├── player.js                        Layer 4 — player.html controller
│   ├── search.js                         Layer 4 — search.html controller
│   ├── login.js                           Layer 4 — login.html controller
│   ├── register.js                         Layer 4 — register.html controller
│   ├── favorites-page.js                    Layer 4 — favorite.html controller
│   └── profile.js                             Layer 4 — profile.html controller
│
└── styles/
    ├── reset.css            Browser default reset (no design tokens — works standalone)
    ├── variables.css         All design tokens (colors, type scale, spacing, shadows, etc.)
    ├── global.css              Base typography, buttons, utility classes, focus ring
    ├── navbar.css                Navigation bar
    ├── footer.css                  Footer
    ├── hero.css                      Home page hero banner
    ├── cards.css                       Movie/show poster card
    ├── rows.css                          Horizontal scrolling content rows
    ├── movie.css                           Movie/TV details page
    ├── player.css                            Video player page
    ├── login.css                               Shared shell for login + register
    ├── register.css                              Register-only additions (avatar upload, password strength)
    ├── search.css                                  Search page + reused by favorite.html's grid
    └── responsive.css                                Shared breakpoints for navbar/footer/containers only
```

> **Note on the original spec:** the initial architecture request listed both `style.css` and `global.css` as separate files. Only `global.css` was built — it absorbed that role (base typography, utility classes, buttons). If you want a literal empty `style.css` for your own page-specific overrides, you can create one and link it last on any page; nothing in this codebase depends on its absence.

**Layering rule** (why files are organized this way): a file may only import from a lower layer. `config.js` has zero dependencies; `utils.js`/`storage.js` depend only on `config.js`; and so on up to the eight page controllers, which never import each other. This is what lets you add a ninth page later without touching any existing file.

---

## 3. Changing Logos

1. Replace `assets/logos/chi-cinema-mark.svg` with your own SVG (referenced in every page's navbar as `<img class="cc-navbar__brand-mark" src="assets/logos/chi-cinema-mark.svg">`).
2. The text portion of the logo ("CHI **CINEMA**") is plain HTML/CSS, not an image — to change the wordmark, edit the `.cc-navbar__brand-text` / `.cc-footer__brand-text` spans directly in each HTML file, or just restyle the existing text via `styles/navbar.css` / `styles/footer.css`.
3. Favicons live in `assets/icons/` (`favicon.ico`, `favicon.svg`, `apple-touch-icon.png`) and PWA icons are declared in `manifest.json`.

---

## 4. Changing Theme

All colors, typography, spacing, shadows, and timing live as CSS custom properties in **`styles/variables.css`** — nothing else in the project hardcodes a raw color or spacing value. To retheme the entire site:

```css
/* styles/variables.css */
--cc-color-crimson: #e63950; /* primary accent — change this for a new brand color */
--cc-color-gold: #d4af6a; /* secondary accent (marquee glow, ratings) */
--cc-color-stage-0: #0b0b0f; /* base background */
--cc-font-display: "Bebas Neue", ...; /* headline/marquee font */
--cc-font-body: "Manrope", ...; /* body/UI font */
```

Changing these tokens re-themes every page automatically, including the signature "marquee glow" effect (`--cc-shadow-marquee`) used on hero/movie titles.

To switch fonts, also update the Google Fonts `@import` at the top of `styles/global.css`.

---

## 5. Replacing the TMDB API Key

The API key lives in exactly one place: **`scripts/config.js`**.

```js
export const TMDB_API_KEY = "60fce5a23a9cb77fbd08a91933047b7d";
```

Replace the string with your own key from [themoviedb.org/settings/api](https://www.themoviedb.org/settings/api). No other file needs to change — `api.js` builds every request URL from this constant.

---

## 6. Adding MP4 Movies

By default, the player plays the official YouTube trailer for every title (`FEATURE_FLAGS.useLocalMp4 = false` in `scripts/config.js`). To play your own local video files instead:

1. Add your `.mp4` files to `assets/movies/`, named by TMDB id: `assets/movies/603.mp4` (for "The Matrix", TMDB id 603).
2. In `scripts/config.js`, set:
   ```js
   export const FEATURE_FLAGS = {
     useLocalMp4: true,
     // ...
   };
   ```
3. That's it — `player.html`/`player.css`/`player.js` require zero further changes. The full custom control bar (scrubber, volume, speed, captions-ready, Picture-in-Picture, fullscreen) automatically takes over, and if a specific movie's file is missing, `player.js` gracefully falls back to that title's YouTube trailer.

---

## 7. Adding Posters / Custom Artwork

Movie and TV posters are loaded live from TMDB's image CDN by default (via `IMG_BASE_URL` in `config.js`) — you don't need to download anything for the site to work.

To use your own artwork instead for a specific title (e.g. a custom poster):

1. Add the image to `assets/posters/`.
2. In the relevant page controller (e.g. `scripts/movie.js`'s `posterUrl()` function), add a lookup that checks a local override map before falling back to the TMDB path.

Fallback images (shown when TMDB returns no poster/backdrop/avatar) live in `assets/images/` and are wired via `FALLBACK_IMAGE` in `config.js` — replace `poster-placeholder.svg`, `backdrop-placeholder.svg`, and `avatar-placeholder.svg` with your own designs any time.

---

## 8. Deployment to GitHub Pages

1. Push the `CHI-CINEMA/` folder contents to a GitHub repository (the repo root should be the folder containing `index.html`).
2. In the repo, go to **Settings → Pages**.
3. Under "Build and deployment", set **Source: Deploy from a branch**, branch: `main`, folder: `/ (root)`.
4. Save. Your site will be live at `https://<your-username>.github.io/<repo-name>/` within a minute or two.
5. Update `sitemap.xml` and the `og:url`/`canonical` meta tags in `index.html` to match your real GitHub Pages URL.

No build step is needed — GitHub Pages serves static files directly, and this project has no server-side code.

---

## 9. Deployment to Vercel

1. Push the project to a GitHub repository (same as above).
2. Go to [vercel.com/new](https://vercel.com/new) and import the repository.
3. Framework Preset: choose **"Other"** (this is a static site — no framework to detect).
4. Build Command: leave blank. Output Directory: leave as the repo root (`.`).
5. Deploy. Vercel will serve the static files directly, and every page (including deep links like `movie.html?id=603&type=movie`) works immediately since there's no routing to configure.

---

## 10. Troubleshooting

**Blank page / "Failed to load module script" error in the console**
You opened `index.html` directly via `file://` instead of through a local server. ES6 modules require `http://`. Use Live Server (see Installation).

**Hero banner or rows never load, console shows a 401 from TMDB**
Your API key in `scripts/config.js` is invalid or has been rate-limited. Verify it at themoviedb.org.

**Login/Register says "An account already exists on this browser"**
This project stores **one account per browser** in `localStorage` (see `storage.js`'s schema — it's a single `chicinema:user` record, not a user list). Log in with the existing account, or clear `localStorage` for the site (DevTools → Application → Local Storage → clear) to start fresh.

**Favorites/Continue Watching disappeared**
All user data is stored in the browser's `localStorage`. Clearing browser data, using a different browser, or private/incognito mode will not persist it. There is no server-side account sync in this build.

**Player always shows the YouTube trailer, never my local MP4**
Confirm `FEATURE_FLAGS.useLocalMp4` is `true` in `config.js`, and that your file is named exactly `assets/movies/<TMDB id>.mp4`. Check the browser console for a `[CHI CINEMA] local MP4 not found` warning, which confirms the fallback path was triggered.

**Fonts look like a generic system font**
The Google Fonts `@import` in `global.css` requires an internet connection. If you're testing fully offline, self-host the font files in `assets/fonts/` and update the `@font-face`/`@import` rule accordingly.

---

## 11. Future Expansion

The architecture was designed so none of the above files need to be rewritten to add:

- **Dedicated Movies/TV Shows listing pages** — currently, "Movies"/"TV Shows" in the navbar scroll-anchor to homepage rows (there was no `movies.html`/`tvshows.html` in the original locked file structure). Adding real listing pages is a Layer-4 addition: a new `movies.html` + `scripts/movies.js` that reuses `api.js`'s `discover()`/`getByGenre()` and the same card/row rendering pattern as `home.js`.
- **A shared UI-components module** — `home.js`, `movie.js`, `search.js`, and `favorites-page.js` each independently implement a small card/row-rendering function (a deliberate, documented trade-off to keep page controllers decoupled — see the comments at the top of `movie.js`). If this duplication becomes a maintenance burden, extract it into a new Layer-3 module (e.g. `scripts/ui-cards.js`) that all Layer-4 controllers import.
- **Real subtitle tracks** — `player.html`'s control bar is captions-ready (the markup/CSS pattern supports adding a `<track kind="subtitles">` and a toggle button), but no `.vtt` files or toggle logic are wired yet.
- **Multi-account support** — today's `storage.js` schema stores one user record per browser by design. Moving to multiple local profiles (like Netflix's profile switcher) means changing `chicinema:user` from a single object to a keyed collection — a schema version bump (`STORAGE_SCHEMA_VERSION`) handles this without breaking existing installs, since `storage.js`'s `ensureSchemaVersion()` migration hook exists for exactly this.
- **A real backend** — swapping `auth.js`'s local SHA-256/localStorage implementation for real server-side authentication only requires changing the internals of `auth.js`'s exported functions (`registerUser`, `loginUser`, `logoutUser`, `isLoggedIn`, `getCurrentSession`) — every page controller already calls these five functions exclusively and never touches storage directly, so the public contract doesn't need to change.

---

**Movie and TV data provided by [TMDB](https://www.themoviedb.org/).** CHI CINEMA is an educational/portfolio project and is not affiliated with Netflix, Disney+, Prime Video, Apple TV, or TMDB.
