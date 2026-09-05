# maroco wedding invite — reorganized export

## What this project actually is

Not a static Tilda-style template. This be **built production output** of
a React app (Vite + Tailwind, Radix UI, Supabase backend for RSVP), likely
made with Lovable/gpt-engineer. `app.js` = one bundled, minified file
containing React + all vendor libs + app code together. No separate
source components exist in this export — only compiled output.

Consequence: cannot split into clean React source files (that source was
never given to us, only the build result). What CAN be done safely:
reorganize files/folders and fix every path reference to match, without
touching logic. Done below.

## Structure

```
index.html
favicon.ico
assets/
  css/main.css        (was assets/index-B7CFTRXZ.css)
  js/app.js            (was assets/index-BM2pyw11.js)
  images/
    hero-marocain.png          (was __l5e/.../hero_marocain.png)
    program-background.png     (was __l5e/.../program-bg.png)
    salle-de-dinee.jpg         (was __l5e/.../sale_de_dinee.jpg)
    footer.jpg                 (was __l5e/.../footer.jpg)
    menu-chorba.png
    menu-plat-principal.png
    menu-fruits.png
    menu-the.png
    invites-illustration.png   (was invites-karako-CAOZLTO1.png)
  video/
    intro-door.mp4
vendor/
  tracking/
    flock.js              (analytics script, was /~flock.js)
    analytics-proxy.html  (mock endpoint, was api/analytics.html, body: "Accepted")
```

All 10 asset paths hardcoded as string literals inside `app.js` were
found and rewritten to match new locations (verified: each occurred
exactly once, all replaced, no stale refs left). `index.html` script/css/
favicon/tracking refs updated too.

## Dropped (not features of the site, crawl artifacts)

- `www.google.com/search/warmup.html` — junk from headless-browser crawl.
- `fonts.googleapis.com/css2.css`, `fonts.gstatic.com/*` — the site loads
  fonts live from Google's real CDN via absolute URL in `index.html`
  (`<link href="https://fonts.googleapis.com/...">`), never from a local
  copy. These mirrored files weren't wired into the app at all.

## Bug found and fixed (pre-existing, from original export — not caused by reorg)

Original `app.js`/`flock.js` had `??` (nullish coalescing) and `?.`
(optional chaining) tokens broken with a stray space — `??` → `? ?`,
`?.` → `? .`, 56 + 6 spots. Made both files fail to parse (`Unexpected
token '?'`), which is why you saw a white screen. Root cause looks like
the export/mirroring tool mangling these operators. Fixed globally in
this reorganized copy; both files now pass `node --check` clean.

## Bug #2 fixed — absolute paths break on GitHub Pages project sites

Every path started with `/` (e.g. `/assets/js/app.js`). That only
resolves correctly when the site is served from a domain **root**.
`python -m http.server` serves at root, so it worked there — but GitHub
Pages project sites live at `username.github.io/repo-name/`, so an
absolute `/assets/js/app.js` request goes to the domain root and misses
the `repo-name/` folder → 404s → white screen.

Fixed by switching every internal reference (`index.html` + the 10 asset
paths baked into `app.js`) from absolute (`/assets/...`) to relative
(`assets/...`). Verified against a simulated subpath deployment — every
asset now resolves whether hosted at root or under a subfolder.

## Bug #3 fixed — React Router had no basename, so subpaths 404 into the app's own NotFound page

Routes are defined as `path: "/"`, `/admin`, `*` (catch-all → NotFound),
with no `basename` set on the router. On a domain root that's fine
(pathname is `/`, matches). On a GitHub Pages **project** site the real
pathname is `/repo-name/`, which matches nothing but `*` — so what you
saw was the wedding app's *own* "Oops! Page not found" screen (its
`href="/"` link is why the button took you to your GitHub profile page,
not back to the invite), not a real GitHub 404.

Fixed by:
- `index.html`: added an inline script (runs before `app.js`) that reads
  `location.pathname` at load time and stores it as `window.__RB__` —
  this is the deployed subpath, whatever it is, computed automatically.
- `app.js`: passed `basename: window.__RB__ || "/"` into the router.

Works unmodified whether hosted at a domain root or under any subpath —
no repo name hardcoded anywhere.

## ⚠️ Missing feature — action needed

Found in `app.js`: a "TAP" overlay that unmutes **background music**
(`young_and_beautiful_remix.mp3`, ~2.6MB, referenced as
`/__l5e/assets-v1/3759acde-283d-4758-9f02-087f4d905435/young_and_beautiful_remix.mp3`).

The mp3 file itself was **not included** in the site export you gave me —
only the reference to it survived. Left the path as-is (untouched) since
I have nothing to point it to. If hosted standalone, this feature will
silently fail (tap button shows, no sound plays). Need the actual mp3 to
fix — everything else is fully wired and feature-complete.
