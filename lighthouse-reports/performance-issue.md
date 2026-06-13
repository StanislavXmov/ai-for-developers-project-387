# Performance regression detected by Lighthouse

The scheduled Lighthouse audit found that **every audited page regressed by far more than the 10% threshold** on LCP and the FID proxy (`max-potential-fid`). CLS is unaffected (0.000 on all pages).

- **Workflow run:** https://github.com/StanislavXmov/ai-for-developers-project-387/actions/runs/27471069080
- **Baseline:** `docs/performance-baseline.md`
- **Full HTML reports:** download the **`lighthouse-reports`** artifact from the workflow run (contains `home.report.html`, `create.report.html`, `admin.report.html`).
- FID is approximated with Lighthouse `max-potential-fid` because real FID needs field data.

## Affected pages and degraded metrics

| Page | Metric | Baseline | Current | Degradation |
| --- | --- | ---: | ---: | ---: |
| `/` | LCP | 2500 ms | 27513 ms | **+1001%** |
| `/` | FID proxy | 100 ms | 120 ms | **+20%** |
| `/create` | LCP | 2500 ms | 27325 ms | **+993%** |
| `/create` | FID proxy | 100 ms | 129 ms | **+29%** |
| `/admin` | LCP | 2500 ms | 27613 ms | **+1005%** |
| `/admin` | FID proxy | 100 ms | 10000 ms | **+9900%** |

## Which pages slowed down and why

- **`/admin` is the worst offender for interactivity.** Its main-thread work jumped to **13.4 s** and JS bootup to **11.1 s** (vs ~0.4–0.5 s on the other pages). A single long task dominates: `react-dom_client.js` runs for **~12.5 s** (11.0 s scripting). That long task is what pushes the FID proxy to the 10000 ms ceiling.
- **All three pages have a ~27 s LCP.** This is consistent across pages and tracks with the very large, unbundled JavaScript payload being parsed/executed before content paints.

### Important context: the audit ran against the Vite **dev** server
The diagnostics reference `http://localhost:5173/@vite/client` and unminified, individually-served `node_modules/.vite/deps/*` modules (**25 separate dev-dependency requests** on `/`). Dev builds are unminified, unsplit, and shipped per-module, which inflates bytes, request count, and execution time dramatically. **First action: confirm whether the audit should target a production `vite build` + `vite preview` instead of the dev server** — the absolute numbers will not be representative until it does. The relative regressions below are still real and worth fixing regardless.

## Which resources became heavier

Total transfer is **~4,620 KiB** on every page. The heaviest resources (from `admin.report.json`, near-identical on all pages):

| Size | Resource |
| ---: | --- |
| 1,084 KiB | `.vite/deps/lucide-react.js` |
| 999 KiB | `.vite/deps/@mantine_core.js` |
| 801 KiB | `.vite/deps/react-dom_client.js` |
| 262 KiB | `@mantine/core/styles.css` |
| 261 KiB | `.vite/deps/@tanstack_react-router.js` |
| 245 KiB | `.vite/deps/react-day-picker.js` |
| 205 KiB | `@vite/client` (dev-only) |
| 97 KiB | `.vite/deps/@tanstack_react-query.js` |

## Extra / wasteful requests Lighthouse flagged

- **Unused JavaScript:** ~**1,580–1,615 KiB** of wasted JS per page. Biggest offenders: `@mantine_core.js` (**793 KiB wasted**), `react-dom_client.js` (320 KiB), `react-day-picker.js` (116 KiB), `@tanstack_react-router.js` (85 KiB), `@tanstack_react-query.js` (84 KiB).
- **Unused CSS:** ~**47 KiB** wasted, primarily from `@mantine/core/styles.css` (46 KiB).
- **Request fan-out:** 56 / 57 / 58 requests on `/` / `/create` / `/admin`, with **25 dev-dependency module requests** on the Home page alone — a side effect of serving unbundled dev modules.

## Recommendations

1. **Audit a production build, not the dev server.** Run Lighthouse against `vite build` + `vite preview` so metrics reflect minified, code-split, bundled output. This alone should collapse byte weight, request count, and bootup time.
2. **Investigate the `/admin` long task.** The 11 s of scripting attributed to `react-dom_client` suggests an expensive synchronous render or a hot loop on mount (e.g., rendering a very large list/table without virtualization, or a heavy effect running on every render). Profile the admin route and defer/virtualize heavy work.
3. **Code-split heavy dependencies.** `lucide-react` (1 MB), `@mantine/core` (1 MB), and `react-day-picker` are largely unused per page. Import icons individually instead of the barrel, lazy-load Mantine-heavy and date-picker components with `React.lazy`/dynamic import, and route-split so `/admin` doesn't ship `/create`'s dependencies and vice versa.
4. **Trim unused JS (~1.6 MB).** Tree-shake `@mantine/core` (import only used components), drop unused `lucide-react` icons, and verify `@tanstack` packages aren't pulled into every route.
5. **Trim unused CSS (~47 KiB).** Scope/purge `@mantine/core/styles.css` to the components actually used.
6. **Reduce request fan-out.** A production build bundles the 25+ dev-dep modules; confirm this is resolved post-build and avoid importing whole libraries.

## Follow-up checklist

- [ ] Confirm Lighthouse target (production preview vs dev server) and re-run if needed.
- [ ] Profile and fix the ~11 s scripting long task on `/admin`.
- [ ] Code-split `lucide-react`, `@mantine/core`, and `react-day-picker`.
- [ ] Remove unused JavaScript (~1.6 MB) and CSS (~47 KiB).
- [ ] Re-run the audit and verify LCP/FID return to baseline (LCP ≤ 2500 ms, FID ≤ 100 ms).
- [ ] If the new numbers are an accepted, intentional change, update `docs/performance-baseline.md`.
