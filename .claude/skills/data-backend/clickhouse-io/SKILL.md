---
name: clickhouse-io
description: ClickHouse analytical query patterns, engine selection, columnar storage, materialized views, partitioning, and time-series analytics for OLAP workloads.
---

# ClickHouse Analytics

ClickHouse-specific patterns for high-performance analytical queries and data pipelines. Design tables for your query patterns — ClickHouse rewards schema decisions made up front.

---

## 1. Engine Selection

Choose the MergeTree family engine based on your write pattern:

| Engine | Use When |
|---|---|
| `MergeTree` | Standard append-only analytics |
| `ReplacingMergeTree` | Data may arrive with duplicate IDs (deduplicate on merge) |
| `AggregatingMergeTree` | Pre-aggregate metrics incrementally |
| `SummingMergeTree` | Simple sum aggregations at merge time |

**MergeTree** (standard):

```sql
CREATE TABLE events (
    date        Date,
    market_id   String,
    volume      UInt64,
    trades      UInt32,
    created_at  DateTime
) ENGINE = MergeTree()
PARTITION BY toYYYYMM(date)
ORDER BY (date, market_id)
SETTINGS index_granularity = 8192;
```

**ReplacingMergeTree** for deduplication:

```sql
CREATE TABLE user_events (
    event_id    String,
    user_id     String,
    event_type  String,
    timestamp   DateTime
) ENGINE = ReplacingMergeTree()
PARTITION BY toYYYYMM(timestamp)
ORDER BY (user_id, event_id, timestamp)
PRIMARY KEY (user_id, event_id);
```

**Rule:** Never use `FINAL` in production queries to force deduplication — it disables parallel processing. Merge data explicitly before querying or use `ReplacingMergeTree` with background merges.

---

## 2. Partitioning Strategy

Partition by time — usually month (`toYYYYMM`) or day (`toYYYYMMDD`) for high-volume tables.

- Keep total partition count under 1000 per table.
- Use `Date` type for partition keys, not `DateTime`.
- ClickHouse prunes entire partitions that fall outside `WHERE` filters on the partition key — always filter on the partition column.

**Rule:** Never partition by a high-cardinality column (e.g., `user_id`). Partition by time, then put high-cardinality columns in the `ORDER BY`.

---

## 3. Ordering Key and Columnar Storage

The `ORDER BY` key is the primary index. Put most-filtered columns first, then range columns:

```sql
-- Queries filter on market_id equality then date range
ORDER BY (market_id, date)
```

Data types for columnar efficiency:

- Use `LowCardinality(String)` for columns with < 10K distinct values (status, country, event_type).
- Use `UInt32` instead of `UInt64` when values fit — halves storage.
- Use `Enum8` or `Enum16` for fixed categorical sets.
- Never use `Nullable` unless the column genuinely has NULL values — it adds a separate null bitmask column.

---

## 4. Efficient Filtering and Aggregations

Filter on `ORDER BY` columns first to enable index skipping:

```sql
-- GOOD: indexed columns first
SELECT *
FROM events
WHERE date >= '2025-01-01'
  AND market_id = 'market-123'
  AND volume > 1000
LIMIT 100;

-- BAD: non-indexed filter first defeats index skipping
SELECT * FROM events WHERE volume > 1000 AND market_name LIKE '%foo%';
```

Use ClickHouse-native aggregation functions — they are significantly faster than generic SQL equivalents:

```sql
SELECT
    toStartOfDay(created_at)  AS day,
    market_id,
    sum(volume)               AS total_volume,
    count()                   AS total_trades,
    uniq(trader_id)           AS unique_traders,
    quantile(0.95)(trade_size) AS p95_size
FROM trades
WHERE created_at >= today() - INTERVAL 7 DAY
GROUP BY day, market_id
ORDER BY day DESC, total_volume DESC;
```

**Rule:** Use `uniq()` instead of `COUNT(DISTINCT ...)`. Use `quantile()` instead of percentile functions. Use `count()` (no argument) instead of `COUNT(*)`.

---

## 5. Window Functions

```sql
-- Running cumulative volume per market
SELECT
    date,
    market_id,
    volume,
    sum(volume) OVER (
        PARTITION BY market_id
        ORDER BY date
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS cumulative_volume
FROM events
WHERE date >= today() - INTERVAL 30 DAY
ORDER BY market_id, date;
```

---

## 6. Materialized Views

Use materialized views to pre-aggregate data in real time as it is inserted. The view fires on every insert to the source table.

```sql
-- Target table (AggregatingMergeTree)
CREATE TABLE market_stats_hourly (
    hour       DateTime,
    market_id  String,
    total_volume AggregateFunction(sum, UInt64),
    total_trades AggregateFunction(count, UInt32),
    unique_users AggregateFunction(uniq, String)
) ENGINE = AggregatingMergeTree()
PARTITION BY toYYYYMM(hour)
ORDER BY (hour, market_id);

-- Materialized view feeds target on insert
CREATE MATERIALIZED VIEW market_stats_hourly_mv
TO market_stats_hourly AS
SELECT
    toStartOfHour(timestamp) AS hour,
    market_id,
    sumState(amount)         AS total_volume,
    countState()             AS total_trades,
    uniqState(user_id)       AS unique_users
FROM trades
GROUP BY hour, market_id;

-- Query: merge aggregate states
SELECT
    hour, market_id,
    sumMerge(total_volume)   AS volume,
    countMerge(total_trades) AS trades,
    uniqMerge(unique_users)  AS users
FROM market_stats_hourly
WHERE hour >= now() - INTERVAL 24 HOUR
GROUP BY hour, market_id;
```

**Rule:** Always pair a materialized view with an `AggregatingMergeTree` target. Never query the MV directly — query the target table using `*Merge()` functions.

---

## 7. Time-Series Patterns

**Daily active users:**

```sql
SELECT
    toDate(timestamp)    AS date,
    uniq(user_id)        AS dau
FROM events
WHERE timestamp >= today() - INTERVAL 30 DAY
GROUP BY date
ORDER BY date;
```

**Cohort retention:**

```sql
SELECT
    toStartOfMonth(signup_date) AS cohort,
    dateDiff('month', cohort, toStartOfMonth(activity_date)) AS months_since_signup,
    count(DISTINCT user_id) AS active_users
FROM (
    SELECT
        user_id,
        min(toDate(timestamp)) OVER (PARTITION BY user_id) AS signup_date,
        toDate(timestamp) AS activity_date
    FROM events
)
GROUP BY cohort, months_since_signup
ORDER BY cohort, months_since_signup;
```

---

## 8. Insertion Patterns

Always batch inserts — individual row inserts trigger a new part on disk each time:

```typescript
// GOOD: batch insert
async function bulkInsert(rows: Row[]) {
  const values = rows.map(r => `('${r.id}', '${r.market_id}', ${r.amount}, '${r.ts}')`).join(',');
  await clickhouse.query(`INSERT INTO trades (id, market_id, amount, timestamp) VALUES ${values}`).toPromise();
}

// BAD: row-by-row insert in a loop
for (const row of rows) {
  await clickhouse.query(`INSERT INTO trades VALUES ('${row.id}', ...)`).toPromise();
}
```

Target batch sizes of 10K–100K rows. For continuous ingestion, use Kafka integration or a streaming insert buffer.

**Rule:** Never insert fewer than 1000 rows per INSERT statement in production. Too many small parts degrade merge performance and query speed.

---

## 9. Performance Monitoring

```sql
-- Slow queries in the last hour
SELECT query_id, query, query_duration_ms, read_rows, read_bytes, memory_usage
FROM system.query_log
WHERE type = 'QueryFinish'
  AND query_duration_ms > 1000
  AND event_time >= now() - INTERVAL 1 HOUR
ORDER BY query_duration_ms DESC
LIMIT 10;

-- Table sizes and part counts
SELECT database, table,
    formatReadableSize(sum(bytes)) AS size,
    sum(rows) AS rows,
    count() AS parts
FROM system.parts
WHERE active
GROUP BY database, table
ORDER BY sum(bytes) DESC;
```

Monitor `parts` count per table. High part counts (> 300) indicate insufficient batching or merge lag.

---

## 10. Anti-Patterns

- Never use `SELECT *` — specify columns to exploit columnar storage.
- Never use `FINAL` in production — use background merges or explicit `OPTIMIZE TABLE`.
- Never join more than 2–3 tables — denormalize for analytics instead.
- Never issue small frequent inserts — always batch.
- Never filter on non-`ORDER BY` columns without also filtering on the partition key.
