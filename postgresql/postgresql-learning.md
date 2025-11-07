# PostgreSQL Learning

## Cheatsheet

Docker Compose file:

```yml
services:
  postgres:
    image: postgres:18-bookworm
    restart: always
    # shared memory is used for caching data, query plans, and managing concurrent connection
    # NOTE: unlike PostgreSQL, MySQL/MariaDB use RAM
    shm_size: 128mb
    # set shared memory limit when deploy via swarm mode
    #volumes:
    #  - type: tmpfs
    #    target: /dev/shm
    #    tmpfs:
    #      size: 134217728 # 128 MB
    environment:
      - POSTGRES_USER=hatchet
      - POSTGRES_PASSWORD=hatchet
      - POSTGRES_DB=hatchet
      - POSTGRES_INITDB_ARGS=--data-checksums
      # more...
    volumes:
      - hatchet_lite_postgres_data:/var/lib/postgresql/data
      # - "$PWD/my-postgre.conf":/etc/postgresql/postgresql.conf
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -d hatchet -U hatchet"]
      interval: 10s
      timeout: 10s
      retries: 5
      start_period: 10s
    # command: postgres -c 'config_file=/etc/postgresql/postgresql.conf'
    command: postgres -c shared_buffers=256MB -c max_connections=200
```
