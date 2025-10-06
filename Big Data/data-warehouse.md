# Data Warehouse

A data warehouse is a centralized, governed store for integrated, historical data optimized for analytical SQL and BI.

---

## Characteristics
- **Schema-on-write**: Modeled upfront (star/snowflake)
- **Structured data**: Conformed dimensions, facts
- **Performance**: MPP engines, query optimizers, indexes
- **Consistency**: Strong data quality and governance

## Modeling
- **Star schema**: Fact tables linked to dimensions
- **Snowflake**: Normalized dimensions for reuse
- **Slowly Changing Dimensions (SCD)**: Type 1/2/3 handling

## Workloads
- Descriptive analytics, KPI reporting, self-serve BI, data marts

## Platforms
- Cloud DW: BigQuery, Snowflake, Redshift, Synapse
- On-prem: Teradata, Exadata, Vertica

## Pros / Cons
- Pros: Reliable, consistent, fast SQL for BI
- Cons: Rigid modeling, higher cost, slower to adapt to new data

## Best Practices
- Clear semantic layer and data contracts
- Batch and streaming ELT into modeled layers (staging → core → marts)
- Govern with RBAC, masking, row-level security, auditing


