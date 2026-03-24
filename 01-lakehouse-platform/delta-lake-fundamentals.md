# Delta Lake Fundamentals

Delta Lake is the storage layer that makes the Lakehouse reliable. It sits on top of Parquet files and adds a **transaction log** that enables ACID guarantees.

---

## How Delta Lake works under the hood

```
Delta Table
├── _delta_log/               ← transaction log (JSON + Parquet checkpoints)
│   ├── 00000000000000000000.json
│   ├── 00000000000000000001.json
│   └── 00000000000000000010.checkpoint.parquet
└── part-00000-*.snappy.parquet   ← actual data files
```

- Every write appends a new JSON entry to `_delta_log/`
- Every 10 commits, Delta creates a **checkpoint** (Parquet summary) for faster log replay
- Reading a table = reading the latest checkpoint + subsequent JSON entries to reconstruct current state

**Exam tip:** The transaction log is what enables time travel, ACID, and schema enforcement — not the Parquet files themselves.

---

## ACID Guarantees

| Property | What it means in Delta |
|---|---|
| **Atomicity** | A write either fully succeeds or fully rolls back — no partial writes |
| **Consistency** | Schema is always valid; constraints are enforced |
| **Isolation** | Concurrent reads and writes don't corrupt each other (optimistic concurrency) |
| **Durability** | Committed data survives failures (log + cloud storage) |

---

## Time Travel

Delta keeps old data files until you explicitly clean them up. You can query past versions:

```sql
-- By version number
SELECT * FROM orders VERSION AS OF 5;

-- By timestamp
SELECT * FROM orders TIMESTAMP AS OF '2024-01-15 12:00:00';
```

```python
# PySpark
df = spark.read.format("delta").option("versionAsOf", 5).load("/delta/orders")
df = spark.read.format("delta").option("timestampAsOf", "2024-01-15").load("/delta/orders")
```

### VACUUM — cleaning old files
```sql
VACUUM orders RETAIN 168 HOURS;  -- default is 7 days (168 hours)
```

- **Default retention: 7 days**
- Running `VACUUM` with less than 7 days requires setting `spark.databricks.delta.retentionDurationCheck.enabled = false`
- After VACUUM, those versions are **no longer queryable** via time travel

**Exam trap:** `VACUUM` does NOT affect the transaction log — it only removes data files no longer referenced by current or retained versions.

---

## Schema Enforcement and Evolution

### Schema Enforcement (default behavior)
Delta **rejects writes** that don't match the existing schema:
```python
# This will FAIL if new_df has extra or differently typed columns
new_df.write.format("delta").mode("append").save("/delta/orders")
```

### Schema Evolution (opt-in)
```python
# Allow new columns to be added automatically
new_df.write.format("delta") \
  .mode("append") \
  .option("mergeSchema", "true") \
  .save("/delta/orders")
```

```sql
-- For ALTER TABLE operations
ALTER TABLE orders ADD COLUMNS (discount DOUBLE);
```

**Key rule:**
- **mergeSchema:** Adds new columns, widens data types — safe
- **overwriteSchema:** Replaces the schema entirely — destructive, requires explicit opt-in

---

## Common Delta Operations

### CREATE TABLE
```sql
CREATE TABLE orders (
  order_id BIGINT,
  customer_id BIGINT,
  amount DOUBLE,
  order_date DATE
)
USING DELTA
LOCATION 'dbfs:/delta/orders';  -- external table
```

Without `LOCATION` → **managed table** (Databricks controls the files).

### INSERT / UPDATE / DELETE
```sql
INSERT INTO orders VALUES (1, 42, 99.99, '2024-01-15');

UPDATE orders SET amount = 109.99 WHERE order_id = 1;

DELETE FROM orders WHERE order_date < '2020-01-01';
```

### MERGE (upsert)
The most important and most tested DML operation:

```sql
MERGE INTO orders AS target
USING updates AS source
ON target.order_id = source.order_id
WHEN MATCHED THEN
  UPDATE SET target.amount = source.amount
WHEN NOT MATCHED THEN
  INSERT (order_id, customer_id, amount, order_date)
  VALUES (source.order_id, source.customer_id, source.amount, source.order_date)
WHEN NOT MATCHED BY SOURCE THEN
  DELETE;  -- optional: remove rows not in source
```

**MERGE use cases:**
- CDC (Change Data Capture) — sync changes from source systems
- Deduplication — update if exists, insert if new
- SCD Type 1 — overwrite old values with new ones

---

## Table Utility Commands

```sql
DESCRIBE DETAIL orders;      -- file count, size, partitioning, format
DESCRIBE HISTORY orders;     -- full transaction log (versions, operations, timestamps)
DESCRIBE EXTENDED orders;    -- schema + table properties + storage info
```

### OPTIMIZE
```sql
OPTIMIZE orders;                      -- compacts small files
OPTIMIZE orders ZORDER BY (customer_id);  -- compacts + co-locates data by column
```

- **Small file problem:** Streaming writes and frequent small appends create many tiny Parquet files → slow reads
- **OPTIMIZE** merges small files into larger ones (~1 GB target)
- **ZORDER:** Multi-dimensional clustering — puts related rows physically together, speeds up filter queries on that column

**Exam tip:** ZORDER doesn't sort; it *clusters*. Best for high-cardinality columns used in WHERE filters.

---

## Managed vs External Tables

| | Managed | External |
|---|---|---|
| Metadata | Databricks/Unity Catalog | Databricks/Unity Catalog |
| Data files | Databricks-controlled location | Your specified location |
| `DROP TABLE` | Deletes metadata **AND** data | Deletes metadata only, **data survives** |

```sql
-- Managed (no LOCATION)
CREATE TABLE managed_orders USING DELTA AS SELECT * FROM raw_orders;

-- External (with LOCATION)
CREATE TABLE external_orders USING DELTA
LOCATION 'abfss://container@storage.dfs.core.windows.net/orders';
```

**Exam trap:** Dropping a managed table = data is gone. Dropping an external table = data is safe.

---

## Delta Table Properties

```sql
-- Set properties
ALTER TABLE orders SET TBLPROPERTIES (
  'delta.logRetentionDuration' = 'interval 30 days',
  'delta.deletedFileRetentionDuration' = 'interval 7 days'
);

-- View properties
SHOW TBLPROPERTIES orders;
```

---

## Key numbers to memorize

| Setting | Default |
|---|---|
| VACUUM retention | 7 days (168 hours) |
| Log checkpoint frequency | Every 10 commits |
| OPTIMIZE target file size | ~1 GB |
| Delta log format | JSON (commits) + Parquet (checkpoints) |

---

## Common exam questions

1. **What enables ACID transactions in Delta Lake?** → The transaction log (`_delta_log/`)
2. **You run VACUUM with 0 hours — what happens?** → Error unless you disable the retention check; time travel history is lost
3. **What's the difference between `mergeSchema` and `overwriteSchema`?** → mergeSchema adds columns; overwriteSchema replaces the entire schema
4. **When should you use MERGE vs INSERT OVERWRITE?** → MERGE for row-level upserts; INSERT OVERWRITE for full partition/table replacement
5. **What does `DESCRIBE HISTORY` show?** → Full audit log of all operations with version numbers and timestamps
6. **Dropping an external table — what happens to the data?** → Data files remain; only the metadata is removed
