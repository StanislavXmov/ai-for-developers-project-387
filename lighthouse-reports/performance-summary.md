# Lighthouse performance audit

Run: https://github.com/StanislavXmov/ai-for-developers-project-387/actions/runs/27471069080
Baseline: `docs/performance-baseline.md`
Artifact: `lighthouse-reports`

## Current metrics

| Page | LCP | FID proxy | CLS |
| --- | ---: | ---: | ---: |
| / | 27513 ms | 120 ms | 0.000 |
| /create | 27325 ms | 129 ms | 0.000 |
| /admin | 27613 ms | 10000 ms | 0.000 |

## Trend check

| Page | Metric | Baseline | Current | Degradation |
| --- | --- | ---: | ---: | ---: |
| / | LCP | 2500 ms | 27513 ms | 1001% |
| / | FID | 100 ms | 120 ms | 20% |
| /create | LCP | 2500 ms | 27325 ms | 993% |
| /create | FID | 100 ms | 129 ms | 29% |
| /admin | LCP | 2500 ms | 27613 ms | 1005% |
| /admin | FID | 100 ms | 10000 ms | 9900% |

FID is compared with Lighthouse `max-potential-fid`, because real FID requires field data.