# Redis Learning

<!-- tl;dr starts -->

Redis is my most favorable Key-value store and NoSQL database in general.

<!-- tl;dr ends -->

## Overview

| Aspect               | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| **Type**             | In-memory database with optional persistance to disk         |
| **Performance**      | 110,000 SET ops/sec, 81,000 GET ops/sec                      |
| **Data Types**       | Rich: strings, hashes, lists, sets, sorted sets, HyperLogLog |
| **Atomicity**        | Single operations atomic; MULTI/EXEC transactions            |
| **Programatically**  | Lua scripts execute multiple Redis commands atomically       |
| **Batch processing** | Pipelining, reduce network round trips (RTT)                 |
| **Limitations**      | Performance degrades when dataset exceeds available memory   |
| **Replication**      | Master-slave replication                                     |
| **Persistance**      | RDB (Redis Database) and AOF (Append Only File)              |

**Use cases:**

- Small, targeted data pieces retrieving with low latency requirements.
- Multiple local, low-usage CPU `GET*` commands execution is more performant complex SQL JOIN operations.
- Cache frequently accessed data or data that requires complex SQL JOIN operations.
- Message queues (Pub/Sub)
- Web user sessions
- Hit counters
- Real-time data.
- Web server integration
- Ephemeral storage for multi-step workflow.

## Naming convention

Redis use **colon-separated hierarchical naming**.

```
# general syntax
<namespace>:<entity>:<id>:<field>

# string
user:1:name
user:1:email
config:max_connections
counter:page_views

# hash
user:1            # a user with multiple fields
session:abc123    # a session with multiple properties

# lists
queue:jobs        # job queue
timeline:user:1   # user #1's timelines
noti:user:1       # user #1's notifications

# sets
tags:article:1    # tags for article #1
followers:user:1  # followers for user #1
online_users

# sorted sets
leaderboard:game:1  # game leaderboard
trending:posts      # posts that are trending on social media
events:2025:06      # everts sorted by year and month

# include type hints
hash:user:1
list:queue:jobs
set:user:1:permission

# include versioning
user:v2:1:profile
config:v1:database
```

## Persistent strategies - RDB and AOF

1. **Redis Database (RDB):** snapshot (create a compressed binary file) entire dataset after specific intervals or a certain number of write operations.

- Pro #1: Small cold boot duration after restart.
- Pro #2: RDB files' size < AOF files' size => Good for backup
- Pro #3: Snapshot is done in background process, minimize impact on Redis performance.

- Con #1: Data loss can occur between two scheduled snapshots.
- Con #2: Backups are infrequent.

=> Best for low-profile application with low data integrity needs.

2. **Append Only File (AOF):** log every write operations received by the Redis server as a command appended to a file. When Redis restarts, it replays these commands to reconstruct the dataset.

- Pro #1: appropriate `fsync()` setting (e.g. `everysec`) data loss can be limited to a few seconds, maybe zero => High data integrity.

- Con #1: Large file size.
- Con #2: Writing command to disk can introduce a performance overhead.
- Con #3: High cold boot duration.

3. **Mixed:** use both of them.

## Cheatsheet

### Configuration

```conf
protected-mode yes
requirepass johndoe123

# Set a sensible maxmemory limit (adjust to your server)
maxmemory 2gb
maxmemory-policy allkeys-lru

# CAUTION: Enable either append-only or Redis database
# append-only file for durability
appendonly yes
appendfsync everysec
# not recommended
appendfsync always

# Redis database
appendonly no
# save a snapshot if at least 1+ write ops occur within 900 seconds
save 900 1
# save a snapshot if at least 10+ write ops occur within 300 seconds
save 300 10
# save a snapshot if at least 10k+ write ops occur within 60 seconds
save 60 10000

# Limit number of connected clients
maxclients 10000

# Set a timeout for idle clients
timeout 300

# Disable dangerous commands (optional, for extra security)
rename-command FLUSHALL ""
rename-command FLUSHDB ""
rename-command CONFIG ""
rename-command SHUTDOWN ""

# Log level and file
loglevel notice
logfile /var/log/redis/redis-server.log

# Set slowlog threshold (in microseconds)
slowlog-log-slower-than 10000
slowlog-max-len 128
```

### Performance tuning

- Install `redis-benchmark` on your production server:

```sh
# cre: https://bitsnacker.com/posts/redis-performance-hmget-pipeline-vs-mget/

# single command
$ redis-benchmark -q

# pipelining, each pipeline run 1000 commands
$ redis-benchmark -q -P 1000

# focus on SET command
$ redis-benchmark -q -t set -P 1000

# latency sampling
# measure + analyze the response time of Redis commands

# continuously sends PING commands and reports RTT latency
$ redis-cli --latency
# track latency over time for trend analysis
$ redis-cli --latency-history
# show latency distribution
$ redis-cli --latency-dist

# hmget
$ hset hash1 key1 "The golden rule of a useful benchmark is to only compare apples and apples"
$ hset hash2 key2 "The golden rule of a useful benchmark is to only compare apples and apples"
$ hset hash3 key3 "The golden rule of a useful benchmark is to only compare apples and apples"
$ redis-benchmark -P 1000 hmget hash1 key1 hmget hash2 key2 hmget hash3 key3

# mget
# NOTE: use Shell to run 1000 commands automatically
$ set hash1:key1 "The golden rule of a useful benchmark is to only compare apples and apples"
$ set hash2:key2 "The golden rule of a useful benchmark is to only compare apples and apples"
$ set hash3:key3 "The golden rule of a useful benchmark is to only compare apples and apples"
$ ...
$ set hash1000:key1000 "The golden rule of a useful benchmark is to only compare apples and apples"

# NOTE: use Shell/C/Python/... anything to generate 1000 arguments for below commands
$ redis-benchmark mget hash1:key1 hash2:key2 hash3:key3 ... hash1000:key1000
# output: # of requests/s to run MGET on 1000 keys at once
# Multiple by 1000 and you know how many ops/s
```

In conclusion: `mget` is faster than `hmget` in pipeline by 8 times.

- Monitor Redis for slow-running commands, especially `EVALSHA`, `HGETALL`, `HMGET`, `MGET`. Lower `slowlog-log-slower-than` variable to capture more slow commands

```sh
> CONFIG SET slowlog-log-slower-than <THRESHOLD_MICROSECONDS>
```

- Avoid `KEYS` command at all cose, scanning entire keyspace is sure to be slow.

- Check the size of keys:

```sh
$ redis-cli --bigkeys
$ redis-cli --hotkeys
```

- Limit the size of range on commands such as `ZREVRANGE, ZRANGE, ZREVRANGEBYSCORE, ...`. Avoid unbound `LRANGE` calls and use a range.

- Delete huge keys usingg asynchronous command `UNLINK` rather than synchronous `DEL`.

- Always use connection pooling and pipelining for batch operatcion.

### Redis Cluster

<!-- TODO: learn more about best practices when using Redis Sharding https://redis.io/kb/doc/1mebipyp1e/performance-tuning-best-practices  -->

### Redis client - `redis-cli`

Redis is built with client-server architecture in mind. A Redis client can send commands, execute Lua scripts, and interact with the database through the Redis protocol.

```sh

redis-cli -h hostname -p port                 # Basic connection
redis-cli -h hostname -p port -a password     # Password authentication
docker run -it --network some-network --rm redis redis-cli -h some-redis  # container name

redis-cli --latency                           # execute 100 PING/s to Redis instance
redis-cli                                     # access interactive shell
```

There are 2 ways to read and write Redis settings:

- Write directly into `/usr/local/etc/redis/redis.conf` file
- Call `redis-cli` on Redis instance and run `CONFIG ...` commands

```ini
> PING                          # test connection, output: PONG
> AUTH <password>               # authenticate to server
> ECHO "message"                # echo a message
> SELECT [0-15]                 # switch current database
> QUIT                          # close connection



# client
> CLIENT LIST                   # show list of clients
> CLIENT GETNAME                # show current connection name
> CLIENT SETNAME <string>       # set current connection name

# show server configuration
> CONFIG GET *                  # get the value of ALL params
> CONFIG GET <param>            # get the value of param
> CONFIG SET <param> <value>    # change the value of param
> CONFIG SET requirepass <pass> # set password on Redis
> CONFIG REWRITE                # rewrite configuration file

# show server configurations
> INFO                          # get all server info
> INFO <section>                # get server info about a section
> DBSIZE                        # number of keys in current db
> DEBUG OBJECT <key>            # get debug info about a key
> FLUSHDB                       # remove all keys from current db
> FLUSHALL                      # remove all keys from all databases

> SAVE                          # backup, or sync save the dataset to disk
                                  # `dump.rdb` is saved in your Redis directory
                                  # e.g. `/data`
                                  # to restore data, move `dump.rdb` to `/data`
> BGSAVE                        # async save the dataset to disk
> BGREWRITEAOF                  # async rewrite append-only file
> LASTSAVE                      # show last successful save's UNIX timestamp
> SHUTDOWN SAVE                 # shutdown with save
> SHUTDOWN NOSAVE               # shutdown without save

> MONITOR                       # listen all requests received by the server
> SLOWLOG <subcommand> <args>   # manage Redis slow queries log
> TIME                          # 1. current UNIX timestamp
                                # 2. ms already elapsed in the current second

# ======================================================================= #
# Replication
# ======================================================================= #

# make server a slave of another instance
# or promotes it as a master (slave of itself)
> SLAVEOF <host> <port>

# what is the role of this instance, in the context of replication
> ROLE

# start replicating
> SYNC
```

## Data types

Redis return types:

- `OK`: the command run successfully.
- (integer) 0, 1, 2, ... N: the command has run successfully against N units
- Special output: `(nil)`, `-1`, ...

### Keys

A key is a string.

```ini
> INFO keyspace: show info about the current database (no. of keys in each db, ...)

# NOTE: use KEYS with extremely care in production environment
# NOTE: it can ruin performance when executed against large databases

> KEYS <regex>                  # show all keys matching the pattern
> KEYS *                        # show all keys inside the current db
> EXISTS <key>                  # check key existance
> DUMP <key>                    # return the serialized version of the key's value
> MOVE <key> <db>               # move the key to another db.
> DEL <key>                     # delete the key

> EXPIRE <key> <seconds>        # set expiry in seconds
> EXPIREAT <key> <seconds>      # same as EXPIRE, in Unix timestamp format.
> PERSIST <key>                 # removes the expiry
> TTL <key>                     # get remaining expiry time in seconds
> PTTL <key>                    # same as TTL, but in ms
```

### Strings

- map a string to another string.
- limit: max string value = 512MB

**Simple examples:**

```
> SET name "johndoe"
OK
> get name
"johndoe"
```

**Notable commands:**

```ini
> APPEND <key> <value>                    # new value = old value + <value>
> STRLEN <key>                            # get value length

> GET <key>                               # get value
> GETRANGE <key> <start> <end>            # get a substring from value
> GETSET <key> <newvalue>                 # set new value but return the old value

> SET <key> <value>                       # set value with no expiry
> SETEX <key> <value>                     # set value with expiry
> SETNX <key> <value>                     # set value only if key not existed before
> SETRANGE <key> <offset> <value>         # overwrite a part of the value, starting at the specified offset
> MSET <key1> <value1> [<key2> <value2> ...]    # set multiple key-value pairs
> MSETNX  <key1> <value1> [<key2> <value2> ...] # set multiple key-value pairs, only if none of the keys existed
> PSETEX <key> <value>                    # same as SETEX, but in ms

> INCR <key>                    # value + 1
> INCRBY <key> <inc>            # value + <inc>
> INCRBYFLOAT <key> <inc>       # same as INCRBY, but value in float

> DECR <key>                    # value - 1
> DECRBY <key> <dec>            # value - <dec>
```

### Hashes

- a list of string field-value pairs
- performance: most commands are O(1), listing-related and exp-related ones are O(n)
- limit: max field-value pairs = 2^32 - 1
- use cases: represent object

**Simple examples:**

```
> HMSET user:1 username johndoe password 123456
OK
> HGETALL user:1
1) "username"
2) "johndoe"
3) "password"
4) "123456z
```

**Notable commands:**

```ini
> HEXISTS <key> <field>                 # check if a hash field exists
> HLEN <key>                            # get the number of fields
> HINCRBY <key>                         # increment the integer value of a hash field

> HKEYS <key>                           # get all fields only
> HVALS <key>                           # get all values only

> HGET <key> <field>                    # get the value of a hash field
> HMGET <key> <field1> [<field2> ...]   # get the values of specified fields
> HGETALL <key>                         # get all the fields and values

> HSET <key> <field> <value>            # set string value of a field
> HSETNX <key> <field> <value>          # set string value of a field, only if field is non-existed
> HMSET <key> <field1> [<field2> ...]   # set new values for specified fields

> HDEL <key> <field1> [<field2> ...]    # delete one or more hash fields

# iterate fields and associated values inside a hash
# <key>: the name of the hash
# <cursor>: starting position (0 for first scan)
# <pattern>: glob-style pattern
# <count>: how many elements to return per iteration (it's a hint, not a strict limit, Redis may return less or more)
# This command is non-blocking and memory-efficient while scanning large hashes
# return: - next cursor (0 means scan completed), use it for the next scan
#         - array of field-value pairs
> HSCAN <key> <cursor> MATCH <pattern> COUNT <count>
```

### Lists

- a.k.a Linked List
- a list of strings, sorted by insertion order
- element can be added on the head or on the tail
- limit: max no. of elements = 2^32 - 1

**Examples:**

```ini

> lpush mocklist redis
(integer) 1
> lpush mocklist is
(integer) 2
> lpush mocklist awesome
(integer) 3
> lrange mocklist 0
(error) ERR wrong number of arguments for 'lrange' command
> lrange mocklist 0 10
1) "awesome"
2) "is"
3) "redis"
```

**Notable commands:**

```ini
> LLEN <key>                            # get length
> LINDEX <key> <index>                  # get an element based on its index
> LRANGE <key> <start> <stop>           # get a range of elements

> LPUSH <key> <value1> [<value2> ...]   # prepend 1 or more values
> LPUSHX <key> <value>                  # same as LPUSH, only if list existed
> RPUSH <key> <value1> [<value2> ...]   # append 1 or more values
> RPUSHX <key> <value>                  # same as RPUSH, only if list existed
> LINSERT <key> [BEFORE|AFTER] <pivot> <value>  # insert element before/after another element in a list

> LSET <key> <index> <value>            # set the value of an element by its index

> LPOP <key>                            # removes + gets the first element
> RPOP <key>                            # removes + gets the last element
> RPOPLPUSH <src> <dest>                # pop from <src>, push into <dest>
> LTRIM <key> <start> <stop>            # list contains only elements in the specified range

# <count>: how many elements to remove, and from which direction
# count > 0: remove <count> elements from the beginning (left/head) of the list
# count < 0: remove <count> elements from the end (right/tail) of the list
# count = 0: remove ALL occurance of the value from the list
# Output: no. of elements it removed
# Q: Why direction matters?
# A: Values can be duplicated in different positions.
> LREM <key> <count> <value>            # remove elements from a list
```

### Sets

- unordeded collection of strings
- add, remove, test set member existance in O(1)
- max no. of members in a set = 2^32 - 1

**Simple examples:**

```
> sadd mockset redis
(integer) 1
> sadd mockset mongodb
(integer) 1
> sadd mockset abcdefg
(integer) 1
> sadd mockset abcdefg
(integer) 0
> smembers mockset
1) "redis"
2) "mongodb"
3) "abcdefg" # there is only one element
```

**Notable examples:**

```ini
> SCARD <key>                           # get no. of members in a set
> SMEMBERS <key>                        # get all the members in a set

> SADD <key> <member1> [<member2> ...]  # add 1 or more members
> SREM <key> <member1> [<member2> ...]  # remove 1 or more members
> SPOP <key>                            # remove + return random member (random?)
> SDIFF <key1> [<key2> [<key3> ...]]    # subtracts multiple sets

> SMOVE <src> <dest> <member>           # move a member from one set to another

TODO: learn the rest commands
```

### Sorted Sets

- Similar to [Sets](#sets)
- Every member is associated with a score
- Using this score, non-numerical members can be sorted from smallest to grestest score.

**Simple examples:**

```
> zadd mocksortedset 0 redis
(integer) 1
> zadd mocksortedset 0 mongodb
(integer) 1
> zadd mocksortedset -1 abcdefg
(integer) 1
> zadd mocksortedset -1 abcdefg
(integer) 0
> zrangebyscore mocksortedset -10 10
1) "abcdefg"
2) "mongodb"
3) "redis"
> zadd mocksortedset 0 abcdefg
(integer) 0
> zrangebyscore mocksortedset 0 10
1) "mongodb"
2) "redis"
3) "abcdefg"
```

**Notable commands:**

```ini
TODO: learn the rest commands
```

### HyperLogLog

It's an algorithm using randomization in order to provide an approximation of the number of unique elements in a set using:

- a constant
- a small amount of memory.

- Limit: 2^64 items.

**Simple examples:**

```ini
> PFADD hyperloglog "redis"
(integer) 1
> PFADD hyperloglog "mongodb"
(integer) 1
> PFADD hyperloglog "mysql"
(integer) 1
> PFCOUNT hyperloglog
(integer) 3
> PFADD hyperloglog "sqlite"
(integer) 1
> PFADD hyperloglog "neo4j"
(integer) 1
> PFCOUNT hyperloglog
(integer) 5
```

**Notable commands:**

```ini
> PFADD <key> <element1> [<element2> ...]     # add element to the special set
> PFCOUNT <key>                               # count the element inside the special set
```

> **NOTE:** if you run `TYPE <key>`, it returns "string"

## Pub/Sub

- A message system where the senders (a.k.a publishers) sends the messages while the receivers (a.k.a subscribers) receive them.
- The link by which the messages are transferred is called channel.

**Simple examples:**

```
# Server
> SUBSCRIBE mockchannel
1) "subscribe"
2) "mockchannel"
3) (integer) 1
1) "message"
2) "mockchannel"
3) "Hello World from mockchannel"

# Client
> PUBLISH mockchannel "Hello World from mockchannel"
```

**Notable commands:**

```ini
# show the state of Pub/Sub system
> PUBSUB <subcommand> [<argument1> [<argument2> ...]]
> PUBSUB CHANNELS     # show list of subscribed channels

# listen for messages on channels
> SUBSCRIBE <channel1> [<channel2> ...]
# same as SUBSCRIBE, but instead of exact match, use regex pattern
> PSUBSCRIBE <regex1> [<regex2> ...]

# post a message to a channel
> PUBLISH <channel> <message>

# remove subscribed channels by specifying exact match
> UNSUBSCRIBE <channel1> [<channel2> ...]
# remove subscribed channels by regex
# NOTE: not PUN-SUBSCRIBE, it's P-UNSUBSCRIBE
> PUNSUBSCRIBE <pattern1> [<pattern2> ...]
```

## Transaction

MySQL has their ACID and Redis has their `MULTI/EXEC` commands.

- All commands from one client are sequentially executed and served by the server. It's not possible for this server to serve requests from another client.
- Transaction is atomic: if one fails, they're all failed.

```ini
> MULTI                     # OK
127.0.0.1:6379(TX)> SET tutorial redis    # QUEUED
127.0.0.1:6379(TX)> GET tutorial          # QUEUED
127.0.0.1:6379(TX)> SET visitors 2000     # QUEUED
127.0.0.1:6379(TX)> INCR visitors         # QUEUED
127.0.0.1:6379(TX)> EXEC
# 1) OK
# 2) "redis"
# 3) OK
# 4) (integer) 2001
> # NOTE: no more (TX)
```

However, while thse commands are being queued, other commands from other clients can come in and change the value of the keys that are part of the transaction.

=> Enter `watch`: monitor a list of keys for any modifications before the execution of `exec`. If by the time `exec` is called and at least 1 of the monitored keys is changed, the `exec` will return an error. Engineer can either run code to retry the transaction, or generate an error (depends on the use case).

```ini
> WATCH tutorial visitors   # OK
> MULTI                     # OK
127.0.0.1:6379(TX)> SET tutorial redis    # QUEUED
127.0.0.1:6379(TX)> GET tutorial          # QUEUED
# in the meantime, some client run `SET tutorial "memcached"`
127.0.0.1:6379(TX)> SET visitors 2000     # QUEUED
127.0.0.1:6379(TX)> INCR visitors         # QUEUED
127.0.0.1:6379(TX)> EXEC                  # nil
> # NOTE: no more (TX)
```

**Notable commands:**

```
> WATCH <k1> [<k2> ...]   # watch the keys to determine the
                                        # exec of MULTI/EXEC block
> MULTI                   # mark the start of transaction block
> UNWATCH                 # forget the watched keys
> DISCARD                 # discard all commands after MULTI
> EXEC                    # execute all commands after MULTI
```

**NOTE:** outside of interactive shell there's a the concept of pipelining and how `watch` change its behavior. Read more in my `redis-py` cheatsheet.

## Scripting

Add programmatical operations with Lua interpreter using `EVAL` command.

- Lua script gives you performance gain
- Lua script is transactional and is executed atomically. No other Redis command can run while a script is executing.
- Lua indexed tables start with 1, not 0.
- Best practice: provide keys which the script uses as `KEYS[]`, and all other arguments as `ARGV[]` instead of specifying `KEYS` as 0 then provide all keys within `ARGV[]` table.

**Examples:**

```
# Redis use no. of keys to create boundary between keys and args

> EVAL "return {KEYS[1], KEYS[2], ARGV[1], ARGV[2]}" 2 key1 key2 arg1 arg2
1) "key1"
2) "key2"
3) "arg1"
4) "arg2"

> SCRIPT LOAD "return {KEYS[1], KEYS[2], ARGV[1], ARGV[2]}"
"c0d2d6f81be75d67523d7c8ac69a932fbe1aa4e2"

> EVALSHA "c0d2d6f81be75d67523d7c8ac69a932fbe1aa4e2" 2 key1 key2 arg1 arg2
1) "key1"
2) "key2"
3) "arg1"
4) "arg2"
```

**Notable commands:**

```ini
> EVAL <script> <no-of-keys> <key1> [<key2> ...] <arg1> [<arg2> ...]    # execute Lua script
> EVALSHA <script> <no-of-keys> <key1> [<key2> ...] <arg1> [<arg2> ...]    # execute Lua script using SHA1, loaded into cache first
> SCRIPT LOAD <script>      # load the script into script cache
> SCRIPT EXISTS <sha1>      # check script existance
> SCRIPT FLUSH              # clear script cache
> SCRIPT KILL               # kill the script currently executed
```

## FAQs

### How to connect to Redis via UNIX Socket on Amazon Linux 2023 ?

```sh
# Cre: https://serverfault.com/a/1129059/1138548
sudo dnf install -y redis6
sudo systemctl start redis6
sudo systemctl enable redis6
sudo systemctl is-enabled redis6
redis6-server --version
redis6-cli ping

# NOTE: add the user who is owner of the application process (e.g. Nginx) to the
# NOTE: same group as `redis6`
sudo usermod -a -G redis6 ec2-user
```

```conf
# /etc/redis6/redis6.conf

# do not listen on a port
bind 0

# NOTE: redis.sock can only be created inside dicectory which allow `redis6` user
# NOTE: to write on. In this case, only `/tmp`
unixsocket /tmp/redis.sock

# set permissions for the socket
# NOTE: socket can be read/write by `redis6` user and group
unixsocketperm 770
```

### How to install Redis inside Docker?

```sh
# NOTE: Redis image drop privileges and switch to `redis` user by default
# NOTE: Don't specify --user="1000:1000" option
# NOTE: Ignore `vm.overcommit_memory` warning if persistence is disabled
# TODO: test if files and directories host permission need to be changed to 524xxx
docker run \
  --name=some-redis \
  --init \
  --rm \
  --read-only \
  --network=none \
  -v "./redis.conf:/usr/local/etc/redis/redis.conf:ro" \
  # persistence volume
  -v "./data:/data" \
  # :latest for Debian-based image
  redis:alpine \
  # persistance strategy
  # if at least 1 write operation was performed
  # save ONE snapshot of the database every 60 seconds
  # more log => stricter log level is better
  redis-server --save 60 1 --loglevel warning

# Connect via redis-cli
docker exec -it some-redis redis-cli

# Run the following command if persistence is enabled
echo "vm.overcommit_memory=1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
cat /proc/sys/vm/overcommit_memory
```
