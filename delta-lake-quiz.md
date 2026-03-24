# Delta Lake Fundamentals Quiz
## Databricks Certified Data Engineer Associate — Practice Questions

---

**Instructions:** Answer all 5 questions. For each question, pick the best answer (A, B, C, or D). At the end, ask me to reveal the answers and explain each one.

---

### Question 1

A data engineer runs the following command on a Delta table:

```sql
VACUUM my_table RETAIN 0 HOURS
```

Immediately after, they try to query a version from 2 days ago using time travel:

```sql
SELECT * FROM my_table VERSION AS OF 5
```

What happens?

**A.** The query succeeds because Delta Lake always retains the last 10 versions regardless of VACUUM.

**B.** The query fails because VACUUM deleted the data files needed for that historical version.

**C.** The query succeeds because VACUUM only removes metadata, not Parquet data files.

**D.** The query fails because `VERSION AS OF` syntax is incorrect — you must use `TIMESTAMP AS OF`.

---

### Question 2

A data engineer wants to improve query performance on a large Delta table that is frequently filtered by `country` and `event_date`. Which command best achieves this?

**A.** `OPTIMIZE my_table`

**B.** `OPTIMIZE my_table ZORDER BY (country, event_date)`

**C.** `VACUUM my_table RETAIN 168 HOURS`

**D.** `ALTER TABLE my_table CLUSTER BY (country, event_date)`

---

### Question 3

Which of the following best describes the role of the Delta Lake transaction log (`_delta_log`)?

**A.** It stores the actual Parquet data files for the table in a compressed format.

**B.** It is an append-only ledger of every change made to a Delta table, enabling ACID transactions and time travel.

**C.** It is a temporary cache that Spark uses to speed up reads and is cleared after each session.

**D.** It stores only schema information and is not used during query execution.

---

### Question 4

A data engineer needs to insert new records into a Delta table but wants to avoid inserting duplicates based on a unique `user_id`. Which approach is most appropriate?

**A.** Use `INSERT INTO` with a `WHERE NOT EXISTS` subquery.

**B.** Use `MERGE INTO` with a condition matching on `user_id`.

**C.** Use `INSERT OVERWRITE` to replace the entire table each time.

**D.** Use `CREATE OR REPLACE TABLE` to rebuild the table with deduplication.

---

### Question 5

A Delta table has accumulated many small files over time due to frequent streaming writes. A data engineer runs `OPTIMIZE` on the table. What is the primary benefit of this operation?

**A.** It deletes all historical versions of the table to free up storage space.

**B.** It rewrites small files into larger ones, improving read performance by reducing the number of files scanned.

**C.** It converts the table from Parquet format to Delta format.

**D.** It enforces the table schema and rejects any records that don't match the defined schema.

---

*When you're done, tell me your answers (e.g., "1-B, 2-A, 3-C, 4-D, 5-B") and ask Claude to score you and explain each answer.*
