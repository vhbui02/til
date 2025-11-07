# Docker Compose Learning

## [Variable interpolation in Docker Compose file"](https://docs.docker.com/compose/how-tos/environment-variables/variable-interpolation/)

DO NOT edit Compose file when doing the following:

- Switch between image tags to test multiple versions
- Adjust volume source to local environment.
- ...

THREE ways to set variables with interpolation:

1. Host's shell environment's variables.
2. Variables set by an `.env` file in the same directory as the Compose file.
3. Variables set by `--env-file path/to/env/file`

The environment variables are for interpolation ONLY, which means they will NOT being populated into container's environment.

> [!TIP]
>
> Check variables interpolation result: `docker compose config --environment`

- `environment:` attribute can reference variables defined in env files (e.g. `- ENV=${ENV}`.
- Using `--env-file`, `.env*` file can be placed in other location (e.g. `./instance`).
- Using `--env-file`, you can override a main `.env` file with `.env.dev`, `.env.test`, ...
- Multiple `--env-file` can be used to specify multiple env files, latters override formers.
- Of course, `-e FOO=bar` provide ad-hoc env var that shouldn't be written into `.env*` files.

> [!CAUTION]
>
> `env_file:` attribute, despite having the same name as `--env-file`, doesn't add env vars for variable interpolation. It's used to set environment variables inside **container's shell environment**.

## [Nick Janetakis's Best Practices](https://nickjanetakis.com/blog/best-practices-around-production-ready-web-apps-with-docker-compose)

0. Leverate variable interpolation for different environments

```yml
web:
  volumes:
    - "${DOCKER_WEB_VOLUME:-./public:/app/public:ro}" # :rw if you app have features such as uploading files and store directly on disk (use S3 is more recommended)
  restart: "${DOCKER_RESTART_POLICY:-unless-stopped}"

# .env.dev
DOCKER_WEB_VOLUME=./src:/app/src
DOCKER_WEB_VOLUME=.:/app # mount the whole directory
DOCKER_RESTART_POLICY=no
```

- Naming convention would be `/public`
- A bind mount is a reasonable choice when the static files are being served via an Nginx server that's not running in a container.
- This bind mount can be either `:ro` or `:rw` (default) if your app have features that uploading files directly to disk.

1. Two ways to define Docker Compose for multiple environments:

- [Profiles](https://docs.docker.com/compose/how-tos/profiles/): ONE `docker-compose.yml` file, multiple services, each service corresponds to a profile, THREE `.env` for 3 environments, each env must direct which profiles to use. Some services will have its dev counterpart (e.g. `web:` service with `profiles: ["web"]` and `web-dev:` service with `profiles: ["web-dev"]`.

```yml
services:
  db:
    profiles: ["db"]
  kv:
    profiles: ["kv"]
  web:
    profiles: ["web"]
    depends_on: ["db", "kv"]
  web-dev:
    profiles: ["web-dev"],
    depends_on: ["db", "kv"] # there is no difference between db in prod vs dev
  worker:
    profiles: ["worker"],
    depends_on: ["db", "kv"]
  assets:
    profiles: ["assets"]
```

```sh
# running custom commands is tedious
docker compose --profile web,worker up
```

```sh
# .env
export COMPOSE_PROFILES=web,worker
# .env.dev
export COMPOSE_PROFILES=db,kv,assets,web,worker
```

Profiles support multi-hosts deployment:

```sh
# .env on web server
export COMPOSE_PROFILES=web

# .env on cron server
export COMPOSE_PROFILES=worker
```

or using `--profile` command line arguments:

```sh
DOCKER_HOST="ssh://user@web" docker compose --profile web ...
DOCKER_HOST="ssh://user@worker" docker compose --profile worker ...
```

> [!NOTE]
>
> "assets", or asset watcher (`esbuild`, `tailwind`), isn't needed, since they have been built and copy into `web` Docker Image.
>
> "db", "kv" isn't needed as well, due to:
>
> - Some teams don't deploy databases and caches inside Docker for production.
> - For production, `.env` I might specify a `DATABASE_URL`, `REDIS_URL`, ... which point to cloud-managed service that take precedence before other environment configurations which point to Docker service.

- Override: TWO files: `docker-compose.base.yml` and `docker-compose.dev.yml`, multiple services, THREE `.env` for 3 environments.

2. Docker Compose or Docker Swarm (Compose v3) or K8s.

- Docker Compose if deploying multi-container stack on a single host.
- Docker Swarm for simple workloads deploying on multi-on-premise hosts.
- K8s: https://doineedkubernetes.com/

3. Advanced DAG settings

```yml
x-app: &default-app
  # some configs
  # ...
  depends_on:
    postgre:
      condition: "service_started"
      required: false
    redis:
      condition: "service_started"
      required: false
```

4. Reduce Service Duplication with Aliases and YAML Anchors

DRY and a production-first, development-overriden practice.

```yml
# syntax
# name the `x-` properties anything you want
x-app: &default-app

x-web: &default-assets

web:
  <<: *default-app
worker:
  <<: *default-app
js:
  <<: *default-assets
css:
  <<: *default-assets
```

5. `HEALTHCHECK` in Docker Compose, NOT Dockerfile

Never make assumptions about the deployment platform: VPS + Docker Compose/K8s Cluster/Heroku. How they're run is different, despite all of them having Docker underneath.

For example, K8s disables `HEALTHCHECK` in Dockerfile because it relies on its custom readiness checks.

=> Therefore, define your `HEALTHCHECK` in Docker Compose. Use can use variable interpolation to conditionally tweak its value according to specific environment.

```yml
web:
  <<: *default-app
  healthcheck:
    # Fact: `/up` is a naming convention chosen by the creator of Redis.
    # I use `/health`
    test: "${DOCKER_WEB_HEALTHCHECK_CMD:-curl localhost:8000/up}"
    interval: "60s"
    timeout: "3s"
    start_period: "5s"
    retries: 3
```

```py
@app.get("/up")
def up():
  redis.ping()
  db.engine.execute("SELECT 1")
  return ""

# very lightweight yet ensure the app is working and can connect to PostgreSQL + Redis.
# it's proof that the database is live, can be login with the current credential and have at least read access
# under normal condition, the API calling takes about 1ms
```

For development environment, you do not want your log shell to be cluttered by the result from curl'ing healthcheck endpoint. It's best to send it to the void.

```env
export DOCKER_WEB_HEALTHCHECK_CMD=/bin/true
```

6. Leverage environment variables in multiple files

- `.env` tend to include sensitive information and should be ignored from version control (unless you use 3rd-party credential manager such as AWS Secret Manager/AWS System Manager Parameter Store).

- Maintain an `.env.example` which would be commited to version control. This is needed to ensure team development experience and write CI workflows.

- Add `.env.dev`, `.env.test`, ... if you have multiple environments.

```env
# comment out default value is a good practice
# avoid going back-and-fort for default value
#export FLASK_ENV=production
#export NODE_ENV=production
export FLASK_ENV=development
export NODE_ENV=development
```

7. Running Nginx outside of Docker directly on the Docker host for single server deployment. For multiple hosts, consider using Traefik + Swarm.

Benefits:

- More performant in static files serving.
- SSL termination (decrypt incoming SSL/TLS encrypted traffic at the load balancer instead of doing so at backend servers will grant you simplified certificate management and app server performance improvement).
- Decompress request, compress response.
- Get country code headers, IP, ...
- Set custom Request Header
- WAF: Control deny/allow lists
- Redirect from HTTP to HTTPS, from `www` to apex domain, ...
- Compiled/Minified JS/CSS and MD5 tagged static files to ensure cache busting (file change ==> hash change ==> signal browser should fetch new version).

When running Nginx directly on host, you can control which ports are exposed to the public network. Typically, only port 80 + 443 are open for web traffic. Nginx will listen on these ports on forward your requests to your web applications running on different ports (`8080`, `5000`, `8000`, ...). These ports can be accessed if:

- `iptables` rules are not configured correctly.
- App bound to `0.0.0.0` when port mapping or using `network_mode: host`.

Memory refresh, Docker networking bypass Linux firewall (`ufw`, `firewalld`, ...).

8. Avoid running Nginx in a Docker Container.

When updating Docker:

- If you need a quick, simple solution in exchange of little downtime: Serve a 503 maintainance page that doesn't depend on Docker.

- If you can't tolerate downtime, a long and complex migration strategy is needed: Spin up a brand new server, update DNS records, wait for DNS to propagate, destroy old server. Mirror your infra then replace DNS.

SSL cert management is more complicated, hard to get DNS challenges working to issue wildcard cert.

9. Nginx

- App should be decoupled from reverse proxies. You might change your deployment infrastructure in the future (today is self-hosted VPS, tomorrow is K8s cluster, why should you be bother)

- Nginx in Docker is NOT for testing production environment on development. One way or another you will run Nginx differently. Configure a dedicated testing/staging environment (testing with mock production user data + simulate user traffic).

10. Resource limiting

```yml
web:
  deploy:
    resources:
      limits:
        cpus: "${DOCKER_WEB_CPUS:-0}"
        memory: "${DOCKER_WEB_MEMORY:-0}"
```

- `0` == use as many resources as they need.
- For single server deploy, leave it as-is, unless you're using eccentric tech stack that can gobble up as many resources as it can.
- Select optimal server hardware specifications based on the results of stress testing.
- Kubernetes works best when you specify the exact resource requirements for a single instance of your application.

11. Web Server config file

`.env` is not the only place to load configuration file. Flask and other Python-based web frameworks have a dozan way to load configurations that aren't limited to `.env` file.

```py
bind = f"0.0.0.0:{os.getenv('PORT', '5000')}"
```

Binding `0.0.0.0` from inside the container allows browser on host to connect to inside of the container.

Gunicorn, uWSGI, ... WSGI app server in general have something called _worker_ and _thread count_, control how many IO-bound + CPU-bound tasks/seconds your app server can serve. The more you have, the more concurrency (not parallelism) you can handle, at the cost of using more CPU and memory.

```py
# NOTE: Heroku naming convention
workers = int(os.getenv("WEB_CONCURRENCY", multiprocessing.cpu_count() * 2))
threads = int(os.getenv("PYTHON_MAX_THREADS", 1))
```

For development environment, set both of them to 1 to debug easier. Setup code reloading as well:

```py
from distutils.util import strtobool
reload = bool(strtobool(os.getenv("WEB_RELOAD", "false")))
```

Finally, always log to stdout, instead of a file on disk when working with Docker:

- First, Docker container are ephemeral. The moment the container is removed, there goes your log as well.
- Second, there are a lot of ways to transfer/persist log to do archive/monitor/analysis/... For small scale, use `journald` to log and explore your log via `journalctl`. AWS CloudWatch is also a good choice.

12. Database configuration

```py
# NOTE: PostgreSQL Docker image naming convention
pg_user = os.getenv("POSTGRES_USER", "hello")
pg_pass = os.getenv("POSTGRES_PASSWORD", "password")
pg_host = os.getenv("POSTGRES_HOST", "postgres")
pg_port = os.getenv("POSTGRES_PORT", "5432")
pg_db = os.getenv("POSTGRES_DB", pg_user)

# override file pattern
db = f"postgresql://{pg_user}:{pg_pass}@{pg_host}:{pg_port}/{pg_db}"
DATABASE_URI = os.getenv("DATABASE_URI", db)
# similar
REDIS_URI = os.getenv("REDIS_URI", "redis://redis:6379/0")
```

By loading `DATABASE_URI` this way, you can use a DaaS outside Docker for production and an on-premise locally running PostgreSQL container for development.

13. Utilize Dockerfile multi-stage built

For a project with front-end using Node-based frameworks (Vite) and back-end using Python-based frameworks (Flask), you can build/bundle static front-end assets and copy to back-end image:

```Dockerfile
# building /app/public in "assets" build stage...
# inside "app" build stage
COPY --chown=python:python --from=assets /app/public /public
```

Python image contains static assets without needing to install a single thing! Now you can volume mount those files from inside image to outside hosts so Nginx on host can serve them (yes, volume mount is bi-directional)

14. Run container as non-root and change file permissions accordingly

```Dockerfile
FROM node:... as assets
WORKDIR /app/assets
RUN mkdir -p /node_modules && chown node:node -R /node_modules /app
USER node
# to make the files owned by 'node', don't forget --chown=node:node
COPY --chown=node:node assets/package.json assets/*yarn* ./
# yarn install as node!
RUN yarn install
```

15. Volume mounts in development environment

- Rootful Docker: write `user: "1000:1000"` or `-u="1000:1000"`.
- Rootless Docker: The files and directory on host will be owned by auxiliary users that host's user can't write into them. When mounting existing directories, user inside container can't write into them.

=> Solution is to either mount as root or creating files and directories at host with corresponding UID/GID first. (`524388` on host will map to `1000` on container)

16. Create custom user in case base image don't have one

Always create a home user, you never know your package's behavior.

```Dockerfile
# Python base image don't have one
RUN useradd --create-home python
ENV USER=python
```

17. Customize where package dependencies get installed:

```Dockerfile
# /node_modules: ensure dependencies can be written into the directory as `node` user
# /app: prevent permission errors when copying files between multi-state
RUN mkdir -p /node_modules && chown node:node -R /node_modules /app
# ...
COPY --chown=node:node assets/package.json assets/*yarn* ./
```

You can see that I've put `/node_modules` in root dir. To install Node dependencies into that folder, inside `.yarnrc` specify `--modules-folder /node_modules`.

Now the benefit of a `/node_modules` directory is that it's not placed in the app's `WORKDIR` so the chance it get accidentally volume mounted is very low.

Note that this pattern doesn't get limited to Node only. Python can also do this.

18. Docker 101 in layer caching: ALWAYS install dependencies first, then copy files.

19. Build arguments: being able to run something specific during build time that can be controlled via command line arguments or Compose `args:` attribute inside `build:` attribute, which can also be adjusted by `.env`

=> Use `.env` as the SSOT for everything, from image building to runtime

Some use cases that need `ARG` build arguments: `FLASK_ENV`, `NODE_ENV`, `UID`, `GID`, ...

20. Bring immutables into environment variables: Some can be used. Other, leave it.

```Dockerfile
ARG FLASK_ENV="production"
ENV FLASK_ENV="${FLASK_ENV}" \
    FLASK_APP="hello.app" \
    FLASK_SKIP_DOTENV="true" \
    # not get buffered will ensure the order of log message
    PYTHONUNBUFFERED="true" \
    # some package may expect this
    # "." implicitly points to WORKDIR
    PYTHONPATH="." \
    PATH="${PATH}:/home/python/.local/bin" \
    USER="python"
```

**EXPOSE**: it's for informational purposes.

**Array** (`exec`) or **Shell** (`sh`) syntax: ALWAYS USE ARRAY, trade variable substitution capability with being able to run the process as PID 1.

**ENTRYPOINT script:** Sometimes you have a list of work that needed to do after running containers, entrypoint script to do that. When does a command be written as a `RUN` instruction vs. as a command inside an entrypoint script?

- If the work is deterministic, put it in as a `RUN` instruction inside Dockerfile.
- If the work is non-deterministic, create an `/app/bin` directory inside container then put the script in there then run it.

**Merge build-time and run-time assets and handle volume mounting behavior.**: `cp -r /public /app`.

- If you mount a Docker volume at `/app` (e.g. for live code reload or local development, it overwrites everything in `/app` inside the container with your host directory)
- Any files or assets that were copied to `/app` during image build will be hidden by the mounted volume

```yml
volumes:
  - ./app:/app
```

=> That's why you don't copy the `public` folder directly into `/app` folder. Always copy built assets at container startup if you expect something to be a mounted volume.

**Construct a good `.gitignore` and `.dockerignore`**:

- Docker Security 101: NEVER copy your `.env` file into image layer. Ignore all of the `.env`. A common practice is to use `scp` to transfer the `.env` file with production configuration onto production server and load it using `env_file:` directive.

- Everything inside `public/` folder shouldn't be version-controlled, but the folder itself can be kept. This is to ensure the directory ends up being owned by the correct user when mounting.

## Dockerfile Naming Convention

1. **Environment-specific:** `dev.Dockerfile`, `prod.Dockerfile`, `test.Dockerfile`
2. **Base image variations:** `alpine.Dockerfile`, `debian.Dockerfile`
3. **Architecture-specific:** `arm64.Dockerfile`, `amd64.Dockerfile`, `cpu.Dockerfile`
4. **Service-specific:** `web.Dockerfile`, `api.Dockerfile`

## Services Naming Convention

1. Name based on **Functional Role** rather than **Technology**.
2. Use consistent naming patterns across services
3. Choose names that make sense in application context.

This creates better abstraction - if you change from application A to application B, the service function remains the same even though the implementation changes.

## Secrets in Compose

Secret - sensitive data (password, cert, API key) - that shouldn't be transmitted over a network, stored unencrypted in Dockerfile, in application source code or version control system.

DO NOT use env var. they're available to all processes, difficult to track access, might be printed in logs when debugging.

Secrets are mounted as a file in `/run/secrets/<secret-name>` inside the container.

By using file, this even allowed granular access control via standard filesystem permissions.

```yml
services:
  my_app:
    image: my_app:latest
    environment:
      MY_SECRET_FILE: /run/secrets/my_secret.txt
      MY_SECRET_2_FILE: /run/secrets/my_secret_2.txt
    secrets:
      - my_secret
      - my_secret_2

# top-level directives
secrets:
  my_secret:
    file: ./my_secret.txt
  my_secret_2:
    file: ./my_secret_2.txt
```

**NOTE:** IMO, use `env_file:` is less verbose yet ensure the same security level. Only use Secret when:

- `env_file:` is being used
- You want to make a quick override.

## Merge Compose files vs `include:` Compose files

### Merge

By default, Compose reads two files:

- `compose.yml`: contain base config
- `compose.override.yml` : optional, contain config overrides for EXISTING services or NEW services.

TWO methods to use multiple override files with different names:

- Use `COMPOSE_FILE=` env var.

```sh
# Linux
export COMPOSE_FILE="my-compose.yaml:my-compose.prod.yaml"

# Windows
# Replace : with ;
```

- Use `-f` option. Path to overriden files are relative to base file

```sh
docker compose -f my-compose.yml -f my-compose.prod.yaml ...
```

**NOTE:** override files need not be valid Compose file, it can contain a small fragment of config.

Merge behavior if a service is defined in both files:

- Single-value option (e.g. `image`, `command`, `mem_limit`, ...): new value replaces old value
- List-type options (e.g. `ports`, `expose`, `dns`, `dns_search`, `external_links`, `tmpfs`, ...): values are concatenated
- Dict-type options (e.g. `volumes`, `environment`, `labels`, `devices`): values are merged, with new value take precedence.

It's best to separate your `docker-compose.yml` to three files:

- `docker-compose.base.yml`: general settings
- `docker-compose.dev.yml`: source code volume mounts, hard-coded credentials in environment variables, unoptimized docker build image, ...
- `docker-compose.prod.yml`: inter-container communication and security management (run as non-root, set filesystem read-only, drop all capabilities, ...)

### Include

Incorporate a separate `compose.yaml` file in the current file.

=> Modularize complex applications into sub-Compose files.

```yml
include:
  - my-compose-include.yaml # serviceB: defined
  - oci://docker.io/username/my-compose-app:latest # OCI artifact, Git repository, ...
  - override.yaml # if somehow my-compose-include.yaml define serviceA, you will need to add an override file
  # not recommended, add override file maintainance overhead!
  # it's best to avoid naming conflicts

services:
  serviceA:
    build: .
    depends_on:
      - serviceB # use serviceB: directly as if it's defined in this file
```

## References

- [Docker Tip #82: Using YAML Anchors and X Properties in Docker Compose](https://nickjanetakis.com/blog/docker-tip-82-using-yaml-anchors-and-x-properties-in-docker-compose)
- [Docker Tip #94: Docker Compose v2 and Profiles Are the Best Thing Ever](https://nickjanetakis.com/blog/docker-tip-94-docker-compose-v2-and-profiles-are-the-best-thing-ever)
- [Optional depends_on with Docker Compose v2.20.2+](https://nickjanetakis.com/blog/optional-depends-on-with-docker-compose-v2-20-2)

- [Docker Docs's "Compose file reference"](https://docs.docker.com/reference/compose-file/)
- [Docker Docs's "Merge Compose files"](https://docs.docker.com/compose/how-tos/multiple-compose-files/merge/)
- [Docker Docs's "Include"](https://docs.docker.com/compose/how-tos/multiple-compose-files/include)
