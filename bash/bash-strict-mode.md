# Bash Strict Mode

<!-- tl;dr starts -->

Best practices when writing shell scripts are my favorite.

<!-- tl;dr ends -->

## `set -euxo pipefail`

Normally, certain common errors are ignored, causing unexpected results that're hard to troubleshoot. But now script can be failed immediately. Avoid hidden bugs in production.

It's a short for the following 4 commands:

```sh
set -e
set -u
set -o pipefail
set -x
```

### `set -e`

This option instructs `sh/bash` to immediately exit if any command has a **non-zero exit status**. This isn't not what a CLI would want, but in a script it's very helpful.

> In other programming languages, run-time errors nuke the program from the defiant line, subsequent lines are not executed. But `sh/bash` is a REPL (Read-Eval-Print Loop) shell so being failed occasionally must supported, or the shell will be logged out everytime something failed. That's why when writing scripts we want a more script-like behavior by using `set-e` option.

> NOTE: Pay attention to short circuiting. It could create unexpected behavior.

### `set -x`

Bebugging: all executed commands are printed to the terminal, therefore help visualizing the execution flow of the script.

This is typically being used in conjunction with `set -e` since the program get nuked before any logs can be echoed into terminal.

### `set -u`

A reference to any variables that haven't been defined previously (except `$*`, `$@` and default syntax `${NOT_EXIT:-0}`).

This is to prevent typo error causing unexpected results:

```sh
#!/bin/bash
firstName="Aaron"
# Typo here! it has to be camelCase
fullName="$firstname Maxwell"
echo "$fullName"
```

### `set -o pipefail`

By default, the pipeline's return code is that of the last command, regardless of prior commands in pipeline.

```sh
grep some-string /non/existent/file | sort
echo $? # 0
```

Using this setting will prevents errors in a pipeline from being masked. If any command in pipeline fails, that return code will be used as the return code of the whole pipeline.

## IFS (Internal Field Separator)

Also known as "word splitter". It can be set to a character or a string and used to governs how `sh/bash` will split a sequence into different words.

```sh
#!/bin/sh
IFS=$' '
items="a b c"
for x in $items; do
    echo "$x"
done

# a
# b
# c

IFS=$'\n'
for y in $items; do
    echo "$y"
done

# a b c
```

Use the correct IFS will lead to less unexpected behaviors when iterating `bash` array.

## References

- [mohanpedala/bash_strict_mode.md](https://gist.github.com/mohanpedala/1e2ff5661761d3abd0385e8223e16425)
