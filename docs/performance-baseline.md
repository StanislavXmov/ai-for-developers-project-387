# Performance Baseline

This file stores accepted Lighthouse baseline metrics for the main pages.
The scheduled workflow compares new audits against these values and opens an issue when LCP, FID proxy, or CLS degrades by more than 10%.

| Page | LCP | FID | CLS |
| --- | ---: | ---: | ---: |
| / | 2500 | 100 | 0.100 |
| /create | 2500 | 100 | 0.100 |
| /admin | 2500 | 100 | 0.100 |

Notes:
- LCP and FID values are in milliseconds.
- FID uses Lighthouse `max-potential-fid` as a lab-data proxy.
- CLS is unitless.
- Update this file only after the team accepts an intentional performance change.
