# Production AI Systems: Deployment Guide

## Table of Contents
1. [System Architecture](#system-architecture)
2. [Model Serving](#model-serving)
3. [RAG Production Setup](#rag-production-setup)
4. [Monitoring & Observability](#monitoring--observability)
5. [Cost Optimization](#cost-optimization)
6. [Security & Compliance](#security--compliance)

---

## System Architecture

### Production RAG System

```
┌─────────────┐
│   Client    │
└──────┬──────┘
       │
┌──────▼──────────────────┐
│   API Gateway           │
│   - Rate Limiting       │
│   - Authentication      │
└──────┬──────────────────┘
       │
┌──────▼──────────────────┐
│   RAG Service           │
│   - Query Processing    │
│   - Embedding           │
│   - Retrieval            │
│   - Generation           │
└──────┬──────────────────┘
       │
┌──────┬──────────────────┐
│      │                  │
▼      ▼                  ▼
┌─────────┐  ┌─────────┐  ┌─────────┐
│ Vector  │  │   LLM   │  │  Cache  │
│   DB    │  │ Service │  │ (Redis) │
└─────────┘  └─────────┘  └─────────┘
```

### Implementation

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import redis
from rag_system import RAGSystem

app = FastAPI()
rag = RAGSystem()
cache = redis.Redis(host='localhost', port=6379)

class QueryRequest(BaseModel):
    query: str
    top_k: int = 5

@app.post("/query")
async def query(request: QueryRequest):
    # Check cache
    cache_key = f"query:{hash(request.query)}"
    cached = cache.get(cache_key)
    if cached:
        return json.loads(cached)
    
    # Process query
    result = rag.query(request.query, top_k=request.top_k)
    
    # Cache result
    cache.setex(cache_key, 3600, json.dumps(result))
    
    return result
```

---

## Model Serving

### Options

**1. vLLM (Recommended):**
```python
from vllm import LLM

llm = LLM(model="gpt-3.5-turbo", max_model_len=4096)
# Fast inference with paged attention
```

**2. TensorRT-LLM:**
```python
# NVIDIA optimized inference
# Best for NVIDIA GPUs
```

**3. OpenAI API:**
```python
# Managed service
# No infrastructure needed
```

### Deployment

```yaml
# docker-compose.yml
version: '3.8'
services:
  vllm:
    image: vllm/vllm-openai:latest
    ports:
      - "8000:8000"
    environment:
      - MODEL=gpt-3.5-turbo
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
```

---

## RAG Production Setup

### Complete Pipeline

```python
class ProductionRAG:
    def __init__(self):
        # Embedding service
        self.embedding_service = EmbeddingService(
            model="text-embedding-ada-002",
            cache=True
        )
        
        # Vector database
        self.vector_db = MilvusClient(
            host="milvus",
            port="19530"
        )
        
        # LLM service
        self.llm = OpenAI(
            api_key=os.getenv("OPENAI_API_KEY"),
            timeout=30
        )
        
        # Cache
        self.cache = Redis(host="redis", port=6379)
    
    def query(self, query: str, top_k: int = 5):
        # 1. Check cache
        cache_key = f"rag:{hash(query)}"
        cached = self.cache.get(cache_key)
        if cached:
            return json.loads(cached)
        
        # 2. Embed query
        query_embedding = self.embedding_service.encode(query)
        
        # 3. Retrieve documents
        docs = self.vector_db.search(
            query_embedding,
            top_k=top_k
        )
        
        # 4. Generate answer
        answer = self.llm.generate(
            query=query,
            context=docs
        )
        
        # 5. Cache result
        result = {"answer": answer, "sources": docs}
        self.cache.setex(cache_key, 3600, json.dumps(result))
        
        return result
```

---

## Monitoring & Observability

### Key Metrics

```python
from prometheus_client import Counter, Histogram, Gauge

# Metrics
query_counter = Counter('rag_queries_total', 'Total queries')
latency_histogram = Histogram('rag_latency_seconds', 'Query latency')
cache_hit_rate = Gauge('rag_cache_hit_rate', 'Cache hit rate')
error_counter = Counter('rag_errors_total', 'Total errors')

@app.post("/query")
async def query(request: QueryRequest):
    query_counter.inc()
    start_time = time.time()
    
    try:
        result = rag.query(request.query)
        latency_histogram.observe(time.time() - start_time)
        return result
    except Exception as e:
        error_counter.inc()
        raise HTTPException(status_code=500, detail=str(e))
```

### Logging

```python
import logging
import structlog

logger = structlog.get_logger()

@app.post("/query")
async def query(request: QueryRequest):
    logger.info("query_received", query=request.query)
    
    try:
        result = rag.query(request.query)
        logger.info("query_success", 
                   query=request.query,
                   latency=time.time() - start_time)
        return result
    except Exception as e:
        logger.error("query_failed",
                    query=request.query,
                    error=str(e))
        raise
```

---

## Cost Optimization

### Strategies

**1. Caching:**
```python
# Cache frequent queries
cache_ttl = 3600  # 1 hour
```

**2. Batch Processing:**
```python
# Process multiple queries together
def batch_embed(queries: List[str]):
    return embedding_model.encode(queries, batch_size=32)
```

**3. Model Selection:**
```python
# Use smaller models when possible
if query_complexity == "simple":
    model = "gpt-3.5-turbo"  # Cheaper
else:
    model = "gpt-4"  # More capable
```

**4. Quantization:**
```python
# Use quantized models
model = load_quantized_model("gpt-3.5-turbo-int8")
```

---

## Security & Compliance

### Best Practices

**1. API Keys:**
```python
# Use environment variables
api_key = os.getenv("OPENAI_API_KEY")
# Never commit to git
```

**2. Input Validation:**
```python
from pydantic import BaseModel, validator

class QueryRequest(BaseModel):
    query: str
    
    @validator('query')
    def validate_query(cls, v):
        if len(v) > 1000:
            raise ValueError('Query too long')
        # Check for injection attacks
        if any(char in v for char in ['<', '>', 'script']):
            raise ValueError('Invalid characters')
        return v
```

**3. Rate Limiting:**
```python
from slowapi import Limiter

limiter = Limiter(key_func=get_remote_address)

@app.post("/query")
@limiter.limit("10/minute")
async def query(request: Request, query_request: QueryRequest):
    return rag.query(query_request.query)
```

---

## Key Takeaways

1. **Use vLLM** for fast inference
2. **Cache aggressively** to reduce costs
3. **Monitor** latency, errors, cache hit rate
4. **Secure** API keys and validate inputs
5. **Optimize** with quantization and batching

---

## Next Steps

- Review all guides for complete understanding
- Set up development environment
- Build your first RAG system

---

## References

- [vLLM Documentation](https://docs.vllm.ai/)
- [Production ML Systems](https://www.oreilly.com/library/view/designing-machine-learning/9781098107956/)

