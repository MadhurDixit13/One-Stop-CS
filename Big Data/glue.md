# AWS Glue Data Catalog

A fully managed, serverless metadata catalog for discovering, organizing, and managing data in AWS (S3, Redshift, RDS, Lake Formation), integrated with Glue ETL and Athena.

---

## What It Provides
- Central metadata store for tables, schemas, partitions, classifiers
- Crawlers to auto-discover schemas and partitions
- Versioned schema registry for streaming (Glue Schema Registry)
- Fine-grained permissions via Lake Formation integration

## Architecture
- Managed control plane; no self-hosted DB
- Integrates with Glue Jobs, Athena, Redshift Spectrum, EMR, Lake Formation
- APIs/SDK/CLI for programmatic catalog operations

## Strengths / Limitations
- Strengths: Serverless, integrated with AWS analytics stack, granular permissions
- Limitations: AWS-centric, opinionated IAM/LF model, per-request pricing

## Best Practices
- Use Lake Formation for centralized access control (table/column/row)
- Tag-based access policies; align with data domains
- Validate schemas at ingestion; automate crawlers with CI/CD

## Common Operations (CLI)
```bash
# Create a database
aws glue create-database --database-input Name=analytics

# Create a table (simplified)
aws glue create-table --database-name analytics --table-input file://table.json

# Start a crawler
aws glue start-crawler --name discover-raw
```

## Alternatives / Complements
- Hive Metastore, Unity Catalog, Apache Iceberg catalogs, AWS Lake Formation
