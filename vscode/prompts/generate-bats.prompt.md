---
mode: 'agent'
description: 'A comprehensive prompt to generate Bash Automated Testing System unit tests.'
tools: ['changes', 'codebase', 'editFiles', 'extensions', 'fetch', 'findTestFiles', 'githubRepo', 'new', 'openSimpleBrowser', 'problems', 'runCommands', 'runNotebooks', 'runTasks', 'runTests', 'search', 'searchResults', 'terminalLastCommand', 'terminalSelection', 'testFailure', 'usages', 'vscodeAPI', 'context7']
---

## Your goal

Set up test cases for the Bash script that's also a command-line application (now referred to as "target script").

## Tech stack

The chosen testing framework is Bash Automated Testing System or BATS, [bats-core](https://bats-core.readthedocs.io/)

The chosen testing auxiliary libraries are:
- [bats-support](https://github.com/bats-core/bats-support/blob/master/README.md)
- [bats-assert](https://github.com/bats-core/bats-assert/blob/master/README.md)
- [bats-file](https://github.com/bats-core/bats-file/blob/master/README.md)

use context7

## Requirements

- You MUST use `$BATS_SUITE_TMPDIR` - a temporary directory common to all tests of a suite - to create files required by multiple tests.
- You MUST use `$BATS_FILE_TMPDIR` - a temporary directory common to all tests of a test file -  to create files required by multiple tests in the same test file.
- You MUST use `$BATS_TEST_TMPDIR` - a temporary directory unique for each test - to create files required only for specific tests.
- You MUST clone the target script and create all mock dependencies in `$BATS_FILE_TMPDIR`. This is to ensure the state of the project directory remains intact after test execution.
- You MUST pay attention to not creating duplicate files and directories while doing provision to `$BATS_FILE_TMPDIR`.
- You MUST leave the `teardown()` function blank:
    ```bash
    teardown() {
        :
    }
    ```
- DO NOT source the target script, unless the test case requires internal function direct invocation.
- DO NOT duplicate test case (i.e. 2 test cases testing the same logic).
- DO NOT use destructive operations (e.g. `mv -f`, `cp -f`, `ln -f`, ...).
- DO NOT back up the original working directory in the `setup()` function.
- DO NOT change current working directory to `$BATS_TEST_TMPDIR`, instead, every file operation (copying, moving, changing owner, changing permissions, executing, etc.) MUST construct its own path using `$BATS_TEST_TMPDIR`.

## Examples

```bats
#!/usr/bin/env bats

setup() {
    export BATS_LIB_PATH="${BATS_TEST_DIRNAME}/test_helper"
    bats_load_library bats-support
    bats_load_library bats-assert
    bats_load_library bats-file

    export SCRIPT_UNDER_TEST=$(realpath "${BATS_TEST_DIRNAME}/../script.sh")
    assert_file_exists "${SCRIPT_UNDER_TEST}"
}

teardown() {
    :
}

@test "Script runs without arguments" {
    run "${SCRIPT_UNDER_TEST}"
    assert_success
}

@test "Script handles -h, --help option" {
    run "${SCRIPT_UNDER_TEST}" --help
    assert_success
    assert_output --partial "Usage:"
    assert_output --partial "Options:"

    run "${SCRIPT_UNDER_TEST}" -h
    assert_success
    assert_output --partial "Usage:"
    assert_output --partial "Options:"
}
```
