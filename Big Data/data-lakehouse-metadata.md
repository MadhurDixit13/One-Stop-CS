# Data Lakehouse Metadata

Lakehouse metadata manages tables, schemas, permissions, quality, and lineage across data lakes using open table formats.

---

## Table Formats
- **Delta Lake**: Transaction log (JSON + checkpoints), ACID, time travel
- **Apache Iceberg**: Snapshot/manifest metadata trees, hidden partitioning
- **Apache Hudi**: Write-optimized (COW/MOR), incremental pulls

## Catalogs
- Hive Metastore, AWS Glue, Unity Catalog, Nessie
- Provide table namespaces, schemas, permissions, and discovery

## Metadata Concepts
- **Schemas**: Column types, nullability, constraints
- **Statistics**: Column min/max, null counts for pruning
- **Partitions**: Physical layout, spec evolution
- **Transactions**: ACID guarantees, snapshot isolation
- **Lineage**: Job- and column-level provenance

## Governance & Security
- RBAC/ABAC, row/column masking, tokenization
- Audit logs, data retention, PII catalogs, DPIA readiness

## Quality & Observability
- Expectations/tests (Great Expectations, Deequ)
- Freshness/volume/anomaly monitors, DQ SLAs

## Best Practices
- Use open table formats (Delta/Iceberg/Hudi) over raw files for managed datasets
- Centralize authz in a unified catalog; avoid bucket-level ACL sprawl
- Automate compaction, vacuum/retention, and checkpointing
- Track lineage and publish a business glossary


