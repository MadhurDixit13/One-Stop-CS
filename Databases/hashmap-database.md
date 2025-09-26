# Hashmap Database 🗂️

[![Hashmap Database](https://img.shields.io/badge/Hashmap%20Database-Key%20Value-red)](https://en.wikipedia.org/wiki/Hash_table)
[![Redis](https://img.shields.io/badge/Redis-In%20Memory-green)](https://redis.io/)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

> **Key-Value Storage** - Hashmap databases are specialized databases that store data as key-value pairs, providing fast lookups, inserts, and deletions using hash table data structures for optimal performance.

## 🎯 Overview

Hashmap databases are specialized databases that:
- **Store Key-Value Pairs** - Simple data model with keys and values
- **Provide Fast Access** - O(1) average time complexity for operations
- **Enable Caching** - Ideal for caching frequently accessed data
- **Support Sessions** - Store user sessions and temporary data
- **Handle Counters** - Efficiently manage counters and statistics
- **Offer Pub/Sub** - Publish-subscribe messaging patterns

## 🚀 Key Features

### Core Capabilities
- **Key-Value Storage** - Store any data type as key-value pairs
- **Fast Operations** - O(1) average time complexity for CRUD operations
- **Data Types** - Support strings, numbers, lists, sets, hashes, and more
- **Atomic Operations** - Ensure data consistency with atomic operations
- **Expiration** - Set time-to-live (TTL) for keys
- **Persistence** - Optional disk persistence for data durability

### Advanced Features
- **Clustering** - Distribute data across multiple nodes
- **Replication** - High availability with master-slave replication
- **Pub/Sub** - Publish-subscribe messaging system
- **Lua Scripting** - Execute complex operations atomically
- **Streams** - Append-only log data structure
- **Modules** - Extend functionality with custom modules

## 🏗️ Architecture

### Hashmap Database Components
```
Hashmap Database Architecture
├── 🗂️ Storage Engine
│   ├── Hash Table
│   ├── Skip List
│   ├── Radix Tree
│   └── Compression
├── 🔍 Query Engine
│   ├── Key Lookup
│   ├── Pattern Matching
│   └── Range Queries
├── 💾 Persistence Layer
│   ├── RDB Snapshots
│   ├── AOF Logs
│   └── Memory Management
├── 🌐 Network Layer
│   ├── TCP Protocol
│   ├── RESP Protocol
│   └── Connection Pooling
└── 🔧 Management
    ├── Clustering
    ├── Replication
    └── Monitoring
```

## 🛠️ Popular Hashmap Databases

### Redis
```bash
# Install Redis
# Ubuntu/Debian
sudo apt update
sudo apt install redis-server

# macOS
brew install redis

# Start Redis
redis-server

# Connect to Redis
redis-cli
```

```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

# Basic operations
r.set('key', 'value')
value = r.get('key')
print(value.decode('utf-8'))  # 'value'

# Set with expiration
r.setex('temp_key', 60, 'temporary_value')  # Expires in 60 seconds

# Check if key exists
exists = r.exists('key')
print(exists)  # 1 if exists, 0 if not

# Delete key
r.delete('key')
```

### Memcached
```bash
# Install Memcached
# Ubuntu/Debian
sudo apt install memcached

# macOS
brew install memcached

# Start Memcached
memcached -d -m 64 -p 11211
```

```python
import memcache

# Connect to Memcached
mc = memcache.Client(['127.0.0.1:11211'])

# Basic operations
mc.set('key', 'value')
value = mc.get('key')
print(value)  # 'value'

# Set with expiration
mc.set('temp_key', 'temporary_value', time=60)  # Expires in 60 seconds

# Delete key
mc.delete('key')
```

### Amazon DynamoDB
```python
import boto3

# Create DynamoDB client
dynamodb = boto3.resource('dynamodb', region_name='us-east-1')

# Create table
table = dynamodb.create_table(
    TableName='my-table',
    KeySchema=[
        {
            'AttributeName': 'id',
            'KeyType': 'HASH'
        }
    ],
    AttributeDefinitions=[
        {
            'AttributeName': 'id',
            'AttributeType': 'S'
        }
    ],
    BillingMode='PAY_PER_REQUEST'
)

# Put item
table.put_item(
    Item={
        'id': '1',
        'name': 'John Doe',
        'email': 'john@example.com'
    }
)

# Get item
response = table.get_item(Key={'id': '1'})
item = response['Item']
print(item)
```

## 📊 Data Types

### Strings
```python
# Redis string operations
r.set('name', 'Alice')
r.set('age', '30')
r.set('score', '95.5')

# Get values
name = r.get('name').decode('utf-8')
age = int(r.get('age'))
score = float(r.get('score'))

# String operations
r.append('name', ' Smith')  # Append to string
r.incr('age')  # Increment by 1
r.incrby('score', 5)  # Increment by 5
r.decr('age')  # Decrement by 1
```

### Lists
```python
# Redis list operations
r.lpush('tasks', 'task1', 'task2', 'task3')  # Push to left
r.rpush('tasks', 'task4', 'task5')  # Push to right

# Get list elements
tasks = r.lrange('tasks', 0, -1)  # Get all elements
first_task = r.lindex('tasks', 0)  # Get first element
last_task = r.lindex('tasks', -1)  # Get last element

# List operations
r.lpop('tasks')  # Remove and return first element
r.rpop('tasks')  # Remove and return last element
r.ltrim('tasks', 0, 2)  # Keep only first 3 elements
```

### Sets
```python
# Redis set operations
r.sadd('tags', 'python', 'redis', 'database')
r.sadd('tags', 'python')  # Duplicate ignored

# Set operations
members = r.smembers('tags')  # Get all members
is_member = r.sismember('tags', 'python')  # Check membership
r.srem('tags', 'redis')  # Remove member

# Set operations
r.sadd('tags1', 'python', 'java', 'c++')
r.sadd('tags2', 'python', 'javascript', 'go')
intersection = r.sinter('tags1', 'tags2')  # Intersection
union = r.sunion('tags1', 'tags2')  # Union
difference = r.sdiff('tags1', 'tags2')  # Difference
```

### Hashes
```python
# Redis hash operations
r.hset('user:1', mapping={
    'name': 'Alice',
    'email': 'alice@example.com',
    'age': '30'
})

# Hash operations
name = r.hget('user:1', 'name')
all_fields = r.hgetall('user:1')  # Get all fields
r.hset('user:1', 'age', '31')  # Update field
r.hdel('user:1', 'email')  # Delete field

# Hash operations
r.hincrby('user:1', 'age', 1)  # Increment field
r.hincrbyfloat('user:1', 'score', 0.5)  # Increment float field
```

### Sorted Sets
```python
# Redis sorted set operations
r.zadd('leaderboard', {
    'player1': 100,
    'player2': 200,
    'player3': 150
})

# Sorted set operations
top_players = r.zrevrange('leaderboard', 0, 2)  # Top 3 players
player_score = r.zscore('leaderboard', 'player1')
r.zincrby('leaderboard', 50, 'player1')  # Increment score

# Range operations
players_100_200 = r.zrangebyscore('leaderboard', 100, 200)
r.zremrangebyscore('leaderboard', 0, 100)  # Remove low scores
```

## 🔍 Advanced Operations

### Transactions
```python
# Redis transactions
pipe = r.pipeline()
pipe.set('key1', 'value1')
pipe.set('key2', 'value2')
pipe.incr('counter')
result = pipe.execute()

# Watch keys for changes
r.watch('key1', 'key2')
pipe = r.pipeline()
pipe.set('key1', 'new_value1')
pipe.set('key2', 'new_value2')
try:
    pipe.execute()
except redis.WatchError:
    print("Key was modified, transaction aborted")
```

### Lua Scripting
```python
# Lua script for atomic operations
lua_script = """
local key = KEYS[1]
local value = ARGV[1]
local ttl = ARGV[2]

redis.call('SET', key, value)
redis.call('EXPIRE', key, ttl)
return redis.call('GET', key)
"""

# Execute Lua script
result = r.eval(lua_script, 1, 'my_key', 'my_value', 60)
print(result)
```

### Pub/Sub
```python
# Publisher
r.publish('channel1', 'Hello, subscribers!')

# Subscriber
pubsub = r.pubsub()
pubsub.subscribe('channel1')

for message in pubsub.listen():
    if message['type'] == 'message':
        print(f"Received: {message['data']}")
```

## 🚀 Use Cases

### Caching
```python
# Cache with expiration
def get_user_data(user_id):
    cache_key = f"user:{user_id}"
    
    # Try to get from cache
    cached_data = r.get(cache_key)
    if cached_data:
        return json.loads(cached_data)
    
    # Get from database
    user_data = database.get_user(user_id)
    
    # Cache for 1 hour
    r.setex(cache_key, 3600, json.dumps(user_data))
    
    return user_data

# Cache invalidation
def update_user_data(user_id, new_data):
    # Update database
    database.update_user(user_id, new_data)
    
    # Invalidate cache
    r.delete(f"user:{user_id}")
```

### Session Management
```python
# Store user session
def create_session(user_id, session_data):
    session_id = str(uuid.uuid4())
    session_key = f"session:{session_id}"
    
    r.hset(session_key, mapping=session_data)
    r.expire(session_key, 3600)  # 1 hour expiration
    
    return session_id

# Get session data
def get_session(session_id):
    session_key = f"session:{session_id}"
    session_data = r.hgetall(session_key)
    
    if not session_data:
        return None
    
    # Convert bytes to strings
    return {k.decode('utf-8'): v.decode('utf-8') for k, v in session_data.items()}

# Extend session
def extend_session(session_id, seconds=3600):
    session_key = f"session:{session_id}"
    r.expire(session_key, seconds)
```

### Rate Limiting
```python
# Rate limiting with sliding window
def is_rate_limited(user_id, limit=100, window=3600):
    key = f"rate_limit:{user_id}"
    current_time = int(time.time())
    
    # Remove old entries
    r.zremrangebyscore(key, 0, current_time - window)
    
    # Count current requests
    current_requests = r.zcard(key)
    
    if current_requests >= limit:
        return True
    
    # Add current request
    r.zadd(key, {str(current_time): current_time})
    r.expire(key, window)
    
    return False
```

### Leaderboards
```python
# Update player score
def update_score(player_id, score):
    r.zadd('leaderboard', {player_id: score})

# Get top players
def get_top_players(limit=10):
    return r.zrevrange('leaderboard', 0, limit - 1, withscores=True)

# Get player rank
def get_player_rank(player_id):
    rank = r.zrevrank('leaderboard', player_id)
    return rank + 1 if rank is not None else None

# Get players around a player
def get_players_around(player_id, range_size=5):
    player_rank = r.zrevrank('leaderboard', player_id)
    if player_rank is None:
        return []
    
    start = max(0, player_rank - range_size // 2)
    end = start + range_size - 1
    
    return r.zrevrange('leaderboard', start, end, withscores=True)
```

## 🔧 Performance Optimization

### Connection Pooling
```python
import redis.connection

# Create connection pool
pool = redis.ConnectionPool(
    host='localhost',
    port=6379,
    db=0,
    max_connections=20
)

# Use connection pool
r = redis.Redis(connection_pool=pool)

# Or use with context manager
with redis.Redis(connection_pool=pool) as r:
    r.set('key', 'value')
```

### Pipelining
```python
# Batch operations with pipeline
pipe = r.pipeline()
for i in range(1000):
    pipe.set(f'key:{i}', f'value:{i}')
pipe.execute()

# Pipeline with transactions
pipe = r.pipeline(transaction=True)
pipe.set('key1', 'value1')
pipe.set('key2', 'value2')
pipe.execute()
```

### Memory Optimization
```python
# Use appropriate data types
# Instead of storing JSON strings, use hashes
r.hset('user:1', mapping={
    'name': 'Alice',
    'email': 'alice@example.com'
})

# Use compression for large values
import gzip
import json

def compress_and_store(key, data):
    compressed = gzip.compress(json.dumps(data).encode())
    r.set(key, compressed)

def decompress_and_get(key):
    compressed = r.get(key)
    if compressed:
        return json.loads(gzip.decompress(compressed))
    return None
```

## 🔧 Best Practices

### Key Design
1. **Use Namespaces** - Prefix keys with namespace (e.g., "user:1", "session:abc")
2. **Avoid Long Keys** - Keep keys short and meaningful
3. **Use Consistent Patterns** - Follow consistent naming conventions
4. **Avoid Special Characters** - Use alphanumeric characters and colons
5. **Consider Key Expiration** - Set appropriate TTL for temporary data

### Data Modeling
1. **Choose Right Data Type** - Use appropriate data types for your use case
2. **Normalize Data** - Avoid storing redundant data
3. **Use Hashes for Objects** - Store object properties as hash fields
4. **Use Sets for Unique Values** - Use sets for unique collections
5. **Use Sorted Sets for Rankings** - Use sorted sets for leaderboards

### Performance
1. **Use Pipelines** - Batch operations for better performance
2. **Monitor Memory Usage** - Keep track of memory consumption
3. **Use Appropriate Expiration** - Set TTL to prevent memory leaks
4. **Optimize Queries** - Use efficient data structures and operations
5. **Monitor Performance** - Track response times and throughput

## 🚀 Advanced Features

### Clustering
```python
# Redis Cluster
from rediscluster import RedisCluster

startup_nodes = [
    {"host": "127.0.0.1", "port": "7000"},
    {"host": "127.0.0.1", "port": "7001"},
    {"host": "127.0.0.1", "port": "7002"}
]

rc = RedisCluster(startup_nodes=startup_nodes, decode_responses=True)

# Cluster operations
rc.set('key', 'value')
value = rc.get('key')
```

### Replication
```python
# Master-Slave replication
# Master
r = redis.Redis(host='localhost', port=6379)

# Slave
slave = redis.Redis(host='localhost', port=6380)
slave.slaveof('localhost', 6379)
```

### Streams
```python
# Redis Streams
# Add to stream
stream_id = r.xadd('mystream', {'field1': 'value1', 'field2': 'value2'})

# Read from stream
messages = r.xread({'mystream': '$'}, count=10)

# Consumer group
r.xgroup_create('mystream', 'mygroup', id='0', mkstream=True)
messages = r.xreadgroup('mygroup', 'consumer1', {'mystream': '>'}, count=10)
```

## 🔍 Monitoring and Debugging

### Performance Monitoring
```python
# Get Redis info
info = r.info()
print(f"Used memory: {info['used_memory_human']}")
print(f"Connected clients: {info['connected_clients']}")
print(f"Total commands processed: {info['total_commands_processed']}")

# Monitor slow queries
slowlog = r.slowlog_get(10)
for entry in slowlog:
    print(f"Command: {entry['command']}, Duration: {entry['duration']}ms")
```

### Debugging
```python
# Check key information
key_info = r.debug_object('my_key')
print(key_info)

# Monitor commands
r.monitor()

# Check memory usage
memory_usage = r.memory_usage('my_key')
print(f"Memory usage: {memory_usage} bytes")
```

## 🐛 Troubleshooting

### Common Issues
1. **Memory Issues** - Monitor memory usage and set appropriate expiration
2. **Slow Queries** - Use slowlog to identify slow operations
3. **Connection Issues** - Check connection pool settings and network
4. **Data Persistence** - Verify RDB and AOF configuration
5. **Replication Issues** - Check master-slave configuration and network

### Debugging Tools
```python
# Check Redis configuration
config = r.config_get()
print(config)

# Check key expiration
ttl = r.ttl('my_key')
print(f"TTL: {ttl} seconds")

# Check key type
key_type = r.type('my_key')
print(f"Key type: {key_type}")
```

## 📚 Learning Resources

### Official Documentation
- [Redis Documentation](https://redis.io/documentation)
- [Memcached Documentation](https://memcached.org/)
- [DynamoDB Documentation](https://docs.aws.amazon.com/dynamodb/)

### Community Resources
- [Redis University](https://university.redis.com/)
- [Redis Community](https://community.redis.com/)
- [Memcached Wiki](https://github.com/memcached/memcached/wiki)

## 🔄 Updates and Roadmap

### Recent Updates
- **Redis 7.0** - Enhanced performance and new features
- **Redis 6.2** - Improved clustering and security
- **Redis 6.0** - Multi-threading and ACL support

### Upcoming Features
- **Redis 8.0** - Enhanced performance and new data structures
- **Better Clustering** - Improved cluster management
- **Enhanced Security** - Better authentication and authorization

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

- **Redis Labs** for creating and maintaining Redis
- **Memcached Team** for the Memcached project
- **Amazon** for DynamoDB
- **Open Source Community** for contributions and feedback

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#hashmap-database-)

</div>
