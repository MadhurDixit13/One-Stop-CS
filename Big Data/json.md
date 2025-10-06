# JSON

JSON is a text, schema-flexible format widely used for APIs and event logs.

---

## Characteristics
- **Human-readable**: Easy to inspect and debug
- **Flexible schema**: Nested structures, optional fields
- **Interoperable**: Ubiquitous tooling and language support

## Pros / Cons
- Pros: Simple, portable, great for ingestion and interchange
- Cons: Larger size, slower scans, no columnar compression or stats

## Best Practices
- Use for raw landing (Bronze) or streaming events
- Validate with JSON Schema or Protobuf/Avro for contracts
- Convert to Parquet/ORC in curated layers for analytics

## JSON vs Parquet (Quick)
- Storage: JSON larger; Parquet compressed
- Read: JSON scans full records; Parquet columnar and pushdown
- Schema: JSON flexible; Parquet typed with evolution


