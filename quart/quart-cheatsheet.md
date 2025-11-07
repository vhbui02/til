---
description: My Quart learning cheatsheet.
applyTo: "**/*.py"
---

# Quart Cheatsheet

## Overview

It's an ASGI web framework using Flask syntax, made by Flask's author (Pallets).

## Background Tasks

```py
# ============================================================================ #
# BACKGROUND TASKS                                                             #
# ============================================================================ #

async def background_task():
  await asyncio.wait(100)

@app.route("/jobs/", methods=["POST"])
async def create_job():
  app.add_background_task(background_task)
  return 'Success'

@app.before_serving
async def startup():
  app.add_background_task(background_task)

# background task have access to app context
# tasks will be awaited during shutsdown to ensure they're completed before the shutdown is completed
# NOTE: set the BACKGROUND_TASK_SHUTDOWN_TIMEOUT less than any server timeout, since tasks will be cancelled if they doesn't complete within BACKGROUND_TASK_SHUTDOWN_TIMEOUT

# CAUTION: if the background tasks do not use `await` (i.e. yield control while waiting for IO), they will block the event loop and other requests can't be processed.
# CAUTION: Background tasks and server share the same CPU, make sure the tasks didn't take too much compute resources. Offload to Serverless via cloud provider is a better choice

async def test_tasks_complete():
  # test_app() context manager will actually wait for any background tasks to
  # complete before the allowing the test to continue
  async with app.test_app():
    app.add_background_task(background_task)
  assert task_has_done_something()

  # the coroutine can be tested by creating an app context and await the functions
  async with app.app_context():
    await background_task()
  assert task_has_dome_something()
```

## Blueprints

```py
# ============================================================================ #
# BLUEPRINTS                                                                   #
# ============================================================================ #

# fascilitate modular architecture
# used when there's an increasing number of API routes

from quart import Blueprint
bp = Blueprint(
  "store",
  __name__,
  # ...
)

# endpoint name: store.index
# usage: url_for('store.index')

@bp.route("/")
async def index():
  return await render_template("index.html")

# Nested

parent = Blueprint(
  "parent",
  __name__,
  url_prefix="/parent"
)

child = Blueprint(
  "child",
  __name__,
  url_prefix="/child"
)

parent.register_blueprint(child)
app.register_blueprint(parent)

@child.route("/")
async def index2():
  return await render_template("index2.html")

# endpoint name: parent.child.index
# endpoint url prefix: /parent/child/
```

## Custom CLI commands

```py
import click

# Click commands are synchronous since they're running outside Quart's app context.
@app.cli.command()
def init_db():
  click.echo("Database initializing....")
  # do some work here

# Of course, you can run async code by manually create an event loop and run coroutines inside it

async def _fetch():
  return await asyncio.wait(10) # simulate external database query

@app.cli.command()
def fetch_db_data():
  loop = asyncio.get_event_loop()
  result = loop.run_until_complete(_fetch())
```

```sh
$ quart init_db
```

## Configuration

There are TWO patterns:

1. Load environment variables (via `python-dotenv`, via export, ...) with `QUART_` prefix into the configuration. **The prefix must be stripped when referenced**.

You don't need `python-dotenv` anymore:

```py
# export QUART_TESTING=true

app = Quart(__name__)
app.config.from_prefixed_env() # load QUART_* env vars
assert app.config["TESTING"] is True
```

2. Class inheritance: define base/common settings with production, then create development overrides.

```py
# config.py
# use `config_class` to change it
class Config:
  DEBUG = False
  TESTING = False
  SECRET_KEY = 'quary-secret-key'

class Development(Config):
  DEBUG = True

class Production(Config):
  SECRET_KEY = 'insert an actual secret key here'
  # NOTE: it's still not a best practice to store secret key inside a Python file
  # NOTE: instead, load it via AWS Secret Manager or Docker Secrets, Docker Compose Secrets, ...

# __init__.py
# Application factory pattern
def create_app(mode='Development'):
  app = Quart(__name__)
  app.config.from_object(f"config.{mode}")
  return app
```

Instance folder: a deployment specific location to store files and config settings:

```
/app.py
/instance/.env
/instance/config.py
```

- API `app.open_resource()` can load files relative to **app root**
- API `app.open_instance_resource()` can load files relative to the `/resource` folder. To load the configuration from this folder, setup:

```py
def create_app(mode='Development'):
  app = Quart(__name__, instance_relative_config=True)
```

## THREE ways to run Quart development server

1. `QUART_APP` env var:

Given `run.py`:

```py
from quart import Quart

app = Quart(__name__)
```

```sh
$ QUART_APP=run:app quart run --host "127.0.0.1" --port 5001 [--certfile] [--keyfile]
```

2. `app.run()`:

Given `run.py`:

```py
from quart import Quart

app = Quart(__name__)

# ...

# running file directly, which ensures that it doesn't run in production
# $ python -m quart run ...
# or $ python run.py
if __name__ == "__main__":
  # NOTE: app.run() API create a new event loop underhood
  # NOTE: it will become a problem if a 3rd-party library also need to be
  # NOTE: initalized by passing a loop which we can't access currently.
  # NOTE: instead, create a separate `loop` then passed it as the first argument
  # NOTE:
  # NOTE: app.run() don't use multiple workers also
  app.run(host="127.0.0.1", port=5001, ...) # debug mode and reloader are enabled
```

3. `app.run_task`

Given `run.py`:

```py
import asyncio
from quart import Quart

app = Quart(__name__)

if __name__ == "__main__":
  # run_task() API returns a Task
  # asyncio.run() awaited the Task, thus the app is run
  # no event loop alteration from start
  asyncio.run(app.run_task(host="127.0.0.1", port=5001, ...))
```

## Detecting disconnection

```py
@app.route('/sse') # WebSockets, streaming the request, ...
async def sse():
  try:
    await ...
  except asyncio.CancelledError:
    # handling disconnection
```

## Customize event loop

In order to use Quart with a 3rd-party async library whilst ensuring both of them use the same loop, **create/initialize the 3rd-party within the loop** created by Quart, using API `@app.before_serving` and `@app.after_serving` decorators:

```py
@app.before_serving
async def startup():
  # return current event loop for the running thread, if no event loop exists, it creates one and sets it as the current loop
  loop = asyncio.get_event_loop()

  # start an async TCP server, returns a coroutine
  # store the instance inside `app` context
  app.smtp_server = loop.create_server(aiosmtpd.smtp.SMTP, port=1025)

  # schedule a coroutine to run asap on the event loop
  loop.create_task(app.smtp_server)

@app.after_serving
async def shutdown()
  # NOTE: that's why you need to store the instance inside app context
  app.smtp_server.close()
```

The ASGI server running Quart owns the event loop that Quart runs within, by default it's Hypercorn (pretty overkill that Quart development server is Hypercorn, a production-grade ASGI server).

Both Quart and Hypercorn allow the loop to be explicitly created and passed:

```py
import asyncio
from hypercorn.asyncio import serve
from hypercorn.config import Config

loop = asyncio.get_event_loop()
third_party = ThirdParty(loop)
# NOTE: app.run() is seen to be reserved for development use case, it's best to use application factory pattern
app.run(loop=loop)
# or
loop.run_until_complete(app.run_task())

# production
config = Config()
loop.run_until_complete(serve(app, config))
# or
await serve(app, config)
```

Disable debug mode + reloader and you can deploy immediately. However, you should use **Uvicorn** since all benchmarks shows superior performance comes from Uvicorn ([a 2024 benchmark between Uvicorn, Hypercorn and Daphne](https://medium.com/@onegreyonewhite/2024-comparing-asgi-servers-uvicorn-hypercorn-and-daphne-addb2fd70c57))

# Migrate from Flask

- Replace `Flask`, `flask.*` imports with `Quart`, `quart.*` imports. Automated it using Find and Replace.
- Insert `async` and `await` keywords. If there are awaitables not being awaited, `RuntimeWarning: coroutine 'XX' was never awaited` will display on log.
