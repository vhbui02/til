# NX learning

Best practices:

- Default layout are recommended, not mandatory. Re-organize projects, including flat structure, nested directory, organization patterns, ...

- Non-JS workspaces have their own convention
+ Gradle multi-project builds: auto-detect, provides caching, analysis, task orchestration, ...
+ uv monorepo.

- Solution-style project references pattern: Three levels of `tsconfig.json` files. Benefits: give IDE accurate type info per-project; enable incremental bujilds; create clear boundary between projects.

Responsiblities:

- NX auto-rediscover projects every time `nx` command is run.

- Package manager workspaces handle linking between projects, technically, create symlinks to local packages/libs into `node_modules` so they can be imported like npm packages.

- NX tracks dependencies, builds project graph, determines build order, knows that's affected by a change. NX DOES NOT install/resolve dependencies, that's package manager's job.
