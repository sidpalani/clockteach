# Copilot instructions for Clock Learning App

This repo is a small, single-file React-based SPA served as static HTML and deployed to Cloud Run (or via Docker/nginx). The goal of this file is to give AI coding agents the minimal, actionable context they need to make sensible edits.

## Big picture (short)
- Single-page app built entirely in `index.html` using UMD React + `@babel/standalone` in-browser transpilation and Tailwind via CDN.
- No Node toolchain, bundler, or package.json in this repo. Changes are made directly to `index.html`.
- Deployment targets:
  - Recommended: Google Cloud Run using `gcloud run deploy` (see `README.md`)
  - Docker: `Dockerfile` (nginx:alpine, serves `/usr/share/nginx/html` on port 8080)
- `nginx.conf` contains SPA routing (`try_files $uri $uri/ /index.html`) — important for client-side routing.

## Where to look (key files)
- `index.html` — the app source (JSX in a `<script type="text/babel">` block) and styling (Tailwind CDN). All interactive logic lives here.
- `Dockerfile` — copies `index.html` and `nginx.conf` into nginx, exposes 8080.
- `nginx.conf` — server configuration for static hosting and SPA fallback.
- `README.md` — deployment steps (Cloud Run). Use it to reproduce the author's recommended deploy flow.

## Codebase-specific patterns & conventions
- React is used via UMD bundles (React/ReactDOM) and transpiled in the browser using Babel: edits to JSX must remain compatible with the in-browser Babel version.
- Styling uses Tailwind via CDN; classes are applied directly in JSX markup.
- State model convention:
  - `hours` and `minutes` store time (note: `hours` is 0–23 internally; UI shows 12-hour formatted value via `formatDigitalTime`).
  - `mode` toggles between `'learn'` and `'quiz'` and controls interaction/visibility.
  - Dragging logic: pointer/touch events compute angles and map to `hour`/`minute` updates (see `getAngleFromEvent`, `getClosestHand`, `handleDrag*`).
- Minute values are often quantized to 5-minute intervals for quiz generation and presets.
- Quick presets are an inline array of objects like `{ h: 3, m: 0, label: '3:00' }` — add similar entries for more preset buttons.

## Build / Run / Debug (concrete commands)
- Local quick test without Docker: open `index.html` in a browser (works with a file:// URL for simple testing, but some browsers restrict the bundled scripts). Prefer a local static server for realistic behavior:
  - Python: `python3 -m http.server 8000` then open `http://localhost:8000`.
- Docker build + run:
  - Build: `docker build -t clockteach .`
  - Run: `docker run --rm -p 8080:8080 clockteach`
- Cloud Run deploy (recommended, from project root):
  - `gcloud run deploy clock-learning --source . --region us-central1 --allow-unauthenticated`
  - After changes to `index.html`, re-run the deploy command.

## Testing guidance (what to check after edits)
- Verify UI works in both modes (`Learn` and `Quiz`); confirm drag behavior for hour and minute hands on both mouse and touch.
- Verify digital time formatting (`formatDigitalTime`) and quiz checking logic (`checkAnswer`) for AM/PM edge cases.
- Confirm responsive layout (mobile <-> desktop) because layout is inlined and Tailwind classes are used.
- If adding JS/CSS dependencies, prefer CDN usage or add a build step and update README to describe it (there is currently no bundler).

## Deployment and infra notes
- `Dockerfile` uses `nginx:alpine` and exposes `8080` — any runtime must serve on `8080` for consistency with current Cloud Run settings.
- `nginx.conf` SPA behavior uses `try_files … /index.html` — preserve this when modifying server config.

## Small, actionable examples for common edits
- Add a new quick time preset: append to the quick presets array in `index.html`:
  - Example: `{ h: 1, m: 30, label: '1:30' }`
- Add a new learning tip: add another `<div className="flex gap-2">…</div>` item inside the Learning Tips block (search `Learning Tips` comment in `index.html`).
- Change default starting time: update the `useState` initializers for `hours` and `minutes` (top of `ClockLearningApp`).

## What not to change without confirmation
- Replacing in-browser Babel + UMD stacks with a bundler or Node build is allowed, but is a major structural change — ask for confirmation before making that migration.
- Changing the server port or nginx SPA fallback without updating `Dockerfile`/Cloud Run settings may break deployment.

## Questions for maintainers (leave as TODOs in PRs)
- Do you want to keep in-browser Babel for quick edits, or migrate to a bundler + CI build? (affects CI and contributor experience)
- Are there planned accessibility (a11y) goals for keyboard interaction with the clock hands?

---
If anything above is unclear or you want extra details (for example, a suggested migration plan to a modern toolchain or a small test harness), tell me which area to expand and I'll iterate. 
