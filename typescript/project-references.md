# TypeScript Project References

- Compiler feature allowing to divide a large TS codebase into smaller, independently compiled and referenced programs.
- No parsing and type-check entire repository as a single monolithic compilation unit, TS compiles projects incrementally using build cache metadata.

## Key components

- `composite: true`: enable composite projects, enforce modular constraints, emit `.d.ts` declaration files, write `.tsbuildinfo` build state file.
- `references: [{ path: "..." }]`: where upstream dependencies are located on the FS.
- `tsc --build/-b`: compiles pre-requisite projects first, skips projects whose outputs and `.tsbuildinfo` files are up-to-date.

```
// libs/shared/tsconfig.json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es2022",
    "composite": true, // enable Composite project
    "declaration": true, // emits .d.ts, mandatory when `composite` is enabled
    "declarationMap": true, // emits .d.ts.amp to allow IDE navigation to source.ts file rather than compiled .d.ts files
    "rootDir": "src",
    "outDir": "dist"
  },
  "include": ["src/**/*"]
}

// apps/api/tsconfig.json
{
  "compilerOptions": {
    "module": "commonjs",
    "target": "es2022",
    "rootDir": "src",
    "outDir": "dist",
    // ...
  },
  "references": [
    {
      "path": "../../libs/shared"
    }
  ]
  "include": "src/**/*"
}
```

## `compilerOptions.paths` can be SSOT, but `references` cannot

- `compilerOptions.paths` is a compiler option. extends shallow-merges `compilerOptions` keys from the base file, so every inheriting project gets the alias map for free.

Path aliases are repository-wide vocabulary (same for everyone), while references are the per-project dependency graph (who consumes whom).

=> Union of all references would force every project to depend on every lib, breaking build order and coupling unrelated project.
=> Per-project edge references are design, not an accident.

- `references` is not a `compilerOptions` entry, it sits alongside `"files"`, `"include"`, `"exclude"`, `"extends"` does not merge or inherit it. **A child TypeScript configuration that omits `"references"` gets an empty list, not the base list**

NOTE: reference path is resolved relative to the directory of the TypeScript configuration that declares it, it's illogical to even think of inheritance logic in the first place.

Q: But then, do we have to manually maintain the per-project references list?
A: Not at all, NX monorepo has synchronization tool for that.
