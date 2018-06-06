# SCHARP-collectd
This is a set of updates for collectd to collect data from Postgres.

The file | Description
-------- | -----------
etc/conf.d/postgresql.conf | SCHARP Postgres Configuration
collectd/types.postgres.db | SCHARP Postgres Types
etc/collectd.conf | Collectd Configuration (Edited)
collectd/postgresql_default.conf | Default Postgres Configuration (Default)
collectd/types.db | Default Types including Postgres (Default)

To install this, just copy the first two files into your collectd installation. The third file is already part of your collectd installation and just needs to be edited to a TypedDB line for the types.postgres.db file and a Include line for the postgresql.conf file as shown in the example collectd.conf file. The last two files are already part of your collectd installation.

You will want to edit the end of the postgresql.conf file to add a Database section for each of your servers and sometimes spevialized ones for specific databases.

## Postgres 9.4-

### Stats for Standalone Server

```apache
  <Database postgres>
    Instance "db-a"
    Host "db-a"
    Port "5432"
    User "postgres"
    Query connections
    Query connection_states
    Query connection_state
    Query database_size
    Query buffercache
    Query buffercache_databases
    Query database_transactions
    Query transactions
    Query concurrent_txns
    Query pg_hit_ratio
    Query pg_xlog
    Query connection_state_by_database
    Query database_commit_ratio_by_database
    Query database_stats_by_database
    Query query_length
    Query query_length_server
    Query wait_length
    Query wait_length_server
    Query transaction_length
    Query transaction_length_server
  </Database>
```

### Stats for Primary Server

Use the Stats for Standalone Server and add the following lines

```apache
    Query a_server_location
```

### Stats for Seconary Server

Use the Stats for Standalone Server and add the following lines

```apache
    Query b_server_location
    Query pg_conflicts
```

### Stats for a specific database

```apache
#Stats for a specific database
  <Database main>
    Instance "main"
    Host "db-a"
    Port "5432"
    User "postgres"
    Query pg_trig_disabled
  </Database>
```

### Stats for a specific database, every 300 seconds aka 5 minutes

```apache
  <Database main>
    Instance "main"
    Host "sqltest"
    Port "5432"
    User "postgres"
    Query stat_tables
    Query planning_time
    Interval 300
  </Database>
```

## Postgres 10+

### Stats for Standalone Server

```apache
  <Database postgres>
    Instance "db-a"
    Host "db-a"
    Port "5432"
    User "postgres"
    Query connections
    Query connection_states_10
    Query connection_state_10
    Query database_size
    Query buffercache
    Query buffercache_databases
    Query database_transactions
    Query transactions
    Query concurrent_txns_10
    Query pg_hit_ratio
    Query pg_xlog_10
    Query connection_state_by_database_10
    Query database_commit_ratio_by_database
    Query database_stats_by_database
    Query query_length
    Query query_length_server
    Query wait_length_10
    Query wait_length_server_10
    Query transaction_length
    Query transaction_length_server
  </Database>
```

### Stats for Primary Server

Use the Stats for Standalone Server and add the following lines

```apache
    Query a_server_location_10
```

### Stats for Seconary Server

Use the Stats for Standalone Server and add the following lines

```apache
    Query b_server_location_10
    Query pg_conflicts
```

My recomendation if you are running both a primary and secondary server is to add the stats for primary and secondary to the both servers for when you swap the two servers.

NOTE: The planning_time requires the planning time function from my [SCHARP-PG-DBA-Tools](https://github.com/LloydAlbin/SCHARP-PG-DBA-Tools) toolset.
