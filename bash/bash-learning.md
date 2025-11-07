# Bash Learning

`bash` (Bourne Again Shell) are two of my favorite _command interpreters and command programming languages_ so I can implement a lot of automation workflows on my machine.

## Parameter expansions

```sh
# if args is set and non-empty, sub with $args value
${args:+$args}

# if args is set (allow empty), sub with $args value
${args+$args}

# if args is set and non-empty, sub with 'default'
${args:-default}

# if args is set (allow empty), sub with 'default'
${args-}
```

## `[[` and `test` statements

> [!TIP]
>
> `test` and `[[` are interchangbly. `[[` is non-POSIX since only Bash, Zsh and Ksh implements it.

```sh
# String tests
[[ -z "${1-}" ]]     # String is empty (zero length)
[[ -n "${1-}" ]]     # String is non-empty (non-zero length)

# File existence tests
[[ -e "${1-}" ]]     # File exists (any type)
[[ -s "${1-}" ]]     # File exists and is not empty (recommended)
[[ -f "${1-}" ]]     # Regular file exists
[[ -d "${1-}" ]]     # Directory exists
[[ -L "${1-}" ]]     # Symbolic link exists
[[ -S "${1-}" ]]     # Socket exists
[[ -p "${1-}" ]]     # Named pipe (FIFO) exists
[[ -b "${1-}" ]]     # Block device exists
[[ -c "${1-}" ]]     # Character device exists

# File permission tests
[[ -r "${1-}" ]]     # File is readable
[[ -w "${1-}" ]]     # File is writable
[[ -x "${1-}" ]]     # File is executable

# File ownership tests
[[ -O "${1-}" ]]     # File is owned by effective UID
[[ -G "${1-}" ]]     # File is owned by effective GID

# File attribute tests
[[ -u "${1-}" ]]     # File has setuid bit set
[[ -g "${1-}" ]]     # File has setgid bit set
[[ -k "${1-}" ]]     # File has sticky bit set

# File comparison tests
[[ "${file1}" -nt "${file2}" ]]  # file1 is newer than file2
[[ "${file1}" -ot "${file2}" ]]  # file1 is older than file2
[[ "${file1}" -ef "${file2}" ]]  # file1 and file2 are same file (hard links)

# Numeric comparisons
[[ "${1-}" -eq 0 ]]   # Equal to
[[ "${1-}" -ne 0 ]]   # Not equal to
[[ "${1-}" -lt 10 ]]  # Less than
[[ "${1-}" -le 10 ]]  # Less than or equal
[[ "${1-}" -gt 10 ]]  # Greater than
[[ "${1-}" -ge 10 ]]  # Greater than or equal

# String comparisons
[[ "${1-}" == "value" ]]    # String equality
[[ "${1-}" != "value" ]]    # String inequality
[[ "${1-}" < "value" ]]     # String less than (lexicographic)
[[ "${1-}" > "value" ]]     # String greater than (lexicographic)

# Pattern matching
[[ "${1-}" == pattern* ]]   # Glob pattern matching
[[ "${1-}" =~ regex ]]      # Regular expression matching

# Combined tests
[[ -n "${1-}" && -f "${1-}" ]]  # Non-empty and is a file
[[ -z "${1-}" || ! -e "${1-}" ]] # Empty or doesn't exist
```

## Naming convention

### Variables

- CLI command: `_cmd*` prefix
- CLI option: `_option*` prefix
- Fish argument: `_flag*` prefix

### Functions

- Single leading underscore (Python): `_` prefix
- Double leading underscore (C++, Fish): `__` prefix

## Strict mode

Add this general one-liner at the start of your script:

```bash
set -euxo pipefail
```

### `set -e` or `set -u errexit`

- By default, if one line in a thousand-line script failed, but the last line succeeds. The whole script will have a successful exit code.
- Immediately exit if any commands has a non-zero exit status code.
- It's useless (actually harmful) in REPL session.
- `if` and `[`/`[[` (`[[` are introduced in Bash) are exceptions - they won't terminate the program when these statements resolve to `false`, but short-circuiting operations (`&&`, `||`) will terminate the script:

```bash
# exit
[[ "${DEBUG:-0}" -eq 1 ]] && echo "Debug mode is activated!"

# engineers used to add another short-circuiting at the end to bypass the behavior
[[ "${DEBUG:-0}" -eq 1 ]] && echo "Debug mode is activated!" || true
[[ "${DEBUG:-0}" -eq 1 ]] && echo "Debug mode is activated!" || echo "Debug mode is deactivated!"

# better, use if statements
if [[ "${DEBUG:-0}" -eq 1 ]]; then
  echo "Debug mode is activated!"
fi
```

### `set -x` or `set -o xtrace`

All executed commands are printed into console. Used when debug only.

### `set -u` or `set -o nounset`

All variable subtitution operations will resolve to failure when handling an unset variables (with the exceptions of `$*` and `$@`)

```bash
#!/bin/bash
firstName="Silver"
lastName="Wick"
greeting="Hello, Mr. $firstname" # failed, $firstName, not $firstname
echo "$fullName"
```

### `set -o pipefail`

Prevent error from a pipeline from being masked. That said, by default the exit status of the whole pipeline is depended solely to the success of the last command. Using this option, any commands that failed inside a pipeline will result in the pipeline being failed.

```bash
$ set -o pipefail
$ grep some-string /non/existent/file | sort
grep: /non/existent/file: No such file or directory
$ echo $?
2
```

### `IFS=`

- Internal Field Seperator, or "delimiter", "word-splitting"
- Which character will be used to define what is a word in Bash.

```bash
# a string
local -r items="a b c"

# set to whitespace character
IFS=$' '
items="a b c"
for x in $items; do
    echo "$x"
done
# Here, anything between two whitespaces is consider a word
# It loops 3 times:
# a
# b
# c

# set to newline character
IFS=$'\n'
for y in $items; do
    echo "$y"
done
# Here anything between two newlines is considered a word
# It loops only once:
# a b c
```

## References

- [silverbullet069/bash_strict_mode.md](https://gist.github.com/Silverbullet069/7d34a2523c001d9382080fc533dbc4fe)
- [Unofficial Bash Strict Mode, Maxwell, A., 2018](http://redsymbol.net/articles/unofficial-bash-strict-mode/)

<!-- TODO: read https://mywiki.wooledge.org/BashGuide/Practices -->
