# Q Fab Works Website — Handoff

Last updated: 2026-07-11
Repo: `github.com/DQDMedia/qfabworks` (branch `main`)

This document covers everything done in this project so far, why each
decision was made, and what's still open.

---

## 1. What this project is

A single-page static website for Q Fab Works (Daniel Durgin's custom metal
fabrication / laser engraving / 3D printing shop, Myrtle Beach, SC). No
build process, no framework — one `index.html` file with inline CSS and a
small vanilla-JS gallery widget, plus `images/` (logo, headshot) and
`photos/` (subfolders for `metal-art`, `laser`, `3d-printing` — currently
empty, gallery uses placeholder color gradients until real photos are
dropped in).

## 2. Timeline of work done

### a. Initial design (pre-existing, commit `1586ee7`)
The site already existed with a dark charcoal theme, Oswald + IBM Plex Mono
fonts, and amber/orange accents — sections for Services, Gallery (with
category filters + a lightbox modal), About, and Contact.

### b. First redesign pass — reskin (superseded)
The owner pointed out the site looked too close to **DQD Media**
(`dqdmedia.com`), a sibling site under the same GitHub org, which used the
same underlying template (same font pairing, same warm-charcoal palette
approach, same header/hero/card/lightbox structure — just different colors).

A first pass changed fonts (Bebas Neue / Work Sans / Space Mono) and the
color palette (near-black + safety-yellow + warning-red), but the owner
correctly called out that the *structure* was still identical to DQD
Media — same sticky header pattern, same "kicker + heading + note" formula
opening every section, same filter-tabs-plus-grid-plus-lightbox gallery
interaction, same photo-left/text-right About layout, same two-button
Contact block. Colors and fonts alone weren't enough.

### c. Full structural rebuild (commit `ba30335`) — current design
Rebuilt around a "work order / shop plate" concept instead of the shared
template:

- **Chamfered panels** (`clip-path` cutting two opposite corners) on the
  hero, service steps, and gallery tabs — meant to evoke a plasma-cut edge.
  Applied via a reusable `.chamfer` / `.chamfer-sm` class.
- **Hero**: single chamfered plate with a stamped "job tag" label, headline,
  sub-copy, CTA — followed by a horizontal scrolling capability **ticker**
  strip (`01 Custom & ready-made / 02 Plasma cutting...`), instead of the
  old split hero-copy + spec-plate sidebar.
- **Services → "process strip"**: three connected plates with large
  outline-only numbers (`01`, `02`, `03`), a red tag line, heading, and
  copy — framed as steps rather than generic feature cards.
- **Gallery → filmstrip + inline detail panel**: category tabs (styled like
  folder tabs) filter a horizontally scrolling strip of tiles. Clicking a
  tile opens an **inline detail panel below the strip** (title, meta, full
  caption, close button) instead of a fullscreen lightbox/modal overlay —
  intentionally a different interaction pattern from DQD Media's modal.
- **About → "dossier" layout**: bio text as the primary column, with a
  small rotated (`-1.2deg`), dashed-border **ID-badge card** (photo, name,
  role, credential list) floated beside it, including a fake "hole punch"
  circle at the top — instead of a plain photo-left/text-right grid.
- **Contact → "ticket" layout**: a dashed vertical perforation line
  separates the message copy from an email/phone "stub," styled like a
  ticket you'd tear off, replacing the old two pill-shaped CTA buttons.
- Palette: `--onyx` (#0c0d0e background), `--chalk` (#f3f4ef text),
  `--hazard` (#f7d417 safety yellow, primary accent), `--warn` (#e8382c,
  secondary accent). Fonts: **Bebas Neue** (headings), **Work Sans**
  (body/nav), **Space Mono** (tags/meta/labels) — all via Google Fonts CDN.

Verified in a browser preview: nav, category filters, inline detail-expand,
mobile responsive layout (375px), and desktop (1280px) — no console errors,
no failed network requests.

### d. Deployment to GitHub Pages + custom domain (commits `f6e43aa`,
`1999ebc`)
- Confirmed remote is `git@github.com:DQDMedia/qfabworks.git`.
- Added a `CNAME` file containing `qfabworks.com` to enable a GitHub Pages
  custom domain.
- Owner configured GitHub → Settings → Pages (source: `main` branch,
  `/root`) and added DNS records on Squarespace (which manages the
  `qfabworks.com` domain/DNS, backed by Google Cloud DNS under the hood —
  Squarespace acquired Google Domains):
  - 4 `A` records at `@` → `185.199.108.153`, `.109.153`, `.110.153`,
    `.111.153` (GitHub Pages' IPs)
  - `CNAME` record `www` → `dqdmedia.github.io`
- **Problem found**: `qfabworks.com`'s DNS zone returns `SERVFAIL` when
  queried directly against its own authoritative nameserver
  (`ns-cloud-a1.googledomains.com`), for every record type. This is not a
  propagation delay and not a records mistake — the zone itself is broken
  on the Squarespace/Google Cloud DNS backend. Confirmed by querying the
  authoritative server directly (bypasses caching/propagation entirely) and
  getting the same SERVFAIL repeatedly, before and after re-checking.
  - Owner tried removing the `_domainconnect` CNAME record (Squarespace
    Domain Connect) hoping it was the cause — did not fix it.
  - **This needs Squarespace support** — recommended contacting them with:
    "DNS queries for qfabworks.com return SERVFAIL directly from our
    authoritative nameserver (ns-cloud-a1.googledomains.com), for all
    record types." This is still unresolved as of this writing.
- Because of a Monday deadline and the DNS issue being out of our control,
  **the `CNAME` file was temporarily removed** (commit `1999ebc`) so the
  site would be reachable somewhere while Squarespace's issue gets sorted.
- **Discovered a reliable working fallback**: `https://dqdmedia.com/qfabworks/`
  serves the site correctly right now (HTTP 200, verified). This works
  because `dqdmedia.com` (DQD Media's own domain) already resolves fine,
  and GitHub Pages automatically serves any other project repo under the
  same GitHub account/org as a subpath of that already-working custom
  domain — independent of this repo's own `CNAME` file or Pages settings.
- **Known cosmetic issue, not urgent**: `https://dqdmedia.github.io/qfabworks/`
  (the free default GitHub Pages URL) still 301-redirects to
  `http://qfabworks.com/` (which doesn't resolve) even after removing the
  `CNAME` file and two subsequent pushes/rebuilds. This means the
  **"Custom domain" field in this repo's own Settings → Pages** is likely
  still separately set to `qfabworks.com` and needs to be manually cleared
  there (removing the file alone wasn't enough to reset the UI field). Low
  priority since `dqdmedia.com/qfabworks/` is unaffected by this and is the
  actual link being used.

### e. Contact info update (commit `561a6bb`)
- Email changed from `daniel@dqdmedia.com` → `daniel@qfabworks.com`
- Phone changed from `843.813.7293` → `843.281.7322`
- Both the visible text and the `mailto:` / `tel:` link targets were updated.

## 3. Current live state (as of last commit `561a6bb`)

| URL | Status |
|---|---|
| `https://dqdmedia.com/qfabworks/` | **Working** — serves the current site, correct contact info |
| `https://qfabworks.com` | **Not working** — DNS zone SERVFAIL (Squarespace backend issue) |
| `https://dqdmedia.github.io/qfabworks/` | 301-redirects to the broken `qfabworks.com` (Settings → Pages custom domain field still set) |

The `CNAME` file is **intentionally absent** from the repo right now. Do
not re-add it until `qfabworks.com` DNS is confirmed actually resolving —
re-adding it will make both the `.github.io` URL and `dqdmedia.com/qfabworks/`
redirect to `qfabworks.com` again, which currently goes nowhere.

## 4. What's left / open items

1. **Follow up with Squarespace support** about the DNS SERVFAIL issue on
   `qfabworks.com`. This is the actual blocker for the real domain going
   live. Not something fixable from the DNS records screen or from this
   repo.
2. **Once Squarespace confirms the zone is fixed**: re-add a `CNAME` file
   containing `qfabworks.com`, commit, and push. Then re-verify the A/CNAME
   records still resolve (`dig qfabworks.com A`, `dig www.qfabworks.com
   CNAME`) before considering it done.
3. **Clear the stale "Custom domain" field** in `github.com/DQDMedia/qfabworks`
   → Settings → Pages (currently still shows `qfabworks.com`, causing the
   `.github.io` redirect issue in section 2d). Low priority — cosmetic only.
4. **Real gallery photos**: the gallery's `SHOTS` array in the `<script>`
   block at the bottom of `index.html` currently uses placeholder CSS
   gradients (`grad: [...]`) instead of real photos. There's a `photos/`
   folder already set up with `metal-art/`, `laser/`, and `3d-printing/`
   subfolders (currently empty) for this. To swap in a real photo, add the
   file to the right subfolder and change that shot's entry to
   `img: "photos/<category>/<filename>.jpg"` (see the comment directly
   above the `SHOTS` array in `index.html`).
5. **Local preview tooling caveat** (not a site issue): my local preview
   tool can't serve directly from `~/Documents/Q Fab Works website` due to
   a macOS Files-and-Folders privacy permission that hasn't been granted to
   whatever process runs it. Doesn't affect the live site at all — only
   affects my ability to preview locally in-session. If this comes up
   again, either grant that permission in System Settings → Privacy &
   Security → Files and Folders (then fully restart the app) or just open
   `index.html` directly in a browser.

## 5. Key decisions worth knowing about

- **Why a full rebuild instead of just new colors/fonts**: the first pass
  only changed surface styling and the owner correctly identified that the
  layout skeleton (header, hero, section formula, gallery interaction,
  about layout, contact layout) was still identical to DQD Media's site.
  Real differentiation required changing the actual structure/interaction
  patterns, not just the paint.
- **Why the `CNAME` file was removed rather than waiting on Squarespace**:
  there was a hard Monday deadline and the DNS issue is on Squarespace's
  backend with an unknown fix timeline. Removing `CNAME` was the fastest
  way to guarantee *something* live, without waiting on a third party.
- **Why `dqdmedia.com/qfabworks/` was adopted as the interim link** rather
  than spinning up a separate host (Netlify/Vercel, etc.): it required zero
  new setup, was already live and correct the moment it was checked, and
  keeps everything on GitHub Pages so no extra accounts/services are
  involved.
