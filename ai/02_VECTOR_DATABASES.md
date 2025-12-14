# Vector Databases: Complete Guide

## Table of Contents
1. [What are Vector Databases?](#what-are-vector-databases)
2. [Choosing the Right Vector Database](#choosing-the-right-vector-database)
3. [Vector Compression & Quantization](#vector-compression--quantization)
4. [Indexing Techniques](#indexing-techniques)
5. [Search Execution Flow](#search-execution-flow)
6. [Milvus Deep Dive](#milvus-deep-dive)
7. [Production Implementation](#production-implementation)

---

## What are Vector Databases?

**Vector Databases** are specialized databases designed to store and search high-dimensional vectors efficiently. They're optimized for similarity search, not exact matches.

### Why Regular Databases Fail

```python
# Regular SQL database
SELECT * FROM documents 
WHERE embedding = [0.23, -0.45, ..., 0.12]  # Exact match - won't work!

# Vector databases support similarity search
SELECT * FROM documents 
ORDER BY cosine_similarity(embedding, query_vector) DESC
LIMIT 10
```

### Key Features

1. **Similarity Search**: Find vectors close to query vector
2. **Scalability**: Handle millions/billions of vectors
3. **Indexing**: Fast approximate nearest neighbor (ANN) search
4. **Metadata Filtering**: Combine vector search with traditional filters

---

## Choosing the Right Vector Database

### Comparison Matrix

| Database | Open Source | Cloud | Best For | Scalability | Performance |
|----------|------------|-------|----------|-------------|-------------|
| **Milvus** | ✅ | ✅ | Large-scale, production | Excellent | Very High |
| **Pinecone** | ❌ | ✅ | Managed, easy setup | Excellent | Very High |
| **Weaviate** | ✅ | ✅ | Graph + Vector search | Good | High |
| **Qdrant** | ✅ | ✅ | Fast, Rust-based | Good | Very High |
| **Chroma** | ✅ | ✅ | Simple, Python-first | Good | Medium |
| **FAISS** | ✅ | ❌ | Research, in-memory | Excellent | Very High |
| **Elasticsearch** | ✅ | ✅ | Text + Vector hybrid | Excellent | High |

### Decision Tree

```
Do you need managed service?
├─ Yes → Pinecone (easiest) or Milvus Cloud
└─ No → Continue

Do you need metadata filtering?
├─ Yes → Milvus, Weaviate, or Qdrant
└─ No → FAISS (in-memory) or Chroma

Scale requirements?
├─ < 1M vectors → Chroma, Qdrant
├─ 1M - 100M → Milvus, Qdrant, Weaviate
└─ > 100M → Milvus, Pinecone

Language preference?
├─ Python → Chroma, Milvus
├─ Rust → Qdrant
└─ Java → Weaviate
```

### Detailed Comparison

#### 1. Milvus

**Best For:** Large-scale production systems, complex queries

**Pros:**
- Excellent scalability (billions of vectors)
- Multiple index types (HNSW, IVF, FLAT)
- Strong metadata filtering
- Active community
- Cloud and self-hosted options

**Cons:**
- Steeper learning curve
- Requires more infrastructure

**Example:**
```python
from pymilvus import connections, Collection, FieldSchema, CollectionSchema, DataType

# Connect
connections.connect("default", host="localhost", port="19530")

# Define schema
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),
    FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=768),
    FieldSchema(name="text", dtype=DataType.VARCHAR, max_length=1000)
]
schema = CollectionSchema(fields, "Document collection")

# Create collection
collection = Collection("documents", schema)

# Create index
index_params = {
    "metric_type": "L2",
    "index_type": "HNSW",
    "params": {"M": 16, "efConstruction": 200}
}
collection.create_index("embedding", index_params)

# Insert data
data = [
    [1, 2, 3],  # IDs
    [[0.1, 0.2, ...], [0.3, 0.4, ...], ...],  # Embeddings
    ["doc1", "doc2", "doc3"]  # Text
]
collection.insert(data)
collection.load()

# Search
search_params = {"metric_type": "L2", "params": {"ef": 64}}
results = collection.search(
    data=[[0.1, 0.2, ...]],  # Query vector
    anns_field="embedding",
    param=search_params,
    limit=10
)
```

#### 2. Pinecone

**Best For:** Quick setup, managed service, production apps

**Pros:**
- Easiest to use
- Fully managed
- Great documentation
- Auto-scaling
- Pay-as-you-go pricing

**Cons:**
- Vendor lock-in
- Can be expensive at scale
- Less control over infrastructure

**Example:**
```python
import pinecone

# Initialize
pinecone.init(api_key="your-api-key", environment="us-west1-gcp")

# Create index
pinecone.create_index("documents", dimension=768, metric="cosine")

# Connect
index = pinecone.Index("documents")

# Upsert vectors
vectors = [
    ("1", [0.1, 0.2, ...], {"text": "document 1"}),
    ("2", [0.3, 0.4, ...], {"text": "document 2"})
]
index.upsert(vectors=vectors)

# Search
results = index.query(
    vector=[0.1, 0.2, ...],
    top_k=10,
    include_metadata=True
)
```

#### 3. Qdrant

**Best For:** Fast, Rust-based, good balance of features

**Pros:**
- Very fast (Rust implementation)
- Good Python client
- Payload filtering
- Open source + cloud

**Cons:**
- Smaller community than Milvus
- Less mature ecosystem

**Example:**
```python
from qdrant_client import QdrantClient
from qdrant_client.models import Distance, VectorParams, PointStruct

# Connect
client = QdrantClient("localhost", port=6333)

# Create collection
client.create_collection(
    collection_name="documents",
    vectors_config=VectorParams(size=768, distance=Distance.COSINE)
)

# Insert vectors
points = [
    PointStruct(
        id=1,
        vector=[0.1, 0.2, ...],
        payload={"text": "document 1"}
    ),
    PointStruct(
        id=2,
        vector=[0.3, 0.4, ...],
        payload={"text": "document 2"}
    )
]
client.upsert(collection_name="documents", points=points)

# Search
results = client.search(
    collection_name="documents",
    query_vector=[0.1, 0.2, ...],
    limit=10
)
```

#### 4. Chroma

**Best For:** Simple Python projects, prototyping

**Pros:**
- Easiest Python API
- Great for prototyping
- Lightweight
- Good documentation

**Cons:**
- Less scalable
- Fewer advanced features
- Not ideal for production at scale

**Example:**
```python
import chromadb

# Create client
client = chromadb.Client()

# Create collection
collection = client.create_collection("documents")

# Add documents
collection.add(
    embeddings=[[0.1, 0.2, ...], [0.3, 0.4, ...]],
    documents=["doc1", "doc2"],
    ids=["1", "2"]
)

# Query
results = collection.query(
    query_embeddings=[[0.1, 0.2, ...]],
    n_results=10
)
```

---

## Vector Compression & Quantization

### Why Compress Vectors?

**Problem:** 
- 1M vectors × 1536 dimensions × 4 bytes (float32) = **6.1 GB**
- At scale: 1B vectors = **6.1 TB**

**Solution:** Compression reduces storage and speeds up search

### Compression Techniques

#### 1. Scalar Quantization (INT8)

**Concept:** Convert float32 (4 bytes) → int8 (1 byte)

```python
# Original: float32 vector
original = np.array([0.234, -0.456, 0.789, ...], dtype=np.float32)
# Size: 1536 × 4 = 6144 bytes

# Quantize to INT8
def quantize_to_int8(vector):
    # Scale to [-128, 127] range
    scale = vector.max() - vector.min()
    offset = vector.min()
    quantized = ((vector - offset) / scale * 255 - 128).astype(np.int8)
    return quantized, scale, offset

quantized, scale, offset = quantize_to_int8(original)
# Size: 1536 × 1 = 1536 bytes (75% reduction!)

# Dequantize for search
def dequantize(quantized, scale, offset):
    return (quantized.astype(np.float32) + 128) / 255 * scale + offset
```

**Trade-off:**
- **Storage:** 75% reduction
- **Accuracy:** ~1-2% loss in search quality
- **Speed:** Faster (less memory bandwidth)

#### 2. Product Quantization (PQ)

**Concept:** Split vector into subvectors, quantize each separately

```python
def product_quantize(vector, m=8, k=256):
    """
    Split vector into m subvectors
    Quantize each subvector to k centroids
    """
    dim = len(vector)
    subvector_dim = dim // m
    
    # Split into m subvectors
    subvectors = vector.reshape(m, subvector_dim)
    
    # For each subvector, find closest centroid
    # (In practice, train centroids on sample data)
    codes = []
    for subvec in subvectors:
        # Find nearest centroid (k-means)
        centroid_id = find_nearest_centroid(subvec, centroids)
        codes.append(centroid_id)
    
    return codes  # m integers (one per subvector)

# Storage: m × log2(k) bits
# Example: 8 subvectors × 8 bits = 64 bits = 8 bytes
# vs original: 1536 × 4 = 6144 bytes
# Reduction: 99.87%!
```

**Use Case:** Very large datasets where storage is critical

#### 3. Binary Quantization (1-bit)

**Concept:** Convert to binary (0 or 1)

```python
def binary_quantize(vector):
    """Convert to binary: 1 if > 0, else 0"""
    return (vector > 0).astype(np.uint8)

# Storage: 1536 bits = 192 bytes
# vs original: 6144 bytes
# Reduction: 96.9%!

# Search: Use Hamming distance (XOR + popcount)
def hamming_distance(binary1, binary2):
    return np.sum(binary1 != binary2)
```

**Trade-off:**
- **Storage:** 97% reduction
- **Accuracy:** Significant loss (only for very large scale)
- **Speed:** Extremely fast (bit operations)

### Implementation in Vector Databases

**Milvus Quantization:**
```python
# Create index with quantization
index_params = {
    "metric_type": "L2",
    "index_type": "IVF_PQ",  # Product Quantization
    "params": {
        "nlist": 1024,  # Number of clusters
        "m": 8,  # Number of subvectors
        "nbits": 8  # Bits per subvector
    }
}
collection.create_index("embedding", index_params)
```

**Pinecone Quantization:**
```python
# Pinecone handles quantization automatically
# Uses optimized compression based on your data
# No manual configuration needed
```

---

## Indexing Techniques

### 1. Flat Index (Brute Force)

**How it works:** Compare query against all vectors

```python
def flat_search(query_vector, all_vectors, top_k=10):
    """Brute force search - exact but slow"""
    similarities = []
    for i, vector in enumerate(all_vectors):
        sim = cosine_similarity(query_vector, vector)
        similarities.append((i, sim))
    
    # Sort and return top k
    similarities.sort(key=lambda x: x[1], reverse=True)
    return similarities[:top_k]
```

**Characteristics:**
- **Accuracy:** 100% (exact)
- **Speed:** O(n) - slow for large datasets
- **Use Case:** Small datasets (< 100K vectors)

### 2. Inverted File Index (IVF)

**How it works:** Cluster vectors, search only relevant clusters

```python
# Step 1: Cluster vectors into nlist clusters
def build_ivf_index(vectors, nlist=1024):
    from sklearn.cluster import KMeans
    
    kmeans = KMeans(n_clusters=nlist)
    kmeans.fit(vectors)
    
    # Assign each vector to cluster
    cluster_assignments = kmeans.predict(vectors)
    
    # Build inverted index: cluster_id -> list of vector IDs
    inverted_index = {}
    for vector_id, cluster_id in enumerate(cluster_assignments):
        if cluster_id not in inverted_index:
            inverted_index[cluster_id] = []
        inverted_index[cluster_id].append(vector_id)
    
    return kmeans, inverted_index

# Step 2: Search
def ivf_search(query_vector, kmeans, inverted_index, vectors, nprobe=10, top_k=10):
    # Find nprobe closest clusters
    distances = kmeans.transform([query_vector])[0]
    closest_clusters = np.argsort(distances)[:nprobe]
    
    # Search only vectors in these clusters
    candidates = []
    for cluster_id in closest_clusters:
        for vector_id in inverted_index[cluster_id]:
            vector = vectors[vector_id]
            sim = cosine_similarity(query_vector, vector)
            candidates.append((vector_id, sim))
    
    # Return top k
    candidates.sort(key=lambda x: x[1], reverse=True)
    return candidates[:top_k]
```

**Characteristics:**
- **Accuracy:** ~95-99% (approximate)
- **Speed:** O(n/nlist × nprobe) - much faster
- **Use Case:** Medium datasets (1M - 100M vectors)

**Parameters:**
- `nlist`: Number of clusters (more = better accuracy, slower)
- `nprobe`: Clusters to search (more = better accuracy, slower)

### 3. HNSW (Hierarchical Navigable Small World)

**How it works:** Build multi-layer graph, navigate from top

```python
# Simplified HNSW concept
class HNSWIndex:
    def __init__(self, m=16, ef_construction=200):
        self.m = m  # Number of connections per node
        self.ef_construction = ef_construction  # Search width during construction
        self.layers = []  # Multiple layers (graph levels)
    
    def insert(self, vector, id):
        # Start from top layer
        current_level = len(self.layers) - 1
        
        # Find entry point (closest node in top layer)
        entry = self.find_entry_point(vector, current_level)
        
        # Navigate down, finding neighbors at each level
        for level in range(current_level, -1, -1):
            # Search for m nearest neighbors
            neighbors = self.search_layer(vector, entry, m, level)
            
            # Connect to these neighbors
            self.connect(vector, id, neighbors, level)
            
            # Move to next level
            entry = neighbors[0]
    
    def search(self, query_vector, ef=64, top_k=10):
        # Start from top layer
        entry = self.layers[-1][0]  # Top layer entry point
        
        # Navigate down to find candidates
        candidates = self.search_layer(query_vector, entry, ef, len(self.layers)-1)
        
        # Refine in bottom layer
        for level in range(len(self.layers)-2, -1, -1):
            candidates = self.search_layer(query_vector, candidates[0], ef, level)
        
        # Return top k
        return sorted(candidates, key=lambda x: x[1], reverse=True)[:top_k]
```

**Characteristics:**
- **Accuracy:** ~98-99% (very good)
- **Speed:** O(log n) - very fast
- **Use Case:** Large datasets, production systems

**Parameters:**
- `M`: Connections per node (16-64, more = better accuracy, more memory)
- `efConstruction`: Search width during build (100-500)
- `ef`: Search width during query (16-512)

### 4. LSH (Locality Sensitive Hashing)

**How it works:** Hash similar vectors to same bucket

```python
import random

class LSHIndex:
    def __init__(self, num_tables=10, num_hashes=10):
        self.num_tables = num_tables
        self.num_hashes = num_hashes
        self.tables = [{} for _ in range(num_tables)]
        
        # Generate random hash functions
        self.hash_functions = []
        for _ in range(num_tables):
            # Random hyperplane hash
            hyperplane = np.random.randn(768)  # Random vector
            self.hash_functions.append(hyperplane)
    
    def hash(self, vector, table_id):
        """Hash vector to bucket"""
        hyperplane = self.hash_functions[table_id]
        # Sign of dot product determines hash
        hash_value = 1 if np.dot(vector, hyperplane) >= 0 else 0
        return hash_value
    
    def insert(self, vector, id):
        """Insert into all hash tables"""
        for table_id in range(self.num_tables):
            bucket = self.hash(vector, table_id)
            if bucket not in self.tables[table_id]:
                self.tables[table_id][bucket] = []
            self.tables[table_id][bucket].append((id, vector))
    
    def search(self, query_vector, top_k=10):
        """Search in buckets that query hashes to"""
        candidates = set()
        
        for table_id in range(self.num_tables):
            bucket = self.hash(query_vector, table_id)
            if bucket in self.tables[table_id]:
                candidates.update(self.tables[table_id][bucket])
        
        # Compute similarity for candidates
        results = []
        for id, vector in candidates:
            sim = cosine_similarity(query_vector, vector)
            results.append((id, sim))
        
        return sorted(results, key=lambda x: x[1], reverse=True)[:top_k]
```

**Characteristics:**
- **Accuracy:** ~80-90% (lower than HNSW)
- **Speed:** O(1) average case - very fast
- **Use Case:** Very large datasets, approximate search

---

## Search Execution Flow

### Complete Search Pipeline

```
1. Query Input
   ↓
2. Query Embedding
   ↓
3. Index Selection
   ↓
4. Candidate Retrieval
   ↓
5. Re-ranking
   ↓
6. Metadata Filtering
   ↓
7. Result Return
```

### Detailed Flow

```python
class VectorSearchEngine:
    def __init__(self, collection, embedding_model):
        self.collection = collection
        self.embedding_model = embedding_model
        self.index = collection.get_index("embedding")
    
    def search(
        self, 
        query_text: str,
        top_k: int = 10,
        filters: dict = None,
        rerank: bool = True
    ):
        # Step 1: Convert query to embedding
        query_vector = self.embedding_model.encode(query_text)
        
        # Step 2: Index search (HNSW/IVF)
        # This returns approximate nearest neighbors
        candidates = self.index.search(
            query_vector,
            top_k=top_k * 10  # Get more candidates for re-ranking
        )
        
        # Step 3: Metadata filtering
        if filters:
            candidates = self.apply_filters(candidates, filters)
        
        # Step 4: Exact re-ranking (optional)
        if rerank:
            # Re-compute exact similarity for top candidates
            candidates = self.rerank(query_vector, candidates)
        
        # Step 5: Return top k
        return candidates[:top_k]
    
    def apply_filters(self, candidates, filters):
        """Apply metadata filters (e.g., date range, category)"""
        filtered = []
        for candidate in candidates:
            metadata = self.get_metadata(candidate.id)
            if self.matches_filters(metadata, filters):
                filtered.append(candidate)
        return filtered
    
    def rerank(self, query_vector, candidates):
        """Re-rank using exact similarity"""
        reranked = []
        for candidate in candidates:
            exact_sim = cosine_similarity(
                query_vector,
                self.get_vector(candidate.id)
            )
            reranked.append((candidate.id, exact_sim))
        return sorted(reranked, key=lambda x: x[1], reverse=True)
```

### Performance Optimization

**1. Two-Stage Search:**
```python
# Stage 1: Fast approximate search (HNSW)
candidates = hnsw_index.search(query, top_k=100)

# Stage 2: Exact re-ranking on top candidates
results = exact_search(query, candidates[:20])
```

**2. Parallel Search:**
```python
from concurrent.futures import ThreadPoolExecutor

def parallel_search(query, partitions):
    """Search multiple partitions in parallel"""
    with ThreadPoolExecutor() as executor:
        futures = [
            executor.submit(search_partition, query, partition)
            for partition in partitions
        ]
        results = [f.result() for f in futures]
    
    # Merge and return top k
    return merge_results(results)
```

**3. Caching:**
```python
from functools import lru_cache

@lru_cache(maxsize=1000)
def cached_search(query_hash, top_k):
    """Cache frequent queries"""
    return vector_db.search(query_vector, top_k)
```

---

## Milvus Deep Dive

### Architecture

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
┌──────▼──────────────────┐
│   Proxy (API Gateway)   │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   Coordinator          │
│   - Root Coordinator    │
│   - Data Coordinator    │
│   - Query Coordinator   │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   Worker Nodes          │
│   - Data Nodes          │
│   - Query Nodes         │
│   - Index Nodes         │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   Object Storage        │
│   (MinIO/S3)            │
└─────────────────────────┘
```

### Key Components

**1. Collections:**
```python
# Collection = Table (like in SQL)
collection = Collection("documents")

# Fields
fields = [
    FieldSchema(name="id", dtype=DataType.INT64, is_primary=True),
    FieldSchema(name="embedding", dtype=DataType.FLOAT_VECTOR, dim=768),
    FieldSchema(name="title", dtype=DataType.VARCHAR, max_length=200),
    FieldSchema(name="category", dtype=DataType.INT64)
]
```

**2. Partitions:**
```python
# Partition = Shard (for horizontal scaling)
collection.create_partition("partition_1")
collection.create_partition("partition_2")

# Insert to specific partition
collection.insert(data, partition_name="partition_1")
```

**3. Indexes:**
```python
# HNSW Index
index_params = {
    "metric_type": "L2",
    "index_type": "HNSW",
    "params": {
        "M": 16,  # Connections per node
        "efConstruction": 200  # Build-time search width
    }
}

# IVF_PQ Index (with quantization)
index_params = {
    "metric_type": "L2",
    "index_type": "IVF_PQ",
    "params": {
        "nlist": 1024,  # Number of clusters
        "m": 8,  # Subvectors
        "nbits": 8  # Bits per subvector
    }
}
```

### Production Setup

**Docker Compose:**
```yaml
version: '3.5'

services:
  etcd:
    image: quay.io/coreos/etcd:v3.5.5
    environment:
      - ETCD_AUTO_COMPACTION_MODE=revision
      - ETCD_AUTO_COMPACTION_RETENTION=1000
      - ETCD_QUOTA_BACKEND_BYTES=4294967296
      - ETCD_SNAPSHOT_COUNT=50000

  minio:
    image: minio/minio:RELEASE.2023-03-20T20-16-18Z
    environment:
      MINIO_ACCESS_KEY: minioadmin
      MINIO_SECRET_KEY: minioadmin
    command: minio server /minio_data

  standalone:
    image: milvusdb/milvus:v2.3.0
    command: ["milvus", "run", "standalone"]
    environment:
      ETCD_ENDPOINTS: etcd:2379
      MINIO_ADDRESS: minio:9000
    depends_on:
      - "etcd"
      - "minio"
    ports:
      - "19530:19530"
      - "9091:9091"
```

---

## Production Implementation

### Complete RAG System with Vector DB

```python
from pymilvus import connections, Collection
from sentence_transformers import SentenceTransformer
import openai

class RAGSystem:
    def __init__(self):
        # Connect to Milvus
        connections.connect("default", host="localhost", port="19530")
        self.collection = Collection("documents")
        self.collection.load()
        
        # Embedding model
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        
        # LLM
        self.llm_client = openai.OpenAI()
    
    def search_documents(self, query: str, top_k: int = 5):
        """Search relevant documents"""
        # Encode query
        query_embedding = self.embedding_model.encode(query)
        
        # Search in Milvus
        search_params = {"metric_type": "L2", "params": {"ef": 64}}
        results = self.collection.search(
            data=[query_embedding],
            anns_field="embedding",
            param=search_params,
            limit=top_k,
            output_fields=["text", "metadata"]
        )
        
        return results[0]
    
    def generate_answer(self, query: str, context_docs: list):
        """Generate answer using RAG"""
        # Build context
        context = "\n\n".join([doc.entity.get("text") for doc in context_docs])
        
        # Prompt
        prompt = f"""Context:
{context}

Question: {query}

Answer based on the context above:"""
        
        # Generate
        response = self.llm_client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}]
        )
        
        return response.choices[0].message.content

# Usage
rag = RAGSystem()
docs = rag.search_documents("What is machine learning?")
answer = rag.generate_answer("What is machine learning?", docs)
```

---

## Key Takeaways

1. **Choose database** based on scale, features, and deployment model
2. **Compression** reduces storage by 75-99% with minimal accuracy loss
3. **Indexing** (HNSW, IVF) enables fast approximate search
4. **Two-stage search** (approximate + exact) balances speed and accuracy
5. **Metadata filtering** combines vector search with traditional queries
6. **Production** requires proper infrastructure (etcd, object storage)

---

## Next Steps

- [LLM Architecture Guide](03_LLM_ARCHITECTURE.md) - Understand how LLMs work
- [RAG Implementation Guide](06_REASONING_AGENTS.md) - Build production RAG systems

---

## References

- [Milvus Documentation](https://milvus.io/docs)
- [Vector Database Comparison](https://www.pinecone.io/learn/vector-database/)
- [FAISS Documentation](https://github.com/facebookresearch/faiss)

