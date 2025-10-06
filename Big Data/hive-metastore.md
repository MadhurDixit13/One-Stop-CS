# Hive Metastore

A central metadata service that stores table schemas, partitions, and locations for data in a data lake or warehouse, used by engines like Hive, Spark, Presto/Trino.

---

## What It Provides
- Central catalog of databases, tables, partitions
- Schema, serde info, storage formats, and locations (HDFS/S3/ADLS)
- Authorization hooks and table/partition statistics

## Architecture
- Metastore service (Thrift) + backing RDBMS (MySQL/Postgres)
- Clients: HiveServer2, Spark SQL, Presto/Trino, Impala
- Deployments: Embedded, Local, Remote metastore service

## Strengths / Limitations
- Strengths: De facto standard, broad engine compatibility
- Limitations: No ACID transactions on files, weak multi-catalog/namespace story, operational management of DB/service

## Best Practices
- Run remote metastore with HA RDBMS
- Back up metastore DB regularly; enable table stats
- Control changes with migrations (e.g., schema registry, CI/CD)

## Alternatives / Successors
- AWS Glue Data Catalog, Unity Catalog, Apache Iceberg catalogs, Nessie
