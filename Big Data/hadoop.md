# Hadoop - Distributed Computing Framework

Hadoop is an open-source framework for distributed storage and processing of large datasets across clusters of computers. It's designed to scale from single servers to thousands of machines, each offering local computation and storage.

---

## What is Hadoop?

Hadoop is a:
- **Distributed Storage System**: HDFS (Hadoop Distributed File System)
- **Processing Framework**: MapReduce for batch processing
- **Ecosystem**: Collection of tools and libraries for big data
- **Scalable**: Handles petabytes of data across thousands of nodes
- **Fault Tolerant**: Automatically handles node failures
- **Cost Effective**: Runs on commodity hardware

### Core Components
- **HDFS**: Distributed file system for storing large datasets
- **MapReduce**: Programming model for processing large datasets
- **YARN**: Resource management and job scheduling
- **Hadoop Common**: Common utilities and libraries

---

## Hadoop Architecture

### HDFS Architecture
```
NameNode (Master)
    ↓
DataNodes (Slaves)
    ↓
Blocks (128MB default)
```

### MapReduce Architecture
```
Client Application
    ↓
JobTracker (Master)
    ↓
TaskTrackers (Slaves)
    ↓
Map Tasks → Reduce Tasks
```

### YARN Architecture
```
ResourceManager (Master)
    ↓
NodeManagers (Slaves)
    ↓
ApplicationMasters
    ↓
Containers
```

---

## Installation & Setup

### Single Node Setup
```bash
# Download Hadoop
wget https://archive.apache.org/dist/hadoop/common/hadoop-3.3.4/hadoop-3.3.4.tar.gz
tar -xzf hadoop-3.3.4.tar.gz
cd hadoop-3.3.4

# Set environment variables
export HADOOP_HOME=/path/to/hadoop-3.3.4
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin

# Configure Java
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

### Configuration Files
```xml
<!-- core-site.xml -->
<configuration>
    <property>
        <name>fs.defaultFS</name>
        <value>hdfs://localhost:9000</value>
    </property>
</configuration>

<!-- hdfs-site.xml -->
<configuration>
    <property>
        <name>dfs.replication</name>
        <value>1</value>
    </property>
    <property>
        <name>dfs.namenode.name.dir</name>
        <value>/path/to/hadoop/data/namenode</value>
    </property>
    <property>
        <name>dfs.datanode.data.dir</name>
        <value>/path/to/hadoop/data/datanode</value>
    </property>
</configuration>

<!-- mapred-site.xml -->
<configuration>
    <property>
        <name>mapreduce.framework.name</name>
        <value>yarn</value>
    </property>
</configuration>

<!-- yarn-site.xml -->
<configuration>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
</configuration>
```

### Starting Hadoop
```bash
# Format HDFS (first time only)
hdfs namenode -format

# Start HDFS
start-dfs.sh

# Start YARN
start-yarn.sh

# Check status
jps
```

---

## HDFS (Hadoop Distributed File System)

### HDFS Commands
```bash
# List files
hdfs dfs -ls /
hdfs dfs -ls /user

# Create directory
hdfs dfs -mkdir /data
hdfs dfs -mkdir -p /user/hadoop/data

# Copy files
hdfs dfs -put localfile.txt /data/
hdfs dfs -copyFromLocal localfile.txt /data/
hdfs dfs -get /data/remotefile.txt ./
hdfs dfs -copyToLocal /data/remotefile.txt ./

# View file content
hdfs dfs -cat /data/file.txt
hdfs dfs -tail /data/file.txt
hdfs dfs -head /data/file.txt

# File operations
hdfs dfs -chmod 755 /data/file.txt
hdfs dfs -chown hadoop:hadoop /data/file.txt
hdfs dfs -mv /data/oldfile.txt /data/newfile.txt
hdfs dfs -rm /data/file.txt
hdfs dfs -rmr /data/olddir

# Check disk usage
hdfs dfs -du /data
hdfs dfs -df -h
```

### HDFS Web UI
- **NameNode UI**: http://localhost:9870
- **DataNode UI**: http://localhost:9864

### HDFS Java API
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.FileSystem;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.fs.FSDataInputStream;
import org.apache.hadoop.fs.FSDataOutputStream;

public class HDFSExample {
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        FileSystem fs = FileSystem.get(conf);
        
        // Create file
        Path filePath = new Path("/data/example.txt");
        FSDataOutputStream out = fs.create(filePath);
        out.writeUTF("Hello HDFS!");
        out.close();
        
        // Read file
        FSDataInputStream in = fs.open(filePath);
        String content = in.readUTF();
        System.out.println(content);
        in.close();
        
        // List files
        Path dirPath = new Path("/data");
        FileStatus[] files = fs.listStatus(dirPath);
        for (FileStatus file : files) {
            System.out.println(file.getPath().getName());
        }
        
        fs.close();
    }
}
```

---

## MapReduce

### MapReduce Programming Model
```java
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.fs.Path;
import org.apache.hadoop.io.IntWritable;
import org.apache.hadoop.io.LongWritable;
import org.apache.hadoop.io.Text;
import org.apache.hadoop.mapreduce.Job;
import org.apache.hadoop.mapreduce.Mapper;
import org.apache.hadoop.mapreduce.Reducer;
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;

public class WordCount {
    
    // Mapper class
    public static class TokenizerMapper
            extends Mapper<LongWritable, Text, Text, IntWritable> {
        
        private final static IntWritable one = new IntWritable(1);
        private Text word = new Text();
        
        public void map(LongWritable key, Text value, Context context)
                throws IOException, InterruptedException {
            
            StringTokenizer itr = new StringTokenizer(value.toString());
            while (itr.hasMoreTokens()) {
                word.set(itr.nextToken());
                context.write(word, one);
            }
        }
    }
    
    // Reducer class
    public static class IntSumReducer
            extends Reducer<Text, IntWritable, Text, IntWritable> {
        
        private IntWritable result = new IntWritable();
        
        public void reduce(Text key, Iterable<IntWritable> values,
                          Context context) throws IOException, InterruptedException {
            
            int sum = 0;
            for (IntWritable val : values) {
                sum += val.get();
            }
            result.set(sum);
            context.write(key, result);
        }
    }
    
    public static void main(String[] args) throws Exception {
        Configuration conf = new Configuration();
        Job job = Job.getInstance(conf, "word count");
        
        job.setJarByClass(WordCount.class);
        job.setMapperClass(TokenizerMapper.class);
        job.setCombinerClass(IntSumReducer.class);
        job.setReducerClass(IntSumReducer.class);
        
        job.setOutputKeyClass(Text.class);
        job.setOutputValueClass(IntWritable.class);
        
        FileInputFormat.addInputPath(job, new Path(args[0]));
        FileOutputFormat.setOutputPath(job, new Path(args[1]));
        
        System.exit(job.waitForCompletion(true) ? 0 : 1);
    }
}
```

### Running MapReduce Jobs
```bash
# Compile and package
javac -classpath $HADOOP_CLASSPATH WordCount.java
jar cf wc.jar WordCount*.class

# Run job
hadoop jar wc.jar WordCount /input /output

# Check job status
hadoop job -list
hadoop job -status job_id

# View results
hdfs dfs -cat /output/part-r-00000
```

### MapReduce with Python (Hadoop Streaming)
```python
# mapper.py
import sys

for line in sys.stdin:
    words = line.strip().split()
    for word in words:
        print(f"{word}\t1")

# reducer.py
import sys

current_word = None
current_count = 0

for line in sys.stdin:
    word, count = line.strip().split('\t')
    count = int(count)
    
    if current_word == word:
        current_count += count
    else:
        if current_word:
            print(f"{current_word}\t{current_count}")
        current_word = word
        current_count = count

if current_word:
    print(f"{current_word}\t{current_count}")
```

```bash
# Run with Hadoop Streaming
hadoop jar $HADOOP_HOME/share/hadoop/tools/lib/hadoop-streaming-*.jar \
    -files mapper.py,reducer.py \
    -mapper mapper.py \
    -reducer reducer.py \
    -input /input \
    -output /output
```

---

## YARN (Yet Another Resource Negotiator)

### YARN Components
- **ResourceManager**: Manages cluster resources
- **NodeManager**: Manages resources on individual nodes
- **ApplicationMaster**: Manages application lifecycle
- **Container**: Resource allocation unit

### YARN Commands
```bash
# List applications
yarn application -list
yarn application -list -appStates RUNNING

# Application details
yarn application -status application_id

# Kill application
yarn application -kill application_id

# Node information
yarn node -list
yarn node -status node_id

# Queue information
yarn queue -status default
```

### YARN Web UI
- **ResourceManager UI**: http://localhost:8088

### YARN Configuration
```xml
<!-- yarn-site.xml -->
<configuration>
    <property>
        <name>yarn.resourcemanager.hostname</name>
        <value>localhost</value>
    </property>
    <property>
        <name>yarn.nodemanager.aux-services</name>
        <value>mapreduce_shuffle</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>4096</value>
    </property>
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>4</value>
    </property>
</configuration>
```

---

## Hadoop Ecosystem

### Hive (Data Warehouse)
```sql
-- Create table
CREATE TABLE users (
    id INT,
    name STRING,
    email STRING
) ROW FORMAT DELIMITED
FIELDS TERMINATED BY ','
STORED AS TEXTFILE;

-- Load data
LOAD DATA INPATH '/data/users.csv' INTO TABLE users;

-- Query data
SELECT name, email FROM users WHERE id > 100;
SELECT COUNT(*) FROM users;
```

### Pig (Data Flow Language)
```pig
-- Load data
users = LOAD '/data/users.csv' USING PigStorage(',') AS (id:int, name:chararray, email:chararray);

-- Filter data
filtered_users = FILTER users BY id > 100;

-- Group data
grouped_users = GROUP filtered_users BY name;

-- Count
user_count = FOREACH grouped_users GENERATE group, COUNT(filtered_users);

-- Store results
STORE user_count INTO '/output/user_count';
```

### HBase (NoSQL Database)
```java
import org.apache.hadoop.hbase.HBaseConfiguration;
import org.apache.hadoop.hbase.client.*;
import org.apache.hadoop.hbase.util.Bytes;

public class HBaseExample {
    public static void main(String[] args) throws Exception {
        Configuration config = HBaseConfiguration.create();
        Connection connection = ConnectionFactory.createConnection(config);
        
        // Create table
        Admin admin = connection.getAdmin();
        TableName tableName = TableName.valueOf("users");
        HTableDescriptor table = new HTableDescriptor(tableName);
        table.addFamily(new HColumnDescriptor("info"));
        admin.createTable(table);
        
        // Insert data
        Table table = connection.getTable(tableName);
        Put put = new Put(Bytes.toBytes("user1"));
        put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("name"), Bytes.toBytes("John"));
        put.addColumn(Bytes.toBytes("info"), Bytes.toBytes("email"), Bytes.toBytes("john@example.com"));
        table.put(put);
        
        // Get data
        Get get = new Get(Bytes.toBytes("user1"));
        Result result = table.get(get);
        byte[] name = result.getValue(Bytes.toBytes("info"), Bytes.toBytes("name"));
        System.out.println("Name: " + Bytes.toString(name));
        
        table.close();
        connection.close();
    }
}
```

### Sqoop (Data Transfer)
```bash
# Import from MySQL to HDFS
sqoop import \
    --connect jdbc:mysql://localhost/database \
    --username user \
    --password password \
    --table users \
    --target-dir /data/users \
    --m 1

# Export from HDFS to MySQL
sqoop export \
    --connect jdbc:mysql://localhost/database \
    --username user \
    --password password \
    --table users \
    --export-dir /data/users \
    --m 1
```

### Flume (Data Ingestion)
```properties
# flume.conf
agent.sources = r1
agent.sinks = k1
agent.channels = c1

agent.sources.r1.type = spooldir
agent.sources.r1.spoolDir = /path/to/spool
agent.sources.r1.channels = c1

agent.sinks.k1.type = hdfs
agent.sinks.k1.hdfs.path = hdfs://localhost:9000/data/flume
agent.sinks.k1.hdfs.fileType = DataStream
agent.sinks.k1.channel = c1

agent.channels.c1.type = memory
agent.channels.c1.capacity = 10000
agent.channels.c1.transactionCapacity = 1000
```

```bash
# Run Flume
flume-ng agent --conf conf --conf-file flume.conf --name agent
```

---

## Monitoring & Management

### Hadoop Metrics
```bash
# HDFS metrics
hdfs dfsadmin -report
hdfs dfsadmin -safemode get

# YARN metrics
yarn node -list
yarn application -list

# Cluster health
hdfs dfsadmin -report | grep "Live datanodes"
```

### Log Files
```bash
# HDFS logs
$HADOOP_HOME/logs/hadoop-hadoop-namenode-*.log
$HADOOP_HOME/logs/hadoop-hadoop-datanode-*.log

# YARN logs
$HADOOP_HOME/logs/yarn-hadoop-resourcemanager-*.log
$HADOOP_HOME/logs/yarn-hadoop-nodemanager-*.log

# Application logs
yarn logs -applicationId application_id
```

### Web UIs
- **HDFS NameNode**: http://localhost:9870
- **HDFS DataNode**: http://localhost:9864
- **YARN ResourceManager**: http://localhost:8088
- **YARN NodeManager**: http://localhost:8042

---

## Performance Tuning

### HDFS Tuning
```xml
<!-- hdfs-site.xml -->
<configuration>
    <!-- Block size -->
    <property>
        <name>dfs.blocksize</name>
        <value>268435456</value> <!-- 256MB -->
    </property>
    
    <!-- Replication factor -->
    <property>
        <name>dfs.replication</name>
        <value>3</value>
    </property>
    
    <!-- NameNode heap size -->
    <property>
        <name>dfs.namenode.heapsize</name>
        <value>4096m</value>
    </property>
</configuration>
```

### MapReduce Tuning
```xml
<!-- mapred-site.xml -->
<configuration>
    <!-- Map tasks -->
    <property>
        <name>mapreduce.map.memory.mb</name>
        <value>2048</value>
    </property>
    
    <!-- Reduce tasks -->
    <property>
        <name>mapreduce.reduce.memory.mb</name>
        <value>4096</value>
    </property>
    
    <!-- JVM options -->
    <property>
        <name>mapreduce.map.java.opts</name>
        <value>-Xmx1638m</value>
    </property>
</configuration>
```

### YARN Tuning
```xml
<!-- yarn-site.xml -->
<configuration>
    <!-- NodeManager memory -->
    <property>
        <name>yarn.nodemanager.resource.memory-mb</name>
        <value>8192</value>
    </property>
    
    <!-- NodeManager CPU -->
    <property>
        <name>yarn.nodemanager.resource.cpu-vcores</name>
        <value>8</value>
    </property>
    
    <!-- Scheduler -->
    <property>
        <name>yarn.resourcemanager.scheduler.class</name>
        <value>org.apache.hadoop.yarn.server.resourcemanager.scheduler.capacity.CapacityScheduler</value>
    </property>
</configuration>
```

---

## Security

### HDFS Security
```xml
<!-- hdfs-site.xml -->
<configuration>
    <!-- Enable security -->
    <property>
        <name>hadoop.security.authentication</name>
        <value>kerberos</value>
    </property>
    
    <!-- Authorization -->
    <property>
        <name>hadoop.security.authorization</name>
        <value>true</value>
    </property>
</configuration>
```

### Access Control Lists (ACLs)
```bash
# Set ACL
hdfs dfs -setfacl -m user:alice:rwx /data
hdfs dfs -setfacl -m group:developers:r-x /data

# View ACL
hdfs dfs -getfacl /data

# Remove ACL
hdfs dfs -setfacl -x user:alice /data
```

---

## Backup & Recovery

### HDFS Backup
```bash
# Create snapshot
hdfs dfsadmin -allowSnapshot /data
hdfs dfs -createSnapshot /data backup1

# List snapshots
hdfs dfs -ls /data/.snapshot

# Restore from snapshot
hdfs dfs -cp /data/.snapshot/backup1/file.txt /data/
```

### NameNode Backup
```bash
# Save namespace
hdfs dfsadmin -saveNamespace

# Checkpoint
hdfs dfsadmin -rollEdits
```

---

## Best Practices

### 1. Cluster Design
- **Hardware**: Use commodity hardware with good network
- **Network**: Ensure high bandwidth between nodes
- **Storage**: Use multiple disks per node for better I/O
- **Memory**: Allocate sufficient memory for NameNode and ResourceManager

### 2. Data Management
- **File sizes**: Use larger files (100MB+) for better performance
- **Compression**: Use compression for storage efficiency
- **Partitioning**: Partition data appropriately for queries
- **Replication**: Set appropriate replication factor

### 3. Performance
- **Tuning**: Tune JVM and Hadoop parameters
- **Monitoring**: Monitor cluster health and performance
- **Balancing**: Keep data balanced across nodes
- **Caching**: Use HDFS caching for frequently accessed data

### 4. Security
- **Authentication**: Use Kerberos for authentication
- **Authorization**: Implement proper access controls
- **Encryption**: Use encryption for data at rest and in transit
- **Auditing**: Enable audit logging

---

## Common Use Cases

### Log Processing
```java
// Process web server logs
public class LogProcessor {
    public static class LogMapper extends Mapper<LongWritable, Text, Text, IntWritable> {
        public void map(LongWritable key, Text value, Context context) {
            String[] fields = value.toString().split(" ");
            String ip = fields[0];
            context.write(new Text(ip), new IntWritable(1));
        }
    }
    
    public static class LogReducer extends Reducer<Text, IntWritable, Text, IntWritable> {
        public void reduce(Text key, Iterable<IntWritable> values, Context context) {
            int sum = 0;
            for (IntWritable value : values) {
                sum += value.get();
            }
            context.write(key, new IntWritable(sum));
        }
    }
}
```

### Data Analysis
```sql
-- Analyze user behavior
SELECT 
    user_id,
    COUNT(*) as page_views,
    AVG(session_duration) as avg_duration
FROM user_sessions
WHERE date >= '2024-01-01'
GROUP BY user_id
HAVING COUNT(*) > 10
ORDER BY page_views DESC;
```

### ETL Pipeline
```bash
# Extract data from source
sqoop import --connect jdbc:mysql://source/db --table users --target-dir /data/users

# Transform data
hive -f transform_users.sql

# Load to destination
sqoop export --connect jdbc:postgresql://dest/db --table users --export-dir /data/transformed_users
```

Hadoop provides a robust foundation for big data processing. Start with single-node setup, understand HDFS and MapReduce, then explore the ecosystem tools. Focus on proper configuration, monitoring, and security for production deployments.
