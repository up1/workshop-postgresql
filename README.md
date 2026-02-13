# Install PostgreSQL database

## 1. Start
```
$docker compose up -d db
$docker compose ps
```

## 2. Access to PostgreSQL database with PgAdmin
* Host: localhost
* Port: 5432
* POSTGRES_USER: monitoring
* POSTGRES_PASSWORD: monitorpass
* POSTGRES_DB: postgres

## 3. Start workshop

Reset statistics
```
SELECT pg_stat_reset();
```

Query statistics
```
SELECT * FROM pg_stat_database;
```

Query indexes never used
```
SELECT * FROM pg_stat_user_indexes WHERE idx_scan = 0;
```

```
SELECT s.schemaname,
       s.relname AS table_name,
       s.indexrelname AS index_name,
       s.idx_scan AS times_used,
       pg_size_pretty(pg_relation_size(t.relid)) AS table_size,
       pg_size_pretty(pg_relation_size(s.indexrelid)) AS index_size,
       idx.indexdef AS index_ddl
  FROM pg_stat_user_indexes s
  JOIN pg_stat_user_tables t 
    ON s.relname = t.relname
  JOIN pg_index i
    ON s.indexrelid = i.indexrelid
  JOIN pg_indexes AS idx 
    ON s.indexrelname = idx.indexname
   AND s.schemaname = idx.schemaname
 WHERE s.idx_scan = 0 -- no scans
   AND 0 <> ALL(i.indkey) -- 0 in the array means this is an expression index
   AND NOT i.indisunique -- no unique index
 ORDER BY pg_relation_size(s.indexrelid) DESC;
```
