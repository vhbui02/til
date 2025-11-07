# Git Best Practices

<!-- tl;dr starts -->

Learn how to use Git properly before doing anything else is my motto.

<!-- tl;dr ends -->

## Cheatsheet

Most commonly used `git` subcommands:

```sh
# mark commit, which can be referred later on merging, rebasing, ...
# versioning is one of its use cases
git tag ...

# delete tag local
git tag -d v1.2.3
# delete tag remote
git push origin :refs/tags/v1.2.3

# add content to commit without updating the SHA-1 of the commit
# super useful
git notes add -m "Note #1"
git notes add -f -m "Note #1 updated"
git push origin "refs/notes/*"

# see the list of files which were updated during 2 commits
git whatchanged ...

# remove a file from version control but keeps it in working directory
git rm --cached /path/to/file

# call `git stash drop` when prior `git stash pop` create conflicts
git stash pop
git stash drop
```

## Branching strategy + Clean commit history

For me, I use **a hybrid of Gitflow and Feature branching**:

## Branch protection rules

Before a PR can be merged:

1. Require pull request reviews: a certain numbers of reviewers approve the PR.
1. Require status checks.

   - Status checks can be divided into 2 types: Checks and Commit Statuses.
   - Checks provide line annotations, more detailed messaging, is only available for use with GitHub Apps.
   - GitHub Actions generates Checks when workflows are run.
   - A repository can be set to automatically request status checks for pushes.
   - Organization owners + Users with Push access can create status checks with GitHub's API.

1. Require conversation resolution: all conversations on a PR must be resolved.
1. Require signed commits: all commits must be signed with a verified signature
1. Require linear history: the commit history of a PR must be linear
1. Require merge queue: the PR are merged using a merge queue using GitHub Actions.
1. Require deployments to succeed: deployments to production environment must succeed.
1. Do not allow bypassing
1. Restrict who can push to matching branches.
1. Disallow force pushes and deletions (default).

## .gitignore

- Install []()
- Logs, caches, temporary files. Including test results, test logs
- Build artifacts
- Sensitive information (`.env` files, SSH private key, ... )
  > NOTE: do not hardcode API Key into codebase, you will regret it later!
- Large files (media, binary, compressed archive, ...). Use Git LFS.
- Install [CodeZombie's VSCode Extension](https://marketplace.visualstudio.com/items?itemName=codezombiech.gitignore)

## .gitmodules

Remove submodule

```sh
# cre: https://gist.github.com/myusuf3/7f645819ded92bda6677

# remove the submodule entry from .git/config
$ git submodule deinit -f path/to/submodule # or nano .git/config and remove the modules

# remove the submodule directory from the suporproject .git/modules directory
$ rm -rf .git/modules/path/to/submodule

# remove the entry in .gitmodules + the submodule directory in filesystem
$ git rm -f path/to/submodule
# or nano .gitmodules, remove the entries + rm -rf path/to/submodule

# (optional) if you have commited the submodule previously, re-commit
$ git commit -m "removed submodule"
```

## GitHub App

A type of integration that

## GItHub Actions for CI/CD

Here are the things that you can automate:

- Build
- Tests
- Deploy
- Releases
- Documentation
- IaC
- Security check
- ... and more

## GitHub Issues + GitHub Projects for Project Management

GitHub Issues is a powerful issue tracking system that allow you to manage and track issues:

- Feature requests.
- Bugs.
- FAQs.
- ...

GitHub Projects is a tool for planning and tracking work on GitHub, more specifically, issues from GitHub Issues.

- Plan and prioritize issues (MoSCoW/Eisenhower Matrix)
- Use Kanban board view.
- Use milestones to manage releases.
- Collaborate and communicate with team members.
- Use labels when categorizing issues.

## GitHub security features

1. Enable Dependency Graph

   - It analyzes the manifest and lock files in a repository, in order to help users understand the upstream packages that their software project depends on.
   - It can analyze both direct dependency and transitive dependency (i.e. the dependency of your direct dependency).
   - GitHub's Dependency Graph can display transitive dependency if package ecosystems support display them in their dependency graph.
   - In some ecosystems, the resolution of transitive dependencies only occurs at build-time, therefore the contents of the repository alone can't help GitHub discovering all dependencies.

   => Enable Automatic Dependency Submission can help this problem. **However, you must maintain a `pom.xml` file in your project, and `pom.xml` is only existed in a Maven project.**

1. Web UI > Security: Security Overview
   GitHub have a very nice dashboard to control every security aspects of this repository, some notables are:

   - Dependabot alerts
   - Code scanning alerts
   - Secret scanning alerts

1. GitHub's Secret Scanning: It's not uncommon to accidentally commit sensitive credentials like API Keys and passwords.

1. Code Scanning: CodeQL is semantic analysis engine that scans your code for potential vulnerabilities (e.g. SQL Injection). A default config is more than enough. 3rd-party code scanning tools can further enhance your code's security such as: SonarCloud, TfSec (Terraform), trivy, Snyk, ...

1. Enable Dependabot Security Updates

   - It monitors your dependencies and sends alerts when it encounters any vulnerabilities. It can automatically update your dependencies to the latest secure version, depending on how serious the vulnerabilities are.

   - Not only security updates does it run but also non-security updates such as bumping out-of-date dependency version.

   - Dependabot uses a YAML file called `dependabot.yml` stored at `.github/dependabot.yml` in the default branch to maintain dependencies using version updates, it can change how Dependabot creates PR for security updates.

   When `dependabot.yml` gets added or updated, Dependabot will immediately trigger status check

   ```yml
   # To get started with Dependabot version updates, you'll need to specify which
   # package ecosystems to update and where the package manifests are located.
   # Please see the documentation for all configuration options:
   # https://docs.github.com/code-security/dependabot/dependabot-version-updates/configuration-options-for-the-dependabot.yml-file

   version: 2
   updates:
     # define a package manager to update
     - package-ecosystem: "npm"
       # Docs: https://github.blog/changelog/2024-04-29-dependabot-multi-directory-configuration-public-beta-now-available/
       directories:
         - "/" # bats-core
         - "/tests/test_helper/bats-*" # bats library
       schedule:
         interval: "weekly"
       open-pull-requests-limit: 2
       reviewers:
         - "Silverbullet069"
       commit-message:
         prefix: "test"
         include: "scope"
       #target-branch: "main"
   ```

## Reference

- [Securing Your Code With GitHub, Marcel L, 2023-09-06](https://dev.to/pwd9000/securing-your-code-with-github-3le0)
- [GitHub Repository Best Practice, Marcel L, 2024-02-07](https://dev.to/pwd9000/github-repository-best-practices-23ck)
