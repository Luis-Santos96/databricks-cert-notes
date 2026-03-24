# Lakehouse Architecture

**Domain weight: 24%** — This is the second-largest domain. Expect questions about *why* the lakehouse exists, how it compares to prior architectures, and where each component fits.

---

## The problem with data warehouses and data lakes

| | Data Warehouse | Data Lake | Lakehouse |
|---|---|---|---|
| Storage | Proprietary / expensive | Cheap object storage | Cheap object storage |
| Schema | Enforced on write | None (schema-on-read) | Enforced, but flexible |
| ACID | Yes | No | Yes (via Delta Lake) |
| BI workloads | Yes | Poor | Yes |
| ML/unstructured | No | Yes | Yes |
| Data freshness | Hours (ETL) | Varies | Near real-time |

**Key insight:** Warehouses gave structure but locked you in and were costly at scale. Lakes gave flexibility but became unreliable swamps. Lakehouse = open format + reliability guarantees.

---

## Medallion Architecture (Bronze / Silver / Gold)

This is the **most heavily tested concept** in this domain.

```
Raw Sources → [Bronze] → [Silver] → [Gold] → BI / ML
```

### Bronze (Raw)
- **What:** Exact copy of source data, as-is
- **Format:** Delta tables (sometimes raw files)
- **Schema:** Minimal enforcement — preserve original structure
- **Purpose:** Audit trail, reprocessing from source
- **Key rule:** Never delete or modify source records here

### Silver (Cleansed & Conformed)
- **What:** Validated, deduplicated, joined data
- **Transformations:** Type casting, null handling, deduplication, light joins
- **Schema:** Enforced — consistent data types and column names
- **Purpose:** Unified view for analysts and data scientists

### Gold (Business-Level Aggregates)
- **What:** Aggregated, business-ready datasets
- **Examples:** Daily revenue by region, customer churn features
- **Consumers:** BI dashboards, ML models, reports
- **Key rule:** Optimized for read performance (ZORDER, partitioning)

### Exam traps
- Bronze is NOT cleaned — don't confuse it with Silver
- Gold doesn't always mean "final" — you can have multiple Gold tables for different use cases
- The medallion pattern is a *recommendation*, not a requirement enforced by Databricks

---

## Databricks Platform Components

### Cluster types

| | All-Purpose | Job Cluster |
|---|---|---|
| Created by | User manually | Workflow/job automatically |
| Lifecycle | Persistent (until terminated) | Spun up for job, torn down after |
| Cost | Higher (idle time) | Lower (ephemeral) |
| Use case | Interactive notebooks, exploration | Production jobs |

**Exam tip:** Job clusters are preferred for production. All-purpose clusters cost more because they sit idle.

### Cluster modes (runtime)
- **Standard:** Single-user or shared (legacy)
- **Single User:** Dedicated to one user, required for Unity Catalog access
- **Shared:** Multiple users on one cluster, resource isolation via Unity Catalog

### Databricks Runtime (DBR)
- Includes Apache Spark + Databricks-specific optimizations
- **Photon:** Vectorized query engine — speeds up SQL and Delta operations significantly
- **ML Runtime:** Includes MLflow, popular ML libraries pre-installed
- **GPU Runtime:** For deep learning workloads

---

## Databricks Repos and Notebooks

- **Repos:** Git-backed version control inside Databricks workspace
- **Notebooks:** Supports Python, SQL, Scala, R — cells can mix languages with `%python`, `%sql` magic commands
- **%run:** Executes another notebook — shares scope (variables, functions)
- **dbutils.notebook.run():** Runs a notebook as a job — isolated scope, returns a string result

### Key dbutils
```python
dbutils.fs.ls("/mnt/data")          # list files
dbutils.fs.cp("/src", "/dst")       # copy
dbutils.secrets.get("scope", "key") # retrieve secrets
dbutils.widgets.text("param", "")   # notebook parameter
```

---

## Workspace, Metastore, and Catalog hierarchy

```
Metastore (one per region, managed by Unity Catalog)
└── Catalog
    └── Schema (Database)
        └── Table / View / Function
```

- **Hive Metastore (legacy):** One per workspace, not shareable
- **Unity Catalog Metastore:** Shared across workspaces, centralized governance

---

## Common exam questions

1. **Which layer stores raw, unmodified data?** → Bronze
2. **What is the benefit of a lakehouse over a data lake?** → ACID transactions, schema enforcement, BI-ready performance
3. **When should you use a Job cluster vs All-Purpose?** → Job cluster for production/scheduled workloads
4. **What does Photon accelerate?** → SQL queries and Delta Lake operations (not arbitrary Python/Pandas)
5. **What's the difference between `%run` and `dbutils.notebook.run()`?** → `%run` shares scope; `dbutils.notebook.run()` is isolated
