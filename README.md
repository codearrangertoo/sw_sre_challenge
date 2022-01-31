# sw_sre_challenge



## Setup
1. Start the postgres server
   - ```docker-compose up -d postgres```
2. Initialize the database
   - ```docker-compose run --rm rails bundle exec rails db:chatwoot_prepare```
3. Start all of the containers
   - ```docker-compose up```

## Usage
- Login to chatwoot http://localhost:8080
  - Create a new account
- Login to Grafana http://localhost:3000
  - login: admin
  - password: admin

## Metrics
- http://localhost:3000/
### Postgres
* Average CPU Usage
  - CPU time in seconds
* Average Memory Usage
  - Resident memory usage in bytes
  - Virtual memory usage in bytes
* Open File Descriptors
  - Count of open file descriptors
* Active Sessions
  - Count of active Postgres client sessions
* Transactions
  - Count of commits and rollbacks
* Update data
  - Number of rows updated by queries per database
* Fetch data (SELECT)
  - Number of rows fetched by queries per database
* Insert data
  - Number of rows inserted by queries per database
* Lock tables
  - Number of locks (by lock type) per database
* Return data
  - Number of rows returned by queries per database
* Idle sessions
  - Count of idle client connections
* Delete data
  - Number of rows deleted by queries per database
* Cache Hit Rate
  - Percentage of times disk blocks were found already in the buffer cache, so that a read was not necessary (this only includes hits in the PostgreSQL buffer cache, not the operating system's file system cache)
* Buffers (bgwriter)
  - buffers_backend - Number of buffers written directly by a backend
  - buffers_alloc - Number of buffers allocated
  - backend_fsync - Number of times a backend had to execute its own fsync call (normally the background writer handles those even when the backend does its own write)
  - buffers_checkpoint - Number of buffers written during checkpoints
  - buffers_clean - Number of buffers written by the background writer
* Conflicts/Deadlocks
  - conflicts - Number of queries canceled due to conflicts with recovery in this database. (Conflicts occur only on standby servers; see pg_stat_database_conflicts for details.)
  - deadlocks - Number of deadlocks detected in this database
* Temp File (Bytes)
  - Total amount of data written to temporary files by queries in this database. All temporary files are counted, regardless of why the temporary file was created, and regardless of the log_temp_files setting.
* Checkpoint Stats
  - write_time - Total amount of time that has been spent in the portion of checkpoint processing where files are written to disk, in milliseconds
  - sync_time - Total amount of time that has been spent in the portion of checkpoint processing where files are synchronized to disk, in milliseconds

### Redis
* Commands per second
  - Total number of calls per command
* Command latency per second
  - Average amount of time in seconds spent per command
* Hit ratio per instance
  - Average hit ratio per instance
* Total Memory Usage
  - used_bytes - Total used memory
  - max_bytes - Configured max memory
  - rss_bytes - Used RSS memory
* Memory fragmentation ratio per instance
  - Ratio of memory used as seen by the operating system ( used_memory_rss ) to memory allocated by Redis ( used_memory )
* Key evictions per second per instance
  - Rate of keys per second that have been evicted
* Connected/Blocked Clients
  - connected_clients - Because access to Redis is usually mediated by an application (users do not generally directly access the database), for most uses, there will be reasonable upper and lower bounds for the number of connected clients. If the number leaves the normal range, this could indicate a problem. If it is too low, upstream connections may have been lost, and if it is too high, the large number of concurrent client connections could overwhelm your server’s ability to handle requests. Regardless, the maximum number of client connections is always a limited resource—whether by operating system, Redis’s configuration, or network limitations. Monitoring client connections helps you ensure you have enough free resources available for new clients or an administrative session.
  - blocked_clients - Redis offers a number of blocking commands which operate on lists. BLPOP, BRPOP, and BRPOPLPUSH are blocking variants of the commands LPOP, RPOP, and RPOPLPUSH, respectively. When the source list is non-empty, the commands perform as expected. However, when the source list is empty, the blocking commands will wait until the source is filled, or a timeout is reached. An increase in the number of blocked clients waiting on data could be a sign of trouble. Latency or other issues could be preventing the source list from being filled. Although a blocked client in itself is not cause for alarm, if you are seeing a consistently nonzero value for this metric you should investigate.

## Alerting
- Alert rules http://localhost:3000/alerting/list
### Postgres
* PostgresNotResponding 
  - Critial alert if Postgres service is not responding
* PostgresHighConnectionCount
  - Warn when connection count is above 70% of configured max_connections
  - Critical when connection count is above 90% of configured max_connections
* PostgreSQLSlowQueries
  - Warn when there are a high number of slow queries. (Running longer than 2 minutes)
* PostgreSQLQPS
  - Warn when there are a high number of queries per second (Greater than 10000 in a 5 minute window)
* PostgreSQLCacheHitRatio
  - Warn on low cache hit rate (Less than 98%)
* PostgresqlExporterError
  - Critial alert when posgres_exporter is unable to collect metrics from Postgres
* PostgresqlTableNotVacuumed
  - Warn when a table has not been vacuumed for 24 hours
### Redis
* RedisDown
  - Critial alert when Redis is down
* RedisOutOfMemory
  - Warn when redis is out of memory (> 90% utilization)
* RedisTooManyConnections
  - Warn when redis has too many connections (> 100 connections)
* RedisNotEnoughConnections
  - Warn when redis has too few connections (< 5 connections)
* RedisRejectedConnections
  - Critical alert when redis has rejected any connections
### All services
* monitor_service_down
  - Critical alert when any of the services are down


## Key Metrics
### Postgres
* Update/Insert/Delete Data
  - Monitoring the number of rows inserted, updated, and deleted can help give you an idea of what types of write queries your database is serving. If you see a high rate of updated and deleted rows, you should also keep a close eye on the number of dead rows, since an increase in dead rows indicates a problem with VACUUM processes, which can slow down your queries.
  - A sudden drop in throughput is concerning and could be due to issues like locks on tables and/or rows that need to be accessed in order to make updates. Monitoring write activity along with other database metrics like locks can help you pinpoint the potential source of the throughput issue.


Ref: https://www.datadoghq.com/blog/postgresql-monitoring/#key-metrics-for-postgresql-monitoring

### Redis
* Commands per second
  - Tracking the throughput of commands processed is critical for diagnosing causes of high latency in your Redis instance. High latency can be caused by a number of issues, from a backlogged command queue, to slow commands, to network link overutilization. You could investigate by measuring the number of commands processed per second—if it remains nearly constant, the cause is not a computationally intensive command. If one or more slow commands are causing the latency issues you would see your number of commands per second drop or stall completely.
  - A drop in the number of commands processed per second as compared to historical norms could be a sign of either low command volume or slow commands blocking the system. Low command volume could be normal, or it could be indicative of problems upstream.
* Hit ratio per instance
  - When using Redis as a cache, monitoring the cache hit rate can tell you if your cache is being used effectively or not. A low hit rate means that clients are looking for keys that no longer exist.
* Memory fragmentation ratio
  - The mem_fragmentation_ratio metric gives the ratio of memory used as seen by the operating system (used_memory_rss) to memory allocated by Redis (used_memory).
* Total Memory Usage
  - Memory usage is a critical component of Redis performance. If used_memory exceeds the total available system memory, the operating system will begin swapping old/unused sections of memory. Every swapped section is written to disk, severely affecting performance. Writing or reading from disk is up to 5 orders of magnitude (100,000x!) slower than writing or reading from memory (0.1 µs for memory vs. 10 ms for disk).
  - You can configure Redis to remain confined to a specified amount of memory. Setting the maxmemory directive in the redis.conf file gives you direct control over Redis’s memory usage. Enabling maxmemory requires you to configure an eviction policy for Redis to determine how it should free up memory.

Ref: https://www.datadoghq.com/blog/how-to-monitor-redis-performance-metrics/#key-redis-metrics
