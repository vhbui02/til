# Vendoring 3rd party source files into private source files

Place a preserved JSDoc header banner containing SPDX license tags and upstream provenance metadata at the top of each file:

```js
/**
 * @file WHEP Client implementation // High-level description of module responsibility.
 * @license MIT                     // Instructs bundlers (esbuild, Rollup, ...) to retain license block during build steps
 * @preserve
 * SPDX-License-Identifier: MIT     // Machine-readable identifier standardizing compliance tracking
 * Package: whip-whep-js            // Upstream distribution identifier
 * Upstream: https://github.com/medooze/whip-whep-js  // Canonical repository location
 * Commit: 1bd7fceda4b51d8a591d8e233e05f46af5936267   // Exact snapshot commit SHA to enable precise diffing and update
 * Vendored: 2026-08-17                               // Timestamp when the snapshot was pulled
 * Modification: Unmodified upstream source           // whether the snapshot was modified
 */
```

