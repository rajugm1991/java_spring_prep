# Vector Embeddings & Semantic Space

## Table of Contents
1. [What are Vector Embeddings?](#what-are-vector-embeddings)
2. [How Vectors are Constructed](#how-vectors-are-constructed)
3. [Semantic Space Understanding](#semantic-space-understanding)
4. [Embedding Models Comparison](#embedding-models-comparison)
5. [Practical Implementation](#practical-implementation)
6. [Use Cases](#use-cases)

---

## What are Vector Embeddings?

**Vector Embeddings** are numerical representations of text, images, or other data in a high-dimensional space where similar items are positioned close together.

### Key Concept
```
Text: "The cat sat on the mat"
      ↓ (Embedding Model)
Vector: [0.23, -0.45, 0.67, ..., 0.12]  (768 or 1536 dimensions)
```

### Why Embeddings Matter

1. **Semantic Similarity**: Words with similar meanings are close in vector space
2. **Mathematical Operations**: Can do arithmetic on concepts
   - `king - man + woman ≈ queen`
3. **Search & Retrieval**: Find similar content quickly
4. **Machine Learning**: Input format for neural networks

---

## How Vectors are Constructed

### 1. Word-Level Embeddings (Word2Vec, GloVe)

**Traditional Approach:**
```python
# Word2Vec Training Process
# 1. Create training pairs from context
sentence = "the cat sat on the mat"
# Context window = 2
# Training pairs:
# (cat, the), (cat, sat), (cat, on)
# (sat, the), (sat, cat), (sat, on), (sat, mat)
# ...

# 2. Neural network learns to predict context
# Input: one-hot encoded word
# Output: probability distribution over vocabulary
# Hidden layer weights become the embedding
```

**Example:**
```python
from gensim.models import Word2Vec

sentences = [
    ["the", "cat", "sat", "on", "the", "mat"],
    ["the", "dog", "ran", "in", "the", "park"],
    ["cats", "and", "dogs", "are", "pets"]
]

model = Word2Vec(sentences, vector_size=100, window=5, min_count=1)
# Each word gets a 100-dimensional vector
cat_vector = model.wv['cat']  # [0.23, -0.45, ..., 0.12]
```

**Limitations:**
- One vector per word (no context)
- Can't handle out-of-vocabulary words
- Fixed vocabulary size

---

### 2. Contextual Embeddings (BERT, ELMo)

**Key Innovation:** Same word gets different vectors based on context

```python
# BERT Architecture
sentence = "I deposited money in the bank"
# "bank" in financial context

sentence2 = "I sat by the river bank"
# "bank" in geographical context

# BERT produces different embeddings for "bank" in each context
```

**How BERT Works:**
```python
# 1. Tokenization
tokens = ["[CLS]", "I", "deposited", "money", "in", "the", "bank", "[SEP]"]

# 2. Token Embeddings
token_emb = embedding_layer(tokens)  # [batch, seq_len, 768]

# 3. Position Embeddings
pos_emb = positional_encoding(seq_len)  # [seq_len, 768]

# 4. Segment Embeddings (for sentence pairs)
seg_emb = segment_embeddings(tokens)  # [seq_len, 768]

# 5. Combined Input
input_emb = token_emb + pos_emb + seg_emb  # [batch, seq_len, 768]

# 6. Transformer Layers (12 or 24 layers)
for layer in transformer_layers:
    input_emb = layer(input_emb)

# 7. Output: Contextual embeddings for each token
# Each token's embedding depends on all other tokens
```

**Code Example:**
```python
from transformers import BertTokenizer, BertModel
import torch

tokenizer = BertTokenizer.from_pretrained('bert-base-uncased')
model = BertModel.from_pretrained('bert-base-uncased')

# Sentence 1: Financial context
text1 = "I deposited money in the bank"
inputs1 = tokenizer(text1, return_tensors='pt')
outputs1 = model(**inputs1)
bank_embedding_1 = outputs1.last_hidden_state[0][6]  # "bank" token

# Sentence 2: Geographical context
text2 = "I sat by the river bank"
inputs2 = tokenizer(text2, return_tensors='pt')
outputs2 = model(**inputs2)
bank_embedding_2 = outputs2.last_hidden_state[0][6]  # "bank" token

# These embeddings are different!
cosine_similarity = torch.nn.functional.cosine_similarity(
    bank_embedding_1.unsqueeze(0),
    bank_embedding_2.unsqueeze(0)
)
# Similarity < 1.0 (different meanings)
```

---

### 3. Sentence-Level Embeddings (Sentence-BERT, OpenAI Embeddings)

**Goal:** Get one vector for entire sentence/document

**Approach 1: Mean Pooling**
```python
# Average all token embeddings
sentence = "The cat sat on the mat"
tokens = tokenizer(sentence)
token_embeddings = model(tokens).last_hidden_state  # [1, 7, 768]
sentence_embedding = token_embeddings.mean(dim=1)  # [1, 768]
```

**Approach 2: CLS Token (BERT)**
```python
# Use [CLS] token embedding
sentence = "The cat sat on the mat"
tokens = tokenizer(sentence)
outputs = model(tokens)
sentence_embedding = outputs.last_hidden_state[0][0]  # [CLS] token
```

**Approach 3: Sentence-BERT (Siamese Network)**
```python
from sentence_transformers import SentenceTransformer

model = SentenceTransformer('all-MiniLM-L6-v2')

# Direct sentence embedding
sentences = [
    "The cat sat on the mat",
    "A feline was sitting on a rug"
]

embeddings = model.encode(sentences)
# embeddings[0] and embeddings[1] are similar (high cosine similarity)
```

**How Sentence-BERT is Trained:**
```python
# Training with sentence pairs
# Positive pairs: Similar sentences
# Negative pairs: Dissimilar sentences

# Loss function: Contrastive loss
# Maximize similarity for positive pairs
# Minimize similarity for negative pairs

def contrastive_loss(anchor, positive, negative, margin=0.5):
    pos_sim = cosine_similarity(anchor, positive)
    neg_sim = cosine_similarity(anchor, negative)
    loss = max(0, margin - pos_sim + neg_sim)
    return loss
```

---

### 4. Modern Embedding Models (OpenAI, Cohere)

**OpenAI text-embedding-ada-002:**
```python
import openai

response = openai.Embedding.create(
    input="The cat sat on the mat",
    model="text-embedding-ada-002"
)

embedding = response['data'][0]['embedding']
# 1536-dimensional vector
# Optimized for semantic similarity
# Good for RAG applications
```

**Characteristics:**
- **Dimension:** 1536
- **Normalized:** L2 normalized (unit vector)
- **Optimized for:** Semantic search, clustering
- **Training:** On large text corpus with contrastive learning

**Cohere Embeddings:**
```python
import cohere

co = cohere.Client(api_key="your-api-key")
response = co.embed(
    texts=["The cat sat on the mat"],
    model="embed-english-v3.0"
)

embedding = response.embeddings[0]
# 1024-dimensional vector
# Supports different input types (search_document, search_query)
```

---

## Semantic Space Understanding

### What is Semantic Space?

**Semantic Space** is the high-dimensional vector space where embeddings live. Points close together have similar meanings.

### Visualization (2D Projection)

```
High-dimensional space (768D) → 2D projection

        "king"
          |
    "queen"  "prince"
          |
    "man"  "woman"
          |
    "cat"  "dog"
          |
    "car"  "truck"
```

### Mathematical Properties

**1. Cosine Similarity**
```python
import numpy as np
from sklearn.metrics.pairwise import cosine_similarity

def cosine_similarity(vec1, vec2):
    """
    Measures angle between vectors
    Range: [-1, 1]
    1.0 = identical
    0.0 = orthogonal
    -1.0 = opposite
    """
    dot_product = np.dot(vec1, vec2)
    norm1 = np.linalg.norm(vec1)
    norm2 = np.linalg.norm(vec2)
    return dot_product / (norm1 * norm2)

# Example
cat = [0.2, 0.3, 0.1, ...]
dog = [0.25, 0.28, 0.12, ...]
car = [0.8, 0.1, 0.9, ...]

similarity_cat_dog = cosine_similarity(cat, dog)  # ~0.85 (high)
similarity_cat_car = cosine_similarity(cat, car)  # ~0.15 (low)
```

**2. Euclidean Distance**
```python
def euclidean_distance(vec1, vec2):
    """
    Straight-line distance in vector space
    Lower = more similar
    """
    return np.linalg.norm(vec1 - vec2)

# For normalized vectors, cosine similarity is preferred
# (more robust to vector magnitude)
```

**3. Semantic Arithmetic**
```python
# Famous example: king - man + woman ≈ queen
king = model.encode("king")
man = model.encode("man")
woman = model.encode("woman")

result = king - man + woman
# Find closest vector to result
queen = model.encode("queen")

similarity = cosine_similarity(result, queen)  # ~0.8
```

---

## Embedding Models Comparison

| Model | Dimensions | Max Length | Use Case | Cost |
|-------|-----------|------------|----------|------|
| **OpenAI ada-002** | 1536 | 8191 tokens | General purpose, RAG | Paid |
| **Cohere v3** | 1024 | 512 tokens | Search, classification | Paid |
| **Sentence-BERT** | 384-768 | 512 tokens | Open source, fast | Free |
| **BGE-large** | 1024 | 512 tokens | Multilingual, open | Free |
| **E5-large** | 1024 | 512 tokens | Instruction-tuned | Free |

### Choosing the Right Model

**For RAG (Retrieval Augmented Generation):**
```python
# Use models optimized for retrieval
# OpenAI ada-002 or Cohere v3
# They're trained specifically for semantic search

query = "What is machine learning?"
documents = ["ML is...", "AI involves...", "Deep learning..."]

# Encode with different instructions
query_embedding = model.encode(query, instruction="query")
doc_embeddings = model.encode(documents, instruction="document")

# Better retrieval performance
```

**For Classification:**
```python
# Sentence-BERT is sufficient and free
model = SentenceTransformer('all-MiniLM-L6-v2')
embeddings = model.encode(texts)
# Use with classifier (SVM, logistic regression)
```

**For Multilingual:**
```python
# Use multilingual models
model = SentenceTransformer('paraphrase-multilingual-MiniLM-L12-v2')
# Supports 50+ languages
```

---

## Practical Implementation

### Complete Example: Building Embeddings Pipeline

```python
from sentence_transformers import SentenceTransformer
import numpy as np
from typing import List, Union

class EmbeddingService:
    def __init__(self, model_name: str = 'all-MiniLM-L6-v2'):
        """
        Initialize embedding model
        
        Args:
            model_name: HuggingFace model identifier
        """
        self.model = SentenceTransformer(model_name)
        self.dimension = self.model.get_sentence_embedding_dimension()
    
    def encode(
        self, 
        texts: Union[str, List[str]], 
        batch_size: int = 32,
        normalize: bool = True
    ) -> np.ndarray:
        """
        Encode text(s) into embeddings
        
        Args:
            texts: Single text or list of texts
            batch_size: Batch size for encoding
            normalize: L2 normalize embeddings
        
        Returns:
            numpy array of embeddings
        """
        if isinstance(texts, str):
            texts = [texts]
        
        embeddings = self.model.encode(
            texts,
            batch_size=batch_size,
            normalize_embeddings=normalize,
            show_progress_bar=True
        )
        
        return embeddings
    
    def similarity(
        self, 
        query: str, 
        documents: List[str]
    ) -> List[tuple]:
        """
        Find most similar documents to query
        
        Args:
            query: Search query
            documents: List of documents to search
        
        Returns:
            List of (document, similarity_score) tuples, sorted
        """
        query_emb = self.encode(query)
        doc_embs = self.encode(documents)
        
        # Compute cosine similarity
        similarities = np.dot(doc_embs, query_emb)
        
        # Sort by similarity
        results = list(zip(documents, similarities))
        results.sort(key=lambda x: x[1], reverse=True)
        
        return results

# Usage
service = EmbeddingService()

# Single text
text = "The cat sat on the mat"
embedding = service.encode(text)
print(f"Embedding shape: {embedding.shape}")  # (384,)

# Multiple texts
texts = [
    "The cat sat on the mat",
    "A feline was sitting on a rug",
    "I love programming in Python"
]
embeddings = service.encode(texts)
print(f"Embeddings shape: {embeddings.shape}")  # (3, 384)

# Similarity search
query = "cat on mat"
documents = [
    "The cat sat on the mat",
    "Dogs are great pets",
    "Cats like to sit on soft surfaces",
    "Python is a programming language"
]

results = service.similarity(query, documents)
for doc, score in results:
    print(f"Score: {score:.3f} - {doc}")
```

### Production Considerations

**1. Caching Embeddings**
```python
import hashlib
import pickle
from functools import lru_cache

class CachedEmbeddingService(EmbeddingService):
    def __init__(self, cache_dir: str = "./embeddings_cache"):
        super().__init__()
        self.cache_dir = cache_dir
        os.makedirs(cache_dir, exist_ok=True)
    
    def _get_cache_key(self, text: str) -> str:
        """Generate cache key from text"""
        return hashlib.md5(text.encode()).hexdigest()
    
    def encode(self, texts: Union[str, List[str]], **kwargs):
        """Encode with caching"""
        if isinstance(texts, str):
            texts = [texts]
        
        results = []
        texts_to_encode = []
        indices = []
        
        for i, text in enumerate(texts):
            cache_key = self._get_cache_key(text)
            cache_path = os.path.join(self.cache_dir, f"{cache_key}.pkl")
            
            if os.path.exists(cache_path):
                with open(cache_path, 'rb') as f:
                    results.append((i, pickle.load(f)))
            else:
                texts_to_encode.append(text)
                indices.append(i)
        
        # Encode uncached texts
        if texts_to_encode:
            new_embeddings = super().encode(texts_to_encode, **kwargs)
            
            # Cache new embeddings
            for text, emb in zip(texts_to_encode, new_embeddings):
                cache_key = self._get_cache_key(text)
                cache_path = os.path.join(self.cache_dir, f"{cache_key}.pkl")
                with open(cache_path, 'wb') as f:
                    pickle.dump(emb, f)
            
            # Add to results
            for idx, emb in zip(indices, new_embeddings):
                results.append((idx, emb))
        
        # Sort by original index and return
        results.sort(key=lambda x: x[0])
        return np.array([emb for _, emb in results])
```

**2. Batch Processing**
```python
def encode_large_dataset(
    texts: List[str],
    batch_size: int = 100,
    model_name: str = 'all-MiniLM-L6-v2'
) -> np.ndarray:
    """Encode large dataset in batches"""
    model = SentenceTransformer(model_name)
    all_embeddings = []
    
    for i in range(0, len(texts), batch_size):
        batch = texts[i:i+batch_size]
        embeddings = model.encode(batch, show_progress_bar=True)
        all_embeddings.append(embeddings)
    
    return np.vstack(all_embeddings)
```

**3. GPU Acceleration**
```python
import torch

# Check if GPU available
device = 'cuda' if torch.cuda.is_available() else 'cpu'
model = SentenceTransformer('all-MiniLM-L6-v2')
model = model.to(device)

# Embeddings will use GPU automatically
embeddings = model.encode(texts)  # Runs on GPU if available
```

---

## Use Cases

### 1. Semantic Search
```python
# Find documents similar to query
query = "machine learning algorithms"
documents = load_documents()  # Large corpus

service = EmbeddingService()
results = service.similarity(query, documents, top_k=10)
```

### 2. Clustering
```python
from sklearn.cluster import KMeans

embeddings = service.encode(texts)
kmeans = KMeans(n_clusters=5)
clusters = kmeans.fit_predict(embeddings)

# Group texts by cluster
for i, cluster_id in enumerate(clusters):
    print(f"Cluster {cluster_id}: {texts[i]}")
```

### 3. Duplicate Detection
```python
def find_duplicates(texts: List[str], threshold: float = 0.95):
    """Find duplicate or near-duplicate texts"""
    embeddings = service.encode(texts)
    duplicates = []
    
    for i in range(len(texts)):
        for j in range(i+1, len(texts)):
            similarity = cosine_similarity(
                embeddings[i].reshape(1, -1),
                embeddings[j].reshape(1, -1)
            )[0][0]
            
            if similarity >= threshold:
                duplicates.append((i, j, similarity))
    
    return duplicates
```

### 4. Recommendation Systems
```python
# User preferences → embedding
# Item descriptions → embeddings
# Recommend items with high similarity to user preferences

user_preferences = "I like action movies with sci-fi elements"
movies = load_movie_descriptions()

recommendations = service.similarity(user_preferences, movies, top_k=10)
```

---

## Key Takeaways

1. **Embeddings convert text to numbers** in a way that preserves semantic meaning
2. **Contextual embeddings** (BERT) are better than static (Word2Vec)
3. **Sentence-level embeddings** are essential for document search
4. **Cosine similarity** is the standard metric for comparing embeddings
5. **Choose models** based on your use case (RAG, classification, multilingual)
6. **Cache embeddings** for production systems
7. **Batch processing** is crucial for large datasets

---

## Next Steps

- [Vector Databases Guide](02_VECTOR_DATABASES.md) - Learn how to store and search embeddings efficiently
- [LLM Architecture Guide](03_LLM_ARCHITECTURE.md) - Understand how LLMs use embeddings

---

## References

- [Sentence Transformers Documentation](https://www.sbert.net/)
- [OpenAI Embeddings Guide](https://platform.openai.com/docs/guides/embeddings)
- [Understanding Word Embeddings](https://towardsdatascience.com/introduction-to-word-embedding-and-word2vec-652d0c2060fa)

