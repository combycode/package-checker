# 📦 PackageChecker

A tiny, self-contained dashboard for tracking packages across **npm, PyPI and crates.io** — downloads, releases, dependents, GitHub stars/PRs, and a heuristic that flags whether traffic looks like **real use vs. bots/mirrors/scanners**.

Everything runs **client-side in the browser**. Your package list, filter choices and the latest result are stored in `localStorage` on your device; nothing is sent to any backend. The app calls public APIs directly:

| Ecosystem | metadata | downloads | dependents |
|---|---|---|---|
| **npm** | registry.npmjs.org | api.npmjs.org (~365d daily) | deps.dev |
| **PyPI** | pypi.org/pypi/json | pypistats.org via corsfix proxy (~180d daily) | deps.dev |
| **crates.io** | crates.io/api/v1 | crates.io /downloads (~90d daily) | deps.dev |

GitHub stars + open-PR counts come from `api.github.com` (unauthenticated) for any package whose repo is known.

## Use

1. Open the app (or your deployed URL).
2. First run shows a **welcome screen** — paste package names or upload a `.txt`.
   - One per line (or comma/space separated).
   - **Prefix the ecosystem**: `pypi:requests`, `crates:serde`. A bare name is npm.
   - Aliases: `py:` / `pip:` → PyPI, `cargo:` / `rs:` → crates.
   - Scoped npm names work (`@scope/name`); versions are ignored (`lodash@4.17.21` → `lodash`).
3. The dashboard loads. Click any column header to sort (defaults to primary-first).
4. **⚙ Settings** — edit the list, upload & append, clear all, and set the **Popular thresholds**. **↻ Reload** re-fetches live data.

While collecting, a blocking loader shows a progress bar and a timestamped activity log of every request (and any retries / rate-limits).

## Filters

A chip bar above the chart filters the **whole view** (KPIs, chart and table all recompute):

- **Registry**: `All · npm · PyPI · crates`
- **Primary only** — hide internal build artifacts and legacy packages.
- **Real usage** — signal "looks real" OR ≥1 direct dependent.
- **Popular** — meets **any** of the thresholds set in Settings: dependents > N, stars > N, or downloads/day > N (downloads/day = 30-day total ÷ 30). Defaults: 10 / 1000 / 1000.
- **Search** by name.

Filters combine (AND) and are remembered between visits. "Clear" resets them.

## Caching & rate limits

- **Daily cache** — results are cached per day. Reopening the same day shows the cached snapshot instantly. Adding packages (paste / upload / append) fetches **only the new ones** and reuses the cache for the rest. **Reload** always re-fetches everything and overwrites the cache.
- **Per-domain request queue** — requests are serialized per host with spacing, so no API is hammered.
- **Retries** — transient failures (network, 5xx, 429) retry with exponential backoff honoring `Retry-After`. A rate-limited host enters a short cooldown so the run fails fast (`rate-limited`) instead of stalling. 404s don't retry.
- **CORS note** — npm, PyPI metadata (pypi.org), deps.dev and GitHub allow cross-origin requests. PyPI **downloads** come from pypistats.org, which sends no CORS header, so they're routed through the free [corsfix](https://corsfix.com) proxy (`proxy.corsfix.com`). crates.io **downloads** may also lack CORS on some hosts; if so that column shows an error. Metadata, dependents and stars are unaffected.

## Reading the dashboard

- **Type** — `primary`, `internal` (npm per-platform build artifact), `legacy` (last publish > 1 year). Auto-detected.
- **Use signal** — estimates adoption vs automation from the *shape* of downloads (weekday rhythm & steady floor = looks real; flat/weekend-equal or single post-publish spike = mostly automated). Heuristic, not proof — **Dependents** is the stronger signal.
- **Dependents** — how many packages depend on this one (total + direct), from [deps.dev](https://deps.dev).
- **Total** — downloads over each registry's available window (shown in days beneath the number).
- **Peak day / bursts** — biggest single day; bursts are highlighted and usually reflect mirror/scanner traffic.

## Deploy

Pure static, no build step.

### Cloudflare Pages

- **Framework preset:** `None`
- **Build command:** *(leave empty)*
- **Build output directory:** `/`

The included `_headers` file is applied automatically (security headers + a CSP scoped to the APIs the app calls). If you add another data source, add its origin to the `connect-src` list in `_headers`.

### Other hosts

- **GitHub Pages:** deploy from branch (root). (`_headers` ignored there; app still works.)
- **Netlify:** drag-and-drop or connect the repo; no build command, publish dir = root.
- **Any web server:** copy the files to the web root.

## Privacy

No accounts, no tracking, no backend. Your list, filters and cached results never leave your browser. API calls go directly from your browser to the public services listed above.

## Credits

[CORS proxy by Corsfix](https://corsfix.com) — used to fetch PyPI download stats (pypistats.org) from the browser.

## License

[MIT](LICENSE) © CombyCode Inc.
