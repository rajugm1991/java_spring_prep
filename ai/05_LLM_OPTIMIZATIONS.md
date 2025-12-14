# LLM Optimizations: Production Guide

## Table of Contents
1. [Paged Attention](#paged-attention)
2. [Flash Attention](#flash-attention)
3. [Mixture of Experts (MoE)](#mixture-of-experts-moe)
4. [Quantization](#quantization)
5. [Sparse Attention](#sparse-attention)
6. [SLM & Distillation](#slm--distillation)
7. [Speculative Decoding](#speculative-decoding)

---

## Paged Attention

### Problem: Memory Fragmentation

**Issue:** KV cache memory is allocated per request, causing fragmentation.

```
Request 1: [KV cache: 1000 tokens] → Allocated
Request 2: [KV cache: 500 tokens]  → Allocated
Request 3: [KV cache: 2000 tokens] → Can't fit! (fragmentation)
```

**Result:** Low GPU utilization, wasted memory.

### Solution: Paged Attention

**Concept:** Manage KV cache in fixed-size pages (like OS virtual memory).

```python
class PagedAttention:
    def __init__(self, page_size=16, num_pages=1000):
        """
        page_size: Tokens per page (e.g., 16)
        num_pages: Total pages in pool
        """
        self.page_size = page_size
        self.num_pages = num_pages
        self.page_pool = [None] * num_pages  # Pool of free pages
        self.allocated_pages = {}  # request_id -> [page_ids]
    
    def allocate_pages(self, request_id, num_tokens):
        """Allocate pages for request"""
        num_pages_needed = (num_tokens + self.page_size - 1) // self.page_size
        
        # Find free pages
        allocated = []
        for i, page in enumerate(self.page_pool):
            if page is None and len(allocated) < num_pages_needed:
                self.page_pool[i] = request_id
                allocated.append(i)
        
        self.allocated_pages[request_id] = allocated
        return allocated
    
    def free_pages(self, request_id):
        """Free pages when request completes"""
        if request_id in self.allocated_pages:
            for page_id in self.allocated_pages[request_id]:
                self.page_pool[page_id] = None
            del self.allocated_pages[request_id]
    
    def get_kv_cache(self, request_id, page_ids):
        """Retrieve KV cache from pages"""
        kv_cache = []
        for page_id in page_ids:
            kv_cache.append(self.get_page(page_id))
        return torch.cat(kv_cache, dim=1)  # Concatenate pages
```

### Benefits

1. **Eliminates Fragmentation:** Fixed-size pages prevent waste
2. **Better Utilization:** Can pack more requests into GPU
3. **Dynamic Allocation:** Allocate pages as sequence grows
4. **Used in:** vLLM, TensorRT-LLM

### Implementation in vLLM

```python
# vLLM uses PagedAttention by default
from vllm import LLM

llm = LLM(model="gpt-3.5-turbo", max_model_len=4096)
# Automatically uses paged attention for efficient memory management
```

---

## Flash Attention

### Problem: Memory Bandwidth Bottleneck

**Standard Attention:**
```python
def standard_attention(Q, K, V):
    # Step 1: Compute S = QK^T
    S = torch.matmul(Q, K.transpose(-2, -1))  # [batch, seq_len, seq_len]
    # Memory: O(seq_len²) - huge for long sequences!
    
    # Step 2: Softmax
    P = F.softmax(S, dim=-1)  # [batch, seq_len, seq_len]
    
    # Step 3: Compute output
    O = torch.matmul(P, V)  # [batch, seq_len, d_model]
    
    return O
```

**Problem:** 
- For seq_len=8192: S matrix = 8192² = 67M elements
- At batch=32: 2.1B elements = 8.4 GB (just for S!)
- Memory bandwidth becomes bottleneck

### Solution: Flash Attention

**Key Idea:** Compute attention in blocks, never materialize full S matrix.

```python
def flash_attention(Q, K, V, block_size=128):
    """
    Flash Attention: Compute attention in blocks
    
    Never materialize full attention matrix
    """
    batch, seq_len, d_model = Q.shape
    O = torch.zeros_like(Q)
    l = torch.zeros(batch, seq_len, 1)  # Row sums
    m = torch.full((batch, seq_len, 1), float('-inf'))  # Row maxes
    
    # Process in blocks
    for j in range(0, seq_len, block_size):
        K_j = K[:, j:j+block_size, :]  # [batch, block_size, d_model]
        V_j = V[:, j:j+block_size, :]
        
        # Compute QK^T for this block
        S_ij = torch.matmul(Q, K_j.transpose(-2, -1))  # [batch, seq_len, block_size]
        
        # Update output incrementally (online softmax)
        m_new = torch.maximum(m, S_ij.max(dim=-1, keepdim=True)[0])
        alpha = torch.exp(m - m_new)
        P_ij = torch.exp(S_ij - m_new)
        
        l_new = alpha * l + P_ij.sum(dim=-1, keepdim=True)
        O = (alpha * O + torch.matmul(P_ij, V_j)) / l_new
        
        l = l_new
        m = m_new
    
    return O
```

### Benefits

1. **Memory Efficient:** O(seq_len) instead of O(seq_len²)
2. **Faster:** Better memory access patterns
3. **Scales:** Works for very long sequences (32K+ tokens)

### Speed Comparison

```
Standard Attention (seq_len=8192):
- Memory: 8.4 GB
- Time: 100ms

Flash Attention (seq_len=8192):
- Memory: 0.5 GB (16x reduction!)
- Time: 25ms (4x faster!)
```

### Usage

```python
# PyTorch 2.0+ has built-in Flash Attention
import torch.nn.functional as F

# Automatically uses Flash Attention if available
output = F.scaled_dot_product_attention(Q, K, V)

# Or explicitly
output = F.scaled_dot_product_attention(
    Q, K, V,
    attn_mask=None,
    is_causal=True,
    scale=None
)
```

---

## Mixture of Experts (MoE)

### Problem: Model Size vs Inference Cost

**Dense Models:**
- GPT-3 (175B): All parameters active for every token
- Inference cost: O(175B) operations per token

**MoE Models:**
- Switch Transformer (1.6T): Only activate subset of experts
- Inference cost: O(10B) operations per token (160x reduction!)

### Concept

**MoE Architecture:**
```
Input Token
    ↓
[Router] → Selects top-k experts (e.g., top-2)
    ↓
Expert 1 (activated) ──┐
Expert 2 (activated) ──┤
Expert 3 (not used)    │ → Weighted Sum
Expert 4 (not used)   │
...                    │
Expert 8 (not used) ───┘
```

### Implementation

```python
class MixtureOfExperts(nn.Module):
    def __init__(self, d_model, num_experts=8, top_k=2):
        super().__init__()
        self.num_experts = num_experts
        self.top_k = top_k
        
        # Router: decides which experts to use
        self.router = nn.Linear(d_model, num_experts)
        
        # Experts: separate feed-forward networks
        self.experts = nn.ModuleList([
            FeedForward(d_model) for _ in range(num_experts)
        ])
    
    def forward(self, x):
        # x: [batch, seq_len, d_model]
        batch, seq_len, d_model = x.shape
        
        # Route to experts
        router_logits = self.router(x)  # [batch, seq_len, num_experts]
        router_probs = F.softmax(router_logits, dim=-1)
        
        # Select top-k experts
        top_k_probs, top_k_indices = torch.topk(router_probs, self.top_k, dim=-1)
        # top_k_probs: [batch, seq_len, top_k]
        # top_k_indices: [batch, seq_len, top_k]
        
        # Process through selected experts
        output = torch.zeros_like(x)
        
        for expert_id in range(self.num_experts):
            # Find positions where this expert is selected
            expert_mask = (top_k_indices == expert_id)
            
            if expert_mask.any():
                # Get inputs for this expert
                expert_input = x[expert_mask]
                
                # Process through expert
                expert_output = self.experts[expert_id](expert_input)
                
                # Get corresponding probabilities
                expert_probs = top_k_probs[expert_mask]
                
                # Weighted sum
                output[expert_mask] += expert_output * expert_probs.unsqueeze(-1)
        
        return output
```

### Load Balancing

**Problem:** Some experts might be overused, others underused.

**Solution:** Add load balancing loss.

```python
def load_balancing_loss(router_probs, top_k_indices):
    """
    Encourage uniform expert usage
    """
    # Compute expert usage
    expert_usage = torch.zeros(num_experts)
    for expert_id in range(num_experts):
        usage = (top_k_indices == expert_id).float().mean()
        expert_usage[expert_id] = usage
    
    # Load balancing loss: variance of usage
    # Lower variance = more balanced
    load_balance_loss = expert_usage.var()
    
    return load_balance_loss

# Add to training loss
total_loss = task_loss + 0.01 * load_balancing_loss(...)
```

### Real-World Examples

- **Switch Transformer:** 1.6T parameters, ~10B active
- **GShard:** Google's MoE model
- **Mixtral 8x7B:** 8 experts, top-2 routing (47B active)

---

## Quantization

### What is Quantization?

**Convert high-precision (FP32) to low-precision (INT8, INT4) to reduce memory and speed up inference.**

### Types of Quantization

#### 1. Post-Training Quantization (PTQ)

**Quantize after training (no retraining needed):**

```python
def quantize_to_int8(tensor):
    """Quantize FP32 to INT8"""
    # Find scale and zero point
    scale = tensor.abs().max() / 127.0
    zero_point = 0
    
    # Quantize
    quantized = torch.round(tensor / scale).clamp(-128, 127).to(torch.int8)
    
    return quantized, scale, zero_point

def dequantize(quantized, scale, zero_point):
    """Dequantize INT8 back to FP32"""
    return (quantized.float() - zero_point) * scale

# Usage
weight_fp32 = model.layer.weight  # [1024, 1024] FP32 = 4 MB
weight_int8, scale, zp = quantize_to_int8(weight_fp32)  # 1 MB (4x reduction!)
```

#### 2. Quantization-Aware Training (QAT)

**Train with quantization simulation:**

```python
class FakeQuantize(nn.Module):
    """Simulate quantization during training"""
    def __init__(self, num_bits=8):
        super().__init__()
        self.num_bits = num_bits
    
    def forward(self, x):
        # Quantize
        scale = x.abs().max() / (2 ** (self.num_bits - 1) - 1)
        quantized = torch.round(x / scale).clamp(
            -(2 ** (self.num_bits - 1)),
            2 ** (self.num_bits - 1) - 1
        )
        
        # Dequantize (straight-through estimator)
        dequantized = quantized * scale
        
        return dequantized

# Use in model
class QuantizedLinear(nn.Module):
    def __init__(self, in_features, out_features):
        super().__init__()
        self.weight = nn.Parameter(torch.randn(out_features, in_features))
        self.fake_quant = FakeQuantize()
    
    def forward(self, x):
        quantized_weight = self.fake_quant(self.weight)
        return F.linear(x, quantized_weight)
```

#### 3. INT4 Quantization

**Even more aggressive compression:**

```python
def quantize_to_int4(tensor):
    """Quantize to 4-bit integers"""
    # Pack two 4-bit values into one byte
    scale = tensor.abs().max() / 7.0  # 4-bit: -8 to 7
    quantized = torch.round(tensor / scale).clamp(-8, 7).to(torch.int8)
    
    # Pack: [a, b, c, d] -> [(a,b), (c,d)]
    packed = pack_int4(quantized)
    
    return packed, scale

# Memory: 4x reduction vs INT8, 16x vs FP32
```

### GPTQ (GPT Quantization)

**State-of-the-art PTQ method:**

```python
# GPTQ quantizes weights layer by layer
def gptq_quantize_layer(layer, calibration_data):
    """
    Quantize layer using GPTQ algorithm
    
    1. Quantize weights to INT4
    2. Correct errors using Hessian matrix
    3. Minimize quantization error
    """
    # Compute Hessian (second-order gradient)
    hessian = compute_hessian(layer, calibration_data)
    
    # Quantize with error correction
    quantized_weights = quantize_with_correction(
        layer.weight,
        hessian
    )
    
    return quantized_weights
```

### Benefits

| Precision | Memory | Speed | Accuracy |
|-----------|--------|-------|----------|
| FP32 | 1x | 1x | 100% |
| FP16 | 0.5x | 2x | ~99.9% |
| INT8 | 0.25x | 4x | ~99% |
| INT4 | 0.125x | 8x | ~95% |

---

## Sparse Attention

### Problem: Quadratic Complexity

**Standard Attention:** O(seq_len²) - expensive for long sequences.

**Solution:** Only attend to subset of tokens.

### Types of Sparse Attention

#### 1. Local Attention (Sliding Window)

**Only attend to nearby tokens:**

```python
def local_attention(Q, K, V, window_size=128):
    """
    Each token only attends to tokens within window
    """
    seq_len = Q.size(1)
    output = torch.zeros_like(Q)
    
    for i in range(seq_len):
        # Define window
        start = max(0, i - window_size // 2)
        end = min(seq_len, i + window_size // 2)
        
        # Attend only to window
        Q_i = Q[:, i:i+1, :]  # [batch, 1, d_model]
        K_window = K[:, start:end, :]
        V_window = V[:, start:end, :]
        
        # Compute attention
        scores = torch.matmul(Q_i, K_window.transpose(-2, -1))
        attn = F.softmax(scores, dim=-1)
        output[:, i, :] = torch.matmul(attn, V_window).squeeze(1)
    
    return output
```

**Complexity:** O(seq_len × window_size) instead of O(seq_len²)

#### 2. Strided Attention

**Attend to every k-th token:**

```python
def strided_attention(Q, K, V, stride=4):
    """Attend to tokens at regular intervals"""
    seq_len = Q.size(1)
    
    # Only compute attention for strided positions
    strided_indices = torch.arange(0, seq_len, stride)
    
    Q_strided = Q[:, strided_indices, :]
    K_strided = K[:, strided_indices, :]
    V_strided = V[:, strided_indices, :]
    
    # Standard attention on strided tokens
    output_strided = scaled_dot_product_attention(Q_strided, K_strided, V_strided)
    
    # Interpolate for non-strided positions
    output = interpolate(output_strided, seq_len)
    
    return output
```

#### 3. Longformer Pattern

**Combine local + global attention:**

```python
def longformer_attention(Q, K, V, window_size=512, global_indices=[0, -1]):
    """
    Local attention + global attention to specific tokens
    
    global_indices: Which tokens attend to all positions
    """
    seq_len = Q.size(1)
    output = torch.zeros_like(Q)
    
    for i in range(seq_len):
        if i in global_indices:
            # Global attention (attend to all)
            Q_i = Q[:, i:i+1, :]
            attn = scaled_dot_product_attention(Q_i, K, V)
            output[:, i, :] = attn.squeeze(1)
        else:
            # Local attention (sliding window)
            start = max(0, i - window_size // 2)
            end = min(seq_len, i + window_size // 2)
            Q_i = Q[:, i:i+1, :]
            K_local = K[:, start:end, :]
            V_local = V[:, start:end, :]
            attn = scaled_dot_product_attention(Q_i, K_local, V_local)
            output[:, i, :] = attn.squeeze(1)
    
    return output
```

---

## SLM & Distillation

### Small Language Models (SLMs)

**Problem:** Large models (GPT-4, Claude) are expensive to run.

**Solution:** Train smaller models that are faster and cheaper.

### Knowledge Distillation

**Concept:** Train small model to mimic large model's behavior.

```python
class DistillationTrainer:
    def __init__(self, teacher_model, student_model):
        self.teacher = teacher_model
        self.student = student_model
    
    def distillation_loss(self, inputs, temperature=3.0):
        """
        Distillation loss: student learns from teacher's soft labels
        """
        # Teacher predictions (soft labels)
        with torch.no_grad():
            teacher_logits = self.teacher(inputs)
            teacher_probs = F.softmax(teacher_logits / temperature, dim=-1)
        
        # Student predictions
        student_logits = self.student(inputs)
        student_probs = F.softmax(student_logits / temperature, dim=-1)
        
        # KL divergence loss (student matches teacher distribution)
        distillation_loss = F.kl_div(
            F.log_softmax(student_logits / temperature, dim=-1),
            teacher_probs,
            reduction='batchmean'
        ) * (temperature ** 2)
        
        # Task loss (student still learns task)
        task_loss = F.cross_entropy(student_logits, labels)
        
        # Combined loss
        total_loss = 0.7 * distillation_loss + 0.3 * task_loss
        
        return total_loss
```

### Examples

- **DistilBERT:** 60% of BERT size, 60% faster, 97% accuracy
- **TinyBERT:** 7.5x smaller, 9.4x faster
- **GPT-4 → GPT-3.5:** Smaller, faster, cheaper

---

## Speculative Decoding

### Problem: LLM Generation is Sequential

**Standard Generation:**
```
Token 1 → Token 2 → Token 3 → Token 4
(sequential, slow)
```

### Solution: Speculate and Verify

**Concept:** Use small model to generate multiple tokens, verify with large model.

```python
class SpeculativeDecoder:
    def __init__(self, small_model, large_model, num_speculative_tokens=4):
        self.small_model = small_model  # Fast, less accurate
        self.large_model = large_model  # Slow, accurate
        self.k = num_speculative_tokens
    
    def decode(self, prompt):
        """Speculative decoding"""
        tokens = [prompt]
        
        while not done:
            # Step 1: Small model generates k tokens
            speculative_tokens = []
            for _ in range(self.k):
                next_token = self.small_model.generate_next(tokens)
                speculative_tokens.append(next_token)
                tokens.append(next_token)
            
            # Step 2: Large model verifies all k tokens in parallel
            # (can parallelize because we know the sequence)
            verified_tokens = self.large_model.verify(tokens[-self.k:])
            
            # Step 3: Accept tokens up to first rejection
            accepted = 0
            for i, (spec, verif) in enumerate(zip(speculative_tokens, verified_tokens)):
                if self.accept_token(spec, verif):
                    accepted += 1
                else:
                    break
            
            # Step 4: If all accepted, continue. Otherwise, correct and continue
            if accepted < self.k:
                # Correct rejected token
                tokens[-1] = verified_tokens[accepted]
        
        return tokens
```

### Speed Improvement

**Standard:** 1 token per large model call
**Speculative:** 2-4 tokens per large model call (2-4x speedup!)

**Used in:** vLLM, TensorRT-LLM

---

## Key Takeaways

1. **Paged Attention:** Eliminates memory fragmentation
2. **Flash Attention:** O(seq_len) memory, 4x faster
3. **MoE:** Activate subset of parameters (160x reduction)
4. **Quantization:** 4-16x memory reduction, 4-8x speedup
5. **Sparse Attention:** O(seq_len) instead of O(seq_len²)
6. **Distillation:** Smaller, faster models with 97% accuracy
7. **Speculative Decoding:** 2-4x speedup for generation

---

## Next Steps

- [Reasoning & Agents Guide](06_REASONING_AGENTS.md) - Chain of Thought, AI Agents
- [Production AI Systems](07_PRODUCTION_AI.md) - Deploying optimized models

---

## References

- [Flash Attention Paper](https://arxiv.org/abs/2205.14135)
- [Paged Attention Paper](https://arxiv.org/abs/2309.06180)
- [Mixture of Experts](https://arxiv.org/abs/1701.06538)

