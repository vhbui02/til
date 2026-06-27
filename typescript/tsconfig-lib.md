# TypeScript configuration: `lib`

`compilerOptions.lib` specify the built-in type definition files (`.d.ts`) included during compilation, providing TypeScript with knowledge of the globals, variables, methods available in JS Runtime Environment.

```ts
{
  "compilerOptions": {
    "lib": [
      "DOM",    // Core browser document object model API: window, document, ...
      "DOM.Iterable", // Iterable implementations for DOM types: run forEach against a NodeList
      "WebWorker",    // Globals and APIs in browser background worker context
      "ES5",    // base
      "ES2015", // Block scoping, Classes, Promises, Maps, Sets
      "ES2016", // Array include feature
      // ...
      "ES2020", // Nullish coalescing, optional chaining
      "ES2021", // String replaceAll, Promise.any, weak references
      "ES2022", // Array .at(), Class fields, top-level await
      // ...
      "ESNext",
    ]
  }
}
```
