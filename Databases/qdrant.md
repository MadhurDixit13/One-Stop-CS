# Qdrant 🔍

[![Qdrant](https://img.shields.io/badge/Qdrant-Vector%20Database-green)](https://qdrant.tech/)
[![Rust](https://img.shields.io/badge/Rust-1.70+-orange)](https://rust-lang.org/)
[![License](https://img.shields.io/badge/License-Apache%202.0-green.svg)](https://opensource.org/licenses/Apache-2.0)

> **Vector Database & Vector Search Engine** - Qdrant is a high-performance vector database and vector similarity search engine designed for machine learning applications, semantic search, and AI-powered systems.

## 🎯 Overview

Qdrant is a powerful vector database that:
- **Stores Vectors** - High-dimensional vector storage and retrieval
- **Enables Similarity Search** - Fast approximate nearest neighbor search
- **Supports ML Workflows** - Integration with machine learning pipelines
- **Provides Filtering** - Hybrid search with metadata filtering
- **Offers Scalability** - Horizontal scaling and distributed deployment
- **Ensures Performance** - Optimized for high-throughput vector operations

## 🚀 Key Features

### Core Capabilities
- **Vector Storage** - Efficient storage of high-dimensional vectors
- **Similarity Search** - Fast approximate nearest neighbor (ANN) search
- **Metadata Filtering** - Combine vector search with structured data filtering
- **Real-time Updates** - Dynamic vector insertion and updates
- **Multiple Distance Metrics** - Cosine, Euclidean, Dot product, and more
- **REST API** - Easy integration with any programming language

### Advanced Features
- **Payload Support** - Store additional metadata with vectors
- **Collections Management** - Organize vectors into named collections
- **Sharding** - Horizontal scaling across multiple nodes
- **Replication** - High availability and fault tolerance
- **Quantization** - Memory optimization with minimal accuracy loss
- **Filtering** - Complex boolean and range filters

## 🏗️ Architecture

### System Components
```
Qdrant Architecture
├── 🎯 Vector Engine
│   ├── HNSW Index
│   ├── IVF Index
│   └── Quantization
├── 💾 Storage Layer
│   ├── Vector Storage
│   ├── Payload Storage
│   └── Index Storage
├── 🔍 Search Engine
│   ├── Similarity Search
│   ├── Filtering Engine
│   └── Ranking Algorithms
├── 🌐 API Layer
│   ├── REST API
│   ├── gRPC API
│   └── WebSocket API
└── 🔧 Management
    ├── Collections
    ├── Sharding
    └── Replication
```

## 🛠️ Installation & Setup

### Docker Installation
```bash
# Pull Qdrant image
docker pull qdrant/qdrant

# Run Qdrant container
docker run -p 6333:6333 -p 6334:6334 \
  -v $(pwd)/qdrant_storage:/qdrant/storage:z \
  qdrant/qdrant

# Run with custom configuration
docker run -p 6333:6333 -p 6334:6334 \
  -v $(pwd)/qdrant_storage:/qdrant/storage:z \
  -v $(pwd)/config.yaml:/qdrant/config/production.yaml:z \
  qdrant/qdrant
```

### Binary Installation
```bash
# Download binary
wget https://github.com/qdrant/qdrant/releases/download/v1.6.1/qdrant
chmod +x qdrant

# Run Qdrant
./qdrant

# Run with configuration
./qdrant --config-path config.yaml
```

### Python Client Installation
```bash
# Install Python client
pip install qdrant-client

# Install with additional dependencies
pip install qdrant-client[fastembed]
```

## 📊 Basic Usage

### Python Client
```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct
import numpy as np

# Initialize client
client = QdrantClient(host="localhost", port=6333)

# Create collection
client.create_collection(
    collection_name="my_collection",
    vectors_config=VectorParams(size=384, distance=Distance.COSINE)
)

# Insert vectors
vectors = np.random.random((100, 384)).tolist()
points = [
    PointStruct(
        id=idx,
        vector=vector,
        payload={"text": f"Document {idx}", "category": "tech"}
    )
    for idx, vector in enumerate(vectors)
]

client.upsert(
    collection_name="my_collection",
    points=points
)

# Search similar vectors
query_vector = np.random.random(384).tolist()
search_result = client.search(
    collection_name="my_collection",
    query_vector=query_vector,
    limit=5
)

print("Search results:")
for result in search_result:
    print(f"ID: {result.id}, Score: {result.score}, Payload: {result.payload}")
```

### REST API
```bash
# Create collection
curl -X PUT "http://localhost:6333/collections/my_collection" \
  -H "Content-Type: application/json" \
  -d '{
    "vectors": {
      "size": 384,
      "distance": "Cosine"
    }
  }'

# Insert points
curl -X PUT "http://localhost:6333/collections/my_collection/points" \
  -H "Content-Type: application/json" \
  -d '{
    "points": [
      {
        "id": 1,
        "vector": [0.1, 0.2, 0.3, 0.4],
        "payload": {"text": "Document 1", "category": "tech"}
      }
    ]
  }'

# Search vectors
curl -X POST "http://localhost:6333/collections/my_collection/points/search" \
  -H "Content-Type: application/json" \
  -d '{
    "vector": [0.1, 0.2, 0.3, 0.4],
    "limit": 5
  }'
```

## 🔍 Advanced Features

### Filtering
```python
from qdrant_client.models import Filter, FieldCondition, MatchValue

# Search with filters
search_result = client.search(
    collection_name="my_collection",
    query_vector=query_vector,
    query_filter=Filter(
        must=[
            FieldCondition(
                key="category",
                match=MatchValue(value="tech")
            )
        ]
    ),
    limit=5
)

# Complex filtering
search_result = client.search(
    collection_name="my_collection",
    query_vector=query_vector,
    query_filter=Filter(
        must=[
            FieldCondition(
                key="category",
                match=MatchValue(value="tech")
            )
        ],
        should=[
            FieldCondition(
                key="rating",
                range={"gte": 4.0}
            )
        ]
    ),
    limit=5
)
```

### Batch Operations
```python
# Batch upsert
batch_size = 1000
for i in range(0, len(vectors), batch_size):
    batch_vectors = vectors[i:i + batch_size]
    batch_points = [
        PointStruct(
            id=idx + i,
            vector=vector,
            payload={"batch": i // batch_size}
        )
        for idx, vector in enumerate(batch_vectors)
    ]
    
    client.upsert(
        collection_name="my_collection",
        points=batch_points
    )
```

### Collection Management
```python
# Get collection info
collection_info = client.get_collection("my_collection")
print(f"Collection size: {collection_info.points_count}")

# Update collection parameters
client.update_collection(
    collection_name="my_collection",
    optimizer_config={
        "indexing_threshold": 20000
    }
)

# Delete collection
client.delete_collection("my_collection")
```

## 🚀 Production Deployment

### Docker Compose
```yaml
# docker-compose.yml
version: '3.8'

services:
  qdrant:
    image: qdrant/qdrant:latest
    ports:
      - "6333:6333"
      - "6334:6334"
    volumes:
      - qdrant_storage:/qdrant/storage
    environment:
      - QDRANT__SERVICE__HTTP_PORT=6333
      - QDRANT__SERVICE__GRPC_PORT=6334
    restart: unless-stopped

  qdrant-cluster:
    image: qdrant/qdrant:latest
    ports:
      - "6335:6333"
      - "6336:6334"
    volumes:
      - qdrant_storage_2:/qdrant/storage
    environment:
      - QDRANT__SERVICE__HTTP_PORT=6333
      - QDRANT__SERVICE__GRPC_PORT=6334
    restart: unless-stopped

volumes:
  qdrant_storage:
  qdrant_storage_2:
```

### Kubernetes Deployment
```yaml
# qdrant-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: qdrant
spec:
  replicas: 3
  selector:
    matchLabels:
      app: qdrant
  template:
    metadata:
      labels:
        app: qdrant
    spec:
      containers:
      - name: qdrant
        image: qdrant/qdrant:latest
        ports:
        - containerPort: 6333
        - containerPort: 6334
        env:
        - name: QDRANT__SERVICE__HTTP_PORT
          value: "6333"
        - name: QDRANT__SERVICE__GRPC_PORT
          value: "6334"
        volumeMounts:
        - name: qdrant-storage
          mountPath: /qdrant/storage
      volumes:
      - name: qdrant-storage
        persistentVolumeClaim:
          claimName: qdrant-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: qdrant-service
spec:
  selector:
    app: qdrant
  ports:
  - port: 6333
    targetPort: 6333
  - port: 6334
    targetPort: 6334
  type: LoadBalancer
```

## 🔧 Configuration

### Production Configuration
```yaml
# config.yaml
service:
  host: 0.0.0.0
  http_port: 6333
  grpc_port: 6334
  enable_cors: true
  max_request_size_mb: 32

storage:
  storage_path: ./storage
  snapshots_path: ./snapshots
  temp_path: ./temp
  on_disk_payload: true
  wal:
    wal_capacity_mb: 32
    wal_segments_ahead: 0

cluster:
  enabled: false
  p2p:
    port: 6335
  consensus:
    tick_period_ms: 100
    bootstrap_timeout_sec: 5

hnsw_config:
  m: 16
  ef_construct: 100
  full_scan_threshold: 10000
  max_indexing_threads: 4
  on_disk: false

optimizer_config:
  deleted_threshold: 0.2
  vacuum_min_vector_number: 1000
  default_segment_number: 0
  max_segment_size_kb: 200000
  memmap_threshold_kb: 50000
  indexing_threshold: 20000
  payload_indexing_threshold: 20000
  flush_interval_sec: 5
  max_optimization_threads: 2
```

## 🔍 Use Cases

### Semantic Search
```python
# Semantic search implementation
def semantic_search(query_text, collection_name, limit=10):
    # Generate query vector using embedding model
    query_vector = generate_embedding(query_text)
    
    # Search in Qdrant
    results = client.search(
        collection_name=collection_name,
        query_vector=query_vector,
        limit=limit
    )
    
    return results

# Usage
results = semantic_search(
    "machine learning algorithms",
    "documents_collection",
    limit=5
)
```

### Recommendation System
```python
# User-based recommendations
def get_user_recommendations(user_id, collection_name, limit=10):
    # Get user vector
    user_vector = get_user_vector(user_id)
    
    # Search for similar users
    similar_users = client.search(
        collection_name=collection_name,
        query_vector=user_vector,
        query_filter=Filter(
            must_not=[
                FieldCondition(
                    key="user_id",
                    match=MatchValue(value=user_id)
                )
            ]
        ),
        limit=limit
    )
    
    return similar_users
```

### Image Similarity Search
```python
# Image similarity search
def search_similar_images(image_path, collection_name, limit=5):
    # Extract image features
    image_vector = extract_image_features(image_path)
    
    # Search similar images
    results = client.search(
        collection_name=collection_name,
        query_vector=image_vector,
        limit=limit
    )
    
    return results
```

## 🔧 Best Practices

### Performance Optimization
1. **Batch Operations** - Use batch upsert for large datasets
2. **Index Configuration** - Tune HNSW parameters for your use case
3. **Memory Management** - Use quantization for memory optimization
4. **Payload Indexing** - Index frequently filtered fields
5. **Collection Sharding** - Distribute large collections across shards

### Data Management
1. **Vector Dimensions** - Keep vector dimensions reasonable (100-1000)
2. **Payload Size** - Minimize payload size for better performance
3. **Collection Organization** - Use multiple collections for different data types
4. **Backup Strategy** - Regular snapshots and backups
5. **Monitoring** - Monitor collection size and performance metrics

### Security
1. **Access Control** - Implement proper authentication and authorization
2. **Network Security** - Use TLS for production deployments
3. **Data Encryption** - Encrypt sensitive data in payloads
4. **Input Validation** - Validate all input data
5. **Rate Limiting** - Implement rate limiting for API endpoints

## 🚀 Advanced Features

### Custom Distance Metrics
```python
from qdrant_client.models import Distance

# Create collection with custom distance
client.create_collection(
    collection_name="custom_distance_collection",
    vectors_config=VectorParams(
        size=384,
        distance=Distance.DOT  # Dot product distance
    )
)
```

### Quantization
```python
# Create collection with quantization
client.create_collection(
    collection_name="quantized_collection",
    vectors_config=VectorParams(
        size=384,
        distance=Distance.COSINE
    ),
    quantization_config=ScalarQuantization(
        scalar=ScalarQuantizationConfig(
            type=ScalarType.INT8,
            quantile=0.99,
            always_ram=True
        )
    )
)
```

### Sharding
```python
# Create sharded collection
client.create_collection(
    collection_name="sharded_collection",
    vectors_config=VectorParams(
        size=384,
        distance=Distance.COSINE
    ),
    shard_number=4,
    replication_factor=2
)
```

## 🔍 Monitoring and Debugging

### Health Checks
```python
# Check cluster health
health = client.get_cluster_info()
print(f"Cluster status: {health.status}")

# Check collection health
collection_info = client.get_collection("my_collection")
print(f"Collection points: {collection_info.points_count}")
print(f"Indexed vectors: {collection_info.indexed_vectors_count}")
```

### Performance Monitoring
```python
# Monitor search performance
import time

start_time = time.time()
results = client.search(
    collection_name="my_collection",
    query_vector=query_vector,
    limit=10
)
search_time = time.time() - start_time

print(f"Search took {search_time:.3f} seconds")
print(f"Found {len(results)} results")
```

## 🐛 Troubleshooting

### Common Issues
1. **Memory Usage** - Monitor memory consumption and use quantization
2. **Search Performance** - Tune HNSW parameters and use proper indexing
3. **Storage Issues** - Check disk space and storage configuration
4. **Network Connectivity** - Verify network configuration and firewall rules
5. **Data Consistency** - Ensure proper error handling and retry logic

### Debugging Tools
```python
# Enable debug logging
import logging
logging.basicConfig(level=logging.DEBUG)

# Check collection statistics
stats = client.get_collection("my_collection")
print(f"Collection stats: {stats}")

# Validate collection
client.validate_collection("my_collection")
```

## 📚 Learning Resources

### Official Documentation
- [Qdrant Documentation](https://qdrant.tech/documentation/)
- [Python Client Guide](https://qdrant.tech/documentation/quick-start/)
- [REST API Reference](https://qdrant.github.io/qdrant/redoc/index.html)

### Community Resources
- [GitHub Repository](https://github.com/qdrant/qdrant)
- [Discord Community](https://discord.gg/qdrant)
- [Blog and Tutorials](https://qdrant.tech/articles/)

## 🔄 Updates and Roadmap

### Recent Updates
- **v1.6.1** - Performance improvements and bug fixes
- **v1.6.0** - Enhanced filtering and new distance metrics
- **v1.5.0** - Improved clustering and replication features

### Upcoming Features
- **v1.7.0** - Advanced analytics and monitoring
- **v1.8.0** - Enhanced security features
- **v1.9.0** - New vector search algorithms

## 🤝 Contributing

### How to Contribute
1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests and documentation
5. Submit a pull request

### Contribution Guidelines
- Follow Rust coding standards
- Write comprehensive tests
- Update documentation
- Ensure backward compatibility

## 📄 License

This project is licensed under the Apache License 2.0 - see the [LICENSE](https://github.com/qdrant/qdrant/blob/master/LICENSE) file for details.

## 🙏 Acknowledgments

- **Qdrant Team** for creating and maintaining Qdrant
- **Open Source Community** for contributions and feedback
- **Users** for providing real-world use cases and improvements
- **Contributors** for advancing vector database technology

---

<div align="center">

**🌟 Star this repository if you find it helpful!**

[⬆ Back to Top](#qdrant-)

</div>
