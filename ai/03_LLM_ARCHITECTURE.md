# LLM Architecture: Complete Guide

## Table of Contents
1. [Introduction to LLMs](#introduction-to-llms)
2. [How LLMs Work](#how-llms-work)
3. [LLM Text Generation](#llm-text-generation)
4. [LLM Improvements](#llm-improvements)
5. [Retrieval Augmented Generation (RAG)](#retrieval-augmented-generation-rag)
6. [Positional Embeddings](#positional-embeddings)
7. [Attention Mechanism](#attention-mechanism)
8. [Transformer Architecture](#transformer-architecture)
9. [KV Cache Optimization](#kv-cache-optimization)

---

## Introduction to LLMs

### What are Large Language Models (LLMs)?

**Large Language Models (LLMs)** are neural networks trained on massive text corpora to predict the next token in a sequence. They can generate human-like text, answer questions, translate languages, and perform various NLP tasks.

### Key Characteristics

1. **Scale:** Billions of parameters (GPT-3: 175B, GPT-4: ~1.7T)
2. **Training Data:** Trained on internet-scale text (trillions of tokens)
3. **Architecture:** Based on Transformer architecture
4. **Capabilities:** Zero-shot and few-shot learning

### Evolution Timeline

```
2017: Transformer (Attention Is All You Need)
  ↓
2018: GPT-1 (117M parameters)
  ↓
2019: GPT-2 (1.5B parameters), BERT
  ↓
2020: GPT-3 (175B parameters)
  ↓
2022: ChatGPT (GPT-3.5), InstructGPT
  ↓
2023: GPT-4, Claude, LLaMA
  ↓
2024: GPT-4 Turbo, Claude 3, Gemini
```

---

## How LLMs Work

### Core Concept: Next Token Prediction

**Fundamental Idea:** Given a sequence of tokens, predict the next token.

```
Input: "The cat sat on the"
  ↓
Model predicts: "mat" (with probability 0.85)
              "floor" (with probability 0.10)
              "sofa" (with probability 0.05)
```

### Training Process

**1. Tokenization:**
```python
# Text → Tokens
text = "The cat sat on the mat"
tokens = tokenizer.encode(text)
# [101, 1996, 4937, 2919, 2006, 1996, 3407, 102]
# [CLS, The, cat, sat, on, the, mat, SEP]
```

**2. Embedding:**
```python
# Tokens → Embeddings
token_embeddings = embedding_layer(tokens)  # [seq_len, d_model]
# Each token becomes a 768 or 4096-dimensional vector
```

**3. Transformer Processing:**
```python
# Process through transformer layers
x = token_embeddings
for layer in transformer_layers:
    x = layer(x)  # Self-attention + FFN
# Output: [seq_len, d_model]
```

**4. Prediction:**
```python
# Final layer: predict next token
logits = output_layer(x[-1])  # Last token's representation
# logits: [vocab_size] - scores for each token in vocabulary

probabilities = softmax(logits)
next_token = argmax(probabilities)  # Most likely token
```

### Complete Forward Pass

```python
class LLM:
    def __init__(self, vocab_size, d_model, n_layers, n_heads):
        self.embedding = Embedding(vocab_size, d_model)
        self.positional_encoding = PositionalEncoding(d_model)
        self.layers = [TransformerBlock(d_model, n_heads) 
                      for _ in range(n_layers)]
        self.output_layer = Linear(d_model, vocab_size)
    
    def forward(self, tokens):
        # 1. Embedding
        x = self.embedding(tokens)  # [batch, seq_len, d_model]
        
        # 2. Positional encoding
        x = self.positional_encoding(x)
        
        # 3. Transformer layers
        for layer in self.layers:
            x = layer(x)  # Self-attention + FFN
        
        # 4. Predict next token
        logits = self.output_layer(x)  # [batch, seq_len, vocab_size]
        
        return logits
```

### Training Objective

**Language Modeling Loss:**
```python
def language_modeling_loss(predictions, targets):
    """
    Cross-entropy loss for next token prediction
    
    predictions: [batch, seq_len, vocab_size]
    targets: [batch, seq_len] - actual next tokens
    """
    # Flatten
    predictions = predictions.view(-1, vocab_size)
    targets = targets.view(-1)
    
    # Cross-entropy
    loss = cross_entropy(predictions, targets)
    return loss

# Example
input_text = "The cat sat on the"
target_text = "cat sat on the mat"

# Model predicts next token for each position
# Loss = average cross-entropy across all positions
```

---

## LLM Text Generation

### Autoregressive Generation

**Process:** Generate one token at a time, using previously generated tokens.

```python
def generate_text(model, prompt, max_length=100, temperature=1.0):
    """
    Generate text autoregressively
    
    Args:
        model: LLM model
        prompt: Initial text
        max_length: Maximum generation length
        temperature: Controls randomness (0.0 = deterministic, 1.0 = creative)
    """
    tokens = tokenizer.encode(prompt)
    generated = tokens.copy()
    
    for _ in range(max_length):
        # Get predictions
        logits = model(torch.tensor([generated]))
        
        # Get next token probabilities
        next_token_logits = logits[0, -1, :]  # Last position
        probabilities = softmax(next_token_logits / temperature)
        
        # Sample next token
        next_token = sample(probabilities)  # Random sampling
        
        # Stop if end token
        if next_token == tokenizer.eos_token_id:
            break
        
        # Append to sequence
        generated.append(next_token)
    
    # Decode to text
    text = tokenizer.decode(generated)
    return text
```

### Sampling Strategies

**1. Greedy Decoding:**
```python
# Always pick most likely token
next_token = argmax(probabilities)
# Deterministic, but can be repetitive
```

**2. Temperature Sampling:**
```python
# Adjust randomness
temperature = 0.7  # Lower = more deterministic
adjusted_logits = logits / temperature
probabilities = softmax(adjusted_logits)
next_token = sample(probabilities)
```

**3. Top-k Sampling:**
```python
# Sample from top k most likely tokens
k = 50
top_k_logits, top_k_indices = torch.topk(logits, k)
top_k_probs = softmax(top_k_logits)
next_token = sample(top_k_probs, indices=top_k_indices)
```

**4. Top-p (Nucleus) Sampling:**
```python
# Sample from tokens whose cumulative probability >= p
p = 0.9
sorted_probs, sorted_indices = torch.sort(probabilities, descending=True)
cumulative_probs = torch.cumsum(sorted_probs, dim=0)
mask = cumulative_probs <= p
top_p_indices = sorted_indices[mask]
top_p_probs = probabilities[top_p_indices]
next_token = sample(top_p_probs, indices=top_p_indices)
```

### Complete Generation Example

```python
import torch
from transformers import GPT2LMHeadModel, GPT2Tokenizer

class TextGenerator:
    def __init__(self, model_name="gpt2"):
        self.model = GPT2LMHeadModel.from_pretrained(model_name)
        self.tokenizer = GPT2Tokenizer.from_pretrained(model_name)
        self.model.eval()
    
    def generate(
        self,
        prompt: str,
        max_length: int = 100,
        temperature: float = 1.0,
        top_k: int = 50,
        top_p: float = 0.9,
        do_sample: bool = True
    ):
        # Encode prompt
        input_ids = self.tokenizer.encode(prompt, return_tensors="pt")
        
        # Generate
        with torch.no_grad():
            output = self.model.generate(
                input_ids,
                max_length=max_length,
                temperature=temperature,
                top_k=top_k,
                top_p=top_p,
                do_sample=do_sample,
                pad_token_id=self.tokenizer.eos_token_id
            )
        
        # Decode
        generated_text = self.tokenizer.decode(output[0], skip_special_tokens=True)
        return generated_text

# Usage
generator = TextGenerator()
text = generator.generate(
    prompt="The future of AI is",
    max_length=50,
    temperature=0.7,
    top_p=0.9
)
print(text)
```

---

## LLM Improvements

### 1. Instruction Tuning

**Problem:** Base LLMs are good at next-token prediction but not at following instructions.

**Solution:** Fine-tune on instruction-response pairs.

```python
# Instruction dataset format
dataset = [
    {
        "instruction": "Explain quantum computing",
        "input": "",
        "output": "Quantum computing uses quantum mechanics..."
    },
    {
        "instruction": "Translate to French",
        "input": "Hello world",
        "output": "Bonjour le monde"
    }
]

# Training
def instruction_tuning_loss(model, batch):
    # Format: "### Instruction: {instruction}\n### Input: {input}\n### Response: {output}"
    prompt = format_instruction(batch)
    targets = batch["output"]
    
    logits = model(prompt)
    loss = cross_entropy(logits, targets)
    return loss
```

**Result:** Models like InstructGPT, ChatGPT follow instructions better.

### 2. Reinforcement Learning from Human Feedback (RLHF)

**Process:**
1. **Supervised Fine-Tuning (SFT):** Train on human-written responses
2. **Reward Modeling:** Train model to predict human preferences
3. **RL Fine-Tuning:** Optimize policy to maximize reward

```python
# Step 1: SFT
sft_model = fine_tune(base_model, human_responses)

# Step 2: Reward Model
def train_reward_model(responses, human_ratings):
    # responses: [response1, response2, ...]
    # human_ratings: [4, 2, 5, ...] (1-5 scale)
    reward_model = train_classifier(responses, human_ratings)
    return reward_model

# Step 3: RL Fine-tuning (PPO)
def ppo_training(policy_model, reward_model):
    for episode in range(num_episodes):
        # Generate responses
        responses = policy_model.generate(prompts)
        
        # Get rewards
        rewards = reward_model.predict(responses)
        
        # Update policy to maximize reward
        policy_model.update_with_ppo(rewards)
```

**Result:** ChatGPT, Claude - models that are helpful, harmless, and honest.

### 3. Chain-of-Thought (CoT) Prompting

**Idea:** Guide model to think step-by-step.

```python
# Without CoT
prompt = "Q: A store has 15 apples. They sell 8. How many left?"
# Model might answer incorrectly: "7" (15 - 8 = 7, but might not reason)

# With CoT
prompt = """Q: A store has 15 apples. They sell 8. How many left?
A: Let's think step by step.
1. Store starts with 15 apples
2. They sell 8 apples
3. Remaining = 15 - 8 = 7 apples
So the answer is 7."""
```

**Implementation:**
```python
def chain_of_thought_prompting(question):
    prompt = f"""Q: {question}
A: Let's think step by step.
"""
    response = model.generate(prompt)
    return response
```

### 4. Few-Shot Learning

**Idea:** Provide examples in prompt.

```python
prompt = """Translate English to French:

English: Hello
French: Bonjour

English: Good morning
French: Bonjour

English: How are you?
French:"""
```

---

## Retrieval Augmented Generation (RAG)

### What is RAG?

**RAG** combines retrieval (finding relevant documents) with generation (LLM creates answer).

### Why RAG?

**Problems with LLMs alone:**
1. **Hallucination:** Make up facts
2. **Outdated knowledge:** Training data cutoff
3. **No source attribution:** Can't cite sources
4. **Domain-specific:** Not trained on your data

**RAG Solution:**
1. Retrieve relevant documents from knowledge base
2. Provide documents as context to LLM
3. LLM generates answer based on context

### RAG Architecture

```
User Query
    ↓
[Embedding Model]
    ↓
Query Vector
    ↓
[Vector Database]
    ↓
Relevant Documents
    ↓
[Context + Query]
    ↓
[LLM]
    ↓
Answer (with sources)
```

### Complete RAG Implementation

```python
from sentence_transformers import SentenceTransformer
from pymilvus import Collection
import openai

class RAGSystem:
    def __init__(self):
        # Embedding model
        self.embedding_model = SentenceTransformer('all-MiniLM-L6-v2')
        
        # Vector database
        connections.connect("default", host="localhost", port="19530")
        self.collection = Collection("documents")
        self.collection.load()
        
        # LLM
        self.llm_client = openai.OpenAI()
    
    def retrieve(self, query: str, top_k: int = 5):
        """Retrieve relevant documents"""
        # Encode query
        query_embedding = self.embedding_model.encode(query)
        
        # Search
        search_params = {"metric_type": "L2", "params": {"ef": 64}}
        results = self.collection.search(
            data=[query_embedding],
            anns_field="embedding",
            param=search_params,
            limit=top_k,
            output_fields=["text", "source", "metadata"]
        )
        
        return results[0]
    
    def generate(self, query: str, context_docs: list):
        """Generate answer using retrieved context"""
        # Build context
        context = "\n\n".join([
            f"Document {i+1}:\n{doc.entity.get('text')}\nSource: {doc.entity.get('source')}"
            for i, doc in enumerate(context_docs)
        ])
        
        # Prompt
        prompt = f"""You are a helpful assistant. Answer the question using ONLY the provided context. If the answer is not in the context, say "I don't know."

Context:
{context}

Question: {query}

Answer:"""
        
        # Generate
        response = self.llm_client.chat.completions.create(
            model="gpt-4",
            messages=[{"role": "user", "content": prompt}],
            temperature=0.0  # Deterministic for factual answers
        )
        
        answer = response.choices[0].message.content
        
        # Extract sources
        sources = [doc.entity.get("source") for doc in context_docs]
        
        return {
            "answer": answer,
            "sources": sources,
            "context_docs": context_docs
        }
    
    def query(self, question: str):
        """Complete RAG pipeline"""
        # Retrieve
        docs = self.retrieve(question, top_k=5)
        
        # Generate
        result = self.generate(question, docs)
        
        return result

# Usage
rag = RAGSystem()
result = rag.query("What is machine learning?")
print(result["answer"])
print(f"Sources: {result['sources']}")
```

### RAG Improvements

**1. Re-ranking:**
```python
def rerank_documents(query, documents, top_k=3):
    """Re-rank using cross-encoder for better accuracy"""
    from sentence_transformers import CrossEncoder
    
    cross_encoder = CrossEncoder('cross-encoder/ms-marco-MiniLM-L-6-v2')
    
    # Score each document
    scores = cross_encoder.predict([
        [query, doc] for doc in documents
    ])
    
    # Sort and return top k
    ranked = sorted(zip(documents, scores), key=lambda x: x[1], reverse=True)
    return [doc for doc, _ in ranked[:top_k]]
```

**2. Hybrid Search:**
```python
def hybrid_search(query, collection, alpha=0.7):
    """Combine vector search with keyword search"""
    # Vector search (semantic)
    vector_results = vector_search(query, collection, top_k=10)
    
    # Keyword search (BM25)
    keyword_results = bm25_search(query, collection, top_k=10)
    
    # Combine scores
    combined = {}
    for doc, score in vector_results:
        combined[doc.id] = alpha * score
    for doc, score in keyword_results:
        combined[doc.id] = combined.get(doc.id, 0) + (1 - alpha) * score
    
    # Sort and return
    return sorted(combined.items(), key=lambda x: x[1], reverse=True)
```

---

## Positional Embeddings

### Why Position Matters

**Problem:** Transformers have no inherent notion of sequence order.

**Solution:** Add positional information to embeddings.

### Types of Positional Embeddings

**1. Learned Positional Embeddings:**
```python
class LearnedPositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000):
        super().__init__()
        self.pos_embedding = nn.Embedding(max_len, d_model)
    
    def forward(self, x):
        # x: [batch, seq_len, d_model]
        seq_len = x.size(1)
        positions = torch.arange(seq_len, device=x.device)
        pos_emb = self.pos_embedding(positions)  # [seq_len, d_model]
        return x + pos_emb.unsqueeze(0)
```

**2. Sinusoidal Positional Embeddings:**
```python
class SinusoidalPositionalEncoding(nn.Module):
    def __init__(self, d_model, max_len=5000):
        super().__init__()
        pe = torch.zeros(max_len, d_model)
        position = torch.arange(0, max_len).unsqueeze(1).float()
        
        div_term = torch.exp(torch.arange(0, d_model, 2).float() *
                           -(math.log(10000.0) / d_model))
        
        pe[:, 0::2] = torch.sin(position * div_term)
        pe[:, 1::2] = torch.cos(position * div_term)
        
        self.register_buffer('pe', pe.unsqueeze(0))
    
    def forward(self, x):
        # x: [batch, seq_len, d_model]
        return x + self.pe[:, :x.size(1)]
```

**3. Rotary Position Embedding (RoPE):**
```python
# Used in LLaMA, GPT-NeoX
# Rotates query and key vectors based on position
# Better for long sequences
```

---

## Attention Mechanism

See [Attention Mechanisms Guide](04_ATTENTION_MECHANISMS.md) for detailed explanation.

**Quick Overview:**
```python
# Self-Attention
def attention(Q, K, V):
    # Q: [batch, seq_len, d_model]
    # K: [batch, seq_len, d_model]
    # V: [batch, seq_len, d_model]
    
    scores = torch.matmul(Q, K.transpose(-2, -1)) / sqrt(d_model)
    attention_weights = softmax(scores)
    output = torch.matmul(attention_weights, V)
    return output
```

---

## Transformer Architecture

See [Attention Mechanisms Guide](04_ATTENTION_MECHANISMS.md) for full architecture.

**Key Components:**
1. **Embedding Layer:** Token + Positional
2. **Transformer Blocks:**
   - Multi-Head Self-Attention
   - Feed-Forward Network
   - Layer Normalization
   - Residual Connections
3. **Output Layer:** Linear projection to vocab size

---

## KV Cache Optimization

### Problem: Redundant Computation

**During generation, we recompute keys and values for all previous tokens:**

```python
# Without KV Cache
def generate_token(prompt_tokens, generated_tokens):
    all_tokens = prompt_tokens + generated_tokens
    
    # Recompute attention for ALL tokens
    for token in all_tokens:
        Q, K, V = compute_attention(token)  # Expensive!
    
    # Only need to compute for NEW token
    new_token = generate_next(all_tokens)
    return new_token
```

### Solution: KV Cache

**Cache keys and values from previous tokens:**

```python
class KVCache:
    def __init__(self):
        self.cached_keys = []
        self.cached_values = []
    
    def forward(self, new_tokens, cached_kv=None):
        # Compute Q, K, V for new tokens only
        Q, K, V = compute_attention(new_tokens)
        
        # Concatenate with cached
        if cached_kv:
            K = torch.cat([cached_kv['keys'], K], dim=1)
            V = torch.cat([cached_kv['values'], V], dim=1)
        
        # Compute attention
        output = attention(Q, K, V)
        
        # Cache for next iteration
        return {
            'output': output,
            'keys': K,
            'values': V
        }

# Usage
cache = KVCache()
kv_cache = None

# First token
kv_cache = cache.forward(prompt_tokens, kv_cache)

# Subsequent tokens (only compute for new token!)
for _ in range(max_length):
    new_token = get_next_token()
    kv_cache = cache.forward([new_token], kv_cache)  # Fast!
```

**Speed Improvement:**
- **Without cache:** O(n²) per token (n = total tokens)
- **With cache:** O(n) per token (only compute for new token)
- **Speedup:** 10-100x for long sequences

---

## Key Takeaways

1. **LLMs predict next token** using transformer architecture
2. **Autoregressive generation** creates text token by token
3. **RAG combines retrieval + generation** for factual answers
4. **Positional embeddings** encode sequence order
5. **KV cache** dramatically speeds up generation
6. **Instruction tuning + RLHF** make models helpful and safe

---

## Next Steps

- [Attention Mechanisms Guide](04_ATTENTION_MECHANISMS.md) - Deep dive into attention
- [LLM Optimizations Guide](05_LLM_OPTIMIZATIONS.md) - Production optimizations
- [Reasoning & Agents Guide](06_REASONING_AGENTS.md) - Advanced reasoning

---

## References

- [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
- [GPT-3 Paper](https://arxiv.org/abs/2005.14165)
- [RAG Paper](https://arxiv.org/abs/2005.11401)

