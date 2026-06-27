# Running Preact at build layer

Preact (a smaller clone of React), but the code everywhere says `import ... from "react"`. The build tool quietly swaps "react" for "preact" when making the real app.

When TypeScript checks your file, the word "react" comes through two different clerks, and each clerk has his own switch on the wall:

Clerk 1: `compileOptions.jsxImportSource`, checks the JSX tags - everything you write as `<div>, <RolesDetail />`.
Clerk 2: `compileOptions.paths` aliases, checks the `import ... from "react"` lines.

Each clerk stamps the things coming through his door with a passport saying which species they are: "React element" or "Preact element". Those two species look almost identical, but not quite - a Preact passport has different required fields.

Q: Why goes to such length?
A: Preact is the one that runs things. When you write code, you met with TypeScript React rule, but when you run, you run as Preact.

Q: Why not just `import ... from "preact"` everywhere?
A: Direct Preact imports would only rename at c4i-fe's side. **The imports inside every 3rd-party dependencies's type definitions cannot be changed to `preact`.** Their `.d.ts` files are written explicitly against the name `"react"`

`paths` alias applies to the whole program: your files and every 3rd-party dependencies' `.d.ts`
E.g., when MUI's `Button.d.ts` does `import ... from "react"`, the import resolves to `preact/compat` instead of `react`.

Q: What if the maintainer written `import { useState } from "preact"` everywhere?
A:

- 1st-party modules - Preact passports
- 3rd-party dependencies (MUI, react-router, react-redux, react-query, ...) - their `.d.ts` say React -> resolve to `@types/react` -> React passports.

=> `ts(2786)` might occured: `VNode` not assignable to `ReactNode/ReactElement` at every single `<Button>`, `<Route>`, `<Provider>`, children prop, hook return, ref...
