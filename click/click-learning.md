# Click Application Development Practice

<!-- tl;dr starts -->

Click is the go-to CLI library for Simon (and me) when he has an idea that can be realized using a CLI application.

<!-- tl;dr ends -->

## Overview

Click is a Python library for building "composable" CLI applications. It leverages Python Decorators to declare and parse commands and parameters, produce help message, prompting for user input, ...

Click is more superior in terms of robustness than `argparse`, Python built-in package for building CLI apps.

## Cheatsheet

```py
# greet.py
@click.command()          # Turn a function into a command
@click.argument("name")   # Register a positional argument
@click.option(            # Register an option argument
  "--count", # if the third argument wasn't specified, the function argument name will be inferred from long format. in this case, it's "count"
  "-c",
  "counts" # argument name
  default=1,
  help="The number of times to greet.",
  type=int,
  count=True,   # -v, -vv, -vvv to increase verbosity
) # Register an option into a command
def greet(name, count):   # NOTE: function's arguments become reference to the positional arguments and options
  """Greet NAME COUNT times"""
  for _ in range(count):
    click.echo(f"Hello, {name}!")

if __name__ == "__main__":
  # dead simple entry point
  greet()
  # usage: python greet.py Alice
  # usage: python greet.py Bob -c 3
  # usage: python greet.py Bob -c 3 -vvv

# Turn a function into a decorator
# Any functions being wrapped by it becomes its subcommands
# a.k.a a dispatcher
@click.group()
def cli():
  pass

# NOTE: these subcommands can be splited across modules for better separation of concerns
@cli.command()
def start():
  click.echo("start")

@cli.command()
def stop():
  click.echo("stop")

# Types and Validations
# - Most common built-in types: int, float, click.Path, click.Choice, DateTime
# - Use callback for custom validation
# - click.BadParameter exception class can be thrown when an error regarding validation failed

# Context
# - Call ctx = click.get_current_context() to get context object
# - `ctx.invoke()` can invoke other commands from within a command
# - `ctx.obj` and `ctx.ensure_object()` can store config while in top-level group for subcommands to use

# Prompts and Confirmation
# click.prompt("Name")        # interactive prompt + type conversion
# click.confirm("Proceed?")   # yes/no confirmation
# click.password_option or click.prompt(hide_input=True)    # secret prompting

# Help
# NOTE: Docstring of command becomes help text with running with --help
# click.option(help="Hello World")
# click.group(help="Hello World")
# click.command(help="Hello World")
```

```py
# ============================================================================ #
# cli.py                                                                       #
# ============================================================================ #

@click.group(   # parent of all subcommands
  context_settings=dict(help_option_names=["-h", "--help"]),
  invoke_without_command=True, # group can be invoked without a subcommand
)
@click.option(  # global option
  "--file",
  "-f",
  "compose_file",
  default="docker-compose.yml",
  help="Path to docker-compose file.",
  type=click.Path(exists=False),
)
@click.option(  # global option
  "--project-name",
  "-p",
  "project_name",
  default=None,
  help="Compose project name (COMPOSE_PROJECT_NAME).",
)
@click.version_option(  # special option
  __version__,
  "-V",                       # short format (NOTE: "-v" is reserved for verbosity level)
  "--version",                # long format
  prod_name="brainfort-auth"
)
# TWO ways to access context inside a group/command
# 1. pass_context decorator: the first function argument will be `ctx` (more modern)
# 2. ctx = click.get_current_context() to access the `ctx` object
@click.pass_context
def cli(ctx, compose_file, project_name):
  """
  Docstring for cli
  """
  ctx.ensure_object(dict) # ensure ctx.obj is a dict for downstream commands
  ctx.obj["compose_file"] = os.path.abspath(compose_file)
  ctx.obj["project_name"] = project_name

  # if invoke without subcommand, show help
  if ctx.invoked_subcommand is None:
    click.echo()

# ============================================================================ #
# commands.py                                                                  #
# ============================================================================ #

def run_compose(ctx, *args):
  """Helper functions that receive arguments from other Click's commands"""

  compose_file = ctx.obj.get("compose_file")
  project_name = ctx.obj.get("project_name")

  # building cmd
  cmd = ["docker", "compose", "-f", compose_file] if compose_file else ["docker", "compose"]

  if project_name:
    cmd += ["-p", project_name]
  cmd += list(args)

  try:
    # streaming output and capture exit codes with proper stdout/stderr handling
    proc = subprocess.Popen(cmd)
    proc.communicate()
    return proc.returncode
  except FileNotFoundError:
    click.echo("'docker compose' executables not found.", error=True)
    raise click.Abort()

@click.command("up")
@click.option("--detach", "-d", is_flag=True, default=False, help="Run containers in the background")
@click.argument("services", nargs=-1) # accept zero-or-more argument values
@click.pass_context
def up(ctx, detach, services):
  """Start services"""
  args = ["up"]
  if detach:
    args.append("-d") # flag option
  args += list(services)
  # if it's a non-flag option
  # write: args += ["--option", option]
  code = run_compose(ctx, *args)
  if code != 0:
    raise click.Abort()

# ============================================================================ #
# entry.py
# ============================================================================ #

cli.add_command(up)
cli.add_command(...)

# main.py
if __name__ == "__main__":
  cli()
```

## Best practices

- Separate CLI from business logic.
- Use sensible defaults. What is the value that users are going to use the most?
- Use sensible option types.
- Put long help text/examples into function's Docstring.
- Use clear option names and short aliases. Some option names are conventional (e.g. `--help`, `-h`, `--version`, `-v`, `--quiet`, `-q`, ...), they should be reserved, unless you have no other options.

### Simon Willison's Blog

1. Build yourself a template. Simon had developed his own Cookiecutter template [simonw/click-app](https://github.com/simonw/click-app).

2. Follow conventions:

Arguments:

- Arguments are positional - they are strings that passed directly into command.
- Arguments can be required or optional.
- Commands can accept an unlimited number of arguments. (`nargs=-1`)

Options:

- Options are usually optional but are occasionally required. Reading a command that has so many positional arguments can be harder than reading one using 1 argument and multiple options for better separability.
- Options can be "flags".
- Multiple flags short options can be combined (e.g `grep -ir` === `grep --ignore-case --recursive`)
- Options can take multiple parameters like arguments.

3. Sub-commands: Each sub-command can have their own family of commands. E.g. `git add`, `git commit`, `git push`,... are all sub-commands of `git` command.

4. Help text: Every command and sub-commands should have `--help` option, the more detailed, the better.

5. Enforce consistency between CLIs

As CLI tools scale, they end up with a growing number of commands, sub-commands, arguments and options. You must strive for "Design Consistency", which means new artifacts should be similar to existing artifacts. If you have multiple related CLI utilities, they must resemblance each other as well.

6. Versioning CLI interfaces like APIs

Practice Semantic Versioning (SemVer):

- Bump major version number on breacking changes.
- Bump minor version number on new features.
- Bump patch version number on bug fixes.

Simon said be cautious when making major changes. However, for inexperienced developers that haven't been exposed to a lot of different designs, making major changes is inevitable.

7. Include usage examples in `--help`

Eventually we will forget how to use our own applications due to not having used it for a long time so usage examples are a significant help.

> **NOTE:** Your terminal history is also your documentation.

> E.g. The output of `--help` for [simonw/sqlite-utils's `convert` command](https://simonwillison.net/2023/Sep/30/cli-tools-python/#include-usage-examples-in---help).

## Reference

- [Simon Willison's Blog "Things I’ve learned about building CLI tools in Python"](https://simonwillison.net/2023/Sep/30/cli-tools-python/)
