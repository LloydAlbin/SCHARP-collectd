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

## Postgres 9.5-

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

The use of stat_tables and planning_time requires editing these sections to your needs.

#### stat_tables

You only need this is you want this information for a specific table, otherwise you can use the following queries out of the postgresql_default.conf file. The only things not covered by the default is the autovacuum and analyze portions of the stat_tables.
* queries_by_table
* query_plans_by_table
* table_states_by_table

You need to edit the query and add the table names, in two places, that you wish to capature the stats, for both the main table and the toast table.

```apache
  <Query stat_tables>
    Statement "SELECT 
	b.schemaname || '_' || b.relname AS table, 
    b.seq_scan,
    b.seq_tup_read,
    b.idx_scan,
    b.idx_tup_fetch,
    b.n_tup_ins,
    b.n_tup_upd,
    b.n_tup_del,
    b.n_tup_hot_upd,
    b.n_live_tup,
    b.n_dead_tup,
    b.n_mod_since_analyze,
    b.vacuum_count,
    b.autovacuum_count,
    b.analyze_count,
    b.autoanalyze_count
FROM pg_stat_all_tables b
WHERE relname IN ('table_name')
UNION ALL
SELECT 
	b.schemaname || '_pg_toast_' || a.relname AS table, 
    b.seq_scan,
    b.seq_tup_read,
    b.idx_scan,
    b.idx_tup_fetch,
    b.n_tup_ins,
    b.n_tup_upd,
    b.n_tup_del,
    b.n_tup_hot_upd,
    b.n_live_tup,
    b.n_dead_tup,
    b.n_mod_since_analyze,
    b.vacuum_count,
    b.autovacuum_count,
    b.analyze_count,
    b.autoanalyze_count
FROM pg_class a 
LEFT JOIN pg_stat_all_tables b ON b.relname = 'pg_toast_' || a.oid
WHERE a.relname IN ('table_name');"
    <Result>
      Type pg_stat_tables
      InstancesFrom "table"
      ValuesFrom seq_scan,seq_tup_read,idx_scan,idx_tup_fetch,n_tup_ins,n_tup_upd,n_tup_del,n_tup_hot_upd,n_live_tup,n_dead_tup,n_mod_since_analyze,vacuum_count,autovacuum_count,analyze_count, autoanalyze_count
    </Result>
  </Query>
```

Note: I need to update this query for you to be able to specify schema. The current queries limited by the fact that all tables with the same name will be reported back no matter which schema they are found.

#### planning_time

In this example I am testing the planning time of quering pg_roles. This is sendign an array of arrays to the planning_time functions, this means that you can test several queries at once within the same transaction block / data snapshot.

```apache
  <Query planning_time>
    Statement "SELECT * FROM ds.planning_time(ARRAY[
ARRAY['pg_roles', E'SELECT * FROM pg_roles']
]);"
    <Result>
      Type pg_planning_time
      InstancesFrom "query"
      ValuesFrom planning_time
    </Result>
  </Query>
```

## Postgres 9.6+

### Stats for Standalone Server

```apache
  <Database postgres>
    Instance "db-a"
    Host "db-a"
    Port "5432"
    User "postgres"
    Query connections
    Query connection_states_96
    Query connection_state_96
    Query database_size
    Query buffercache
    Query buffercache_databases
    Query database_transactions
    Query transactions
    Query concurrent_txns_96
    Query pg_hit_ratio
    Query pg_xlog_96
    Query connection_state_by_database_96
    Query database_commit_ratio_by_database
    Query database_stats_by_database
    Query query_length
    Query query_length_server
    Query wait_length_96
    Query wait_length_server_96
    Query transaction_length
    Query transaction_length_server
    Query autovacuums_96
    Query autovacuums_server_96
    Query transaction_length_10
    Query autovacuum_length_server_10
  </Database>
```

### Stats for Primary Server

Use the Stats for Standalone Server and add the following lines

```apache
    Query a_server_location_96
```

### Stats for Seconary Server

Use the Stats for Standalone Server and add the following lines

```apache
    Query b_server_location_96
    Query pg_conflicts
```

My recomendation if you are running both a primary and secondary server is to add the stats for primary and secondary to the both servers for when you swap the two servers.

NOTE: The planning_time requires the planning time function from my [SCHARP-PG-DBA-Tools](https://github.com/LloydAlbin/SCHARP-PG-DBA-Tools) toolset.
