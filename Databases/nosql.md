# NoSQL Databases 🗄️

[![NoSQL](https://img.shields.io/badge/NoSQL-Non%20Relational-orange)](https://en.wikipedia.org/wiki/NoSQL)
[![MongoDB](https://img.shields.io/badge/MongoDB-Document%20DB-green)](https://mongodb.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Non-Relational Databases** - NoSQL databases are non-tabular databases that store data differently than relational tables, providing flexible schemas, horizontal scaling, and high performance for modern applications.

## 🎯 Overview

NoSQL databases are non-relational databases that:
- **Store Flexible Data** - Handle unstructured and semi-structured data
- **Scale Horizontally** - Distribute data across multiple machines
- **Provide High Performance** - Optimized for specific use cases
- **Offer Schema Flexibility** - Dynamic schema evolution
- **Support Big Data** - Handle large volumes of data efficiently
- **Enable Rapid Development** - Faster development cycles

## 🚀 Key Features

### Core Capabilities
- **Schema Flexibility** - No fixed schema requirements
- **Horizontal Scaling** - Scale across multiple machines
- **High Performance** - Optimized for specific workloads
- **Data Variety** - Handle different data types and structures
- **Global Distribution** - Multi-region deployment support
- **Developer Friendly** - Easy to use and integrate

### Advanced Features
- **ACID Transactions** - Some NoSQL databases support ACID properties
- **Real-time Analytics** - Built-in analytics and reporting
- **Machine Learning** - Integration with ML frameworks
- **Graph Processing** - Support for graph data structures
- **Time Series** - Optimized for time-series data
- **Search Integration** - Built-in full-text search capabilities

## 🏗️ Architecture

### NoSQL Database Types
```
NoSQL Database Types
├── 📄 Document Databases
│   ├── MongoDB
│   ├── CouchDB
│   ├── Amazon DocumentDB
│   └── Azure Cosmos DB
├── 🗂️ Key-Value Stores
│   ├── Redis
│   ├── DynamoDB
│   ├── Riak
│   └── Memcached
├── 📊 Column-Family Stores
│   ├── Cassandra
│   ├── HBase
│   ├── ScyllaDB
│   └── Amazon Keyspaces
└── 🌐 Graph Databases
    ├── Neo4j
    ├── Amazon Neptune
    ├── ArangoDB
    └── OrientDB
```

## 🛠️ Document Databases

### MongoDB
```bash
# Install MongoDB
# Ubuntu/Debian
sudo apt update
sudo apt install mongodb

# macOS
brew install mongodb-community

# Start MongoDB
sudo systemctl start mongod
```

```python
from pymongo import MongoClient
from bson import ObjectId

# Connect to MongoDB
client = MongoClient('mongodb://localhost:27017/')
db = client['myapp']
collection = db['users']

# Insert document
user = {
    'name': 'Alice',
    'email': 'alice@example.com',
    'age': 30,
    'address': {
        'street': '123 Main St',
        'city': 'New York',
        'zip': '10001'
    },
    'hobbies': ['reading', 'swimming', 'coding']
}
result = collection.insert_one(user)
print(f"Inserted user with ID: {result.inserted_id}")

# Find documents
users = collection.find({'age': {'$gte': 25}})
for user in users:
    print(user)

# Update document
collection.update_one(
    {'name': 'Alice'},
    {'$set': {'age': 31}}
)

# Delete document
collection.delete_one({'name': 'Alice'})

# Aggregation pipeline
pipeline = [
    {'$match': {'age': {'$gte': 25}}},
    {'$group': {'_id': '$city', 'count': {'$sum': 1}}},
    {'$sort': {'count': -1}}
]
results = collection.aggregate(pipeline)
for result in results:
    print(result)
```

### CouchDB
```python
import couchdb

# Connect to CouchDB
server = couchdb.Server('http://localhost:5984/')
db = server['myapp']

# Create document
doc = {
    'name': 'Alice',
    'email': 'alice@example.com',
    'age': 30
}
doc_id, doc_rev = db.save(doc)
print(f"Saved document with ID: {doc_id}")

# Retrieve document
doc = db[doc_id]
print(doc)

# Update document
doc['age'] = 31
db.save(doc)

# Delete document
db.delete(doc)
```

## 🗂️ Key-Value Stores

### Redis
```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

# Basic operations
r.set('name', 'Alice')
name = r.get('name').decode('utf-8')
print(name)

# Hash operations
r.hset('user:1', mapping={
    'name': 'Alice',
    'email': 'alice@example.com',
    'age': '30'
})
user_data = r.hgetall('user:1')
print(user_data)

# List operations
r.lpush('tasks', 'task1', 'task2', 'task3')
tasks = r.lrange('tasks', 0, -1)
print(tasks)

# Set operations
r.sadd('tags', 'python', 'redis', 'database')
tags = r.smembers('tags')
print(tags)
```

### Amazon DynamoDB
```python
import boto3
from boto3.dynamodb.conditions import Key, Attr

# Create DynamoDB client
dynamodb = boto3.resource('dynamodb', region_name='us-east-1')
table = dynamodb.Table('Users')

# Put item
table.put_item(
    Item={
        'user_id': '1',
        'name': 'Alice',
        'email': 'alice@example.com',
        'age': 30
    }
)

# Get item
response = table.get_item(Key={'user_id': '1'})
item = response['Item']
print(item)

# Query items
response = table.query(
    KeyConditionExpression=Key('user_id').eq('1')
)
items = response['Items']
print(items)

# Scan items
response = table.scan(
    FilterExpression=Attr('age').gte(25)
)
items = response['Items']
print(items)
```

## 📊 Column-Family Stores

### Apache Cassandra
```python
from cassandra.cluster import Cluster
from cassandra.auth import PlainTextAuthProvider

# Connect to Cassandra
cluster = Cluster(['localhost'])
session = cluster.connect()

# Create keyspace
session.execute("""
    CREATE KEYSPACE IF NOT EXISTS myapp
    WITH REPLICATION = {
        'class': 'SimpleStrategy',
        'replication_factor': 1
    }
""")

# Use keyspace
session.set_keyspace('myapp')

# Create table
session.execute("""
    CREATE TABLE IF NOT EXISTS users (
        user_id UUID PRIMARY KEY,
        name TEXT,
        email TEXT,
        age INT
    )
""")

# Insert data
from cassandra.util import uuid_from_time
user_id = uuid_from_time(time.time())
session.execute("""
    INSERT INTO users (user_id, name, email, age)
    VALUES (%s, %s, %s, %s)
""", (user_id, 'Alice', 'alice@example.com', 30))

# Query data
rows = session.execute("SELECT * FROM users WHERE user_id = %s", (user_id,))
for row in rows:
    print(row)
```

### HBase
```python
from happybase import Connection

# Connect to HBase
connection = Connection('localhost')
table = connection.table('users')

# Insert data
table.put(b'user1', {
    b'info:name': b'Alice',
    b'info:email': b'alice@example.com',
    b'info:age': b'30'
})

# Get data
row = table.row(b'user1')
print(row)

# Scan data
for key, data in table.scan():
    print(key, data)
```

## 🌐 Graph Databases

### Neo4j
```python
from neo4j import GraphDatabase

# Connect to Neo4j
driver = GraphDatabase.driver("bolt://localhost:7687", auth=("neo4j", "password"))

# Create nodes and relationships
def create_user(tx, name, email):
    tx.run("CREATE (u:User {name: $name, email: $email})", name=name, email=email)

def create_relationship(tx, user1, user2, relationship):
    tx.run("""
        MATCH (u1:User {name: $user1})
        MATCH (u2:User {name: $user2})
        CREATE (u1)-[r:KNOWS]->(u2)
        SET r.type = $relationship
    """, user1=user1, user2=user2, relationship=relationship)

# Execute operations
with driver.session() as session:
    session.write_transaction(create_user, "Alice", "alice@example.com")
    session.write_transaction(create_user, "Bob", "bob@example.com")
    session.write_transaction(create_relationship, "Alice", "Bob", "friend")

# Query data
def get_friends(tx, name):
    result = tx.run("""
        MATCH (u:User {name: $name})-[:KNOWS]->(friend:User)
        RETURN friend.name, friend.email
    """, name=name)
    return [record for record in result]

with driver.session() as session:
    friends = session.read_transaction(get_friends, "Alice")
    for friend in friends:
        print(friend)

driver.close()
```

## 🚀 Use Cases

### E-commerce Platform
```python
# Product catalog with MongoDB
class ProductCatalog:
    def __init__(self, db):
        self.products = db['products']
        self.categories = db['categories']
    
    def add_product(self, product_data):
        return self.products.insert_one(product_data)
    
    def get_products_by_category(self, category):
        return list(self.products.find({'category': category}))
    
    def search_products(self, query):
        return list(self.products.find({
            '$text': {'$search': query}
        }))
    
    def update_inventory(self, product_id, quantity):
        self.products.update_one(
            {'_id': product_id},
            {'$inc': {'inventory': quantity}}
        )

# User session management with Redis
class SessionManager:
    def __init__(self, redis_client):
        self.redis = redis_client
    
    def create_session(self, user_id, session_data):
        session_id = str(uuid.uuid4())
        self.redis.hset(f"session:{session_id}", mapping=session_data)
        self.redis.expire(f"session:{session_id}", 3600)  # 1 hour
        return session_id
    
    def get_session(self, session_id):
        return self.redis.hgetall(f"session:{session_id}")
    
    def extend_session(self, session_id, seconds=3600):
        self.redis.expire(f"session:{session_id}", seconds)
```

### Social Media Platform
```python
# User relationships with Neo4j
class SocialGraph:
    def __init__(self, driver):
        self.driver = driver
    
    def add_friend(self, user1, user2):
        with self.driver.session() as session:
            session.run("""
                MATCH (u1:User {name: $user1})
                MATCH (u2:User {name: $user2})
                CREATE (u1)-[:FRIENDS]->(u2)
                CREATE (u2)-[:FRIENDS]->(u1)
            """, user1=user1, user2=user2)
    
    def get_friends(self, user):
        with self.driver.session() as session:
            result = session.run("""
                MATCH (u:User {name: $user})-[:FRIENDS]->(friend:User)
                RETURN friend.name
            """, user=user)
            return [record['friend.name'] for record in result]
    
    def get_mutual_friends(self, user1, user2):
        with self.driver.session() as session:
            result = session.run("""
                MATCH (u1:User {name: $user1})-[:FRIENDS]->(mutual:User)<-[:FRIENDS]-(u2:User {name: $user2})
                RETURN mutual.name
            """, user1=user1, user2=user2)
            return [record['mutual.name'] for record in result]

# Activity feed with Cassandra
class ActivityFeed:
    def __init__(self, session):
        self.session = session
    
    def add_activity(self, user_id, activity_type, data):
        activity_id = uuid_from_time(time.time())
        self.session.execute("""
            INSERT INTO activities (user_id, activity_id, activity_type, data, timestamp)
            VALUES (%s, %s, %s, %s, %s)
        """, (user_id, activity_id, activity_type, data, time.time()))
    
    def get_activities(self, user_id, limit=20):
        result = self.session.execute("""
            SELECT * FROM activities
            WHERE user_id = %s
            ORDER BY timestamp DESC
            LIMIT %s
        """, (user_id, limit))
        return [row for row in result]
```

### IoT Data Platform
```python
# Time series data with InfluxDB
from influxdb import InfluxDBClient

class IoTDataStore:
    def __init__(self, host, port, database):
        self.client = InfluxDBClient(host=host, port=port, database=database)
    
    def write_sensor_data(self, device_id, sensor_type, value, timestamp):
        data = [{
            'measurement': 'sensor_data',
            'tags': {
                'device_id': device_id,
                'sensor_type': sensor_type
            },
            'fields': {
                'value': value
            },
            'time': timestamp
        }]
        self.client.write_points(data)
    
    def get_sensor_data(self, device_id, sensor_type, start_time, end_time):
        query = f"""
            SELECT * FROM sensor_data
            WHERE device_id = '{device_id}'
            AND sensor_type = '{sensor_type}'
            AND time >= '{start_time}'
            AND time <= '{end_time}'
        """
        result = self.client.query(query)
        return list(result.get_points())

# Device metadata with MongoDB
class DeviceManager:
    def __init__(self, db):
        self.devices = db['devices']
    
    def register_device(self, device_data):
        return self.devices.insert_one(device_data)
    
    def update_device_status(self, device_id, status):
        self.devices.update_one(
            {'device_id': device_id},
            {'$set': {'status': status, 'last_seen': time.time()}}
        )
    
    def get_devices_by_location(self, location):
        return list(self.devices.find({'location': location}))
```

## 🔧 Best Practices

### Data Modeling
1. **Choose Right Database Type** - Select appropriate NoSQL database for your use case
2. **Design for Queries** - Model data based on how you'll query it
3. **Denormalize When Needed** - Duplicate data for better query performance
4. **Use Appropriate Data Types** - Choose right data types for your data
5. **Plan for Growth** - Design for future data growth and scaling

### Performance Optimization
1. **Index Strategically** - Create indexes on frequently queried fields
2. **Use Connection Pooling** - Reuse database connections
3. **Batch Operations** - Group multiple operations together
4. **Cache Frequently** - Cache frequently accessed data
5. **Monitor Performance** - Track query performance and optimize

### Security
1. **Encrypt Data** - Encrypt sensitive data at rest and in transit
2. **Use Authentication** - Implement proper authentication and authorization
3. **Validate Input** - Validate all input data
4. **Use HTTPS** - Encrypt network communication
5. **Regular Updates** - Keep database software updated

## 🚀 Advanced Features

### Multi-Model Databases
```python
# ArangoDB - Document and Graph database
from arango import ArangoClient

client = ArangoClient(hosts='http://localhost:8529')
db = client.db('myapp')

# Document operations
users = db.collection('users')
user = users.insert({
    'name': 'Alice',
    'email': 'alice@example.com'
})

# Graph operations
graph = db.graph('social')
graph.create_vertex_collection('people')
graph.create_edge_collection('friends')

# Add vertices
alice = graph.vertex_collection('people').insert({
    'name': 'Alice',
    'age': 30
})

bob = graph.vertex_collection('people').insert({
    'name': 'Bob',
    'age': 25
})

# Add edge
graph.edge_collection('friends').insert({
    '_from': alice['_id'],
    '_to': bob['_id'],
    'since': '2023-01-01'
})
```

### Real-time Analytics
```python
# Apache Kafka with NoSQL
from kafka import KafkaConsumer
import json

class RealTimeAnalytics:
    def __init__(self, db):
        self.db = db
        self.consumer = KafkaConsumer(
            'user_events',
            bootstrap_servers=['localhost:9092'],
            value_deserializer=lambda x: json.loads(x.decode('utf-8'))
        )
    
    def process_events(self):
        for message in self.consumer:
            event = message.value
            
            # Store event in NoSQL database
            self.db['events'].insert_one(event)
            
            # Update real-time counters
            self.update_counters(event)
    
    def update_counters(self, event):
        # Update counters in Redis
        redis_client.incr(f"event_count:{event['type']}")
        redis_client.incr(f"user_activity:{event['user_id']}")
```

## 🔍 Monitoring and Debugging

### Performance Monitoring
```python
class NoSQLMonitor:
    def __init__(self, databases):
        self.databases = databases
    
    def collect_metrics(self):
        metrics = {}
        for db_name, db in self.databases.items():
            if hasattr(db, 'server_info'):
                # MongoDB
                info = db.server_info()
                metrics[db_name] = {
                    'connections': info['connections']['current'],
                    'operations': info['opcounters'],
                    'memory': info['mem']['resident']
                }
            elif hasattr(db, 'info'):
                # Redis
                info = db.info()
                metrics[db_name] = {
                    'connected_clients': info['connected_clients'],
                    'used_memory': info['used_memory'],
                    'total_commands': info['total_commands_processed']
                }
        return metrics
    
    def check_health(self):
        health_status = {}
        for db_name, db in self.databases.items():
            try:
                if hasattr(db, 'admin'):
                    # MongoDB
                    db.admin.command('ping')
                    health_status[db_name] = 'healthy'
                elif hasattr(db, 'ping'):
                    # Redis
                    db.ping()
                    health_status[db_name] = 'healthy'
            except Exception as e:
                health_status[db_name] = f'unhealthy: {str(e)}'
        return health_status
```

## 🐛 Troubleshooting

### Common Issues
1. **Connection Issues** - Check network connectivity and authentication
2. **Performance Problems** - Monitor query performance and optimize
3. **Data Consistency** - Handle eventual consistency in distributed systems
4. **Memory Issues** - Monitor memory usage and optimize queries
5. **Scaling Issues** - Plan for horizontal scaling and sharding

### Debugging Tools
```python
class NoSQLDebugger:
    def __init__(self, db):
        self.db = db
    
    def analyze_query_performance(self, query):
        # MongoDB
        if hasattr(self.db, 'explain'):
            return self.db.explain(query)
        
        # Redis
        if hasattr(self.db, 'slowlog_get'):
            return self.db.slowlog_get(10)
    
    def check_data_consistency(self):
        # Implement consistency checks
        pass
    
    def monitor_connections(self):
        # Monitor active connections
        pass
```

## 📚 Learning Resources

### Official Documentation
- [MongoDB Documentation](https://docs.mongodb.com/)
- [Redis Documentation](https://redis.io/documentation)
- [Cassandra Documentation](https://cassandra.apache.org/doc/latest/)
- [Neo4j Documentation](https://neo4j.com/docs/)

### Community Resources
- [NoSQL Database Comparison](https://db-engines.com/en/ranking)
- [NoSQL Patterns](http://horicky.blogspot.com/2009/11/nosql-patterns.html)
- [CAP Theorem](https://en.wikipedia.org/wiki/CAP_theorem)

## 🔄 Updates and Roadmap

### Recent Updates
- **2024** - Enhanced performance and new features
- **2023** - Better cloud integration and scalability
- **2022** - Improved security and compliance features

### Upcoming Features
- **2024** - Advanced analytics and machine learning integration
- **2025** - Enhanced security and compliance features
- **2026** - Improved performance and cost optimization

## 🤝 Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

### Contribution Guidelines
- Follow coding standards
- Write comprehensive tests
- Update documentation
- Ensure backward compatibility

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](https://opensource.org/licenses/MIT) file for details.

## 🙏 Acknowledgments

- **NoSQL Database Teams** for creating and maintaining these tools
- **Open Source Community** for contributions and feedback
- **Users** for providing real-world use cases and improvements
- **Contributors** for advancing NoSQL technology

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#nosql-databases-)

</div>
