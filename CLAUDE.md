# subscriptions-tracker — build CLAUDE.md

Build-specific context only. Global rules (pre-flight, terminology, write safety, behavioral) inherit from `~/.claude/CLAUDE.md`.

## Header

- **Purpose** — Zero-backend personal subscription tracker with a cost-optimization engine. All data lives in the user's browser `localStorage`; nothing leaves the device.
- **Owner** — Personal.
- **GitHub repo** — `gwaiblade/subscriptions-tracker` (public).
- **Local path** — `/Users/mujiri/Library/CloudStorage/Dropbox/+Work/AI Works/subscriptions-tracker`.

## Stack

- Vanilla HTML / CSS / JS in a single `index.html` (~84 KB).
- No build system. No `package.json`. No bundler. No transpile step.
- Chart.js **4.4.1** via jsDelivr CDN (donut + bar charts).
- Google Fonts: Instrument Serif (display), DM Sans (body), JetBrains Mono (numerics).
- Frankfurter API for FX (ECB rates, no key, no auth).
- Persistence: `localStorage` keys `ledger:subs:v1`, `ledger:settings:v1`, `ledger:fx:v1`.

## Commands

- **Install** — none. No dependencies to install.
- **Run / dev** — open `index.html` directly in a browser, or `python3 -m http.server 8000` from the repo root and visit `http://localhost:8000`.
- **Test** — no test suite.
- **Build** — none. The file ships as-is.
- **Deploy** — `git push` to `main`. GitHub Pages auto-builds from `/` on `main`. Live in ~30–60s; CDN cache another 1–3 min.

## Architecture

- **Entry point** — `index.html`. Single file with embedded `<style>` and one `<script>` block (~1050 lines of JS).
- **State** — module-level `state` object: `{ subs, settings, fx, editId, calCursor, sortKey, sortDir }`. Persisted to `localStorage` on every mutation.
- **Optimize engine** — a series of rules in `runOptimize()` that produce findings ranked by annual savings (annual-switch, duplicate-category, low-usage, family-plan, free-alternative, sub-$10 creep, business separation). Always uses out-of-pocket cost; 100%-subsidized subs are skipped.
- **FX** — fetched from Frankfurter on load with USD as base; cached in `localStorage` for 24h; fallback static rates if the API fails.
- **Charts** — Chart.js instances created/destroyed on tab switch to avoid memory leaks.

## External services

- **GitHub Pages** — hosting.
- **Frankfurter API** (`api.frankfurter.app`) — FX rates, no key, no auth, ECB-backed.
- **Google Fonts** — Instrument Serif, DM Sans, JetBrains Mono.
- **jsDelivr CDN** — Chart.js 4.4.1.

No backend, no database, no third-party auth.

## Secrets

None. The app has no API keys, no auth, no env vars. All state is client-side. There is no `.env`.

## Gotchas

- **Single static file is the whole product.** Never introduce a build step, bundler, or backend without explicit approval — zero-backend is the selling point.
- **Brand mismatch.** Repo name and page title say "Subscriptions Tracker"; the in-app brand and copy still say "Ledger" (the original name). `localStorage` keys are namespaced `ledger:*`. Don't rename the keys — would orphan existing user data.
- **Currency default.** State default is `SGD`. Users with existing `localStorage` keep their old setting (typically USD); changing the default only affects fresh visitors.
- **Dropbox sync.** The repo lives under `~/Library/CloudStorage/Dropbox/...`. If `git status` shows phantom modifications or hangs, let Dropbox finish syncing before troubleshooting.
- **CDN cache after deploy.** Pages serves stale HTML for 1–3 min after a push. Hard-refresh (Cmd+Shift+R) to verify a new version.
- **HTML validators flag XHTML-style self-closing.** The file uses `<meta ... />` and `<input ... />`. Naive parsers report mismatches; this is cosmetic and intentional.
- **claude.ai re-uploads can revert local tweaks.** Replacing `index.html` with a fresh download from a claude.ai chat will overwrite any local edits made via Code (e.g. font swaps, size bumps). Re-apply or diff before assuming the upload is a superset.

## Status

- **Live** — https://gwaiblade.github.io/subscriptions-tracker/
- **Body font** — DM Sans, base 15px, `letter-spacing: 0.01em`, `line-height: 1.55`. All other font sizes bumped ~10% over the upstream baseline.
- **Last shipped** — Work subsidy field + Full / Out-of-pocket topbar toggle (commit `7c83e45`).
- **Discussed, not built** — GitHub Actions scraper for Singapore service pricing → `sg-pricing.json` consumed by the Optimize engine. Would add freshness without breaking zero-backend at runtime.
