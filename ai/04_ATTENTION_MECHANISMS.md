# Attention Mechanisms: Complete Guide

## Table of Contents
1. [What is Attention?](#what-is-attention)
2. [Scaled Dot-Product Attention](#scaled-dot-product-attention)
3. [Multi-Head Attention](#multi-head-attention)
4. [Masked Attention](#masked-attention)
5. [Self-Attention vs Cross-Attention](#self-attention-vs-cross-attention)
6. [Attention Visualization](#attention-visualization)

---

## What is Attention?

**Attention** is a mechanism that allows models to focus on relevant parts of the input when making predictions.

### Intuition

**Human Attention Analogy:**
When reading "The cat sat on the mat", you focus on:
- "cat" and "sat" when understanding the action
- "mat" when understanding where

**Machine Attention:**
Model learns to assign weights to different tokens:
```
"The cat sat on the mat"
 0.1  0.3  0.4  0.1  0.1  0.3
      ↑              ↑
   Focus on "cat" and "mat" when predicting next word
```

### Why Attention Matters

**Before Attention (RNNs):**
- Process tokens sequentially
- Information bottleneck
- Hard to parallelize
- Vanishing gradients

**With Attention (Transformers):**
- Process all tokens in parallel
- Direct connections between any tokens
- Better long-range dependencies
- Parallelizable

---

## Scaled Dot-Product Attention

### Mathematical Formulation

```
Attention(Q, K, V) = softmax(QK^T / √d_k) V
```

Where:
- **Q (Query):** What am I looking for?
- **K (Key):** What do I have?
- **V (Value):** What information do I provide?
- **d_k:** Dimension of keys (scaling factor)

### Step-by-Step Implementation

```python
import torch
import torch.nn.functional as F
import math

def scaled_dot_product_attention(Q, K, V, mask=None):
    """
    Scaled Dot-Product Attention
    
    Args:
        Q: [batch, seq_len_q, d_k] - Query matrix
        K: [batch, seq_len_k, d_k] - Key matrix
        V: [batch, seq_len_v, d_v] - Value matrix
        mask: [batch, seq_len_q, seq_len_k] - Attention mask
    
    Returns:
        output: [batch, seq_len_q, d_v]
        attention_weights: [batch, seq_len_q, seq_len_k]
    """
    d_k = Q.size(-1)
    
    # Step 1: Compute attention scores
    # QK^T: [batch, seq_len_q, d_k] × [batch, d_k, seq_len_k]
    #      = [batch, seq_len_q, seq_len_k]
    scores = torch.matmul(Q, K.transpose(-2, -1))
    
    # Step 2: Scale by √d_k (prevent softmax saturation)
    scores = scores / math.sqrt(d_k)
    
    # Step 3: Apply mask (if provided)
    if mask is not None:
        scores = scores.masked_fill(mask == 0, -1e9)
    
    # Step 4: Apply softmax to get attention weights
    attention_weights = F.softmax(scores, dim=-1)
    # attention_weights: [batch, seq_len_q, seq_len_k]
    # Each row sums to 1 (probability distribution)
    
    # Step 5: Apply attention weights to values
    output = torch.matmul(attention_weights, V)
    # [batch, seq_len_q, seq_len_k] × [batch, seq_len_v, d_v]
    # = [batch, seq_len_q, d_v]
    
    return output, attention_weights

# Example
batch_size = 2
seq_len = 5
d_k = 64
d_v = 64

Q = torch.randn(batch_size, seq_len, d_k)
K = torch.randn(batch_size, seq_len, d_k)
V = torch.randn(batch_size, seq_len, d_v)

output, attention_weights = scaled_dot_product_attention(Q, K, V)
print(f"Output shape: {output.shape}")  # [2, 5, 64]
print(f"Attention weights shape: {attention_weights.shape}")  # [2, 5, 5]
```

### Why Scale by √d_k?

**Problem:** Without scaling, dot products can be very large, causing softmax to saturate.

```python
# Without scaling
scores = Q @ K.T  # Can be very large
attention = softmax(scores)  # Saturated (all probability on one token)

# With scaling
scores = (Q @ K.T) / sqrt(d_k)  # Normalized
attention = softmax(scores)  # Better distribution
```

**Mathematical Reason:**
- Dot product variance = d_k
- Scaling by √d_k normalizes variance to 1
- Prevents extreme values

---

## Multi-Head Attention

### Concept

**Single-head attention** uses one set of Q, K, V projections.

**Multi-head attention** uses multiple parallel attention "heads", each with different learned projections.

### Why Multiple Heads?

Different heads learn to attend to different aspects:
- **Head 1:** Syntactic relationships (subject-verb)
- **Head 2:** Semantic relationships (synonyms)
- **Head 3:** Long-range dependencies
- **Head 4:** Positional relationships

### Implementation

```python
class MultiHeadAttention(nn.Module):
    def __init__(self, d_model, num_heads):
        super().__init__()
        assert d_model % num_heads == 0
        
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        
        # Linear projections for Q, K, V
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        
        # Output projection
        self.W_o = nn.Linear(d_model, d_model)
    
    def forward(self, Q, K, V, mask=None):
        batch_size = Q.size(0)
        
        # 1. Linear projections
        Q = self.W_q(Q)  # [batch, seq_len, d_model]
        K = self.W_k(K)
        V = self.W_v(V)
        
        # 2. Reshape for multi-head
        # Split d_model into num_heads × d_k
        Q = Q.view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        # [batch, num_heads, seq_len, d_k]
        K = K.view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        V = V.view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        
        # 3. Apply attention to each head
        attention_outputs = []
        for head in range(self.num_heads):
            Q_head = Q[:, head, :, :]  # [batch, seq_len, d_k]
            K_head = K[:, head, :, :]
            V_head = V[:, head, :, :]
            
            output, _ = scaled_dot_product_attention(Q_head, K_head, V_head, mask)
            attention_outputs.append(output)
        
        # 4. Concatenate heads
        # [batch, num_heads, seq_len, d_k] → [batch, seq_len, d_model]
        concat = torch.cat(attention_outputs, dim=-1)
        # [batch, seq_len, num_heads × d_k] = [batch, seq_len, d_model]
        
        # 5. Final linear projection
        output = self.W_o(concat)
        
        return output

# Optimized version (process all heads in parallel)
class MultiHeadAttentionOptimized(nn.Module):
    def __init__(self, d_model, num_heads):
        super().__init__()
        assert d_model % num_heads == 0
        
        self.d_model = d_model
        self.num_heads = num_heads
        self.d_k = d_model // num_heads
        
        self.W_q = nn.Linear(d_model, d_model)
        self.W_k = nn.Linear(d_model, d_model)
        self.W_v = nn.Linear(d_model, d_model)
        self.W_o = nn.Linear(d_model, d_model)
    
    def forward(self, Q, K, V, mask=None):
        batch_size, seq_len = Q.size(0), Q.size(1)
        
        # Project and reshape
        Q = self.W_q(Q).view(batch_size, seq_len, self.num_heads, self.d_k).transpose(1, 2)
        K = self.W_k(K).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        V = self.W_v(V).view(batch_size, -1, self.num_heads, self.d_k).transpose(1, 2)
        
        # Attention (all heads in parallel)
        scores = torch.matmul(Q, K.transpose(-2, -1)) / math.sqrt(self.d_k)
        
        if mask is not None:
            scores = scores.masked_fill(mask == 0, -1e9)
        
        attention_weights = F.softmax(scores, dim=-1)
        attention_output = torch.matmul(attention_weights, V)
        
        # Concatenate and project
        attention_output = attention_output.transpose(1, 2).contiguous()
        attention_output = attention_output.view(batch_size, seq_len, self.d_model)
        
        output = self.W_o(attention_output)
        return output
```

### Head Visualization

```python
def visualize_attention_heads(model, input_text, head_id=0):
    """Visualize what a specific attention head focuses on"""
    tokens = tokenizer.encode(input_text)
    embeddings = model.embedding(tokens)
    
    # Get attention weights
    _, attention_weights = model.attention(embeddings)
    # attention_weights: [batch, num_heads, seq_len, seq_len]
    
    # Visualize specific head
    head_attention = attention_weights[0, head_id, :, :]
    
    # Plot heatmap
    import matplotlib.pyplot as plt
    plt.imshow(head_attention.detach().numpy(), cmap='Blues')
    plt.xlabel('Key Position')
    plt.ylabel('Query Position')
    plt.title(f'Attention Head {head_id}')
    plt.show()

# Example: "The cat sat on the mat"
# Head 0 might focus on: subject-verb (cat-sat)
# Head 1 might focus on: object relationships (sat-mat)
```

---

## Masked Attention

### Why Mask?

**Problem:** During training, model sees future tokens (data leakage).

**Solution:** Mask future tokens so model can only attend to past tokens.

### Types of Masks

**1. Causal Mask (Decoder):**
```python
def create_causal_mask(seq_len):
    """
    Create mask that prevents attending to future tokens
    
    Example for seq_len=5:
    [[1, 0, 0, 0, 0],
     [1, 1, 0, 0, 0],
     [1, 1, 1, 0, 0],
     [1, 1, 1, 1, 0],
     [1, 1, 1, 1, 1]]
    """
    mask = torch.triu(torch.ones(seq_len, seq_len), diagonal=1)
    mask = mask.masked_fill(mask == 1, float('-inf'))
    mask = mask.masked_fill(mask == 0, float(0.0))
    return mask

# Usage
seq_len = 5
causal_mask = create_causal_mask(seq_len)
# [[0, -inf, -inf, -inf, -inf],
#  [0, 0, -inf, -inf, -inf],
#  [0, 0, 0, -inf, -inf],
#  [0, 0, 0, 0, -inf],
#  [0, 0, 0, 0, 0]]

output, _ = scaled_dot_product_attention(Q, K, V, mask=causal_mask)
```

**2. Padding Mask:**
```python
def create_padding_mask(sequences, pad_token_id=0):
    """
    Mask padding tokens
    
    sequences: [batch, seq_len]
    Returns: [batch, 1, 1, seq_len]
    """
    mask = (sequences != pad_token_id).unsqueeze(1).unsqueeze(2)
    # Convert to attention mask format (0 = mask, 1 = attend)
    mask = mask.float()
    mask = mask.masked_fill(mask == 0, float('-inf'))
    mask = mask.masked_fill(mask == 1, float(0.0))
    return mask

# Example
sequences = torch.tensor([
    [1, 2, 3, 0, 0],  # Padded with 0s
    [4, 5, 6, 7, 8]   # No padding
])
padding_mask = create_padding_mask(sequences)
```

**3. Combined Mask:**
```python
def create_combined_mask(seq_len, padding_mask):
    """Combine causal and padding masks"""
    causal_mask = create_causal_mask(seq_len)
    # Expand padding mask to match causal mask shape
    padding_mask = padding_mask.expand(-1, seq_len, -1)
    # Combine (use most restrictive)
    combined = torch.minimum(causal_mask, padding_mask)
    return combined
```

---

## Self-Attention vs Cross-Attention

### Self-Attention

**Same input for Q, K, V:**
```python
def self_attention(x):
    """
    x: [batch, seq_len, d_model]
    Q, K, V all come from x
    """
    Q = linear_q(x)  # [batch, seq_len, d_k]
    K = linear_k(x)  # [batch, seq_len, d_k]
    V = linear_v(x)  # [batch, seq_len, d_v]
    
    output = attention(Q, K, V)
    return output

# Use case: Understanding relationships within a sequence
# Example: "The cat sat on the mat"
# Model learns: cat-sat relationship, sat-mat relationship
```

### Cross-Attention

**Different inputs for Q vs K, V:**
```python
def cross_attention(query, key_value):
    """
    query: [batch, seq_len_q, d_model] - What am I looking for?
    key_value: [batch, seq_len_kv, d_model] - Where do I look?
    
    Q comes from query
    K, V come from key_value
    """
    Q = linear_q(query)  # [batch, seq_len_q, d_k]
    K = linear_k(key_value)  # [batch, seq_len_kv, d_k]
    V = linear_v(key_value)  # [batch, seq_len_kv, d_v]
    
    output = attention(Q, K, V)
    return output

# Use case: Encoder-Decoder attention
# Encoder output → K, V
# Decoder input → Q
# Model learns: Which encoder tokens to focus on for each decoder position
```

### Encoder-Decoder Architecture

```python
class EncoderDecoder(nn.Module):
    def __init__(self, d_model, num_heads):
        super().__init__()
        self.encoder = TransformerEncoder(d_model, num_heads)
        self.decoder = TransformerDecoder(d_model, num_heads)
    
    def forward(self, encoder_input, decoder_input):
        # Encoder: Self-attention
        encoder_output = self.encoder(encoder_input)
        
        # Decoder: Self-attention + Cross-attention
        decoder_output = self.decoder(
            decoder_input,
            encoder_output  # Used as K, V in cross-attention
        )
        
        return decoder_output

class TransformerDecoder(nn.Module):
    def __init__(self, d_model, num_heads):
        super().__init__()
        self.self_attention = MultiHeadAttention(d_model, num_heads)
        self.cross_attention = MultiHeadAttention(d_model, num_heads)
        self.ffn = FeedForward(d_model)
    
    def forward(self, x, encoder_output):
        # Self-attention (masked)
        x = self.self_attention(x, x, x, mask=causal_mask)
        
        # Cross-attention
        # Q from decoder, K and V from encoder
        x = self.cross_attention(x, encoder_output, encoder_output)
        
        # Feed-forward
        x = self.ffn(x)
        
        return x
```

---

## Attention Visualization

### Visualizing Attention Weights

```python
import matplotlib.pyplot as plt
import seaborn as sns

def plot_attention_weights(attention_weights, tokens, head_id=0):
    """
    Plot attention weights as heatmap
    
    attention_weights: [num_heads, seq_len, seq_len]
    tokens: List of token strings
    """
    # Get specific head
    head_weights = attention_weights[head_id].detach().numpy()
    
    # Create heatmap
    plt.figure(figsize=(10, 8))
    sns.heatmap(
        head_weights,
        xticklabels=tokens,
        yticklabels=tokens,
        cmap='Blues',
        annot=True,
        fmt='.2f'
    )
    plt.title(f'Attention Head {head_id}')
    plt.xlabel('Key Position')
    plt.ylabel('Query Position')
    plt.tight_layout()
    plt.show()

# Example
text = "The cat sat on the mat"
tokens = tokenizer.tokenize(text)
attention_weights = model.get_attention_weights(text)
plot_attention_weights(attention_weights, tokens, head_id=0)
```

### Attention Patterns

**Different heads learn different patterns:**

```python
def analyze_attention_patterns(model, text):
    """Analyze what different attention heads focus on"""
    tokens = tokenizer.tokenize(text)
    attention_weights = model.get_attention_weights(text)
    
    for head_id in range(model.num_heads):
        head_weights = attention_weights[head_id]
        
        # Find strongest attention for each token
        for i, token in enumerate(tokens):
            strongest_attention = head_weights[i].argmax()
            attended_token = tokens[strongest_attention]
            strength = head_weights[i, strongest_attention]
            
            print(f"Head {head_id}: '{token}' → '{attended_token}' (strength: {strength:.2f})")

# Example output:
# Head 0: 'cat' → 'sat' (strength: 0.45)  # Syntactic: subject-verb
# Head 1: 'sat' → 'mat' (strength: 0.38)  # Semantic: action-object
# Head 2: 'The' → 'cat' (strength: 0.52)  # Determiner-noun
```

---

## Key Takeaways

1. **Attention allows models to focus** on relevant parts of input
2. **Scaled dot-product attention** prevents softmax saturation
3. **Multi-head attention** learns different relationships in parallel
4. **Masked attention** prevents data leakage during training
5. **Self-attention** within sequence, **cross-attention** between sequences
6. **Visualization** helps understand what models learn

---

## Next Steps

- [LLM Optimizations Guide](05_LLM_OPTIMIZATIONS.md) - Flash Attention, Paged Attention
- [Transformer Architecture](03_LLM_ARCHITECTURE.md) - Complete transformer implementation

---

## References

- [Attention Is All You Need](https://arxiv.org/abs/1706.03762)
- [The Illustrated Transformer](https://jalammar.github.io/illustrated-transformer/)

