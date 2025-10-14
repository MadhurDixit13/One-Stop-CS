# BigQuery

BigQuery is Google Cloud's fully-managed, serverless, and highly scalable data warehouse designed for fast SQL analytics on massive datasets.

---

## What Problem Does It Solve?

BigQuery addresses:
- **Scalability Challenges**: Query petabytes of data without managing infrastructure
- **Performance Bottlenecks**: Fast interactive analysis using distributed columnar storage and execution
- **Cost of Data Warehousing**: Pay only for storage and queries run (serverless pricing)
- **Complex Infrastructure**: No need to provision, tune, or maintain servers
- **Data Silos**: Integrates with GCP services and external data sources

---

## Core Concepts

### Architecture
- **Serverless**: No infrastructure to manage; auto-scales based on query demand
- **Columnar Storage**: Optimized for analytical queries (reads specific columns, not entire rows)
- **Separation of Compute and Storage**: Storage and query execution scale independently
- **Dremel Engine**: Google's distributed query engine powers BigQuery

### Key Features
- **Standard SQL**: ANSI SQL 2011 compliant with extensions (arrays, structs, window functions)
- **Partitioning & Clustering**: Improve performance and reduce costs
- **Nested & Repeated Fields**: Native support for JSON-like structures
- **Streaming Inserts**: Real-time data ingestion
- **ML Integration**: Built-in machine learning with BigQuery ML
- **Federated Queries**: Query external data sources (Cloud Storage, Bigtable, Spanner)

### Pricing Model
- **Storage**: $0.02 per GB/month (active), $0.01 per GB/month (long-term)
- **Queries**: $5 per TB processed (on-demand) or flat-rate slots
- **Streaming**: $0.01 per 200 MB inserted

---

## Common Use Cases

- **Business Intelligence & Analytics**: Interactive dashboards and reports (Looker, Tableau, Data Studio)
- **Data Lake Analytics**: Query data in Cloud Storage without loading into BigQuery
- **Log Analytics**: Analyze application and infrastructure logs at scale
- **Real-Time Analytics**: Stream data from Pub/Sub or Dataflow for immediate insights
- **Machine Learning**: Train and deploy ML models using BigQuery ML
- **Data Warehousing**: Replace traditional on-prem data warehouses

---

## Best Practices

### Performance Optimization
- **Partition Tables**: By date/timestamp to reduce scanned data
- **Cluster Tables**: By frequently filtered columns (e.g., region, customer_id)
- **Avoid `SELECT *`**: Query only needed columns to minimize data scanned
- **Use `LIMIT` in Development**: Test queries on small subsets first
- **Materialize Results**: Use `CREATE TABLE AS SELECT` for frequently accessed aggregations

### Cost Optimization
- **Partition & Cluster**: Reduce data scanned per query
- **Use Flat-Rate Pricing**: For consistent high query volumes
- **Monitor Query Costs**: Use `INFORMATION_SCHEMA` and Cloud Billing reports
- **Cache Results**: BigQuery caches query results for 24 hours (free)
- **Expire Old Data**: Set table expiration or partition expiration

### Security & Governance
- **Column-Level Security**: Restrict access to sensitive fields
- **Row-Level Security**: Apply policies to filter rows per user
- **Audit Logs**: Track all queries and data access
- **Data Masking**: Use views or BigQuery's dynamic data masking
- **Encryption**: Data encrypted at rest and in transit by default

---

## Example Queries

### Basic Query
```sql
SELECT
  name,
  COUNT(*) AS total_orders,
  SUM(amount) AS revenue
FROM `project.dataset.orders`
WHERE order_date >= '2024-01-01'
GROUP BY name
ORDER BY revenue DESC
LIMIT 10;
```

### Partitioned Table Query
```sql
SELECT
  event_name,
  COUNT(*) AS event_count
FROM `project.dataset.events`
WHERE _PARTITIONTIME BETWEEN '2024-01-01' AND '2024-01-31'
GROUP BY event_name;
```

### BigQuery ML Example
```sql
-- Train a logistic regression model
CREATE OR REPLACE MODEL `project.dataset.churn_model`
OPTIONS(model_type='logistic_reg', input_label_cols=['churned']) AS
SELECT
  age,
  account_balance,
  num_purchases,
  churned
FROM `project.dataset.customers`;

-- Make predictions
SELECT
  customer_id,
  predicted_churned,
  predicted_churned_probs[OFFSET(0)].prob AS churn_probability
FROM ML.PREDICT(MODEL `project.dataset.churn_model`,
  TABLE `project.dataset.new_customers`);
```

---

## Integration & Tools

- **Data Ingestion**: Cloud Storage, Pub/Sub, Dataflow, Cloud Logging, Fivetran, Airbyte
- **BI & Visualization**: Looker, Tableau, Power BI, Google Data Studio
- **ETL/ELT**: Dataflow, dbt, Apache Airflow
- **Notebooks**: Colab, Vertex AI Workbench, Jupyter
- **APIs**: REST API, Python client library, Java client library

---

## Further Reading

- [BigQuery Official Documentation](https://cloud.google.com/bigquery/docs)
- [BigQuery Pricing](https://cloud.google.com/bigquery/pricing)
- [BigQuery ML Documentation](https://cloud.google.com/bigquery-ml/docs)
- [Best Practices for Query Performance](https://cloud.google.com/bigquery/docs/best-practices-performance-overview)
- [Partitioning and Clustering Guide](https://cloud.google.com/bigquery/docs/partitioned-tables)
- [BigQuery SQL Reference](https://cloud.google.com/bigquery/docs/reference/standard-sql/query-syntax)

