# Testing tools comparison

1. Jest (v30+)
- unit/integration testing tool
- use cases: business logic, utility functions, custom hooks, UI components
- in watch mode, it scans the full project to rebuild a module map. Seconds to wait on large test suites.
- typescript and esm support: ts-jest

2. Vitest (v4+)
- unit/integration testing tool
- if using Vite already, you don't need for another configuration file, reuse existing `vite.config.js`
- watch mode is enabled by default, use HMR which is 8-10x faster than Jest
- cover common features when setting up unit tests: mocking, snapshots, coverage, ...
- performant: use Worker threads as much as possible.

3. Playwright

- browser-based/e2e testing tool
- test complete user workflows (e.g., logging in, navigating app, submitting forms, verifying results, ...) and cross-browser compatibility

4. React Testing Library

- Component testing tool
- test individual React components worked correctly in isolation
- dev can render a single component (e.g., a button, a form, a nav bar), simulate user actions and assert the output

- note that it's much simpler than E2E testing tool: 
+ no browser overhead: it uses `jsdom`, a JS-based mock browser. Tests don't run inside a browser.
+ no network lag, tests run in ms.
+ no flaky artificial sleep timers via `setTimeout()` like E2e tools. Virtual DOM updates instantly.
+ no need to log-in or navigate through 10 pages just to test a single button.
