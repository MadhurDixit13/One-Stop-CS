# Data Mart

A data mart is a subject-oriented subset of a data warehouse optimized for a particular team or business function.

---

## Types
- **Dependent**: Sourced from the enterprise data warehouse
- **Independent**: Built directly from source systems (less common for governance)

## Design
- Narrow scope: specific domain (e.g., sales, finance)
- Denormalized star schemas for performance and simplicity

## Use Cases
- Team dashboards, ad-hoc analysis, sandbox experimentation

## Pros / Cons
- Pros: Faster delivery, relevant metrics, performance
- Cons: Risk of silos/metric drift if not aligned to central models

## Best Practices
- Derive from governed core models
- Publish semantic definitions and metric catalog
- Automate refresh and document lineage


