# Apache Spark - Unified Analytics Engine

Apache Spark is a unified analytics engine for large-scale data processing. It provides high-level APIs in Java, Scala, Python, and R, and an optimized engine that supports general computation graphs for data analysis.

---

## What is Apache Spark?

Apache Spark is a:
- **Unified Analytics Engine**: Single platform for batch, streaming, SQL, ML, and graph processing
- **In-Memory Computing**: Caches data in memory for faster processing
- **Fault Tolerant**: Automatically recovers from node failures
- **Scalable**: Runs on clusters with thousands of nodes
- **Developer Friendly**: High-level APIs in multiple languages

### Key Components
- **Spark Core**: Basic functionality and RDDs
- **Spark SQL**: SQL and structured data processing
- **Spark Streaming**: Real-time data processing
- **MLlib**: Machine learning library
- **GraphX**: Graph processing
- **SparkR**: R language integration

---

## Architecture

### Cluster Architecture
```
Driver Program
    ↓
Spark Context
    ↓
Cluster Manager (YARN/Mesos/Standalone)
    ↓
Worker Nodes (Executors)
    ↓
Tasks
```

### Key Concepts
- **Driver**: Main program that creates SparkContext
- **Cluster Manager**: Manages cluster resources (YARN, Mesos, Standalone)
- **Worker Node**: Machine that runs application code
- **Executor**: Process on worker node that runs tasks
- **Task**: Unit of work sent to executor
- **Job**: Parallel computation of multiple tasks
- **Stage**: Set of tasks that can be executed together

---

## Installation & Setup

### Local Installation
```bash
# Download Spark
wget https://archive.apache.org/dist/spark/spark-3.5.0/spark-3.5.0-bin-hadoop3.tgz
tar -xzf spark-3.5.0-bin-hadoop3.tgz
cd spark-3.5.0-bin-hadoop3

# Set environment variables
export SPARK_HOME=/path/to/spark-3.5.0-bin-hadoop3
export PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin

# Start Spark shell
./bin/spark-shell  # Scala
./bin/pyspark      # Python
```

### Python Setup
```bash
# Install PySpark
pip install pyspark

# Or with specific version
pip install pyspark==3.5.0
```

### Docker Setup
```dockerfile
FROM openjdk:11-jre-slim

# Install Python
RUN apt-get update && apt-get install -y python3 python3-pip

# Install Spark
RUN wget https://archive.apache.org/dist/spark/spark-3.5.0/spark-3.5.0-bin-hadoop3.tgz && \
    tar -xzf spark-3.5.0-bin-hadoop3.tgz && \
    mv spark-3.5.0-bin-hadoop3 /opt/spark

ENV SPARK_HOME=/opt/spark
ENV PATH=$PATH:$SPARK_HOME/bin:$SPARK_HOME/sbin

# Install PySpark
RUN pip3 install pyspark

EXPOSE 8080 7077 6066

CMD ["/opt/spark/bin/spark-class", "org.apache.spark.deploy.master.Master"]
```

---

## SparkContext & SparkSession

### SparkContext (Legacy)
```python
from pyspark import SparkContext, SparkConf

# Create configuration
conf = SparkConf().setAppName("MyApp").setMaster("local[*]")

# Create SparkContext
sc = SparkContext(conf=conf)

# Stop context
sc.stop()
```

### SparkSession (Modern)
```python
from pyspark.sql import SparkSession

# Create SparkSession
spark = SparkSession.builder \
    .appName("MyApp") \
    .master("local[*]") \
    .config("spark.sql.adaptive.enabled", "true") \
    .config("spark.sql.adaptive.coalescePartitions.enabled", "true") \
    .getOrCreate()

# Get SparkContext from SparkSession
sc = spark.sparkContext

# Stop session
spark.stop()
```

---

## RDDs (Resilient Distributed Datasets)

### Creating RDDs
```python
from pyspark import SparkContext

sc = SparkContext("local[*]", "RDD Example")

# From collection
data = [1, 2, 3, 4, 5]
rdd = sc.parallelize(data)

# From file
rdd = sc.textFile("path/to/file.txt")

# From multiple files
rdd = sc.textFile("path/to/files/*.txt")

# From HDFS
rdd = sc.textFile("hdfs://namenode:9000/path/to/file")
```

### RDD Transformations
```python
# Map transformation
rdd = sc.parallelize([1, 2, 3, 4, 5])
squared_rdd = rdd.map(lambda x: x * x)
print(squared_rdd.collect())  # [1, 4, 9, 16, 25]

# Filter transformation
even_rdd = rdd.filter(lambda x: x % 2 == 0)
print(even_rdd.collect())  # [2, 4]

# FlatMap transformation
words_rdd = sc.parallelize(["hello world", "spark is great"])
words = words_rdd.flatMap(lambda line: line.split(" "))
print(words.collect())  # ['hello', 'world', 'spark', 'is', 'great']

# Distinct transformation
duplicates = sc.parallelize([1, 2, 2, 3, 3, 3])
unique = duplicates.distinct()
print(unique.collect())  # [1, 2, 3]

# Union transformation
rdd1 = sc.parallelize([1, 2, 3])
rdd2 = sc.parallelize([4, 5, 6])
union_rdd = rdd1.union(rdd2)
print(union_rdd.collect())  # [1, 2, 3, 4, 5, 6]

# Intersection transformation
rdd1 = sc.parallelize([1, 2, 3, 4])
rdd2 = sc.parallelize([3, 4, 5, 6])
intersection_rdd = rdd1.intersection(rdd2)
print(intersection_rdd.collect())  # [3, 4]

# Subtract transformation
subtract_rdd = rdd1.subtract(rdd2)
print(subtract_rdd.collect())  # [1, 2]

# Cartesian transformation
rdd1 = sc.parallelize([1, 2])
rdd2 = sc.parallelize(['a', 'b'])
cartesian_rdd = rdd1.cartesian(rdd2)
print(cartesian_rdd.collect())  # [(1, 'a'), (1, 'b'), (2, 'a'), (2, 'b')]
```

### RDD Actions
```python
# Collect action
rdd = sc.parallelize([1, 2, 3, 4, 5])
print(rdd.collect())  # [1, 2, 3, 4, 5]

# Count action
print(rdd.count())  # 5

# First action
print(rdd.first())  # 1

# Take action
print(rdd.take(3))  # [1, 2, 3]

# Reduce action
sum_result = rdd.reduce(lambda x, y: x + y)
print(sum_result)  # 15

# Fold action
fold_result = rdd.fold(0, lambda x, y: x + y)
print(fold_result)  # 15

# Aggregate action
def seq_op(acc, val):
    return (acc[0] + val, acc[1] + 1)

def comb_op(acc1, acc2):
    return (acc1[0] + acc2[0], acc1[1] + acc2[1])

sum_count = rdd.aggregate((0, 0), seq_op, comb_op)
print(sum_count)  # (15, 5)

# Save actions
rdd.saveAsTextFile("output/path")
rdd.saveAsSequenceFile("output/path")
```

### Key-Value RDDs
```python
# Creating key-value RDDs
kv_rdd = sc.parallelize([(1, "a"), (2, "b"), (3, "c")])

# Group by key
grouped = kv_rdd.groupByKey()
for key, values in grouped.collect():
    print(f"{key}: {list(values)}")

# Reduce by key
data = sc.parallelize([("a", 1), ("b", 1), ("a", 1), ("b", 1)])
reduced = data.reduceByKey(lambda x, y: x + y)
print(reduced.collect())  # [('a', 2), ('b', 2)]

# Sort by key
sorted_rdd = kv_rdd.sortByKey()
print(sorted_rdd.collect())

# Join operations
rdd1 = sc.parallelize([(1, "a"), (2, "b")])
rdd2 = sc.parallelize([(1, "x"), (3, "y")])

# Inner join
inner_join = rdd1.join(rdd2)
print(inner_join.collect())  # [(1, ('a', 'x'))]

# Left outer join
left_join = rdd1.leftOuterJoin(rdd2)
print(left_join.collect())  # [(1, ('a', 'x')), (2, ('b', None))]

# Right outer join
right_join = rdd1.rightOuterJoin(rdd2)
print(right_join.collect())  # [(1, ('a', 'x')), (3, (None, 'y'))]

# Full outer join
full_join = rdd1.fullOuterJoin(rdd2)
print(full_join.collect())  # [(1, ('a', 'x')), (2, ('b', None)), (3, (None, 'y'))]

# Cogroup
cogrouped = rdd1.cogroup(rdd2)
for key, (vals1, vals2) in cogrouped.collect():
    print(f"{key}: {list(vals1)}, {list(vals2)}")
```

---

## DataFrames & Spark SQL

### Creating DataFrames
```python
from pyspark.sql import SparkSession
from pyspark.sql.types import StructType, StructField, StringType, IntegerType, DoubleType

spark = SparkSession.builder.appName("DataFrame Example").getOrCreate()

# From RDD
rdd = spark.sparkContext.parallelize([(1, "Alice", 25), (2, "Bob", 30)])
df = spark.createDataFrame(rdd, ["id", "name", "age"])

# From list of dictionaries
data = [
    {"id": 1, "name": "Alice", "age": 25},
    {"id": 2, "name": "Bob", "age": 30}
]
df = spark.createDataFrame(data)

# From CSV file
df = spark.read.csv("path/to/file.csv", header=True, inferSchema=True)

# From JSON file
df = spark.read.json("path/to/file.json")

# From Parquet file
df = spark.read.parquet("path/to/file.parquet")

# With explicit schema
schema = StructType([
    StructField("id", IntegerType(), True),
    StructField("name", StringType(), True),
    StructField("age", IntegerType(), True)
])
df = spark.read.schema(schema).csv("path/to/file.csv", header=True)
```

### DataFrame Operations
```python
# Basic operations
df.show()
df.printSchema()
df.describe().show()
df.count()
df.columns
df.dtypes

# Select columns
df.select("name", "age").show()
df.select(df.name, df.age).show()
df.select(df["name"], df["age"]).show()

# Filter data
df.filter(df.age > 25).show()
df.filter("age > 25").show()
df.where(df.age > 25).show()

# Add columns
df.withColumn("age_plus_one", df.age + 1).show()
df.withColumn("is_adult", df.age >= 18).show()

# Rename columns
df.withColumnRenamed("name", "full_name").show()

# Drop columns
df.drop("age").show()

# Order by
df.orderBy("age").show()
df.orderBy(df.age.desc()).show()

# Group by and aggregate
df.groupBy("age").count().show()
df.groupBy("age").agg({"id": "count", "age": "avg"}).show()

# Distinct
df.select("age").distinct().show()
df.distinct().show()
```

### SQL Queries
```python
# Register DataFrame as temporary view
df.createOrReplaceTempView("people")

# SQL queries
spark.sql("SELECT * FROM people WHERE age > 25").show()
spark.sql("SELECT age, COUNT(*) as count FROM people GROUP BY age").show()
spark.sql("SELECT name, age FROM people ORDER BY age DESC").show()

# Complex queries
spark.sql("""
    SELECT 
        age,
        COUNT(*) as count,
        AVG(age) as avg_age
    FROM people 
    WHERE age > 20 
    GROUP BY age 
    HAVING COUNT(*) > 1
    ORDER BY age
""").show()
```

### Joins
```python
# Create sample DataFrames
df1 = spark.createDataFrame([(1, "Alice"), (2, "Bob")], ["id", "name"])
df2 = spark.createDataFrame([(1, 25), (2, 30), (3, 35)], ["id", "age"])

# Inner join
inner_join = df1.join(df2, "id")
inner_join.show()

# Left join
left_join = df1.join(df2, "id", "left")
left_join.show()

# Right join
right_join = df1.join(df2, "id", "right")
right_join.show()

# Full outer join
full_join = df1.join(df2, "id", "outer")
full_join.show()

# Cross join
cross_join = df1.crossJoin(df2)
cross_join.show()

# Join with different column names
df3 = spark.createDataFrame([(1, "Engineer"), (2, "Manager")], ["user_id", "role"])
join_different_cols = df1.join(df3, df1.id == df3.user_id)
join_different_cols.show()
```

---

## Spark Streaming

### DStreams (Legacy)
```python
from pyspark import SparkContext
from pyspark.streaming import StreamingContext

# Create streaming context
sc = SparkContext("local[*]", "Streaming Example")
ssc = StreamingContext(sc, 1)  # 1 second batch interval

# Create DStream from socket
lines = ssc.socketTextStream("localhost", 9999)

# Process stream
words = lines.flatMap(lambda line: line.split(" "))
pairs = words.map(lambda word: (word, 1))
word_counts = pairs.reduceByKey(lambda x, y: x + y)

# Output
word_counts.pprint()

# Start streaming
ssc.start()
ssc.awaitTermination()
```

### Structured Streaming (Modern)
```python
from pyspark.sql import SparkSession
from pyspark.sql.functions import *

spark = SparkSession.builder.appName("Structured Streaming").getOrCreate()

# Read from socket
lines = spark \
    .readStream \
    .format("socket") \
    .option("host", "localhost") \
    .option("port", 9999) \
    .load()

# Process stream
words = lines.select(explode(split(lines.value, " ")).alias("word"))
word_counts = words.groupBy("word").count()

# Write to console
query = word_counts \
    .writeStream \
    .outputMode("complete") \
    .format("console") \
    .start()

query.awaitTermination()
```

### Kafka Integration
```python
# Read from Kafka
df = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "topic_name") \
    .load()

# Process data
processed_df = df.select(
    col("key").cast("string"),
    col("value").cast("string"),
    col("timestamp")
)

# Write to Kafka
query = processed_df \
    .writeStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("topic", "output_topic") \
    .option("checkpointLocation", "/path/to/checkpoint") \
    .start()
```

---

## Machine Learning with MLlib

### Data Preparation
```python
from pyspark.ml.feature import VectorAssembler, StringIndexer, OneHotEncoder
from pyspark.ml import Pipeline

# Sample data
data = spark.createDataFrame([
    (1, "A", 25, 50000),
    (2, "B", 30, 60000),
    (3, "A", 35, 70000),
    (4, "C", 28, 55000)
], ["id", "category", "age", "salary"])

# String indexing
indexer = StringIndexer(inputCol="category", outputCol="categoryIndex")

# One-hot encoding
encoder = OneHotEncoder(inputCol="categoryIndex", outputCol="categoryVec")

# Vector assembler
assembler = VectorAssembler(
    inputCols=["age", "categoryVec"],
    outputCol="features"
)

# Pipeline
pipeline = Pipeline(stages=[indexer, encoder, assembler])
model = pipeline.fit(data)
transformed_data = model.transform(data)
```

### Linear Regression
```python
from pyspark.ml.regression import LinearRegression
from pyspark.ml.evaluation import RegressionEvaluator

# Prepare data
lr_data = transformed_data.select("features", "salary")

# Split data
train_data, test_data = lr_data.randomSplit([0.7, 0.3])

# Create model
lr = LinearRegression(featuresCol="features", labelCol="salary")

# Train model
lr_model = lr.fit(train_data)

# Make predictions
predictions = lr_model.transform(test_data)

# Evaluate
evaluator = RegressionEvaluator(labelCol="salary", predictionCol="prediction")
rmse = evaluator.evaluate(predictions)
print(f"RMSE: {rmse}")
```

### Classification
```python
from pyspark.ml.classification import LogisticRegression, RandomForestClassifier
from pyspark.ml.evaluation import MulticlassClassificationEvaluator

# Create binary classification data
classification_data = transformed_data.withColumn(
    "high_salary", 
    when(col("salary") > 60000, 1).otherwise(0)
).select("features", "high_salary")

# Split data
train_data, test_data = classification_data.randomSplit([0.7, 0.3])

# Logistic Regression
lr = LogisticRegression(featuresCol="features", labelCol="high_salary")
lr_model = lr.fit(train_data)
lr_predictions = lr_model.transform(test_data)

# Random Forest
rf = RandomForestClassifier(featuresCol="features", labelCol="high_salary")
rf_model = rf.fit(train_data)
rf_predictions = rf_model.transform(test_data)

# Evaluate
evaluator = MulticlassClassificationEvaluator(labelCol="high_salary", predictionCol="prediction")
accuracy = evaluator.evaluate(lr_predictions)
print(f"Accuracy: {accuracy}")
```

### Clustering
```python
from pyspark.ml.clustering import KMeans
from pyspark.ml.evaluation import ClusteringEvaluator

# Prepare data
clustering_data = transformed_data.select("features")

# K-means clustering
kmeans = KMeans(featuresCol="features", k=3)
kmeans_model = kmeans.fit(clustering_data)
predictions = kmeans_model.transform(clustering_data)

# Evaluate
evaluator = ClusteringEvaluator()
silhouette = evaluator.evaluate(predictions)
print(f"Silhouette score: {silhouette}")
```

---

## Performance Optimization

### Caching and Persistence
```python
# Cache DataFrame
df.cache()
df.persist()

# Different storage levels
from pyspark import StorageLevel

df.persist(StorageLevel.MEMORY_ONLY)
df.persist(StorageLevel.MEMORY_AND_DISK)
df.persist(StorageLevel.DISK_ONLY)

# Unpersist
df.unpersist()
```

### Partitioning
```python
# Repartition
df.repartition(10)

# Coalesce
df.coalesce(5)

# Partition by column
df.write.partitionBy("year", "month").parquet("output/path")

# Bucketing
df.write.bucketBy(10, "user_id").saveAsTable("bucketed_table")
```

### Configuration
```python
# Spark configuration
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.sql.adaptive.skewJoin.enabled", "true")
spark.conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")

# Memory configuration
spark.conf.set("spark.executor.memory", "4g")
spark.conf.set("spark.driver.memory", "2g")
spark.conf.set("spark.executor.memoryFraction", "0.8")
```

---

## Deployment

### Local Mode
```bash
# Start Spark in local mode
./bin/spark-submit --master local[*] --class MyClass myapp.jar

# With specific configuration
./bin/spark-submit \
  --master local[4] \
  --executor-memory 2g \
  --driver-memory 1g \
  --conf spark.sql.adaptive.enabled=true \
  myapp.py
```

### Standalone Cluster
```bash
# Start master
./sbin/start-master.sh

# Start worker
./sbin/start-worker.sh spark://master-host:7077

# Submit job
./bin/spark-submit \
  --master spark://master-host:7077 \
  --executor-memory 2g \
  --total-executor-cores 4 \
  myapp.py
```

### YARN
```bash
# Submit to YARN
./bin/spark-submit \
  --master yarn \
  --deploy-mode cluster \
  --executor-memory 2g \
  --num-executors 4 \
  --executor-cores 2 \
  myapp.py
```

### Kubernetes
```yaml
# spark-pi.yaml
apiVersion: v1
kind: Pod
metadata:
  name: spark-pi
spec:
  containers:
  - name: spark
    image: apache/spark:3.5.0
    command: ["/opt/spark/bin/spark-submit"]
    args:
      - "--master"
      - "k8s://https://kubernetes.default.svc:443"
      - "--deploy-mode"
      - "cluster"
      - "--conf"
      - "spark.kubernetes.container.image=apache/spark:3.5.0"
      - "--class"
      - "org.apache.spark.examples.SparkPi"
      - "local:///opt/spark/examples/jars/spark-examples_2.12-3.5.0.jar"
      - "10"
```

---

## Monitoring & Debugging

### Spark UI
```python
# Access Spark UI
# http://localhost:4040 (local mode)
# http://driver-host:4040 (cluster mode)

# Enable history server
./sbin/start-history-server.sh
```

### Logging
```python
import logging

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

# Log in Spark application
logger.info("Starting Spark application")
```

### Performance Monitoring
```python
# Get Spark context
sc = spark.sparkContext

# Get application ID
app_id = sc.applicationId
print(f"Application ID: {app_id}")

# Get application name
app_name = sc.appName
print(f"Application Name: {app_name}")

# Get default parallelism
default_parallelism = sc.defaultParallelism
print(f"Default Parallelism: {default_parallelism}")
```

---

## Best Practices

### 1. Data Processing
- **Use DataFrames over RDDs**: Better optimization and SQL support
- **Avoid collect()**: Use take() or show() for small results
- **Cache frequently used DataFrames**: Improve performance
- **Use appropriate file formats**: Parquet for analytics, JSON for flexibility

### 2. Performance
- **Optimize partitioning**: Avoid too many small files
- **Use broadcast variables**: For small lookup tables
- **Tune memory settings**: Based on your data size
- **Use adaptive query execution**: Enable AQE for better performance

### 3. Development
- **Use local mode for development**: Faster iteration
- **Test with small datasets**: Before running on large data
- **Use unit tests**: Test individual transformations
- **Monitor resource usage**: Use Spark UI and logs

### 4. Production
- **Use cluster mode**: For production workloads
- **Configure proper resource allocation**: Based on cluster capacity
- **Implement proper error handling**: Handle failures gracefully
- **Use checkpointing**: For streaming applications

---

## Common Use Cases

### ETL Pipeline
```python
# Extract
raw_data = spark.read.json("s3://bucket/raw-data/*.json")

# Transform
processed_data = raw_data \
    .filter(col("status") == "active") \
    .withColumn("processed_date", current_timestamp()) \
    .select("id", "name", "processed_date")

# Load
processed_data.write \
    .mode("overwrite") \
    .parquet("s3://bucket/processed-data/")
```

### Real-time Analytics
```python
# Stream processing
streaming_df = spark \
    .readStream \
    .format("kafka") \
    .option("kafka.bootstrap.servers", "localhost:9092") \
    .option("subscribe", "events") \
    .load()

# Real-time aggregations
aggregated = streaming_df \
    .groupBy(window(col("timestamp"), "1 minute")) \
    .agg(count("*").alias("event_count"))

# Write to sink
query = aggregated \
    .writeStream \
    .outputMode("complete") \
    .format("console") \
    .start()
```

### Machine Learning Pipeline
```python
# Feature engineering
feature_pipeline = Pipeline(stages=[
    StringIndexer(inputCol="category", outputCol="categoryIndex"),
    OneHotEncoder(inputCol="categoryIndex", outputCol="categoryVec"),
    VectorAssembler(inputCols=["feature1", "feature2", "categoryVec"], outputCol="features")
])

# Model training
model_pipeline = Pipeline(stages=[
    feature_pipeline,
    LogisticRegression(featuresCol="features", labelCol="label")
])

# Train and predict
model = model_pipeline.fit(train_data)
predictions = model.transform(test_data)
```

Apache Spark is a powerful tool for big data processing. Start with simple RDD operations, move to DataFrames for better performance, and explore streaming and MLlib for advanced use cases. Always consider performance optimization and proper resource management for production deployments.
