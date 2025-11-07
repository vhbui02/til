# Semantic Commit Message

<!-- tl;dr starts -->

There are 2 subjects regarding the practice of writing a good commit messages: Semantic Versioning (SemVer) and Conventional Commits.

<!-- tl;dr ends -->

## Semantic Versioning (SemVer)

_Definition:_ Given the version number `MAJOR.MINOR.PATCH`, increment the:

- `MAJOR` version when you make incompatible API changes.
- `MINOR` version when you add functionality in a backward compatible manner.
- `PATCH` version when you make backward compatible bug fixes.

## Conventional Commits

Each commit message consists of:
- Header: a type, an optional scope and a description
- Body.
- Footer: less common, reserved for `BRECKING CHANGE: ` or `note: ` 

```txt
<type>(<scope>): <description>
<BLANK LINE>
<body>
<BLANK LINE>
<footer>
```

### Type

A single noun indicating the subject of the change:

<!-- prettier-ignore -->
| Type      | SemVer | Description |
|-----------|--------|-------------|
| `<type>!:`| MAJOR  | Introduce a breaking API change. |
| `feat:`   | MINOR  | Introduce a feature for the user (not a new feature for build script). |
| `test:`   | MINOR  | Add new tests for new features, change old tests to fix regression tests, add/correct missing tests. |
| `docs:`   | MINOR  | Change to the documentation. |
| `fix:`    | PATCH  | Patch a bug in implementation (aside build script). |
| `chore:`  | PATCH  | Anything that doesn't change the state of the codebase (e.g. renaming, formatting, whitespace fixes). |

Examples:

```
feat!: send an email to the customer when a product is shipped

--------------------------------------------------------------------------------

feat: allow provided config object to extend other configs

BREAKING CHANGE: `extends` key in config file is now used for extending other config files

--------------------------------------------------------------------------------

chore!: drop support for Node 6

BREAKING CHANGE: use JavaScript features not available in Node 6.
```

> **NOTE:** Refrain from making breaking changes before considering the cost of users having to migrate.

##### Angular types

<!-- prettier-ignore -->
| Type        | SemVer | Description |
|-------------|--------|-------------|
| `build:`    | MINOR  | Changes that affect the build system or external dependencies. |
| `ci:`       | MINOR  | Changes to CI configuration files and scripts. |
| `pref:`     | MINOR  | Changes that improve performance. |
| `refactor:` | PATCH  | Rename variables, deduplicate code, etc. |
| `style:`    | PATCH  | Formatting, remove whitespace, etc. |
| `revert:`   | PATCH  | Revert a previous commit. The commit SHAs being reverted go in footer. |

> IMO, `style:` + `refactor:` is a sub-set of `chore:`

```
revert: let us never again speak of the noodle incident

Refs: 676104e, a215868
```

```
docs: correct spelling of CHANGELOG
```

### Scope

The name of the dependency affected if you're developing and maintaining for a framework that consists of multiple libraries/packages. If you're developing a zero/few dependencies project, you can ignore scope.

```
feat(lang): add Vietnamese language
```

### Description

Rules:

- one succint sentence written in imperative mood (i.e. verb in present tense, no pronoun)
- no capitalize the first letter
- no full stop.
- refs, closes to issues, bugzilla tickets (if any). By doing so, you can access issue/PRs link in Web UI.

**NOTE:** Each line in a commit message should not exceed 80/100 characters since it allows the message to be easier to read on GitHub and various git tools.

> I've seen long commit messages hidden on Web UI.

````
chore: initial release    # good
chore: Initial release.   # bad

feat: add ```--no-colour``` option, closes #1    # good
feat: added ```--no-colour``` option, closes #1  # bad
````

### Body and Footer

A full-fledge commit message:

```
fix: prevent racing of requests

Introduce a request id and a reference to latest request. Dismiss
incoming responses other than from latest request.

Remove timeouts which were used to mitigate the racing issue but are
obsolete now.

Reviewed-by: Z
Refs: #123
```

### Tips

You should use a commit message linter, such as [conventional-changelog/commitlint](https://github.com/conventional-changelog/commitlint) to force you (and your team members) to write conventional commit messages.

Keeping the commit messages following conventional can allow you to automatically generate beautiful changelog and release note.

## Reference

- [Samuel-Zacharie Faure's "How atomic Git commits dramatically increased my productivity - and will increase yours too"](https://dev.to/samuelfaure/how-atomic-git-commits-dramatically-increased-my-productivity-and-will-increase-yours-too-4a84)
- [Semantic Versioning 2.0.0](https://semver.org)
- [joshbuchea/semantic-commit-messages.md](https://gist.github.com/joshbuchea/6f47e86d2510bce28f8e7f42ae84c716)
