Run a performance audit for the main pages of the application.
Compare key metrics (LCP, FID, CLS) with previous results stored in docs/performance-baseline.md.
If any metric degraded by more than 10%, create or update a GitHub issue with specific recommendations for improvement.

Context:
- GitHub Actions run: https://github.com/StanislavXmov/ai-for-developers-project-387/actions/runs/27471069080
- Lighthouse artifact: lighthouse-reports
- Baseline file: docs/performance-baseline.md
- Audited pages: /, /create, /admin
- FID is represented by Lighthouse max-potential-fid because real FID requires field data.

Detected regressions:

| Page | Metric | Baseline | Current | Degradation |
| --- | --- | ---: | ---: | ---: |
| / | LCP | 2500 ms | 27513 ms | 1001% |
| / | FID | 100 ms | 120 ms | 20% |
| /create | LCP | 2500 ms | 27325 ms | 993% |
| /create | FID | 100 ms | 129 ms | 29% |
| /admin | LCP | 2500 ms | 27613 ms | 1005% |
| /admin | FID | 100 ms | 10000 ms | 9900% |

Initial recommendations from Lighthouse diagnostics:

- Review total transferred bytes on Home: 4620 KiB.
- Check heavy resources on Home:
- http://localhost:5173/node_modules/.vite/deps/lucide-react.js?v=f41cbdf9: 1085 KiB
- http://localhost:5173/node_modules/.vite/deps/@mantine_core.js?v=974302d6: 1000 KiB
- http://localhost:5173/node_modules/.vite/deps/react-dom_client.js?v=ab90914d: 802 KiB
- http://localhost:5173/node_modules/@mantine/core/styles.css: 262 KiB
- http://localhost:5173/node_modules/.vite/deps/@tanstack_react-router.js?v=ecfa5aa6: 262 KiB
- Split or remove unused JavaScript on Home.
- Remove unused CSS rules on Home.
- Review total transferred bytes on Create meeting: 4620 KiB.
- Check heavy resources on Create meeting:
- http://localhost:5173/node_modules/.vite/deps/lucide-react.js?v=f41cbdf9: 1085 KiB
- http://localhost:5173/node_modules/.vite/deps/@mantine_core.js?v=974302d6: 1000 KiB
- http://localhost:5173/node_modules/.vite/deps/react-dom_client.js?v=ab90914d: 802 KiB
- http://localhost:5173/node_modules/@mantine/core/styles.css: 262 KiB
- http://localhost:5173/node_modules/.vite/deps/@tanstack_react-router.js?v=ecfa5aa6: 262 KiB
- Split or remove unused JavaScript on Create meeting.
- Remove unused CSS rules on Create meeting.
- Review total transferred bytes on Admin: 4620 KiB.
- Check heavy resources on Admin:
- http://localhost:5173/node_modules/.vite/deps/lucide-react.js?v=f41cbdf9: 1085 KiB
- http://localhost:5173/node_modules/.vite/deps/@mantine_core.js?v=974302d6: 1000 KiB
- http://localhost:5173/node_modules/.vite/deps/react-dom_client.js?v=ab90914d: 802 KiB
- http://localhost:5173/node_modules/@mantine/core/styles.css: 262 KiB
- http://localhost:5173/node_modules/.vite/deps/@tanstack_react-router.js?v=ecfa5aa6: 262 KiB
- Split or remove unused JavaScript on Admin.
- Remove unused CSS rules on Admin.

Create the issue with:
- A concise title, such as "Performance regression detected by Lighthouse".
- The affected pages and degraded metrics.
- Specific recommendations covering slower pages, heavier resources, and extra or wasteful requests.
- A link to the workflow run and a note to download the lighthouse-reports artifact for full HTML reports.
- A checklist of follow-up tasks.
