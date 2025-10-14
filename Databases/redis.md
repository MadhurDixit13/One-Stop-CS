# Redis

Redis (REmote DIctionary Server) is an open-source, in-memory data structure store used as a database, cache, message broker, and streaming engine.

---

## What Problem Does It Solve?

Redis addresses:
- **Slow Database Queries**: Sub-millisecond response times by keeping data in memory
- **Scalability Bottlenecks**: Offload read-heavy workloads from primary databases
- **Session Management**: Fast, distributed session storage for web applications
- **Real-Time Analytics**: Leaderboards, counters, time-series data
- **Message Queuing**: Pub/Sub and stream-based messaging

---

## Core Concepts

### In-Memory Storage
- All data stored in RAM for ultra-fast access (microsecond latency)
- Optional persistence to disk (RDB snapshots, AOF logs)
- **Trade-off**: Limited by RAM capacity but extremely fast

### Data Structures
Redis supports rich data types beyond simple key-value:
- **Strings**: Binary-safe strings, integers, floats
- **Lists**: Linked lists for queues, stacks
- **Sets**: Unordered unique elements
- **Sorted Sets**: Sets ordered by score (leaderboards)
- **Hashes**: Key-value pairs within a key (objects)
- **Bitmaps**: Bit-level operations
- **HyperLogLogs**: Probabilistic cardinality estimation
- **Streams**: Append-only log for event streaming
- **Geospatial**: Location-based queries

### Persistence Options
- **RDB (Redis Database)**: Point-in-time snapshots
- **AOF (Append-Only File)**: Log every write operation
- **Hybrid**: Combine RDB + AOF for durability

---

## Common Use Cases

### Caching
- **Database Query Cache**: Store frequently accessed data
- **Page Cache**: Full HTML or JSON responses
- **CDN Origin Cache**: Reduce backend load
- **Example**: Cache user profiles, product catalogs, API responses

### Session Store
- Distributed session management for web apps
- Fast access, automatic expiration (TTL)
- Shared across multiple app servers

### Real-Time Analytics
- **Counters**: Page views, API requests, votes
- **Leaderboards**: Game scores, top users (sorted sets)
- **Time-Series**: Metrics, sensor data (Redis TimeSeries module)

### Message Broker
- **Pub/Sub**: Real-time notifications, chat applications
- **Streams**: Event sourcing, log aggregation, message queues
- **Task Queues**: Background job processing (Celery, Bull, Sidekiq)

### Rate Limiting
- Track API request counts per user/IP
- Sliding window or token bucket algorithms

### Distributed Locks
- Coordinate access to shared resources across services
- Use Redlock algorithm for multi-instance environments

---

## Example Commands

### Basic Operations
```bash
# Strings
SET user:1000:name "Alice"
GET user:1000:name
INCR page:views
EXPIRE session:abc123 3600  # TTL in seconds

# Lists (Queue)
LPUSH queue:tasks "task1"
RPOP queue:tasks

# Sets
SADD tags:post:1 "redis" "database" "nosql"
SMEMBERS tags:post:1

# Sorted Sets (Leaderboard)
ZADD leaderboard 1000 "player1" 950 "player2"
ZREVRANGE leaderboard 0 9 WITHSCORES  # Top 10

# Hashes
HSET user:1000 name "Alice" email "alice@example.com" age 30
HGETALL user:1000

# Pub/Sub
PUBLISH notifications "New message!"
SUBSCRIBE notifications
```

### Advanced Patterns
```bash
# Atomic Counter
MULTI
INCR counter
EXPIRE counter 60
EXEC

# Rate Limiting (Sliding Window)
ZADD ratelimit:user:123 1705334400 "req1"
ZREMRANGEBYSCORE ratelimit:user:123 0 1705330800  # Remove old entries
ZCARD ratelimit:user:123  # Count requests

# Cache with Expiration
SET cache:product:456 '{"name":"Laptop","price":999}' EX 3600

# Distributed Lock
SET lock:resource:789 "token123" NX EX 10
# Release lock
DEL lock:resource:789
```

---

## Architecture & Deployment

### Deployment Options
- **Standalone**: Single Redis instance (development, small apps)
- **Replication**: Master-replica for read scaling and failover
- **Sentinel**: Automatic failover and monitoring
- **Cluster**: Horizontal sharding across multiple nodes (16,384 hash slots)
- **Managed Services**: AWS ElastiCache, Azure Cache, Google Cloud Memorystore, Redis Cloud

### High Availability
- **Master-Replica Replication**: Async replication for read scaling
- **Redis Sentinel**: Monitors master, performs automatic failover
- **Redis Cluster**: Distributed, sharded, multi-master setup

### Persistence Trade-offs
| Method | Durability | Performance | Use Case |
|--------|-----------|-------------|----------|
| **None** | ❌ None | 🚀 Fastest | Pure cache, disposable data |
| **RDB** | ⚠️ Periodic | ⚡ Fast | Backup, disaster recovery |
| **AOF** | ✅ High | 🐢 Slower | Critical data, audit logs |
| **RDB + AOF** | ✅ Highest | 🐢 Slowest | Maximum durability |

---

## Best Practices

### Performance
- **Use Pipelining**: Send multiple commands in one round-trip
- **Avoid Large Keys**: Split large hashes/lists into smaller chunks
- **Use Connection Pooling**: Reuse connections (avoid connection overhead)
- **Monitor Memory**: Use `INFO memory`, set `maxmemory` and eviction policies

### Security
- **Enable AUTH**: Set `requirepass` in redis.conf
- **Disable Dangerous Commands**: FLUSHDB, FLUSHALL, CONFIG in production
- **Use TLS**: Encrypt client-server communication
- **Network Isolation**: Bind to private IPs, use firewalls

### Operations
- **Monitor Slow Queries**: Use `SLOWLOG` to identify performance issues
- **Set Eviction Policies**: `allkeys-lru`, `volatile-lru`, `noeviction`
- **Regular Backups**: Schedule RDB snapshots
- **Monitor Replication Lag**: Check `INFO replication`

---

## Redis vs. Memcached

| Feature | Redis | Memcached |
|---------|-------|-----------|
| **Data Structures** | ✅ Rich (lists, sets, hashes, etc.) | ❌ Key-value only |
| **Persistence** | ✅ RDB + AOF | ❌ In-memory only |
| **Replication** | ✅ Master-replica | ❌ No native support |
| **Pub/Sub** | ✅ Yes | ❌ No |
| **Transactions** | ✅ MULTI/EXEC | ❌ No |
| **Use Case** | Cache + database + broker | Pure cache |

---

## Further Reading

- [Redis Official Documentation](https://redis.io/docs/)
- [Redis Commands Reference](https://redis.io/commands/)
- [Redis University](https://university.redis.com/) - Free courses
- [Redis Best Practices](https://redis.io/docs/manual/patterns/)
- [Redis Cluster Tutorial](https://redis.io/docs/management/scaling/)
- [Redis GitHub Repository](https://github.com/redis/redis)
- [Redis Modules](https://redis.io/modules/) - RedisJSON, RedisTimeSeries, RedisGraph, RediSearch

