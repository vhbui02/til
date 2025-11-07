---
description: My Flask learning cheatsheet.
applyTo: "**/*.py"
---

# Flask Cheatsheet

This is my favorite Python WSGI web framework. It has simple API, fast enough and receive community support widely

## Dependencies

**THREE** direct dependencies:

1. **Werkzeug:** a WSGI web application library.
1. **Jinja:** a template engine.
1. **Click:** a Python CLI framework, provides in-house `flask` command. User can add custom commands as well.

**SIX** transitive dependencies:

- MarkupSafe: comes with Jinja, escapes untrusted input when it's being used to be rendered on client-side to avoid XSS attacks.
- ItsDangerous: protect Flask's session cookie by securely signs data to ensure data integrity.
- Blinker: provides support for Signals - a way to notify subscribers of certain events during the lifecycle of the application and each request.
- `python-dotenv`: enable support for Environment Variables from dotenv when running `flask` command.
- Watchdog: faster + more efficient reloader for development server.
- `venv`: create virtual environment to manage project dependencies.

## Production Deployment

Local development environment:

- Development Server
- Debugger
- Reloader

Production environment: **dedicated WSGI server/hosting platform**.
I suggest use `uWSGI` + `nginx`

## CLI

Python module: `*.py`
Python package: `package_name/__init__.py`

```sh
# debug mode
# auto reload server when code changes, show interactive debugger inside browser
# NOTE: allow executing arbitrary Python code from browser, make sure your local network is secure
python -m flask --app package_name run --debug
flask --app package_name run

# if module name is app.py or wsgi.py, omit --app option
python -m flask run

# expose to the whole local network
# now OS listens on all public IPs
python -m flask run --host=0.0.0.0
```

> IMO, use `python -m` syntax to utilize `firejail`

## Development with Flask

```py
import json
import secrets
from flask import Flask, abort, make_response, redirect, render_template, request, session, url_for
from markupsafe import escape
from werkzeug.utils import secure_filename
from werkzeug.middleware.proxy_fix import ProxyFix

# `app` object
# central registry for view functions, URL rules, template configs
app = Flask(__name__)

# the 1st param is used to determine the location of the "resource" directory
# - if module, the package the module is contained in is the "resource" directory.
# - if package, the package itself is the "resource" directory

# Utilize resource folder
with app.test_request_context():
    with simple_page.open_resource('static/style.css') as f:
        code = f.read()

# View functions
# The code responding to requests inside application
@app.route("/")
def show_index_page():
    return "Index page"  # Flask convert returned string into res obj

@app.route("/hello")
def hello():
    return "<h1>Hello World</h1>" # def content type = HTML

# ============================================================================ #
# Variable rules + Converter
# ============================================================================ #

# syntax: <variable-name> or <converter:variable-name>
# syntax: def func(<variable-name>)
@app.route("/api/posts/<int:post_id>")
def get_post(post_id):
    """Show post given specific id"""
    return f"Post {escape(post_id)}" # prevent XSS, use Jinja2 for auto-escaped

# 5 conversion types
# - string: any text without a slash
# - int: positive integer
# - float: positive floating point value
# - path: string but accept slash
# - uuid: accept uuid strings

# ============================================================================ #
# Canonical URL
# ============================================================================ #

# Submit a URL without trailing slash will redirect to the URL with one instead
@app.route("/projects/")
def show_projects():
    return "The project page"

# submit a URL with trailing slash will produce a 404 Not Found error
# - keep URLs unique
# - avoid search engine indexing the same page twice
@app.route("/about")
def about():
    return "The about page"

# ============================================================================ #
# HTTP Methods
# ============================================================================ #
@app.route("/login", methods=["GET", "POST"])  # all methods are kept inside 1 function, useful for sharing common logic
def login():
    error = None
    if request.method == "POST":
        # NOTE: Dict key reference throws KeyError if key not existed
        # NOTE: if extension not handled, a 400 Bad Request is returned
        if request.form["username"] == "admin" and request.form["password"] == "admin":
            return "Login successfully. Welcome: admin"
        else:
            error = "Invalid username/password"

    # only access URL params with `get` or by catching KeyError with try/catch block
    searchword = request.args.get("key", "")
    print(searchword)

    # executed if GET or credentials is invalid
    return f"Return the login form. Error: {error}"


# separated view functions
# CAUTION: you can't build URL around separated view functions
# NOTE: HEAD and OPTIONS are implicitly supported
@app.get("/login2")
def get_login():
    return "Return the login2 form"

@app.post("/login2")
def post_login():
    return "Do the login2..."

# ============================================================================ #
# Jinja2 - Template Engine
# ============================================================================ #

# auto-escaped by default, multiple format supports: HTML, Markdown, Plaintext for emails, ...
@app.route("/hello2/")
@app.route("/hello2/<name>") # nested decorators indicate multiple API endpoints supported
def hello2(name=None):
    # 1st param: name of the template
    # >=2nd param: variables passed to the template engine
    return render_template("hello2.html", person=name)

# for a module, `templates` directory is next to that module
# for an app, `templates` directory is next to __init__.py file

# ============================================================================ #
# Context Manager
# ============================================================================ #

# mock request processing behavior right inside Python shell
with app.test_request_context():
    print(url_for("about"))
    print(url_for("hello", next="/", abc="def"))
    print(url_for("show_user_profile", username="John Doe"))

    # url_for(_external=True) API # def=False
    # use case: generate links for emails, redirects, or APIs that require the absolute URL.
    # NOTE: if Flask is running locally, the domain will be 'localhost' instead

# a specific API endpoint is passed as context
with app.test_request_context("/login", method="POST"):
    # bind a request object inside `with` block
    assert request.path == "/login"
    assert request.method == "POST"

# FIVE reason to build URL instead of hard-coded URL inside template files:
# - more descritive
# - better maintainability, changing route outcome is dynamically applied to anywhere use url_for
# - URL building escapes special characters (e.g. 'John Doe' => 'John%20Doe')
# - Path generated is absolute, avoid unexpected behavior from rel path
# - Flexible root URL change

# ============================================================================ #
# File Uploading
# ============================================================================ #
# NOTE: remember to set <form enctype="multipart/form-data"> attribute
@app.route("/upload", methods=["GET", "POST"])
def upload_file():
    if request.method == "POST":
        # NOTE: request.files is a Dict
        f = request.files["the_file"]  # in-memory storage via request.files
        # craft your own filename
        f.save("/tmp/uploaded_file.txt")  # store as a file inside server filesystem
        # use client's filename, securely
        f.save(f"/tmp/{secure_filename(f.filename or 'uploaded_file.txt')}")
        return "File received"

    return "Show the form to upload file"

# ============================================================================ #
# Cookies
# ============================================================================ #

@app.route("/cookies")
def cookies():
    # again, use get() to prevent KeyError throwing if the cookie is missing
    username = request.cookies.get("username")

    if not username:
        # use make_response() API to set Cookies and Response Header
        res = make_response("Making cookies: username=johndoe...")
        res.headers["X-Something"] = "x-something's value"
        res.set_cookie("username", "johndoe")
        return res

    return f"Reading cookies: {username}..."

# NOTE: cookies can exist before response obj, use Deferred Request Callbacks

# ============================================================================ #
# Redirects and Custom Error Page
# ============================================================================ #

@app.route("/redirect")
def test_redirect():
    return redirect(url_for("login"))

@app.route("/unknown")
def test_abort():
    abort(401)  # Response Body: 401 Unauthorized...
    abort(404)  # Response Body: Custom error page
    return "This is never executed"

@app.errorhandler(404)
def page_not_found(error):
    # return render_template("page_not_found.html", 404)
    return "Custom error page", 404

# ============================================================================ #
# API with JSON
# ============================================================================ #

@app.route("/me")
def get_me():
    return {
        "username": "John Wick",
        "theme": "dark",
        "image": "https://github.com/shadcn.png",
    }  # return a Dict => jsonify()

@app.route("/users")
def get_users():
    users = ["Alpha", "Beta", "Charlie", "Delta", "Echo"] # mock data fetching ...
    return [json.dumps(user) for user in users]  # return a List => jsonify()

# TIPS: for complex data types (database models, ...), use Flask Extension

# ============================================================================ #
# Session
# ============================================================================ #

app.secret_key = secrets.token_hex()

@app.route("/index3")
def index3():
    if "username" in session:
        return f"Logged in as {session["username"]}"
    return "You're not logged in"


@app.route("/login3", methods=["GET", "POST"])
def login3():

    if request.method == "POST":
        # assume login is success ...
        # store session
        session["username"] = request.form["username"]  # <input type="text" name="username" />
        return redirect(url_for("index3"))

    return """
        <form method="post">
            <input type="text" name="username" />
            <input type="submit" value="Login" />
        </form>
    """

# ============================================================================ #
# Logging
# ============================================================================ #

# used built-in lib "logging"
app.logger.debug("This is a debug message")
app.logger.warning("This is a warning message: %d apples", 1)
app.logger.error("This is an error message", exc_info=True)
app.logger.exception("This is an exception message") # exc_info=True by default

# ============================================================================ #
# WSGI Middleware
# ============================================================================ #

# apply Werkzeug's ProxyFix middleware for running Flask behind Nginx
app.wsgi_app = ProxyFix(app.wsgi_app)
# NOTE: wrap app.wsgi_app, not app itself. `app` must point at the Flask application

# ============================================================================ #
# Extensions
# ============================================================================ #
# Take a look at this awesome list: https://github.com/humiaozuzu/awesome-flask
```

## Project: A simple blog with Flask

### Project Directory Structure

```
/my-awesome-app         # kebab-case
├── my_awesome_app      # snake_case
│   ├── __init__.py
│   ├── schema.sql
│   ├── db.py           # module 1
│   ├── auth.py         # module 2
│   ├── blog.py         # module 3
│   ├── *.py            # module N
│   ├── templates/
│   │   ├── base.html
│   │   ├── auth/
│   │   │   ├── login.html
│   │   │   └── register.html
│   │   └── blog/
│   │   │   ├── create.html
│   │   │   ├── index.html
│   │   │   └── update.html
│   │   └── foo/
│   │   │   ├── bar.html
│   │   │   └── baz.html
│   └── static/
│       └── style.css
├── tests/
│   ├── conftest.py
│   ├── data.sql
│   ├── test_factory.py
│   ├── test_db.py
│   ├── test_auth.py
│   └── test_blog.py
├── .github
│   └── workflows
│       ├── release.yml
│       └── test.yml
├── instance
│   ├── db.sqlite
│   └── ...
├── .venv/
├── pyproject.toml
├── LICENSE
├── MANIFEST.in
└── README.md
```

`.gitignore` file:

```gitignore
.venv/

*.pyc
__pycache__/

instance/

.pytest_cache/
.coverage
htmlcov/

dist/
build/
*.egg-info/
```

### Configuration Inheritance Pattern

`config.py`:

```py
class Config(object):
    TESTING = False

class ProductionConfig(Config):
    DATABASE_URI = "mysql://user@localhost/foo"

class DevelopmentConfig(Config):
    DATABASE_URI = "sqlite:////tmp/foo.db"

class TestingConfig(Config):
    TESTING = True
    DATABASE_URI = "sqlite:///:memory:"
```

`app.py`

```py
# static Class
app.config.from_object('config.ProductionConfig')
```

---

`config.py`

```py
class Config(object):
    """Base config"""
    TESTING = False
    DB_SERVER = '192.168.1.56' # staging database server

    @property
    def DATABASE_URI(self):
        return f"mysql://user@{self.DB_SERVER}/foo"

class ProductionConfig(Config):
    DB_SERVER = '192.168.19.32'

class DevelopmentConfig(Config):
    DB_SERVER = 'localhost'

class TestingConfig(Config):
    DB_SERVER = 'localhost'
    DATABASE_URI = 'sqlite:///:memory:'
```

`app.py`

```py
# instantiated

```

Configuration best practice:

1. Never write code that needs config at import time.
2. Load the config early, before the extensions initialization.
3. Always keep a default config in version control. Either populate the config with it, or import it into your dedicated config file then override them.
4. Use an environment variable to switch between configurations.

### Instance folder

Via `Flask.root_path`, we can refer to paths relative to app folder, doing so allowing developers to load configurations stored next to the application.

=> works if applications are not packages, in which case, the root path refers to the package directory.

Use `/instance` folder, it's not under version controlled and be deployment specific. **Store things change at runtime and configuration files here.**

```py
app = Flask(__name__, instance_path="/path/to/instance/directory")
```

If `instance_path` is not specified, here are the default behavior:

```
# uninstalled module
/app.py
/instance

# uninstalled package
/my_app
    /__init__.py
/instance

# installed module or package
$PREFIX/lib/pythonX.Y/site-packages/my_app
$PREFIX/var/my-app-instance

# prefix is the prefix of the Python installation, it can be `/usr`, or path to your `.venv` Python virtualenv.
```

### Application Factory Pattern

`my_awesome_app/__init__.py`

```py
import os
from flask import Flask
from celery import Celery, Task

def celery_init_app(app: Flask) -> Celery:
    """
    Configure Celery with Flask.

    Celery follows a similar approach to Flask: it uses an application object to manage configuration.

    However, you cannot pass the Flask app object directly to the Celery app constructor, because Celery is not a Flask extension—it is a standalone application.
    """
    class FlaskTask(Task):
        def __call__(self, *args: object, **kwargs: object) -> object:
            # run task functions inside Flask's app context
            # why: services like DB connections are available
            with app.app_context():
                return self.run(*args, **kwargs)

    celery_app = Celery(app.name, task_cls=FlaskTask)
    celery_app.config_from_object(app.config["CELERY"]) # taken from CELERY key in app config
    celery_app.set_default() # accessible during each request
    app.extensions["celery"] = celery_app
    return celery_app


def create_app(mode="Production", test_config=None):
    """Flask application instance initialization using Application Factory pattern"""

    # instance/: a directory that's not under version controlled, a standardized place to put config files, env files, database files, ... inside.
    app = Flask(__name__) # the directory is either next to the module file or next to the package directory (not INSIDE it)
    app = Flask(__name__, instance_path='/path/to/instance/folder') # NOTE: it must be an absolute path
    app = Flask(__name__, instance_relative_config=True)
    # loading /instance/application.cfg
    with app.open_instance_resource('application.cfg') as f:
        config = f.read()

    # Method #1: load from a Mapping object (a Python dictionary) or a list of keyword arguments
    app.config.from_mapping(
        SECRET_KEY="dev", # must be replaced with other
        DATABASE=os.path.join(app.instance_path, "db.sqlite"),
        CELERY=dict(
            broker_url="redis://localhost",
            result_backend="redis://localhost",
            task_ignore_result=True,
        ),
    )
    app.config.from_mapping({
        "DEBUG": True,
        "SECRET_KEY": "secret-key",
    })

    # NOTE: Ideally, the configuration should be stored OUTSIDE of the app package, conventionally inside the /instance folder
    # NOTE: application deployment now can be tailored with external configuration

    # Method #2: load from a Python object (class/module)
    # E.g. 'config.py' content:
    #
    # DEBUG = True
    # SECRET_KEY = "your-secret-key"
    # SQLALCHEMY_DATABASE_URI = "sqlite:///mydb.sqlite"
    #
    # from_object() API support relative filename (relative to root_path)
    app.config.from_object('my_app.config') # import the module 'config.py' that share the same parent directory with this file
    app.config.from_object(config) # import an object inside the module

    # Method #3
    # e.g. $ export PATH_TO_CONFIG_FILE="/path/to/settings.cfg"
    # settings.cfg:
    #
    # SECRET_KEY = 'my-secret-key'
    #
    app.config.from_envvar("PATH_TO_SETTING_FILE")

    # Method #4: load FLASK_* env vars from OS env
    # e.g. FLASK_SECRET_KEY="my-secret-key"
    app.config.from_prefixed_env()
    # CAUTION: when accessed, the key must have `FLASK_` stripped off
    app.config["SECRET_KEY"] # not `FLASK_SECRET_KEY`

    # Method #5: data file
    import tomllib
    app.config.from_file("config.toml", load=tomllib.load, text=False)

    import json
    app.config.from_file("config.json", load=json.load)

    if test_config is None:
        # Method #6: configuration from Python files
        app.config.from_pyfile("test_config.py", silent=True)
    else:
        # passed to the factory so it gets used instead of the instance config
        # NOTE: use case: setup test environment
        app.config.from_mapping(test_config)

    # ensure main/instance exists
    try:
        os.makedirs(app.instance_path)
    except OSError:
        pass

    # Database module registration
    from . import db
    db.init_app(app)

    # Load Celery config from Flask app config, initialize Celery app object then store it into app.extensions
    celery_init_app(app)

    # Authentication module registation
    from . import auth
    app.register_blueprint(auth.bp)

    # Blog module registration
    from . import blog
    app.register_blueprint(blog.bp)


    # url_for('index')
    app.add_url_rule('/', endpoint='index') # endpoint's name, not URL

    # alternative: give the blog blueprint a url_prefix, then define a separate index view, then the `index` and `blog.index` endpoints will be different
    @app.route('/blogs')
    def blog_index():
        pass

    # simple route for testing
    @app.route('/')
    def hello():
        return {
            "status": "success",
            "data": {
                "msg": "Welcome to MagicPost."
            }
        }

    return app
```

Workflow of a normal request:

```
req process => create a connection => query => close it => res created
```

**NOTE:** The built-in sqlite3 module performs sequential writes to the database. If multiple write requests occur, each subsequent request must wait for the previous ones to complete.

**`g` object:**

In order to incorporate database connection in the request's context, we use `g` object:

- Unique for each request
- Store data that might be accessed by multiple functions during the request.
- Apply Singleton pattern to initialize and reuse DB connection (during the context of a request though, since different requests make separate connections).

**`app.extensions` and `current_app.extensions` object:**

A centralized place for Flask extensions to register themselves, technically speaking, a Python Dict store their initialized instances. The instance can be accessed and managed at app-level, not request-level like `g` object.

E.g.

```py
app.extensions['sqlalchemy'] = SQLAlchemy()
```

**`current_app` object:**

When using Application Factory pattern `create_app()`, the Flask application instance `app` is not available as a global variable throughout the codebase, due to `app` only existed during runtime. Modules or functions that are imported before the app is created must refer to an app object somehow, therefore Flask provides `current_app` proxy obj that always points to the Flask application that handles the request.

Using `current_app`, users can access app config safely, such as path to database file in filesystem.

Here is the database schema `my_awesome_app/schema.sql`:

```sql
DROP TABLE IF EXISTS user;
DROP TABLE IF EXISTS post;

CREATE TABLE user (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  username TEXT UNIQUE NOT NULL,
  password TEXT NOT NULL
);

CREATE TABLE post (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  author_id INTEGER NOT NULL,
  created TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
  title TEXT NOT NULL,
  body TEXT NOT NULL,
  FOREIGN KEY (author_id) REFERENCES user (id)
);
```

Database module: `my_awesome_app/db.py`

```py
import sqlite3
from datetime import datetime

import click
from flask import current_app, g

# Singleton pattern
def get_db():
    if 'db' not in g:
        g.db = sqlite3.connect(
            # get_db() must be called during a request so current_app can refer to the Flask application instance
            current_app.config['DATABASE'], # NOTE: the file doesn't have to existed yet, don't forget this is called during runtime when processing a request
            detect_types=sqlite3.PARSE_DECLTYPES
        )
        g.db.row_factory = sqlite3.Row # return rows behave like Dict => access columns by name
    return g.db

def close_db():
    db = g.pop('db', None)

    # prevent closing twice
    if db is not None:
        db.close()

def init_db():
    """Create database tables and populate mock data"""
    db = get_db()

    # open a file relative to package directory
    with current_app.open_resouce('schema.sql') as f:
        db.executescript(f.read().decode('utf8'))

@click.command('init-db')
def init_db_command():
    """Clear the existing data and create new tables.
    syntax: $ flask --app my_awesome_app init-db
    """

    init_db()
    click.echo('Initialize the database.')

# ============================================================================ #

def init_app(app):
    """Register init_db_command and close_db with the app instance
    App object is instantiated via application factory pattern, you must
    specify app as argument
    """

    app.cli.add_command(init_db_command)

    # modify the response object before it's sent to client
    # e.g. add headers, set cookies, log response details, ...
    # CAUTION: if an exception is raised inside the callback, subsequent callbacks
    # CAUTION: registered by app.after_request() are ignored.
    # CAUTION: DO NOT THROW exception if you want the subsequent callbacks to be called
    app.after_request(callback)

    # invoke callback function when request context is popped, which is after the returning the Response
    # NOTE: if exception is raised yet being handled by errorhandler, `exception` passing to these callbacks will always be None
    # NOTE: it's best to handle it the rollback/commit right inside the errorhandl
    app.teardown_request(callback)

    # invoke callback function when app context is popped, which are:
    # - after the request context for each request
    # - at the end of CLI commands
    # - after a manually pushed context ends.
    # NOTE: if exception is raised yet being handled by errorhandler, `exception` passing to these callbacks will always be None
    # NOTE: it's best to handle it the rollback/commit right inside the errorhandl
    app.teardown_appcontext(close_db_connections)

    # CAUTION: if you planced to close resources like db/cache connections
    # CAUTION: do it app.teardown_request() or app.teardown_appcontext()


# tell Python how to interpret timestamp values in the database
# more specifically, convert the value in 'timestamp' column to a datetime.datetime
sqlite3.register_converter(
    "timestamp", lambda v: datetime.fromisoformat(v.decode())
)
```

#### Blueprints

Authentication module: `my_awesome_app/auth.py`

```py
import functools
from flask import (
    Blueprint, flash, g, redirect, render_template, request, session, url_for
)
from werkzeug.security import check_password_hash, generate_password_hash
from my_awesome_app.db import get_db

# Blueprint. a way to group related views under a single roof

bp = Blueprint('auth', __name__, url_prefix='/auth')
# 1st param: name of the blueprint
# 2nd param: name of the application's package/module
# 3rd param: the URL prefix that's prepended to all the URLs associated with the blueprint

# NOTE: it's not `app` anymore
@bp.route('/register', methods=('GET', 'POST'))
def register():
    if request.method == 'POST':
        # again, request.form is a Dict
        username = request.form['username']
        password = request.form['password']
        db = get_db()
        error = None

        # simple validation logic
        # NOTE: for more complex POST validation logic or GET query parameter, use a validation library, such as pydantic
        if not username:
            error = 'Username is required'
        elif not password:
            error = 'Password is required'

        if error is None:
            try:
                # create new account
                db.execute(
                    "INSERT INTO user (username, password) VALUES (?, ?)",
                    (username, generate_password_hash(password)),
                ) # prepared statement
                db.commit()
            except db.IntegrityError:
                error = f"User {username} is already registered."
            else:
                return redirect(url_for("auth.login"))

        # store messages that can retrived when rendering the template.
        flash(error)

    return render_template('auth/register.html') # or JSON if your FE is using CSR

@bp.route("/login", methods=('GET', 'POST'))
def login():
    error = None
    if request.method == 'POST':
        username = request.form['username']
        password = request.form['password']

        db = get_db()
        user = db.execute(
            'SELECT * FROM user WHERE username = ?', (username,)
        ).fetchone()

        if user is None:
            error = 'Incorrect username.'
        elif not check_password_hash(user['password'], password):
            error = 'Incorrect password.'

        # sessions initialization
        if error is None:
            session.clear()
            session['user_id'] = user['id']
            flash("You're sucessfully logged in")
            return redirect(url_for('index'))

        # message flash, next req can access using get_flashed_messages() API inside template engine or Python request processing code
        flash(error)

    # or transfer the error direcly into login.html template
    return render_template('auth/login.html', error=error)

def login_required(view):
    """Restrict access to certain views unless the user is authenticated
    Usage: @login_required
           def fn(): ...
    """

    @functools.wraps(view) # preserve original view function's metadata (name, docstring)
    def wrapped_view(**kwargs):
        if g.user is None:
            return redirect(url_for('auth.login')) # 'auth' blueprint, 'login()' view fn
        return view(**kwargs)

    return wrapped_view

# NOTE: invoke a callback BEFORE any view function regardless what URL is requested
@bp.before_app_request
def load_logged_in_user():
    # use session to populate user data
    user_id = session.get('user_id')

    if user_id is None:
        g.user = None
    else:
        # NOTE: `g` only lasts for the length of the request, therefore before any request it must be re-initialized by re-queried.
        # NOTE: this can add a lot of overhead to the SQLite db
        g.user = get_db().execute(
            'SELECT * FROM user WHERE id = ?', (user_id,)
        ).fetchone()

@bp.route('/logout')
def logout():
    session.clear()
    return redirect(url_for('index'))
```

Blog module: `/my_awesome_app/blog.py`

```py
from flask import (
    Blueprint, flash, g, redirect, render_template, request, url_for
)
from werkzeug.exceptions import abort

from my_awesome_app.auth import login_required
from my_awesome_app.db import get_db

bp = Blueprint('blog', __name__) # no url_prefix=. Therefore blog index is also main index

def get_post(id, check_author=True):
    post = get_db().execute(
        'SELECT p.id, title, body, created, author_id, username'
        ' FROM post p JOIN user u ON p.author_id = u.id'
        ' WHERE p.id = ?',
        (id,)
    ).fetchone()

    if post is None:
        abort(404, f"Post id {id} doesn't exist.")

    if check_author and post['author_id'] != g.user['id']:
        abort(403)

    return post

@bp.get('/')
@bp.get('/blogs')
def index()
    db = get_db()
    posts = db.execute(
        'SELECT p.id, title, body, created, author_id, username'
        'FROM post p JOIN user u ON p.author_id = u.id'
        'ORDER BY created DESC'
    ).fetchall()
    return {
        "..."
    }

@bp.post('/blogs')
@login_required
def post_blogs():
    title = request.form['title']
    body = request.form['body']
    error = None

    if not title:
        error = 'Title is required'

    if error is not None:
        flash(error)
    else:
        db = get_db()
        db.execute(
            '''
            INSERT INTO post (title, body, author_id) VALUES (?, ?, ?)
            ''',
            (title, body, g.user['id'])
        )
        db.commit() # IMPORTANT: very crucial to make changes to DB
        return redirect(url_for('blog.index'))

    return {
        "..."
    }

def get_post(id, check_author=True):
    post = get_db().execute(
        """
        SELECT p.id, title, body, created, author_id, username
        FROM post p
        JOIN user u ON p.author_id = u.id
        WHERE p.id = ?
        """,
        (id,)
    ).fetchone()

    if post is None:
        abort(404, f"Post {id} does not exist.")

    if check_author and post['author_id'] != g.user['id']:
        abort(403)

    return post

@bp.put('/blogs/<int:id>')
def put_blogs(id):
    post = get_post(id)
    title = request.form['title']
    body = request.form['body']
    error = None

    if error is not None:
        flash(error)
    else:
        db = get_db()
        db.execute(
            """
            UPDATE post SET title = ?, body = ? WHERE id = ?
            """,
            (title, body, id)
        )
        db.commit()

        return redirect(url_for('blog.index'))

    return {
        "..."
    }

@bp.delete('/blogs/<int:id>')
def delete_blogs(id):
    get_post(id) # utilize the function to check if the post is existed inside DB
    db = get_db()
    db.execute(
        """
        DELETE FROM post WHERE id = ?
        """,
        (id,)
    )
    db.commit() # NEVER FORGET THIS
    return redirect(url_for('blog.index'))
```

#### Templates

There are 6 APIs that a template can access:

Global obj:

- `config`
- `request`
- `session`
- `g`

Functions:

- `url_for()`
- `get_flashed_messages()` - Render message sent from `flash()` API.

---

`/my_awesome_app/templates/base.html`

```html
<!DOCTYPE html>
<head>
  <title>{% block title %}{% endblock %}</title>
  <link
    rel="stylesheet"
    href="{{ url_for('static', filename='styles.css') }}"
  />
  <!-- <script>...</script> -->
</head>

<body>
  <nav>
    <h1>My Awesome App</h1>
    <ul>
      <!-- from load_logged_in_user() API -->
      {% if g.user %}
      <li><p>{{ g.user["username"] }}</p></li>
      <li><a href="{{ url_for('auth.logout') }}">Logout</a></li>
      {% else %}
      <li><a href="{{ url_for('auth.register') }}">Register</a></li>
      <li><a href="{{ url_for('auth.login') }}">Login</a></li>
      {% endif %}
    </ul>
  </nav>
  <main>
    <section>
      <!-- Q: why <header> tag isn't inside the block? -->
      <!-- A: deduplicate to the fullest -->
      <header>{% block header %}{% endblock %}</header>
      <!-- basic use of flash messages -->
      <!-- flash msg can be serialized and parsed to identify which one should be displayed and how should it be displayed -->
      {% for message in get_flashed_messages() %}
      <div class="flash">{{ message }}</div>
      {% endfor %} {% block content %}{% endblock %}
    </section>
  </main>
</body>
```

Register template: `my_awesome_app/templates/auth/register.html`

```html
{% extends 'base.html' %}
<!-- the path is relative to /templates, not /templates/auth -->

<!-- Nested blocks -->
{% block header %}
<h1>{% block title %}My Awesome App{% endblock %}</h1>
{% endblock %} {% block content %}
<form method="post">
  <label for="username">Username</label>
  <input name="username" id="username" required />
  <label for="password">Password</label>
  <input type="password" name="password" id="password" required />
  <input type="submit" value="Register" />
</form>
{% endblock %}
```

Log in template is the same as Register template, except the title.

---

Blog index template:

```html

```

#### Setup build

Build a wheel file and install that in another environment. Pros:

- Call your project from anywhere (as long as you still activate the virtual env)
- Manage project dependencies.
- Test tools can isolate test env with dev env.

```toml
[project]
name = "my-awesome-app"
version = "1.0.0"
description = "Flask tutorial project"
dependencies = [
    "flask",
    ... # add more here
]

[build-system]
requires = ["flit-core<4"]
build-backend = "flit_core.buildapi"
```

Editable mode: as devs make changes to local code, they only need to re-install if they change the metadata about the project (such as "dependencies")

```sh
# look for `pyproject.toml` inside current working directory
# install the project in editable/development mode.
pip install -e .
pip list            # check installation location
```

<!-- TODO: read https://packaging.python.org/en/latest/tutorials/packaging-projects/ -->

#### Testng

<!-- TODO: read https://flask.palletsprojects.com/en/stable/tutorial/tests/ -->

#### Deploying

```sh
pip install build
python -m build --wheel
# output: dist/my-awesome-app-1.0.0-py3-none-any.whl
# syntax: {project name}-{version}-{python tag}-{abi tag}-{platform tag}

# copy this file into another directory and setup a new virtual env
pip install my-awesome-app-1.0.0-py3-none-any.whl
# output .venv/var/my-awesome-app-instance
flask --app my-awesome-app init-db
```

Generate secret key:

```sh
python -c 'import secrets; print(secrets.token_hex())'
# output: 192b9bdd22ab9ed4d12e236c78afcb9a393ec15f71bbf5dc987d54727823bcbf
```

Copy secret key into `.venv/var/my-awesome-app-instance/config.py¶`

```sh
SECRET_KEY=192b9bdd22ab9ed4d12e236c78afcb9a393ec15f71bbf5dc987d54727823bcbf
# more config here...
```

Install a production WSGI server on on-premise server or use cloud solution such as Amazon Beanstalk, ...

## Response

The return value from a view function is converted into a response object.

Conversion logic:

1. res obj itself => nothing
2. string => res obj created
3. Iterator/Generator returns `str/bytes` => streaming response
4. Dict/List => res obj created via jsonify() # CAUTION: all data inside Dict/List must be JSON serializable
5. structured tuple `(response, status)`, `(response, headers)`, `(response, status, headers)` status is int and override the preset status code, headers is a list/dict
6. other => convert valid WSGI application object into res obj.

## Session

Flask built-in support for session is Client-side based sessions, or technically, "server-side signed client storage"

- Session data is serialized into cookies and sent back to client. This data is cryptographically signed.
- User can read session data inside cookies in their browser, but they can't modify it, since doing so will cause signature mismatch the next time a request is sent to the server

## Blueprints

A Blueprint obj works similarly to a Flask application obj, but it defines how to construct or extend an application. If you find yourself in need of 2 Flask application objects, 2 blueprints are for you.

**Use cases:**

- Register a Blueprint on an application at a specific URL prefix and/or subdomain. Parameters in the URL prefix/subdomain become **common** view arguments, or become accessible across all view functions in the same blueprint.
- On the same application, register multiple blueprints with different URL rules.
- Provide template filters, static files, templates, ... Blueprint doesn't have to implement applications/view functions.

**NOTE:** each blueprint should be registered ONCE.

**Blueprints over multiple app objects:**

- Multiple app objects == multiple app configs, app objects are managed by WSGI layer.
- One app object, multiple Blueprints == share app config, change app object as necessary. You can't unregister a Blueprint without destroying the whose app object.

### Basic Syntax

```py
simple_page = Blueprint("simple_page", __name__)

# prefix the endpoint of the function with the name of the blueprint
# i.e. 'simple_page.show'
# NOTE: blueprint name doesn't modify URL
@simple_page.route('/', defaults={'page': 'index'})
@simple_page.route('/<page>')
def show(page):
    try:
        print(url_for("simple_page.show2")) # external blueprint linking
        print(url_for(".show2"))            # internal blueprint linking

        return render_template(f'pages/{page}.html')
    except TemplateNotFound:
        abort(404)

@simple_page.errorhandler(404)
def page_not_found(e):
    return render_template("pages/custom_error.html")

# ============================================================================ #

# app.py
app = Flask(__name__)
app.register_blueprint(
    simple_page,
    url_prefix="...", # override blueprint's url_prefix
)
```

### Advanced Syntax

```py
print(app.url_map)
# output: Map([<Rule '/static/<filename>' (HEAD, OPTIONS, GET) -> static>,
#  <Rule '/<page>' (HEAD, OPTIONS, GET) -> simple_page.show>,
#  <Rule '/' (HEAD, OPTIONS, GET) -> simple_page.show>])

# - 1st rule: from the app object itself to serve static files
# - 2nd rule: from Blueprintt `simple_page`, note the endpoint name `simple_page.show`
# - 3rd rule: same as 2nd rule, but different URL

# Nested Blueprint
parent = Blueprint('parent', __name__, url_prefix='/parent')
child = Blueprint('child', __name__, url_prefix='/child')
parent.register_blueprint(child)
app.register_blueprint(parent)

# child endpoint name: parent.child.*
# child url prefix: /parent/child/*

parent = Blueprint('parent', __name__, subdomain='parent')
child = Blueprint('child', __name__, subdomain='child')
parent.register_blueprint(child)
app.register_blueprint(parent)

# child's /create endpoint
print(url_for('parent.child.create', _external=True))
# output: "child.parent.domain.tld/parent/child/create"

@parent.before_request
def parent_before_request():
    pass

@child.before_request
def child_before_request():
    pass

# child's route will also executed parent_before_request()

# Blueprint API
simple_page = Blueprint(
    name="simple_page", # prefixed into endpoint name later, e.g. 'simple_page.create', 'simple_page.delete'
    import_name=__name__, # used to infer "resource" directory
    url_prefix="simple" # prefixed into URL, e.g. @simple_page.route("/create") => /simple/create
    # Blueprint static folder is different from application static route '/static'
    # Accessible as a built-in endpoint 'simple.static' ONLY WHEN url_prefix IS SPECIFIED
    static_folder="static", # abs or rel to blueprint's resource folder
    template_folder="templates", # abs or rel to blueprint's resource folder
)

# utilize static_folder
print(url_for("simple.static", filename="styles.css"))
print(url_for("simple.static", filename="simple.js"))

# path to resource folder
print(simple_page.root_path)
```

## Extensions

### Flask-MySQLdb

List of `cursor` methods:

- `execute(query, params=None)`
- `executemany(query, seq_of_params)`
- `fetchone()`
- `fetchall()`
- `fetchmany(size=None)`
- `close()`
- callproc(procname, args=())
- nextset()
- scroll(value, mode='relative')
- setinputsizes(sizes)
- setoutputsizes(size, column=None)
- mogrify(query, args=None)
- description (attribute, not a method)
- rowcount (attribute, not a method)
- lastrowid (attribute, not a method)

```py
from flask_mysqldb import MySQL

app = Flask(__name__)
app.config["MYSQL_HOST"] = "127.0.0.1"
app.config["MYSQL_USER"] = "brainfort"
app.config["MYSQL_PASSWORD"] = "XJo8b%57qVxjmkNo"
app.config["MYSQL_DB"] = "brainfort_db"
app.config["MYSQL_CURSORCLASS"] = "DictCursor"  # fetchone() and fetchall() return Dict instead of Tuples
# app.config["MYSQL_CUSTOM_OPTIONS"] = {"ssl": {"ca": "/path/to/ca-file"}}  # cre: https://mysqlclient.readthedocs.io/user_guide.html#functions-and-attributes

mysql = MySQL(app)
# mysql = MySQL()
# mysql.init_app(app)

with app.test_request_context():
    cur = mysql.connection.cursor()
    cur.execute(
        """
        SELECT * FROM users;
        """
    )
    results = cur.fetchall()
    cur.close()
```

### Flask-JWT-Extended

- JWT support to Flask for protected routes.
- Custom claims to JWT and validate custom claims on received tokens.
- Auto user loading `current_user`
- Refresh token
- First class support for fresh tokens for making sensitive changes
- Token revoking/blocklisting
- Storing tokens in cookies and CSRF protection

```py
import redis
from datetime import datetime, timedelta, timezone
from flask import Flask, jsonify, request
from flask_bcrypt import Bcrypt
from flask_mysqldb import MySQL
from flask_jwt_extended import (
    create_access_token,
    create_refresh_token,
    get_jwt,
    get_jwt_identity,
    jwt_required,
    JWTManager,
    get_current_user,
    set_access_cookies,
    unset_jwt_cookies,
)

app = Flask(__name__)
app.config["MYSQL_HOST"] = "127.0.0.1"
app.config["MYSQL_USER"] = "brainfort"
app.config["MYSQL_PASSWORD"] = "XJo8b%57qVxjmkNo"
app.config["MYSQL_DB"] = "brainfort_db"
app.config["MYSQL_CURSORCLASS"] = "DictCursor"  # fetchone() and fetchall() return Dict instead of Tuples
# app.config["MYSQL_CUSTOM_OPTIONS"] = {"ssl": {"ca": "/path/to/ca-file"}}  # cre: https://mysqlclient.readthedocs.io/user_guide.html#functions-and-attributes

app.config["JWT_ACCESS_TOKEN_EXPIRES"] = timedelta(hours=1)  # 15m? IDK
app.config["JWT_REFRESH_TOKEN_EXPIRES"] = timedelta(
    days=30
)  # CAUTION: introduce refresh token means complex revocation logic
# app.config["JWT_IDENTITY_CLAIM"] = "sub" # name of the key that stores the identity claim (key pair, remember?)
# app.config["JWT_PRIVATE_KEY"] = ... # Asymmetrical key
# app.config["JWT_PUBLIC_KEY"] = ...  # Asymmetrical key

app.config["JWT_REFRESH_TOKEN_EXPIRES"] = timedelta(
    days=7
)  # def=30, datetime.timedelta, dateutil.relativedelta, or number of seconds.

app.config["JWT_SECRET_KEY"] = "this-is-not-a-secret-key"  # CAUTION: always use a dedicated key in production

app.config["JWT_TOKEN_LOCATION"] = [
    "headers",
    "cookies",
]  # def="headers", supported: "headers", "cookies", "query_string", "json"

app.config["JWT_COOKIE_SAMESITE"] = "Strict"
app.config["JWT_COOKIE_SECURE"] = False  # CAUTION: always set to True in production
app.config["JWT_CSRF_CHECK_FORM"] = True

mysql = MySQL(app)
# mysql = MySQL()
# mysql.init_app(app)

bcrypt = Bcrypt(app)
# bcrypt.init_app(app)

jwt = JWTManager(app)


# ============================================================================ #
# AUTOMATIC USER LOADING                                                       #
# ============================================================================ #

# NOTE: convert identity into string
@jwt.user_identity_loader
def user_identity_lookup(user):
    if isinstance(user, dict):
        return user["id"]
    return user


# NOTE: convert a JWT into a Python object that can be used in a protected endpoint
@jwt.user_lookup_loader
def user_lookup_callback(jwt_header, jwt_data):
    identity = jwt_data["sub"] # short of "subject claim"
    cur = mysql.connection.cursor()
    cur.execute(
        """
        SELECT * FROM google_users WHERE id = %s
        """,
        (identity,),
    )
    user = cur.fetchone()
    cur.close()
    return user


@app.route("/login", methods=["POST"])
def login():
    username = request.json.get("username", None)
    password = request.json.get("password", None)

    cur = mysql.connection.cursor()
    cur.execute(
        """
        SELECT * FROM users WHERE username = %s
        """,
        (username,),
    )
    user = cur.fetchone()  # REMEMBER: user is a Python Dict
    cur.close()

    app.logger.debug(user)

    if not user or not bcrypt.check_password_hash(pw_hash=user["password_hash"], password=password):
        return jsonify({"msg": "Wrong username or password"}), 401

    # if username != "test" or password != "test":
    #     return jsonify({"msg": "Incorrect username/password"}), 401

    # ======================================================================== #
    # STORING ADDITIONAL DATA IN JWT                                           #
    # ======================================================================== #

    additional_claims = {"aud": "some audience", "foo": "bar"}
    access_token = create_access_token(user, additional_claims=additional_claims)  # override default claims

    return jsonify(access_token=access_token)


# NOTE: it got merge with above additional_claims
# NOTE: I don't know the order, so it's best not to use this decorator
@jwt.additional_claims_loader
def add_claims_to_access_token(identity):
    return {"alpha": "a", "beta": "b", "charlie": "c"}

# ============================================================================ #
# JWT LOCATIONS: HEADERS, COOKIES, JSON BODY, QUERY STRING,                    #
# ============================================================================ #

# NOTE: make sure to set the JWT_TOKEN_LOCATION appropriately to allow the tokens to be sent

@app.route("/login-without-cookies", methods=["POST"])
def login_without_cookies():
    access_token = create_access_token(identity="user with no cookies")

    # Access Token in HTTP Response Body in JSON format
    # Front-end code will have to manage the access token manually (not recommended)
    return jsonify(access_token=access_token)


@app.route("/login-with-cookies", methods=["POST"])
def login_with_cookies():
    response = jsonify({"msg": "Login successful"})
    access_token = create_access_token(identity="user with cookies")

    # modify a Flask response to set a cookie containing a JWT in `access_token` format
    set_access_cookies(response, access_token)
    return response


@app.route("/logout-with-cookies", methods=["POST"])
def logout_with_cookies():
    response = jsonify({"msg": "Logout successful"})

    # a Flask response is needed to delete the cookies containing:
    # - Access Token
    # - Refresh Token
    # - CSRF Access Token
    # - CSRF Refresh Token
    unset_jwt_cookies(response)
    return response


# JWT location-specific route
@app.route("/only-headers")
@jwt_required(locations=["headers"])
def only_headers():
    return jsonify(foo="bar")


# ============================================================================ #
# PROTECTED ROUTE                                                              #
# ============================================================================ #

@app.route("/protected", methods=["GET"])
@jwt_required()
def protected():
    current_identity = (
        get_jwt_identity()
    )  # the identity is taken from user_identity_lookup(), not from create_access_token anymore
    jwt_payload = get_jwt()  # return Python Dict
    return jsonify(logged_in_as=current_identity, foo=jwt_payload["foo"], alpha=jwt_payload["alpha"]), 200


# ============================================================================ #
# PARTIAL PROTECTED ROUTE                                                      #
# ============================================================================ #

@app.route("/optional-protected", methods=["GET"])
@jwt_required(optional=True)
def optional_protected():
    current_identity = get_jwt_identity()
    if current_identity:
        return jsonify(logged_in_as=current_identity)
    else:
        return jsonify(logged_in_as="guest")


@app.route("/me", methods=["GET"])
@jwt_required()
def me():
    # access User dict set by user_lookup_callback()
    current_user = get_current_user()
    return jsonify(id=current_user["id"], username=current_user["username"], email=current_user["email"])
```

Client-side JS code handling JWT:

```js
// Version 1: Web Service API (XSS-vulnerable)
async function login() {
  const response = await fetch("/login-without-cookies", { method: "post" });
  const result = await response.json();
  localStorage.setItem("jwt", result.access_token);
}

function logout() {
  localStorage.removeItem("jwt");
}

// JWT in Authorization Header
async function makeRequestWithJWT() {
  const options = {
    method: "post",
    headers: {
      Authorization: `Bearer ${localStorage.getItem("jwt")}`,
    },
  };
  const response = await fetch("/protected", options);
  const result = await response.json();
  return result;
}

// JWT in GET Query String
async function makeRequestWithJWTUsingQueryString() {
  const jwt = localStorage.getItem("jwt");
  // Sending password reset link
  const response = await fetch(`/reset-password?jwt=${jwt}`, {
    method: "post",
  });
  const result = await response.json();
  return result;
}

// JWT in HTTP Response Body
async function makeRequestWithJWTUsingJSONBody() {
  const options = {
    method: "post",
    body: JSON.stringify({
      // naming convention
      access_token: localStorage.getItem("jwt"),
    }),
    headers: "application/json",
  };
  const response = await fetch("/protected", options);
  const result = await response.json();
  return result;
}

// Version 2: Double Submit Verification (secure)

async function login() {
  await fetch("/login-with-cookies", { method: "POST" });
}

async function logout() {
  await fetch("/logout-with-cookies", { method: "POST" });
}

function getCookie(name) {
  const value = `; ${document.cookie}`;
  const parts = value.split(`; ${name}=`);
  if (parts.length === 2) return parts.pop().split(";").shift();
}

async function makeRequestWithJWT() {
  const options = {
    method: "POST",
    credentials: "same-origin",
    headers: {
      // naming convention: https://flask-jwt-extended.readthedocs.io/en/stable/options.html#cross-site-request-forgery-options
      "X-CSRF-TOKEN": getCookie("csrf_access_token"),
    },
  };
  const response = await fetch("/protected", options);
  const result = await response.json();
  return result;
}

// NOTE: <form> element can also include double submit token, ideal for login/signup form
```

**Refresh token implementation:**

```py
# Method 1: Implicit refreshing with cookies without refresh tokens
@app.after_request
def refresh_expiring_jwts(response):
    try:
        jwt = get_jwt()
        exp_timestamp = jwt["exp"]
        now = datetime.now(timezone.utc)
        target_timestamp = datetime.timestamp(now + timedelta(minutes=30))  # arbitrary
        if target_timestamp > exp_timestamp:
            access_token = create_access_token(identity=get_jwt_identity())

            # 1st time is at the end of the '/login' view function
            # set the 2nd time
            set_access_cookies(response, access_token)
            return
    except (RuntimeError, KeyError):
        return response


# Method 2: Explicit refreshing with refresh token
@app.route("/login2", methods=["POST"])
def login2():
    # skip validation and querying and checking...
    access_token = create_access_token(identity="test refresh token")
    refresh_token = create_refresh_token(identity="test refresh token")

    # Both Access Token and Refresh Token is returned inside HTTP Response Body in JSON format
    # NOTE: make sure to set the JWT location to "json"
    return jsonify(access_token=access_token, refresh_token=refresh_token)


@app.route("/refresh", methods=["POST"])
@jwt_required(refresh=True)  # only allow refresh tokens to access the route
def refresh():
    identity = get_jwt_identity()
    access_token = create_access_token(identity=identity)
    return jsonify(access_token=access_token)
```

**JWT Revoking**

Use Redis if your use case is to simply check if a JWT has been revoked.

**Pros:** fast, can be configured to persist data, can automatically clear JWTs after they expire by utilizing Redis's TTL functionality

```py
# Setup Redis connection
# IMPORTANT: Redis must be configured to persist data to disk
jwt_redis_blocklist = redis.StrictRedis(host="localhost", port=6379, db=0, decode_responses=True)

# NOTE: this function will be called whenever a valid JWT is used to access a protected route
@jwt.token_in_blocklist_loader
def check_if_token_is_revoked(jwt_header, jwt_payload):
    jti = jwt_payload["jti"]
    token_in_redis = jwt_redis_blocklist.get(jti)
    return token_in_redis is not None  # True if JWT has been revoked


@app.route("/logout2", methods=["DELETE"])
@jwt_required()
def logout2():
    jwt = get_jwt()  # jwt_payload data
    jti = jwt["jti"]

    # save the JWTs unique identifier (JTI) inside Redis
    # set a TTL when storing the JWT
    # the key-value pair will be cleared out after an hour
    # empty string value is perfectly fine
    jwt_redis_blocklist.set(jti, "", ex=timedelta(hours=1))  # same as Access Token
    return jsonify(msg="Access token revoked")


@app.route("/protected3", methods=["GET"])
@jwt_required()
def protected3():
    # a blocklisted access token can no longer access this route
    return jsonify(hello="world")
```

---

When logged out, both Access Token and Refresh Token must be revoked.

- Normally, Frontend must send 2 separate requests to 2 seperate API endpoints to revoke both tokens.

- By defining a single endpoint with `@jwt_required(verify_type=False)`, one route is enough. But front-end still have to send 2 requests.

- Send the Access Token in the header, and send the Refresh Token in the HTTP Request Body. Still requires front-end manual front-end code.

- (Most recommended) Embed the refresh token's JTI in the access token. The revoke route is authenticated with access token, then extract the both the access token's JTI and the refresh token itself.

```py
@app.route("/logout3", methods=["DELETE"])
@jwt_required(verify_type=False)
def logout3():
    jwt = get_jwt()  # the value of this is depending on the client's request
    jti = jwt["jti"]
    ttype = jwt["type"]
    jwt_redis_blocklist.set(jti, "", ex=timedelta(hours=1))
    return jsonify(msg=f"{ttype.capitalize()} token successfully revoked.")
```

## WSGI Middleware

...

## References

- [Flask's Official Documentation > Quick Start](https://flask.palletsprojects.com/en/stable/quickstart/)
- [Flask's Official Documentation > Tutorial](https://flask.palletsprojects.com/en/stable/tutorial/)
- [](https://flask.palletsprojects.com/en/stable/patterns/celery/)
