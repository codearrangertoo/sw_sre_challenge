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
