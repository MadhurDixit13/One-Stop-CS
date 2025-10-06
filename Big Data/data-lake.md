# Data Lake

A data lake is a centralized repository that stores raw, granular data from many sources in its native format for large-scale analytics and ML.

---

## Characteristics
- **Schema-on-read**: Define structure at analysis time
- **All data types**: Structured, semi-structured, unstructured
- **Low-cost storage**: Object stores (S3, ADLS, GCS)
- **Decoupled compute**: Query with Spark, Presto/Trino, Athena, etc.

## Architecture Layers
- **Ingestion (Bronze)**: Raw, append-only landing
- **Curation (Silver)**: Cleaned, conformed, deduplicated
- **Serving (Gold)**: Aggregated, optimized for BI

## Governance & Management
- **Catalog**: Hive Metastore, Glue, Unity Catalog
- **Security**: IAM, lake-level ACLs, table/column/row masking
- **Quality**: Expectations (Great Expectations, Deequ)
- **Lineage**: OpenLineage, built-in platform lineage

## File Formats
- Use columnar formats (Parquet, ORC) for analytics; JSON/CSV for ingestion.

## Pros / Cons
- Pros: Flexibility, cost-effective, ML-friendly
- Cons: Risk of "data swamp" without governance; slower for strict BI

## Common Tools
- Storage: S3, ADLS, GCS
- Compute: Spark, Databricks, Flink, Trino/Presto
- Orchestration: Airflow, Dagster
- Catalog/Governance: Glue, Hive, Unity Catalog

## Best Practices
- Separate raw/curated/serving zones
- Enforce naming, partitioning, and lifecycle policies
- Prefer Parquet with partitioning and Z-ordering/compaction
- Track metadata and lineage from day one


