# @nx/eslint plugin

The `@nx/eslint` plugin will create a task for any project that has an ESLint configuration file present (e.g., `eslint.config.js`, ...) and files to lint.

ESLint applies configuration files to all subdirectories, the `@nx/eslint` plugin will also infer tasks for projects in subdirectories.

=> A task in root ESLint config will be inferred in every project.

Inferred ESLint task is inferred if there are files to lint. Otherwise, the task will not be created.

CAUTION: Since NX v18, the plugin bypasses the need for `lintFilePatterns` inside `project.json` and dynamically tells Nx how to run ESLint based on ESLint configuration file
