# Update on local table for ReplicatedMergeTree Table Engine

<!-- toc -->

- [Create Local and Distributed Table](#create-local-and-distributed-table)
- [Insert Data](#insert-data)
- [❌Update on Distributed Table Engine](#%E2%9D%8Cupdate-on-distributed-table-engine)
- [✅Update on Local Table Engine](#%E2%9C%85update-on-local-table-engine)
- [Refs](#refs)

<!-- tocstop -->

# Create Local and Distributed Table

```sql
-- Create Local Table
CREATE TABLE t.t_local ON CLUSTER `{cluster}` (
    timestamp DateTime Codec(DoubleDelta, LZ4),
    log String,
    path String
)
ENGINE = ReplicatedMergeTree(
  '/clickhouse/{installation}/{cluster}/tables/{shard}/{database}/{table}', '{replica}'
)
PARTITION BY toYYYYMM(timestamp)
ORDER BY (timestamp, path)

-- Create Distributed Table
CREATE TABLE t.t_all ON CLUSTER `{cluster}`
ENGINE = Distributed('{cluster}', t, t_local, rand())
```

# Insert Data

Insert data through local table:

NOTE: Run each sql line in `clickhouse-client` prompt, otherwise only one record is inserted.

```sql
INSERT INTO t.t_local (timestamp, log, path) VALUES ('2023-10-17 09:00:00', 'First log entry', '/var/logs/file1.log');
INSERT INTO t.t_local (timestamp, log, path) VALUES ('2023-10-17 10:30:00', 'Second log entry', '/var/logs/file2.log');
INSERT INTO t.t_local (timestamp, log, path) VALUES ('2023-10-17 12:15:00', 'Third log entry', '/var/logs/file3.log');
```

Insert data through distributed table:

```sql
INSERT INTO t.t_all (timestamp, log, path) VALUES ('2023-10-18 09:00:00', 'First log entry', '/var/logs/file1.log');
INSERT INTO t.t_all (timestamp, log, path) VALUES ('2023-10-18 10:30:00', 'Second log entry', '/var/logs/file2.log');
INSERT INTO t.t_all (timestamp, log, path) VALUES ('2023-10-18 12:15:00', 'Third log entry', '/var/logs/file3.log');
```

# ❌Update on Distributed Table Engine

```sql
ALTER TABLE t.t_all UPDATE log = 'Updated log entry <-'
WHERE timestamp = '2023-10-17 09:00:00' AND path = '/var/logs/file1.log';
```

Error:

```sql
ALTER TABLE t.t_all
    UPDATE log = 'Updated log entry <-' WHERE (timestamp = '2023-10-17 09:00:00') AND (path = '/var/logs/file1.log')

Query id: fcd17813-087c-496b-ba2e-0e5c7fbff6ea


0 rows in set. Elapsed: 0.019 sec.

Received exception from server (version 23.3.8):
Code: 48. DB::Exception: Received from localhost:9000. DB::Exception: Table engine Distributed doesn't support mutations. (NOT_IMPLEMENTED)

```

# ✅Update on Local Table Engine

Let's update data through local table(non-distributed table):

```sql
ALTER TABLE t.t_local ON CLUSTER `{cluster}` UPDATE log = 'Updated log entry <-'
WHERE timestamp = '2023-10-17 09:00:00' AND path = '/var/logs/file1.log';
```

Results:

```sql
ALTER TABLE t.t_local ON CLUSTER `{cluster}`
    UPDATE log = 'Updated log entry <-' WHERE (timestamp = '2023-10-17 09:00:00') AND (path = '/var/logs/file1.log')

Query id: 0a119cbf-d244-4d3c-be12-be7201d25d6c

┌─host──────────────┬─port─┬─status─┬─error─┬─num_hosts_remaining─┬─num_hosts_active─┐
│ chi-logs-logs-1-1 │ 9000 │      0 │       │                   3 │                0 │
│ chi-logs-logs-0-0 │ 9000 │      0 │       │                   2 │                0 │
│ chi-logs-logs-0-1 │ 9000 │      0 │       │                   1 │                0 │
│ chi-logs-logs-1-0 │ 9000 │      0 │       │                   0 │                0 │
└───────────────────┴──────┴────────┴───────┴─────────────────────┴──────────────────┘

4 rows in set. Elapsed: 0.116 sec.
```

Check if data is updated:

```sql
SELECT *
FROM t_all
GROUP BY (timestamp, log, path)

Query id: 1cc2811e-cdb1-4fc2-b94e-6d9d4320ede8

┌───────────timestamp─┬─log──────────────────┬─path────────────────┐
│ 2023-10-17 10:30:00 │ Second log entry     │ /var/logs/file2.log │
│ 2023-10-17 12:15:00 │ Third log entry      │ /var/logs/file3.log │
│ 2023-10-18 12:15:00 │ Third log entry      │ /var/logs/file3.log │
│ 2023-10-17 09:00:00 │ Updated log entry <- │ /var/logs/file1.log │
│ 2023-10-18 10:30:00 │ Second log entry     │ /var/logs/file2.log │
│ 2023-10-18 09:00:00 │ First log entry      │ /var/logs/file1.log │
└─────────────────────┴──────────────────────┴─────────────────────┘

6 rows in set. Elapsed: 0.007 sec.
```

We can see the record with timestamp `2023-10-17 09:00:00` is updated!

# Refs

https://clickhouse.com/docs/en/guides/developer/mutations

```sql
ALTER TABLE website.clicks
UPDATE url = substring(url, position(url, '://') + 3), visitor_id = new_visit_id
WHERE visit_date < '2022-01-01'
```
