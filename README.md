# databricks-cert-notes

Study notes and exercises for the **Databricks Certified Data Engineer Associate** exam.

[![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=flat-square&logo=databricks&logoColor=white)](https://databricks.com)
[![Target](https://img.shields.io/badge/Target-June_2026-f59e0b?style=flat-square)]()
[![Status](https://img.shields.io/badge/Status-In_Progress-22c55e?style=flat-square)]()

---

## Why this repo

Structured notes from my certification preparation — organized by exam domain, with practical examples from real-world experience. Built as a reference for myself and for other data engineers on the same path.

## Exam domains

| Domain | Weight | Status |
|--------|--------|--------|
| Databricks Lakehouse Platform | 24% | 📝 In progress |
| ELT with Spark SQL and Python | 29% | 📝 In progress |
| Incremental Data Processing | 22% | ⬜ Not started |
| Production Pipelines | 16% | ⬜ Not started |
| Data Governance | 9% | ⬜ Not started |

## Structure

```
databricks-cert-notes/
├── 01-lakehouse-platform/
│   ├── architecture.md
│   ├── delta-lake-fundamentals.md
│   └── unity-catalog.md
├── 02-elt-spark-sql-python/
│   ├── spark-sql-essentials.md
│   ├── python-for-spark.md
│   └── data-transformations.md
├── 03-incremental-processing/
│   ├── structured-streaming.md
│   └── auto-loader.md
├── 04-production-pipelines/
│   ├── delta-live-tables.md
│   └── workflows.md
├── 05-data-governance/
│   └── unity-catalog-governance.md
├── exercises/
│   └── practice-questions.md
└── README.md
```

## Key concepts covered

- **Delta Lake** — ACID transactions, time travel, schema enforcement and evolution
- **Lakehouse architecture** — medallion pattern (bronze/silver/gold), when to use each layer
- **Spark SQL & PySpark** — joins, aggregations, window functions, UDFs
- **Structured Streaming** — Auto Loader, trigger modes, watermarks, checkpointing
- **Delta Live Tables** — declarative pipelines, expectations, monitoring
- **Unity Catalog** — data governance, access control, lineage tracking

## Resources

- [Databricks Academy — Data Engineer Associate course](https://www.databricks.com/learn/certification/data-engineer-associate)
- [Databricks documentation](https://docs.databricks.com/)
- [Delta Lake documentation](https://docs.delta.io/)

## Author

**Luis Ricardo** — Data Engineer  
[Portfolio](https://luis-santos96.github.io) · [LinkedIn](https://www.linkedin.com/in/luisr-santos/) · [GitHub](https://github.com/Luis-Santos96)
