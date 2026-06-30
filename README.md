# 📦 NpmChecker

A tiny, self-contained dashboard for tracking npm packages — downloads, releases, dependents, GitHub stars/PRs, and a heuristic that flags whether traffic looks like **real use vs. bots/mirrors/scanners**.

Everything runs **client-side in the browser**. Your package list and the latest result are stored in `localStorage` on your device; nothing is sent to any backend. The app calls public APIs directly:

- `registry.npmjs.org` — package metadata (latest version, publish date, repo link, description)
- `api.npmjs.org` — daily download history (last 12 months)
- `api.deps.dev` — dependent counts (Google Open Source Insights)
- `api.github.com` — stars and open pull-request count (unauthenticated)

## Use

1. Open the app (or your deployed URL).
2. First run shows a **welcome screen** — paste package names (one per line, or comma/space separated) or upload a `.txt`. Scoped names (`@scope/name`) work; versions are ignored (`lodash@4.17.21` → `lodash`).
3. The dashboard loads. Click any column header to sort.
4. **⚙ Settings** — paste a new list to *replace*, upload to *append*, or *clear all*. **↻ Reload** re-fetches live data.

While collecting, a blocking loader shows a progress bar and a timestamped activity log of every request (and any retries / rate-limits).

## Caching & rate limits

- **Daily cache** — results are cached per day. Reopening the app the same day shows the cached snapshot instantly. Changing the package list refetches automatically. **↻ Reload** always collects fresh data and overwrites the cache.
- **Per-domain request queue** — requests are serialized per host with spacing, so no API is hammered.
- **Retries** — transient failures (network, HTTP 5xx, 429) retry with exponential backoff honoring `Retry-After`. A rate-limited host enters a short cooldown so the rest of the run fails fast (shown as `rate-limited`) instead of stalling. 404s don't retry.

## Reading the dashboard

- **Type** — `primary` (user-facing), `internal` (per-platform build artifact installed automatically by its parent), `legacy` (last publish > 1 year ago). Auto-detected from the package description/name and publish date. The table defaults to primary-first.
- **Use signal** — estimates adoption vs automation from the *shape* of downloads:
  - **looks real** — weekday rhythm (fewer installs on weekends) and a steady non-zero floor.
  - **mostly automated** — flat/weekend-equal traffic, very few active days, or a single post-publish spike — the signature of registry mirrors and security scanners (Socket, Snyk, jsDelivr, npm replication, corporate proxies).
  - **mixed** / **none**. It's a heuristic, not proof — the **Dependents** column is the stronger signal.
- **Dependents** — how many other packages depend on this one (total, with direct count below), from [deps.dev](https://deps.dev). Non-zero direct dependents indicate genuine real-world use.
- **Peak day / bursts** — biggest single-day count and date; bursts are highlighted and usually reflect mirror/scanner traffic rather than installs.

## Deploy

Pure static, no build step. Any static host works.

### Cloudflare Pages

- **Framework preset:** `None`
- **Build command:** *(leave empty)*
- **Build output directory:** `/`

The included `_headers` file is applied automatically (security headers + a Content-Security-Policy scoped to the APIs the app calls). If you add another data source, add its origin to the `connect-src` list in `_headers`.

### Other hosts

- **GitHub Pages:** deploy from branch (root). (`_headers` is ignored there; the app still works.)
- **Netlify:** drag-and-drop or connect the repo; no build command, publish dir = root. (`_headers` works.)
- **Any web server:** copy the files to the web root.

## Privacy

No accounts, no tracking, no backend. The package list and cached results never leave your browser. API calls go directly from your browser to the public services listed above.

## License

[MIT](LICENSE) © CombyCode Inc.
