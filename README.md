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
* Average Memory Usage
* Open File Descriptors
* Active Sessions
* Transactions
* Update data
* Fetch data (SELECT)
* Insert data
* Lock tables
* Return data
* Idle sessions
* Delete data
* Cache Hit Rate
* Buffers (bgwriter)
* Conflicts/Deadlocks
* Temp File (Bytes)
* Checkpoint Stats

### Redis
* Commands per second
* Command latency per second
* Hit ratio per instance
* Total Memory Usage
* Memory fragmentation ratio per instance
* Key evictions per second per instance
* Connected/Blocked Clients
* Total Items per DB
* Expiring vs Not-Expiring Keys
* Connected slaves by instance
* Time since last master connection

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
