# Unity Catalog

A unified governance layer for data and AI across data lakes and warehouses, providing centralized metadata, permissions, lineage, and auditing (notably in Databricks).

---

## What It Provides
- Central, multi-cloud catalog with metastore(s), catalogs, schemas, tables
- Fine-grained permissions (table, column, row masking), data lineage
- Governance for files, tables (Delta/Iceberg), ML models, functions

## Architecture
- Metastore → Catalog → Schema (Database) → Table/View hierarchy
- Works with external locations/volumes referencing object stores
- Integrates with workspace identity, SCIM/SCIM groups, audit logs

## Strengths / Limitations
- Strengths: Central governance, cross-workspace catalogs, strong lineage
- Limitations: Platform-coupled, feature parity differs by cloud/edition

## Best Practices
- Establish data domains as catalogs; use schemas per product/team
- Enforce least privilege; use grants at catalog/schema level
- Use external locations with storage credentials for object-store RBAC
- Enable column/row policies; document ownership and data contracts

## Common Operations (SQL)
```sql
-- Create catalog and schema
CREATE CATALOG sales;
CREATE SCHEMA sales.core;

-- Create managed/external table
CREATE TABLE sales.core.orders (
  order_id BIGINT,
  customer_id BIGINT,
  order_ts TIMESTAMP,
  total_amount DECIMAL(18,2)
) USING DELTA;

-- Grants
GRANT SELECT ON TABLE sales.core.orders TO `analyst_role`;
```

## Alternatives / Complements
- Glue Data Catalog + Lake Formation, Hive Metastore, Nessie, OpenLineage
