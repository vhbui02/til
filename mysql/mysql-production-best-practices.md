# MySQL Production Best Practices

## MySQL server configurations

> [!IMPORTANT]
>
> All values mentioned here are for reference; the optimal values depend on your application logic, technology stack, hardware resources, etc. You must design comprehensive testing strategies, including load and stress testing, to determine the optimal values.

At `/etc/mysql/conf.d/custom.cnf`:

```cnf
[mysqld]
wait_timeout = 300
interactive_timeout = 36000
net_read_timeout = 20
net_write_timeout = 30

mysqlx_wait_timeout = 300
mysqlx_interactive_timeout = 36000
mysqlx_read_timeout = 20
mysqlx_write_timeout = 30

# ignore if you don't use SSL
ssl_session_cache_timeout = 300

# monitor thread_created value to determine the optimal number
thread_cache_size = 32

# def=151
max_connections = 200

# enable event schedular to create a cron-like behavior in MySQL server
event_scheduler = ON

# allocate memory for InnoDB storage engine to cache recently accessed data and indexes in RAM, reduce disk IO
# def=128M, recommended: min(50-75% RAM, data size)
innodb_buffer_pool_size = 2147483648

# for best efficiency, each buffer pool instance is at least 1GB
# def=min(innodb_buffer_pool_size/innodb_buffer_pool_chunk_size, nproc/4)
innodb_buffer_pool_instances = 2

# you need to size the redo logs to hold approximately an hour's worth of logs
# def=48MB, recommended: 128MB -> 1GB
innodb_log_file_size = 150994944

# 0=for dev database store mock data only
# (def, slow) 1=bank, use with `sync_binlog=1`, slower than
# (faster) 2=blog/stats/e-commerse, a successfuly `commit` could be lost in case of OS crash/power outage can erase the last second of transactions.
innodb_flush_log_at_trx_commit = 1
sync_binlog = 1

# flush method to InnoDB data files and log files
# affect I/O throughput
# fsync/0 = use fsync() syscall
# 0_DSYNC/1 = use 0_SYNC to open and flush log files, fsync() to flush data file. Too many problems and not recommended on UNIX
# littlesync/2 = internal performance testing. not for production
# nosync/3 = internal performance testing. not for production
# O_DIRECT/4 = use directio() to open data files, fsync() to flush both data + log files. Recommended.
innodb_flush_method=O_DIRECT

# seperate tablespaces per table, beneficial for certain workloads
# def=ON
innodb_file_per_table=ON

# prevent frequent updates to InnoDB statistics and improve read speeds
# def=OFF
innodb_stats_on_metadata=OFF
```

MySQL CLI commands:

```sql
-- ========================================================================== --
-- Inspect
-- ========================================================================== --

-- show the number of existing database connections
-- some are actively used, others might be lingering from an unclosed pool
SHOW PROCESSLIST;

-- show the list of status variables that provide information about its operation
SHOW GLOBAL STATUS;

-- list sleeping connections older than 300s
SELECT id,user,host,time,info FROM INFORMATION_SCHEMA.PROCESSLIST WHERE command='Sleep' AND time>300;

-- ========================================================================== --
-- Configuration
-- CAUTION: run the following commands as root
-- ========================================================================== --

-- NOTE; the "GLOBAL" keyword
-- NOTE: if not specified, the settings will be applied to the current session/connection
SHOW GLOBAL VARIABLES LIKE '%timeout%';

SET GLOBAL wait_timeout = 300; -- def=28800
-- SET GLOBAL interactive_timeout = 3600; -- def=28800 (8 hours)
-- SET GLOBAL ssl_session_cache_timeout = 900; -- def=300
SET GLOBAL net_read_timeout = 20;     -- def=30
SET GLOBAL net_write_timeout = 30;    -- def=60
SET GLOBAL mysqlx_wait_timeout = 300; -- def=28800
SET GLOBAL mysqlx_read_timeout = 20;  -- def=30
SET GLOBAL mysqlx_write_timeout = 30; -- def=60
-- SET GLOBAL mysqlx_interactive_timeout = 3600; -- def=28800

-- SET GLOBAL replica_net_timeout = ...
-- SET GLOBAL slave_net_timeout = ...

SET GLOBAL thread_cache_size = 16;    -- def=9
SET GLOBAL max_connections = 200;     -- def=151

-- enable Event Scheduler immediately (requires SUPER or SYSTEM_VARIABLES_ADMIN)
SET GLOBAL event_scheduler = ON;

-- Kill long-running SLEEP connections automatically.
-- Adjust the threshold and schedule to suit your workload.
SET @SLEEP_THRESHOLD_SECONDS = 300; -- 5 minutes

DELIMITER $$
CREATE EVENT IF NOT EXISTS kill_old_sleep_connections
ON SCHEDULE EVERY 1 MINUTE
COMMENT 'Kill client connections in Sleep state older than threshold'
DO
BEGIN
  DECLARE done INT DEFAULT 0;
  DECLARE cur_id BIGINT;

  DECLARE cur CURSOR FOR
    SELECT ID
    FROM INFORMATION_SCHEMA.PROCESSLIST
    WHERE COMMAND = 'Sleep'
      AND TIME > @SLEEP_THRESHOLD_SECONDS
      AND ID <> CONNECTION_ID(); -- avoid killing the session running the event

  DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

  OPEN cur;
  fetch_loop: LOOP
    FETCH cur INTO cur_id;
    IF done THEN
      LEAVE fetch_loop;
    END IF;
    SET @kill_stmt = CONCAT('KILL ', cur_id);
    PREPARE stmt FROM @kill_stmt;
    EXECUTE stmt;
    DEALLOCATE PREPARE stmt;
  END LOOP;
  CLOSE cur;
END$$
DELIMITER ;
```

Client settings're also important, the common rule of thumb is to set **client's timeout < server's timeout** to prevent client reuse something that has been terminated at server, which can results in critical application failure

## OS-level TCP keepalive settings

Lowering the TCP keepalive can help the kernel detech broken client connections and notify MySQL

```sh
# temporary
# by default, a connection must remain idle in 7200 seconds before the first keepalive probe is sent
# keep this short will make OS check the dead connection ASAP
sudo sysctl -w net.ipv4.tcp_keepalive_time=120

# subsequent keepalive probes are sent between an amount of time
# keep this short will make the OS check more regularly
sudo sysctl -w net.ipv4.tcp_keepalive_intvl=15

# the maximum number of probes to be sent before the connection is considered dead and closed
sudo sysctl -w net.ipv4.tcp_keepalive_probes=5

# persistent
# or
cat <<EOF | sudo tee /etc/sysctl.d/99-mysql-keepalive.conf
net.ipv4.tcp_keepalive_time = 120
net.ipv4.tcp_keepalive_intvl = 15
net.ipv4.tcp_keepalive_probes = 5
EOF
sudo sysctl --system
```

## Advanced features

- Choose isolation level. Maybe you don't need `REPEATABLE READ` and you just need `READ COMMITED`?

- Create Updateable View to enforce database-level security. Use this in case you don't trust application code to leave restricted columns/rows.

- Create Materialized View to store the result of complex, long-running query.

- Create trigger. People always forget this amazing feature. DO NOT execute query from application that can be automated via trigger.

- Create stored procedure for reusable complex queries/encapsulate business logic and reduce RTT. Remember: it doesn't return value, that's for function. Always remember: function can return value in the middle of the query.
