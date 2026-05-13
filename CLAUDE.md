# CLAUDE.md — SubscriptionsTracker

## 1. Build identity

SubscriptionsTracker — Personal track. Zero-backend static HTML subscription tracker for Henry; all state in browser `localStorage`.
SPEC: `gwaiblade/context-brain` → `working/subscriptions-tracker/SPEC_SubscriptionsTracker.md`.

## 2. Local conventions

- No package manager. No dependencies to install.
- No language version pins. Single `index.html`, runs in any modern browser.
- No formatter or linter. Match existing style in `index.html`: 2-space indent, single quotes in JS, double quotes for HTML attributes.
- Dev: `python3 -m http.server 8000` from repo root, or open `index.html` directly in a browser.
- Test: none. Smoke-test manually in the browser after changes.
- Build: none.

## 3. Deployment guardrails

- Deploy by pushing to `main` only. There is no staging branch.
- After every push, confirm Pages deploy with `gh run list --workflow=pages-build-deployment --limit 1 --repo gwaiblade/subscriptions-tracker`.
- CDN cache propagates 1–3 min after build success. Hard-refresh (Cmd+Shift+R) to verify a new version is being served.
- Do not introduce a build step, bundler, transpiler, or backend. The zero-backend single-file architecture is the product.
- Do not change GitHub Pages source (currently `main` / `/`).
- Do not force-push to `main`. Use `git revert` for rollbacks.

## 4. Repository conventions

- Trunk-based on `main`. No feature branches in solo mode.
- Commit messages: imperative one-line summary; bullet body for multi-part changes.
- Never modify or commit: `.DS_Store`, `.claude/settings.local.json`. Both gitignored.
- No runtime files are gitignored; the product is self-contained in `index.html`.
- The seeded sample subscriptions in `init()` (Claude Pro, Spotify, Netflix, etc.) mirror Henry's real subs. Do not "tidy" them.

## 5. Known footguns

- **localStorage keys are `ledger:*`** (`ledger:subs:v1`, `ledger:settings:v1`, `ledger:fx:v1`), not `subscriptions-tracker:*`. Do not rename — would orphan existing user data.
- **Brand mismatch is intentional.** Repo and page title say "Subscriptions Tracker"; in-app brand says "Ledger". Both refer to the same product.
- **Default currency is SGD.** Existing users keep their prior `localStorage` setting; only fresh visitors see SGD.
- **claude.ai re-uploads overwrite local edits.** When replacing `index.html` with a fresh download from a claude.ai chat, diff first — font swaps, currency defaults, and other local tweaks may be lost.
- **HTML uses XHTML self-closing tags** (`<meta />`, `<input />`). Naive validators report tag mismatches; ignore.
- **Dropbox sync lag.** Repo lives under `~/Library/CloudStorage/Dropbox/...`. Phantom `git status` modifications usually mean Dropbox is still syncing.
- **FX fallback rates are static.** When Frankfurter API fails, hardcoded fallback rates in `index.html` take over. Refresh these annually.

## 6. Escalate-to-Henry triggers

Stop and ask before:

- Adding any external dependency (CDN script, font, API beyond Frankfurter).
- Introducing a build step, bundler, or backend.
- Renaming `localStorage` keys or changing the storage schema.
- Changing the GitHub Pages source branch or build mode.
- Force-pushing to `main`.
- Modifying the seeded sample subscriptions in `init()`.
- Any `rm -rf` outside obvious build outputs (there are none here, so effectively any `rm -rf`).
- Spec'ing the build under a newer canon version than v1.2.
