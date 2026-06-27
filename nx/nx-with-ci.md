# NX with CI

- Remote Caching (NX replay): code is never rebuilt/retested unnecessary
- Distributed Task Execution (NX agents): allocates tasks across CI machines
- Atomizer: split test suites and parallelize them in NX agents.
- Flaky Task Detection: identify flaky tests (often unit/e2e) and re-runs them

```
myshop/
|-- apps/
|   |-- shop/          # Main application
|   |-- admin/         # Admin interface
|   |-- docs/          # Documentation site
|   |-- landing/       # Marketing site
|   \-- api/           # Backend (JS)
\-- packages/
    |-- products/     # Shared product domain (used by both front and backend)
    |-- orders/       # Shared order management
    \-- shared/       # Common utilities and types
```
