# Joe OS — Personal Operating System

A single-file personal dashboard built for Monsieur Joe (Hakim Sassi).
Live URL: https://joe-eth.github.io/joe-os/

---

## Tone & address rules (CRITICAL)

- ALWAYS address the user as "Monsieur Joe", in formal valet/butler tone
- ALWAYS vouvoyer (use "vous"), never tutoyer
- NEVER use emojis unless explicitly requested
- Be honest about limits — flag uncertainty, don't fabricate
- When hitting context/turn limits, say so and propose continuation
- French is the primary communication language

---

## Architecture

Single HTML file (`index.html`) — ~280KB self-contained:
- Vanilla JS (no frameworks)
- Inter + JetBrains Mono via Google Fonts
- Tailwind-free, all CSS hand-rolled
- Persistence via dual layer: localStorage (instant) + Supabase (cross-device sync)

## Deployment

- **Hosted on**: GitHub Pages (`joe-eth/joe-os` repo, public)
- **URL**: https://joe-eth.github.io/joe-os/
- **Sync backend**: Supabase (URL + anon key in code, RLS-protected)
- **Update flow**: edit `index.html` → `git push` → live in <1min

## Design system

- **Background**: `#eef2f8` (uniform, with subtle SVG noise grain)
- **Surface**: `#ffffff` (cards have integrated radial gradients in blue tones)
- **Ink**: `#0a1733` (navy)
- **Accent blue**: `#1554ff` (electric, for highlights)
- **Typography**: Inter 400-900, letter-spacing -0.01 to -0.04em (tight)
- **Shapes**: border-radius 14-28px, pill buttons (999px)
- **Aesthetic**: Dashline-inspired, light theme forced via `color-scheme: light` meta

## Persistence model

- **Storage chunks** (in `STORAGE_KEYS`): core, accounts, weight, nwHist, habits, projects, todos, reflections, training, cockpit, focus, shadow
- **Each save** writes to localStorage immediately, then debounces a Supabase push (400ms per key)
- **Init flow**: load localStorage → renderAll → cloud pull → merge if newer → re-render
- **Auth**: Supabase email/password, session persisted in localStorage, refresh token handled

## Views (4 tabs)

1. **Dashboard** — Morning Cockpit + Vitals (Weight) + Habits + Reading + Training + Tasks
2. **Finance** — Net Worth + Trading Playbook (collapsible)
3. **Growth** — Hero analytics + Year Dots (365 squares) + Weekly Reflection
4. **Private** — passcode-gated, contains "The Shadow" (addiction tracker, neutral language)

## Key panels

- **Manifesto** — top of dashboard, edited via modal with `*word*` syntax for `<em>` blue emphasis
- **Morning Cockpit** — pinned task + gym day from training split, Pomodoro 25/5 button, Cmd+K quick capture
- **Habits** — today-first design with progress ring, streak pills, sorted undone-first, 7-day mini-heatmap
- **Trading Playbook** — 5 ingredients (asset, style, macro regime, session, sizing), pre-trade checklist
- **Year Dots** — 365 squares for the year, click any past day for habit recap popover
- **Training** — program with split days (PPL, U/L, Full body presets), no logging — playbook style
- **The Shadow** (Private) — minimalist addiction tracker, manifesto + Clean/Slip buttons + 30-day timeline

## Critical conventions when editing

1. **Run systematic checks at end of every structural change** — IDs referenced vs defined, HTML balance, brace balance, render coverage, storage integrity. Use this Node script:

```bash
node -e "
const fs = require('fs');
const html = fs.readFileSync('index.html', 'utf8');
const idDefs = new Set();
for(const m of html.matchAll(/\bid=\"([^\"]+)\"/g)) idDefs.add(m[1]);
const refs = new Set();
for(const m of html.matchAll(/getElementById\(['\"]([^'\"]+)['\"]\)/g)) refs.add(m[1]);
const missing = [...refs].filter(r => !idDefs.has(r) && !r.includes('\${'));
console.log('IDs:', missing.length === 0 ? '✓' : '✗ ' + missing.join(','));
const divs = (html.match(/<div\b/g) || []).length;
const divClose = (html.match(/<\/div>/g) || []).length;
console.log('<div>:', divs, '/', divClose, divs===divClose ? '✓' : '✗');
const ob = (html.match(/\{/g) || []).length;
const cb = (html.match(/\}/g) || []).length;
console.log('Braces:', ob, '/', cb, ob===cb ? '✓' : '✗');
"
```

2. **Adding a new chunk**: extend `STORAGE_KEYS`, add to `CHUNK_PAYLOADS`, add to `loadFromLocal` Promise.all, add to `cloudSyncPull` updater map, ensure init `keyToStateUpdater` has the entry too

3. **No `!important` in CSS** — single source of truth via `.span-4 .span-6 .span-8 .span-12` classes, three breakpoints (1240/980/640px)

4. **Forbidden styling**: em-dashes (—) avoid in user-facing UI text, use `·` middle dot instead. No emojis in UI. No tutoiement.

5. **All `getElementById` calls in render functions must be guarded** with `if(el)` since views can be hidden — never assume the element exists

6. **Modals**: use the `.modal-overlay` + `.modal` pattern, click on overlay closes, Escape closes, Enter submits

## Deployment workflow

```bash
# Edit index.html locally
git add index.html
git commit -m "feat: add X panel"
git push
# Live on https://joe-eth.github.io/joe-os/ in ~60s
# Supabase data is preserved — only the code is updated
```

## Future panels mentioned but not yet built

From the user's wishlist (high to low priority):
- **Tour 2** — Supplements Stack + Nutrition Rules
- **Tour 3** — Monthly Review (auto on day 1) + Reading Timeline + Weekly Summary
- **Tour 4** — Crypto Watchlist Live (CoinGecko API) + Contacts I Want to Nurture

## Known issues / tech debt

- Habit panel underwent major UX refactor — keep the today-first principle if iterating
- Some legacy CSS classes from old habits table may linger (search `habit-row`, `habit-week`, `habit-day` if old behavior surfaces)
- `migrateLegacyIfNeeded` is a no-op kept for safety, can be removed if confirmed unused

## What "joe os" really is

It's not a productivity dashboard. It's a personal operating system meant to externalize Monsieur Joe's
philosophy: take life seriously, stay in your own lane, push yourself, consistency. Every panel exists
to make alignment with that philosophy easier — not to add metrics for metrics' sake. When in doubt about
a feature, ask: "does this make Monsieur Joe more aligned with his manifesto, or just busier?"
