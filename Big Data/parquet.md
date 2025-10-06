# Parquet

Parquet is a columnar, compressed, and splittable storage format optimized for analytical queries on large datasets.

---

## Why Parquet
- **Columnar**: Read only needed columns
- **Compression**: Column-specific codecs (Snappy, ZSTD, Gzip)
- **Encoding**: Dictionary, RLE, bit packing reduce size and speed scans
- **Predicate pushdown**: Prune row groups with min/max statistics
- **Schema evolution**: Add columns, compatible with lake engines

## Best Practices
- Partition by low-cardinality columns (e.g., date)
- Avoid many tiny files; compact with optimize/merge jobs
- Prefer Snappy/ZSTD for balanced speed and size
- Store with stable schemas and clear data types

## When to Use
- Analytics, BI, ML feature stores, lakehouse tables

## When Not Ideal
- High write/update frequency at row-level (use table formats like Delta/Iceberg)
- Interchange of semi-structured data across services (use JSON/Avro)


