# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev       # Vite dev server
npm run build      # Build static site to dist/ (this is what gets deployed)
npm run preview    # Preview the production build locally
```

There is no test suite (`npm test` is a placeholder) and no linter configured.

## Architecture

This is a static marketing/booking site for Elite Home Fitness (front-end brand "Elite", powered by "Montra"), built as a set of standalone HTML pages bundled by Vite — there is no SPA framework or component system.

- **Multi-page Vite build**: every page is a separate HTML entry point listed individually in `vite.config.js` under `build.rollupOptions.input`. **Any new top-level `.html` page must be added there**, or it will not be emitted into `dist/` and will 404 on Firebase Hosting (this exact bug was fixed in a recent commit for the trainer pages).
- **No shared layout/includes**: the `<head>` boilerplate (Tailwind CDN + config, Google Fonts, shared `<style>` overrides for `.active-nav` / `.dark-nav`) and the `<nav>` markup are duplicated at the top of every HTML file rather than templated. When changing nav/header/footer markup, you generally need to edit every page individually.
- **Styling**: Tailwind is loaded via the CDN script (`cdn.tailwindcss.com`) with an inline `tailwind.config` per page (brand color `#E85D04`, font `Inter`) — there is no Tailwind build step, PostCSS config, or `tailwind.config.js` file. `assets/js/nav.js` (active-nav highlighting via `DOMContentLoaded`) exists but is not currently `<script src>`-included by any page — nav active-state is instead hardcoded with classes per page.
- **Deployment**: Firebase Hosting serves the `dist/` directory (`firebase.json`'s `hosting.public`). `.github/workflows/*.yml` deploys to Firebase Hosting (project `montra-27532`, channel `live`) on every push to `main` via `npm ci && npm run build`, then `FirebaseExtended/action-hosting-deploy`. `dist/` and `.firebase/` are gitignored build artifacts — never hand-edit files under `dist/`.
- **Backend integration**: Several pages (`quiz.html`, `trainer-onboarding.html`, `trainer-application.html`, `find-a-coach.html`) talk directly to a separate live backend via inline `<script>` blocks (no bundler-managed JS modules, no shared API client). Each defines its own `const BACKEND_URL = 'https://montra-production.up.railway.app'` and calls REST endpoints directly with `fetch`, e.g.:
  - `GET /api/firebase/client-config` — fetches Firebase web config at runtime, then initializes Firebase Auth client-side (`firebase-auth-compat.js` in `quiz.html`, modular `firebase-auth.js` SDK in `trainer-onboarding.html` — these two pages use different Firebase SDK styles, be consistent with whichever the file already uses).
  - `GET /api/trainers` — trainer directory data used to drive matching/search UI.
  - `POST /api/client/requests` — client booking/request submission (quiz flow), gated behind Firebase email/password auth (sign up or sign in) on the client.
  - `GET /api/trainers/my-profile`, `POST /api/trainers/apply` — trainer onboarding/application flow, also behind Firebase Auth.
  - `POST /api/trainers/provision` — trainer account provisioning (`trainer-application.html`).
  - There is no `.env`/config file for `BACKEND_URL` — it's a literal hardcoded string per file; if the backend URL changes, update each occurrence individually.
- **Static sample data**: `data/trainers.json` is local mock/sample trainer data (specialties, availability, certifications) — distinct from the live `/api/trainers` backend data fetched in the booking flow. Check which a given page actually uses before assuming it's live or static.
- **Images**: all imagery lives flat in `assets/images/` (logos, service icons, hero photos, partner/client logos). `assets/css`, `assets/fonts`, and `assets/icons` exist but are currently empty.
