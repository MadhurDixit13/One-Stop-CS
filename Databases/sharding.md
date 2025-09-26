# Database Sharding 🔀

[![Database Sharding](https://img.shields.io/badge/Database%20Sharding-Horizontal%20Scaling-blue)](https://en.wikipedia.org/wiki/Shard_(database_architecture))
[![MongoDB](https://img.shields.io/badge/MongoDB-Sharded%20Database-green)](https://mongodb.com/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Horizontal Database Scaling** - Database sharding is a method of horizontal partitioning where data is distributed across multiple database instances, enabling better performance, scalability, and fault tolerance for large-scale applications.

## 🎯 Overview

Database sharding is a scaling technique that:
- **Distributes Data** - Spreads data across multiple database instances
- **Improves Performance** - Reduces query load on individual databases
- **Enables Scalability** - Allows horizontal scaling beyond single machine limits
- **Provides Fault Tolerance** - Isolates failures to specific shards
- **Optimizes Resources** - Better resource utilization across machines
- **Supports Growth** - Handles increasing data volumes and user loads

## 🚀 Key Features

### Core Capabilities
- **Horizontal Partitioning** - Split data across multiple database instances
- **Load Distribution** - Distribute read and write operations
- **Independent Scaling** - Scale individual shards independently
- **Fault Isolation** - Isolate failures to specific shards
- **Data Locality** - Keep related data together for better performance
- **Transparent Access** - Hide sharding complexity from applications

### Advanced Features
- **Dynamic Sharding** - Add or remove shards without downtime
- **Rebalancing** - Automatically redistribute data across shards
- **Cross-Shard Queries** - Query data across multiple shards
- **Consistent Hashing** - Efficient data distribution algorithms
- **Shard Monitoring** - Monitor individual shard performance
- **Backup and Recovery** - Backup and restore individual shards

## 🏗️ Architecture

### Sharding Components
```
Database Sharding Architecture
├── 🎯 Shard Key
│   ├── Hash-based
│   ├── Range-based
│   ├── Directory-based
│   └── Consistent Hashing
├── 🔀 Shard Router
│   ├── Query Router
│   ├── Load Balancer
│   ├── Connection Pool
│   └── Failover Handler
├── 💾 Shard Storage
│   ├── Shard 1
│   ├── Shard 2
│   ├── Shard N
│   └── Metadata Store
├── 🌐 Application Layer
│   ├── Shard-aware Client
│   ├── Query Optimizer
│   └── Transaction Manager
└── 🔧 Management
    ├── Shard Monitor
    ├── Rebalancer
    └── Backup Manager
```

## 🛠️ Sharding Strategies

### Hash-based Sharding
```python
import hashlib

class HashSharding:
    def __init__(self, num_shards):
        self.num_shards = num_shards
    
    def get_shard(self, key):
        """Get shard ID using hash function"""
        hash_value = int(hashlib.md5(str(key).encode()).hexdigest(), 16)
        return hash_value % self.num_shards
    
    def add_shard(self):
        """Add new shard (requires rebalancing)"""
        self.num_shards += 1
        # Trigger rebalancing process
    
    def remove_shard(self, shard_id):
        """Remove shard (requires rebalancing)"""
        if shard_id < self.num_shards:
            self.num_shards -= 1
            # Trigger rebalancing process

# Usage
sharding = HashSharding(4)
shard_id = sharding.get_shard("user_123")
print(f"User 123 belongs to shard {shard_id}")
```

### Range-based Sharding
```python
class RangeSharding:
    def __init__(self, shard_ranges):
        self.shard_ranges = sorted(shard_ranges)
    
    def get_shard(self, key):
        """Get shard ID using range lookup"""
        for i, (start, end) in enumerate(self.shard_ranges):
            if start <= key <= end:
                return i
        raise ValueError(f"Key {key} not in any shard range")
    
    def add_shard(self, start, end):
        """Add new shard with range"""
        self.shard_ranges.append((start, end))
        self.shard_ranges.sort()
    
    def remove_shard(self, shard_id):
        """Remove shard"""
        if 0 <= shard_id < len(self.shard_ranges):
            del self.shard_ranges[shard_id]

# Usage
sharding = RangeSharding([(0, 1000), (1001, 2000), (2001, 3000)])
shard_id = sharding.get_shard(1500)
print(f"Key 1500 belongs to shard {shard_id}")
```

### Directory-based Sharding
```python
class DirectorySharding:
    def __init__(self):
        self.shard_map = {}
        self.shard_servers = {}
    
    def get_shard(self, key):
        """Get shard ID using directory lookup"""
        return self.shard_map.get(key)
    
    def add_mapping(self, key, shard_id):
        """Add key to shard mapping"""
        self.shard_map[key] = shard_id
    
    def add_shard(self, shard_id, server_info):
        """Add new shard server"""
        self.shard_servers[shard_id] = server_info
    
    def remove_shard(self, shard_id):
        """Remove shard"""
        if shard_id in self.shard_servers:
            del self.shard_servers[shard_id]
            # Remove all keys mapped to this shard
            keys_to_remove = [k for k, v in self.shard_map.items() if v == shard_id]
            for key in keys_to_remove:
                del self.shard_map[key]

# Usage
sharding = DirectorySharding()
sharding.add_shard(0, "server1:3306")
sharding.add_shard(1, "server2:3306")
sharding.add_mapping("user_123", 0)
shard_id = sharding.get_shard("user_123")
print(f"User 123 belongs to shard {shard_id}")
```

### Consistent Hashing
```python
import hashlib
import bisect

class ConsistentHashing:
    def __init__(self, nodes=None, replicas=3):
        self.replicas = replicas
        self.ring = {}
        self.sorted_keys = []
        
        if nodes:
            for node in nodes:
                self.add_node(node)
    
    def _hash(self, key):
        """Generate hash for key"""
        return int(hashlib.md5(str(key).encode()).hexdigest(), 16)
    
    def add_node(self, node):
        """Add node to hash ring"""
        for i in range(self.replicas):
            key = self._hash(f"{node}:{i}")
            self.ring[key] = node
            bisect.insort(self.sorted_keys, key)
    
    def remove_node(self, node):
        """Remove node from hash ring"""
        for i in range(self.replicas):
            key = self._hash(f"{node}:{i}")
            if key in self.ring:
                del self.ring[key]
                self.sorted_keys.remove(key)
    
    def get_node(self, key):
        """Get node for key"""
        if not self.ring:
            return None
        
        hash_key = self._hash(key)
        idx = bisect.bisect_right(self.sorted_keys, hash_key)
        
        if idx == len(self.sorted_keys):
            idx = 0
        
        return self.ring[self.sorted_keys[idx]]

# Usage
hashing = ConsistentHashing(['server1', 'server2', 'server3'])
node = hashing.get_node("user_123")
print(f"User 123 belongs to {node}")
```

## 🔧 Implementation Examples

### MongoDB Sharding
```javascript
// Enable sharding on database
sh.enableSharding("myapp")

// Create shard key index
db.users.createIndex({ "user_id": 1 })

// Shard collection
sh.shardCollection("myapp.users", { "user_id": 1 })

// Add shards
sh.addShard("shard1/mongodb1:27017")
sh.addShard("shard2/mongodb2:27017")
sh.addShard("shard3/mongodb3:27017")

// Check shard status
sh.status()
```

### MySQL Sharding
```sql
-- Create sharded tables
CREATE TABLE users_shard_1 (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
) ENGINE=InnoDB;

CREATE TABLE users_shard_2 (
    id INT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
) ENGINE=InnoDB;

-- Create shard router function
DELIMITER //
CREATE FUNCTION get_shard_id(user_id INT) RETURNS INT
READS SQL DATA
DETERMINISTIC
BEGIN
    RETURN (user_id % 2) + 1;
END//
DELIMITER ;

-- Insert with shard routing
INSERT INTO users_shard_1 (id, name, email) 
VALUES (1, 'Alice', 'alice@example.com');

INSERT INTO users_shard_2 (id, name, email) 
VALUES (2, 'Bob', 'bob@example.com');
```

### PostgreSQL Sharding
```sql
-- Create sharded tables
CREATE TABLE users_shard_1 (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE users_shard_2 (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

-- Create shard router function
CREATE OR REPLACE FUNCTION get_shard_id(user_id INTEGER) 
RETURNS INTEGER AS $$
BEGIN
    RETURN (user_id % 2) + 1;
END;
$$ LANGUAGE plpgsql;

-- Create view for transparent access
CREATE VIEW users AS
SELECT * FROM users_shard_1
UNION ALL
SELECT * FROM users_shard_2;
```

## 🚀 Sharding Patterns

### Application-level Sharding
```python
class ShardedDatabase:
    def __init__(self, shard_configs):
        self.shards = {}
        self.sharding_strategy = HashSharding(len(shard_configs))
        
        for i, config in enumerate(shard_configs):
            self.shards[i] = self.connect_to_shard(config)
    
    def connect_to_shard(self, config):
        """Connect to individual shard"""
        return pymongo.MongoClient(config['host'], config['port'])
    
    def get_shard(self, key):
        """Get shard for key"""
        shard_id = self.sharding_strategy.get_shard(key)
        return self.shards[shard_id]
    
    def insert(self, collection, document):
        """Insert document into appropriate shard"""
        shard = self.get_shard(document['_id'])
        return shard[collection].insert_one(document)
    
    def find(self, collection, query):
        """Find documents across shards"""
        results = []
        for shard in self.shards.values():
            shard_results = shard[collection].find(query)
            results.extend(list(shard_results))
        return results
    
    def find_one(self, collection, query):
        """Find one document in appropriate shard"""
        if '_id' in query:
            shard = self.get_shard(query['_id'])
            return shard[collection].find_one(query)
        else:
            # Search all shards
            for shard in self.shards.values():
                result = shard[collection].find_one(query)
                if result:
                    return result
            return None

# Usage
shard_configs = [
    {'host': 'localhost', 'port': 27017},
    {'host': 'localhost', 'port': 27018},
    {'host': 'localhost', 'port': 27019}
]

db = ShardedDatabase(shard_configs)
db.insert('users', {'_id': 'user_123', 'name': 'Alice'})
user = db.find_one('users', {'_id': 'user_123'})
```

### Proxy-based Sharding
```python
from flask import Flask, request, jsonify
import pymongo

app = Flask(__name__)

class ShardProxy:
    def __init__(self, shard_configs):
        self.shards = {}
        self.sharding_strategy = HashSharding(len(shard_configs))
        
        for i, config in enumerate(shard_configs):
            self.shards[i] = pymongo.MongoClient(config['host'], config['port'])
    
    def get_shard(self, key):
        shard_id = self.sharding_strategy.get_shard(key)
        return self.shards[shard_id]
    
    def route_request(self, operation, collection, data):
        if operation == 'insert':
            shard = self.get_shard(data['_id'])
            return shard[collection].insert_one(data)
        elif operation == 'find':
            if '_id' in data:
                shard = self.get_shard(data['_id'])
                return shard[collection].find_one(data)
            else:
                # Search all shards
                results = []
                for shard in self.shards.values():
                    results.extend(list(shard[collection].find(data)))
                return results

# Initialize proxy
shard_configs = [
    {'host': 'localhost', 'port': 27017},
    {'host': 'localhost', 'port': 27018}
]
proxy = ShardProxy(shard_configs)

@app.route('/api/users', methods=['POST'])
def create_user():
    data = request.json
    result = proxy.route_request('insert', 'users', data)
    return jsonify({'id': str(result.inserted_id)})

@app.route('/api/users/<user_id>', methods=['GET'])
def get_user(user_id):
    result = proxy.route_request('find', 'users', {'_id': user_id})
    return jsonify(result)

if __name__ == '__main__':
    app.run(debug=True)
```

## 🔍 Cross-Shard Operations

### Cross-Shard Queries
```python
class CrossShardQuery:
    def __init__(self, shards):
        self.shards = shards
    
    def aggregate(self, collection, pipeline):
        """Aggregate data across all shards"""
        results = []
        for shard in self.shards:
            shard_results = shard[collection].aggregate(pipeline)
            results.extend(list(shard_results))
        
        # Merge results
        return self.merge_results(results)
    
    def merge_results(self, results):
        """Merge results from multiple shards"""
        # Simple merge - in practice, you'd implement proper aggregation
        return results
    
    def count(self, collection, query):
        """Count documents across all shards"""
        total_count = 0
        for shard in self.shards:
            count = shard[collection].count_documents(query)
            total_count += count
        return total_count
    
    def distinct(self, collection, field, query=None):
        """Get distinct values across all shards"""
        distinct_values = set()
        for shard in self.shards:
            if query:
                values = shard[collection].distinct(field, query)
            else:
                values = shard[collection].distinct(field)
            distinct_values.update(values)
        return list(distinct_values)
```

### Cross-Shard Transactions
```python
class CrossShardTransaction:
    def __init__(self, shards):
        self.shards = shards
        self.operations = []
    
    def add_operation(self, shard_id, operation):
        """Add operation to transaction"""
        self.operations.append((shard_id, operation))
    
    def execute(self):
        """Execute cross-shard transaction"""
        try:
            # Prepare phase
            for shard_id, operation in self.operations:
                shard = self.shards[shard_id]
                operation.prepare(shard)
            
            # Commit phase
            for shard_id, operation in self.operations:
                shard = self.shards[shard_id]
                operation.commit(shard)
            
            return True
        except Exception as e:
            # Rollback phase
            for shard_id, operation in self.operations:
                shard = self.shards[shard_id]
                operation.rollback(shard)
            raise e

class ShardOperation:
    def __init__(self, collection, operation_type, data):
        self.collection = collection
        self.operation_type = operation_type
        self.data = data
        self.prepared = False
    
    def prepare(self, shard):
        """Prepare operation on shard"""
        if self.operation_type == 'insert':
            # Validate data
            pass
        elif self.operation_type == 'update':
            # Check if document exists
            pass
        self.prepared = True
    
    def commit(self, shard):
        """Commit operation on shard"""
        if not self.prepared:
            raise Exception("Operation not prepared")
        
        if self.operation_type == 'insert':
            shard[self.collection].insert_one(self.data)
        elif self.operation_type == 'update':
            shard[self.collection].update_one(
                self.data['filter'], 
                self.data['update']
            )
    
    def rollback(self, shard):
        """Rollback operation on shard"""
        # Implement rollback logic
        pass
```

## 🔧 Best Practices

### Shard Key Design
1. **Choose High Cardinality** - Use keys with many unique values
2. **Avoid Hotspots** - Distribute load evenly across shards
3. **Consider Query Patterns** - Design for common query patterns
4. **Use Composite Keys** - Combine multiple fields for better distribution
5. **Avoid Monotonic Keys** - Don't use auto-incrementing IDs as shard keys

### Data Distribution
1. **Monitor Shard Sizes** - Keep shards roughly the same size
2. **Plan for Growth** - Design for future data growth
3. **Consider Data Locality** - Keep related data together
4. **Use Consistent Hashing** - For dynamic shard addition/removal
5. **Implement Rebalancing** - Automatically redistribute data

### Performance Optimization
1. **Use Connection Pooling** - Reuse database connections
2. **Implement Caching** - Cache frequently accessed data
3. **Optimize Queries** - Minimize cross-shard queries
4. **Use Read Replicas** - Distribute read load
5. **Monitor Performance** - Track shard performance metrics

## 🚀 Advanced Features

### Dynamic Sharding
```python
class DynamicSharding:
    def __init__(self, initial_shards=2):
        self.shards = {}
        self.shard_count = initial_shards
        self.sharding_strategy = ConsistentHashing()
        
        # Initialize shards
        for i in range(initial_shards):
            self.add_shard(f"shard_{i}")
    
    def add_shard(self, shard_name):
        """Add new shard"""
        self.sharding_strategy.add_node(shard_name)
        self.shards[shard_name] = self.connect_to_shard(shard_name)
        self.shard_count += 1
        
        # Trigger rebalancing
        self.rebalance()
    
    def remove_shard(self, shard_name):
        """Remove shard"""
        if shard_name in self.shards:
            # Move data to other shards
            self.migrate_data(shard_name)
            
            # Remove from hash ring
            self.sharding_strategy.remove_node(shard_name)
            del self.shards[shard_name]
            self.shard_count -= 1
    
    def rebalance(self):
        """Rebalance data across shards"""
        # Implement rebalancing logic
        pass
    
    def migrate_data(self, from_shard):
        """Migrate data from one shard to others"""
        # Implement data migration logic
        pass
```

### Shard Monitoring
```python
class ShardMonitor:
    def __init__(self, shards):
        self.shards = shards
        self.metrics = {}
    
    def collect_metrics(self):
        """Collect metrics from all shards"""
        for shard_name, shard in self.shards.items():
            metrics = {
                'connection_count': shard.server_info()['connections']['current'],
                'operation_count': shard.server_info()['opcounters'],
                'memory_usage': shard.server_info()['mem']['resident'],
                'disk_usage': shard.server_info()['storageEngine']['dataSize']
            }
            self.metrics[shard_name] = metrics
    
    def check_health(self):
        """Check health of all shards"""
        healthy_shards = []
        unhealthy_shards = []
        
        for shard_name, shard in self.shards.items():
            try:
                shard.admin.command('ping')
                healthy_shards.append(shard_name)
            except Exception as e:
                unhealthy_shards.append((shard_name, str(e)))
        
        return healthy_shards, unhealthy_shards
    
    def get_load_balance(self):
        """Get load balance across shards"""
        self.collect_metrics()
        
        loads = []
        for shard_name, metrics in self.metrics.items():
            load = metrics['operation_count']['total']
            loads.append((shard_name, load))
        
        return sorted(loads, key=lambda x: x[1], reverse=True)
```

## 🔍 Monitoring and Debugging

### Performance Monitoring
```python
class ShardPerformanceMonitor:
    def __init__(self, shards):
        self.shards = shards
        self.query_times = {}
        self.error_counts = {}
    
    def track_query_time(self, shard_name, query_time):
        """Track query execution time"""
        if shard_name not in self.query_times:
            self.query_times[shard_name] = []
        self.query_times[shard_name].append(query_time)
    
    def track_error(self, shard_name, error):
        """Track errors per shard"""
        if shard_name not in self.error_counts:
            self.error_counts[shard_name] = 0
        self.error_counts[shard_name] += 1
    
    def get_average_query_time(self, shard_name):
        """Get average query time for shard"""
        if shard_name in self.query_times:
            return sum(self.query_times[shard_name]) / len(self.query_times[shard_name])
        return 0
    
    def get_error_rate(self, shard_name):
        """Get error rate for shard"""
        if shard_name in self.error_counts:
            total_queries = len(self.query_times.get(shard_name, []))
            if total_queries > 0:
                return self.error_counts[shard_name] / total_queries
        return 0
```

## 🐛 Troubleshooting

### Common Issues
1. **Hotspots** - Uneven data distribution causing performance issues
2. **Cross-shard Queries** - Slow queries that need to access multiple shards
3. **Rebalancing** - Data migration causing temporary performance degradation
4. **Connection Issues** - Network problems between application and shards
5. **Data Consistency** - Inconsistent data across shards

### Debugging Tools
```python
class ShardDebugger:
    def __init__(self, shards):
        self.shards = shards
    
    def analyze_data_distribution(self):
        """Analyze data distribution across shards"""
        distribution = {}
        for shard_name, shard in self.shards.items():
            count = shard['users'].count_documents({})
            distribution[shard_name] = count
        return distribution
    
    def find_hotspots(self):
        """Find shards with high load"""
        loads = []
        for shard_name, shard in self.shards.items():
            # Get operation count
            ops = shard.server_info()['opcounters']
            load = ops['total']
            loads.append((shard_name, load))
        
        return sorted(loads, key=lambda x: x[1], reverse=True)
    
    def check_data_consistency(self):
        """Check data consistency across shards"""
        # Implement consistency checks
        pass
```

## 📚 Learning Resources

### Official Documentation
- [MongoDB Sharding](https://docs.mongodb.com/manual/sharding/)
- [MySQL Sharding](https://dev.mysql.com/doc/refman/8.0/en/partitioning.html)
- [PostgreSQL Sharding](https://www.postgresql.org/docs/current/ddl-partitioning.html)

### Community Resources
- [Database Sharding Patterns](https://microservices.io/patterns/data/database-per-service.html)
- [Sharding Best Practices](https://www.mongodb.com/blog/post/building-with-patterns-the-sharding-pattern)
- [Horizontal Scaling Guide](https://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/CHAP_Scaling.html)

## 🔄 Updates and Roadmap

### Recent Updates
- **2024** - Enhanced sharding algorithms and performance
- **2023** - Better cross-shard transaction support
- **2022** - Improved rebalancing and monitoring

### Upcoming Features
- **2024** - Advanced sharding strategies and automation
- **2025** - Better cross-shard query optimization
- **2026** - Enhanced monitoring and debugging tools

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

- **Database Vendors** for implementing sharding features
- **Open Source Community** for contributions and feedback
- **Users** for providing real-world use cases and improvements
- **Contributors** for advancing database scaling technology

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#database-sharding-)

</div>
