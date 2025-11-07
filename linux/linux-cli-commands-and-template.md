# Linux CLI templates

<!-- tl;dr starts -->

There are a lot of useful CLI commands and templates that I would like to archive for later use. Later, it's better to add the CLI commands into [`tealdeer-rs/tealdeer`](https://github.com/tealdeer-rs/tealdeer) or use Fish history or Bash history `tac ~/.bash_history | grep "pattern" | less`

<!-- tl;dr ends -->

## Extract the Top Ten Largest Directories by Scanning from Root

```sh
sudo du -h --exclude=/mnt / 2>/dev/null | sort -hr | head -10
sudo du -sh ~/.local/share/docker
mv docker docker.bak
sudo btrfs subvolume create $PWD/docker
sudo cp -a docker.bak/. docker/

# do the same with large directory
```

## Use both Parameter Expansion `${}` and Double Quotes `""`

### Parameter expansion `${}`

- Delimits variable names clearly, makes it easier to append text to variables:
  ```sh
  echo "${var}text"   # correct
  echo "$vartext"     # incorrect
  ```
- Required for accessing array elements, variable indirection, and parameter substitutions:

  ```sh
  # Array access
  echo "${array[0]}"

  # Variable indirection (using the value of a variable as another variable name)
  var_name="my_var"
  echo "${!var_name}"

  # Parameter substitution
  echo "${filename%.txt}"   # Remove .txt extension
  echo "${path##*/}"        # Get filename from path
  echo "${var:-default}"    # Use default if var is unset
  ```

- Helps prevent ambiguity in variable names:

  ```sh
  # Without ${}, this would be unclear
  user="admin"
  echo "${user}_home"  # Outputs: admin_home

  # Prevents errors with non-existent variables
  empty=""
  echo "Value: ${empty:-Not provided}"  # Outputs: Value: Not provided

  # Using nested variables
  prefix="my"
  suffix="var"
  echo "${!prefix_$suffix}"  # Accesses the variable named my_var
  ```

### Double Quotes `""`

> **NOTE:** Double quotes are still necessary even with ${}

- Without quotes, values with spaces or special characters will be word-split and glob-expanded:

  ```sh
  # Word splitting with spaces
  name="John Doe"
  echo $name      # Outputs: John Doe as separate arguments
  echo "$name"    # Outputs: John Doe as one string

  # Globbing with wildcards
  file="*.txt"
  echo $file      # Lists all .txt files in current directory
  echo "$file"    # Outputs: *.txt as literal text

  # Command substitution
  output=$(ls -l)
  echo $output    # Flattens output, loses formatting
  echo "$output"  # Preserves line breaks and spacing

  # Preserving special characters
  path="/tmp/my file.txt"
  cat $path       # Tries to open two files: "/tmp/my" and "file.txt"
  cat "$path"     # Correctly opens "/tmp/my file.txt"
  ```

## Differences between `local -r VARIABLE_NAME` and `readonly VARIABLE_NAME`

| Feature        | `local -r`            | `readonly`                 |
| -------------- | --------------------- | -------------------------- |
| Scope          | Function-local        | Global                     |
| Usage location | Functions only        | Anywhere inside the script |
| Persistence    | Until function exists | Until shell exists         |
| Can be unset   | No                    | No                         |

## Differences between `curl -o [FILE]` and `curl > [FILE]`:

| Feature                                  | `curl -o [FILE]` | `curl > [FILE]` |
| ---------------------------------------- | ---------------- | --------------- |
| Built-in?                                | Yes              | No, shell       |
| Create parent directories?               | Yes              | No              |
| Retain file's permission and timestamps? | Yes              | No, umask       |
| Clean leftover files if download failed? | Yes              | No              |

## Run a command against a set of files

`find -print0` outputs the names of found files separated by ASCII NULL character, instead of newlines. This is useful when handling filenames that contain spaces, newlines, ... whitespaces in general.

`xargs -0` safely process filename with special character by parsing them using ASCII NULL

```sh
find /path/to/search -name "[PATTERN]" -print0 | xargs -0 COMMAND
```

## Initial CLI options

```sh
while [ -n "$1" ]
do
  case "$1" in
    "-a")
      shift
      OPT_FOO=1
      ;;
    "-b")
      shift
      OPT_BAR=1
      ;;
    # ... more options here
    *)
      echo "Unrecognized option: $1"
      shift
      OPT_SHOWHELP=1
      ;;
  esac
done
```

## Show help

```sh
if [ -n "$OPT_SHOWHELP" ]
then
  echo
  echo "USAGE: $(basename "$0") [-a] [-b]"
  echo
  echo "     -a blah blah blah"
  echo "     -b bloo bloo bloo"
  echo
  exit 1
fi
```

## Check dependency

```sh
# redirect both stdout and stderr to /dev/null
# suppress all possible output
if ! which jq &>/dev/null
then
  echo "The 'jq' utility is required"
  echo "    Most package managers should have a 'jq' package available"
  EXIT=1
fi

# ... more dependencies check
```

## Extract the directory that contains the script via script name

`dirname "$(realpath "$(which "$0")")"`

## Heredoc

```sh
cat << EOF > "/your/path/to/file"
blah
blah
blah
...
blah
EOF
```

## Fetch -> Temp -> New file

```sh
curl ...
if [ ... ]
then
	echo "Error, save to .temp"
	exit 1
else
	# write from .temp to new file
fi
```

## Short-circuit

```sh
[COMMAND THAT CAN BE FAILED] || exit 1
```

## Logging system

```sh
#!/bin/sh

# Colors
readonly RED='\033[0;31m'
readonly YELLOW='\033[1;33m'
readonly BLUE='\033[0;34m'
readonly GRAY='\033[0;37m'
readonly GREEN='\033[0;32m'
readonly NC='\033[0m'

# Format settings
readonly TIME_FORMAT='%Y-%m-%d %H:%M:%S'
# %b: ARGUMENT as a string with '\' escapes interpreted, except that octal escapes are of the form \0 or \0NNN
# Cre: https://stackoverflow.com/a/5412825
readonly LOG_FORMAT='%-19s %s: [%b%-5s%b] %s\n'

log() {
    local level=$1; shift
    local color=$1; shift
    local timestamp=$(date +"$TIME_FORMAT")
    local script_name=$(basename "$0")
    # local line_num=${BASH_LINENO[0]}

    printf "$LOG_FORMAT" \
        "$timestamp" \
        "$script_name" \
        "$color" "$level" "$NC" \
        "$*"
}

error()   { log "ERROR" "$RED" "$@" >&2; }
warning() { log "WARN" "$YELLOW" "$@" >&2; }
info()    { log "INFO" "$BLUE" "$@"; }
debug()   { [ "${DEBUG:-0}" -eq 1 ] && log "DEBUG" "$GRAY" "$@" >&2 || return 0; }
success() { log "SUCC" "$GREEN" "$@"; }
```
