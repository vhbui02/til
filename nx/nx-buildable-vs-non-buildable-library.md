# NX buildable vs non-buildable library

Decision matrix:

| Decision Factor | Buildable library | Non-buildable library |
| --- | --- | --- |
| Primary Goal | Distribution & Caching | Code Organization & Speed |
| Consumer Scope | Multiple independent apps or teams (e.g., mobile, webs, ...) with different build pipelines | Single app (libs splitted for cleaning up a large dir structure) |
| Publishing | NPM/Artifactory | Internal only |
| Build Time | Slow initial build; Fast re-builds | Zero overhead; bundled with the App |
| Dependency Graph | Must define `package.json` exports, strict | Shared app's deps, loose |

=> Start with a non-buildable: simpler, no need for rebuilding libraries during development. Refactor to buildable when: publishing is needed, or faster incremental builds in large workspaces is needed.
